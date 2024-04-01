---
date: "2023-05-05T07:58:11+07:00"
description: Sự linh hoạt của Airflow giúp cho chúng ta có thể tùy biến tối đa mã nguồn để đáp ứng các mục đích khác nhau. Bài viết này liệt kê một số điểm cần lưu ý để có thể tổ chức cũng như tạo mã nguồn tối ưu để sử dụng chung với Airflow.
layout: post
title:
  "M\u1ED9t s\u1ED1 \u0111i\u1EC3m c\u1EA7n l\u01B0u \xFD khi s\u1EED d\u1EE5\
  ng Airflow - Ph\u1EA7n 2"
categories:
  - Một vài tip nhỏ
tags:
  - Data Engineering
  - Data Analyst
---

Sự linh hoạt của Airflow giúp cho chúng ta có thể tùy biến tối đa mã nguồn để đáp ứng các mục đích khác nhau. Bài viết này liệt kê một số điểm cần lưu ý để có thể tổ chức cũng như tạo mã nguồn tối ưu để sử dụng chung với Airflow.

# Import đúng chỗ, tạo biến đúng chỗ

Để tối ưu hiệu suất của Airflow, ta tốt nhất là chỉ viết mã cần thiết để tạo các operator, định nghĩa các hàm callable và định nghĩa quan hệ giữa chúng. Điều này là do Airflow sẽ import toàn bộ file định nghĩa DAG trong lúc đọc nội dung nhằm để cập nhật thông tin các DAG, tức là những đoạn code không thuộc ba thứ trên sẽ được thực thi ngay lập tức và điều đó quá trình xử lý sẽ không đúng kì vọng và tất nhiên quá trình phân tích cú pháp sẽ chậm đi đáng kể

Ví dụ, DAG đầu tiên trong mã dưới đây sẽ được phân tích chậm hơn do NumPy được import mỗi khi tệp DAG được phân tích cú pháp, điều này sẽ dẫn đến hiệu suất kém trong việc xử lý tệp DAG. Để cải thiện vấn đề này, ta nên để NumPy chỉ được import khi task được thực thi như ở ví dụ thứ hai

```python
import pendulum
from airflow import DAG
from airflow.decorators import task

import numpy as np  # <-- THIS IS A VERY BAD IDEA! DON'T DO THAT!

with DAG(
    dag_id="example_python_operator",
    schedule=None,
    start_date=pendulum.datetime(2021, 1, 1, tz="UTC"),
    catchup=False,
    tags=["example"],
) as dag:

    @task()
    def print_array():
        """Print Numpy array."""

        a = np.arange(15).reshape(3, 5)
        print(a)
        return a

    print_array()
```

Good example:

```python
import pendulum
from airflow import DAG
from airflow.decorators import task

with DAG(
    dag_id="example_python_operator",
    schedule=None,
    start_date=pendulum.datetime(2021, 1, 1, tz="UTC"),
    catchup=False,
    tags=["example"],
) as dag:

    @task()
    def print_array():
        """Print Numpy array."""

        import numpy as np  # <- THIS IS HOW NUMPY SHOULD BE IMPORTED IN THIS CASE!a = np.arange(15).reshape(3, 5)
        print(a)
        return a

    print_array()
```

Tương tự, chúng ta không nên đặt đoạn mã sử dụng các Airflow Variable bên ngoài các operator và các python_callable của chúng. Do các đoạn mã này được thực thi khi lập lịch các task, nên phiên bản của các Variable được sử dụng sẽ là phiên bản tại thời điểm task được lập lịch chứ không phải lúc task được thực thi. Việc đặt chúng ở ngoài như vậy cũng khiến cho hiệu năng giảm sút và quá trình parse các file định nghĩa dag có thể bị timeout. Ta có thể dễ dàng hình dung thông qua hai ví dụ sau đây:

Đây là một ví dụ về mã không tối ưu:

```python
from airflow.models import Variable

foo_var = Variable.get("foo")  # DON'T DO THAT
bash_use_variable_bad_1 = BashOperator(
    task_id="bash_use_variable_bad_1", bash_command="echo variable foo=${foo_env}", env={"foo_env": foo_var}
)

bash_use_variable_bad_2 = BashOperator(
    task_id="bash_use_variable_bad_2",
    bash_command=f"echo variable foo=${Variable.get('foo')}",  # DON'T DO THAT
)

bash_use_variable_bad_3 = BashOperator(
    task_id="bash_use_variable_bad_3",
    bash_command="echo variable foo=${foo_env}",
    env={"foo_env": Variable.get("foo")},  # DON'T DO THAT
)
```

Nó sẽ nên như thế này

```python
bash_use_variable_good = BashOperator(
    task_id="bash_use_variable_good",
    bash_command="echo variable foo=$**{foo_env}**",
    env={"foo_env": "{{ var.value.get('foo') }}"},
)

@task
def my_task():
    var = Variable.get("foo")  # this is fine, because func my_task called only run task, not scan DAGs.
    print(var)
```

# Giảm độ phức tạp của DAG xuống ít nhất có thể

Airflow rất tốt trong việc xử lý nhiều DAG với nhiều task, tuy nhiên, nếu bạn có nhiều DAG phức tạp, điều này có thể ảnh hưởng đến hiệu suất của quá trình lịch trình. Để giữ cho Airflow của bạn hoạt động hiệu quả, tốt nhất là đơn giản hóa và tối ưu hóa DAG của bạn mỗi khi có thể. Nếu bạn muốn tối ưu hóa DAG của mình, đây là một số điều bạn có thể làm:

- Làm cho DAG của bạn tải nhanh hơn. Thông thường việc kiểm tra quá trình load một dag có thể thực hiện qua câu lệnh chẳng hạn như sau `time python airflow/example_dag.py` kết quả sẽ cho thông tin về việc Airflow cần bao lâu để đọc được nội dung DAG đã được cài đặt. Đôi khi mã tối ưu và không tối ưu chỉ chênh nhau vài trăm mili giây, tuy nhiên nếu số lượng DAG tăng lên theo thời gian thì đây lại là vấn đề lớn đấy.
- Làm cho DAG của bạn tạo ra một cấu trúc đơn giản hơn. Rất rõ ràng ta có thể thấy rằng DAG có cấu trúc tuyến tính đơn giản `A-> B-> C` sẽ mất ít thời gian hơn trong quá trình lập lịch cũng như thực thi. Điều đó lại hoàn toàn ngược lại với một DAG có cấu trúc cây sâu với số lượng công việc phụ thuộc tăng theo cấp số nhân.
- Có số lượng DAG nhỏ hơn trên mỗi file cài đặt. Trong khi Airflow 2 được tối ưu hóa cho trường hợp có nhiều DAG trong một tệp, có một số phần của hệ thống làm cho nó ít hiệu quả hoặc gây ra nhiều sự chậm trễ hơn so với việc có những DAG đó chia thành nhiều tệp. Vậy nên, nếu bạn có nhiều DAG được tạo ra từ một tệp, hãy xem xét chia chúng nếu bạn quan sát thấy nó mất nhiều thời gian để phản ánh các thay đổi trong tệp DAG của bạn trong giao diện người dùng của Airflow.
- Don't repeat yourself: Thông thường ta sẽ có một hoặc một vài phần mã được sử dụng chung trong toàn bộ các DAG. Tất nhiên là nếu số lần lặp lại ít thì cũng chẳng làm sao cả, tuy nhiên tốt hơn hết ta nên tách chúng thành các Operator riêng biệt. Điều này sẽ giúp ta (1) quản lý source code dễ hơn, (2) kiểm thử tốt hơn, (3) tránh lặp code và hạn chế các vấn đề phát sinh trong quá trình phát triển

# Kiểm thử DAG

Airflow về cơ bản cũng là một phần mềm (mặc dù nó hơi lớn) Vậy nên để đảm bảo source code luôn hoạt động như những gì ta kì vọng cũng như giảm thiểu tối đa công sức thực hiện thì việc viết test cho chúng sẽ là cần thiết. Phần dưới đây mình sẽ liệt kê một số loại test mà mình hay viết (nếu rảnh)

# \***\*DAG Loader Test\*\***

Đầu tiên tất nhiên là chúng ta sẽ kiểm tra xem các file cài đặt DAG có thể load được thành công hay không. Cách đơn giản nhất sẽ là gọi lệnh `python đường_dẫn_tới_tệp_DAG_của_bạn.py`

và việc chạy lệnh trên mà không gây ra lỗi nào sẽ đảm bảo rằng DAG của bạn không chứa bất kỳ phụ thuộc chưa được cài đặt, lỗi cú pháp, vv.

Đây cũng là cách tuyệt vời để kiểm tra xem DAG của ta có tải nhanh hơn sau một tối ưu hóa hay không, nếu bạn muốn thử tối ưu hóa thời gian tải DAG. Đơn giản chạy DAG và đo thời gian nó mất, có nhiều cách để đo thời gian xử lý, một trong số đó trong môi trường Linux là sử dụng lệnh `time` tích hợp sẵn. Hãy đảm bảo chạy nó nhiều lần liên tiếp để tính đến các hiệu ứng bộ nhớ cache. So sánh kết quả trước và sau tối ưu hóa (trong cùng điều kiện - sử dụng cùng một máy, môi trường v.v.) để đánh giá tác động của việc tối ưu hóa`time python airflow/example_dag.py`

Một phiên bản nâng cấp hơn cho việc kiểm tra quá trình load các DAG sẽ là viết test bằng pytest chẳng hạn như sau:

```python
def get_import_errors():
    dag_bag = DagBag(include_examples=False)
    def strip_path_prefix(path):
        return os.path.relpath(path, os.environ.get("AIRFLOW_HOME"))
    return [(None, None)] + [
        (strip_path_prefix(k), v.strip()) for k, v in dag_bag.import_errors.items()
    ]

@pytest.mark.parametrize(
    "rel_path,rv", get_import_errors(), ids=[x[0] for x in get_import_errors()]
)
def test_file_imports(rel_path, rv):
    if rel_path and rv:
        raise Exception(f"{rel_path} failed to import with message \n {rv}")
```

Với đoạn mã trên đơn giản mình chỉ kiểm tra rằng liệu tất cả các DAG được cài đặt có được load thành công hay không, tuy vậy thì ta cũng có thể có thêm một số unit test khác nữa chẳng hạn như:

### Kiểm tra xem DAG có được khởi tạo đúng cách hay không

```python
import pytest
from airflow.models import DagBag


@pytest.fixture()
def dagbag():
    return DagBag()


def test_dag_loaded(dagbag):
    dag = dagbag.get_dag(dag_id="hello_world")
    assert dagbag.import_errors == {}
    assert dag is not None
		assert len(dag.tasks) == 1
```

### Kiểm tra xem cấu trúc DAG có giống như kì vọng hay không

```python
def assert_dag_dict_equal(source, dag):
    assert dag.task_dict.keys() == source.keys()
    for task_id, downstream_list in source.items():
        assert dag.has_task(task_id)
        task = dag.get_task(task_id)
        assert task.downstream_task_ids == set(downstream_list)


def test_dag():
    assert_dag_dict_equal(
        {
            "DummyInstruction_0": ["DummyInstruction_1"],
            "DummyInstruction_1": ["DummyInstruction_2"],
            "DummyInstruction_2": ["DummyInstruction_3"],
            "DummyInstruction_3": [],
        },
        dag,
    )
```

### Kiểm tra xem Custom Operator

```python
import datetimeimport pendulumimport pytestfrom airflow import DAG
from airflow.utils.state import DagRunState, TaskInstanceState
from airflow.utils.types import DagRunType

DATA_INTERVAL_START = pendulum.datetime(2021, 9, 13, tz="UTC")
DATA_INTERVAL_END = DATA_INTERVAL_START + datetime.timedelta(days=1)

TEST_DAG_ID = "my_custom_operator_dag"
TEST_TASK_ID = "my_custom_operator_task"


@pytest.fixture()
def dag():
    with DAG(
        dag_id=TEST_DAG_ID,
        schedule="@daily",
        start_date=DATA_INTERVAL_START,
    ) as dag:
        MyCustomOperator(
            task_id=TEST_TASK_ID,
            prefix="s3://bucket/some/prefix",
        )
    return dag


def test_my_custom_operator_execute_no_trigger(dag):
    dagrun = dag.create_dagrun(
        state=DagRunState.RUNNING,
        execution_date=DATA_INTERVAL_START,
        data_interval=(DATA_INTERVAL_START, DATA_INTERVAL_END),
        start_date=DATA_INTERVAL_END,
        run_type=DagRunType.MANUAL,
    )
    ti = dagrun.get_task_instance(task_id=TEST_TASK_ID)
    ti.task = dag.get_task(task_id=TEST_TASK_ID)
    ti.run(ignore_ti_state=True)
    assert ti.state == TaskInstanceState.SUCCESS
```

Nói chung là việc kiểm thử với Airflow khá là mất thời gian, nhất là khi các Operator thường tương tác với các dịch vụ bên ngoài, nếu không phải các loại database, S3 các thứ thì cũng là Slack để gửi report chẳng hạn. Trong những lúc này các bạn sẽ cần nhiều thứ hơn ví dụ như [mock](https://docs.python.org/dev/library/unittest.mock.html) các function để giả lập các chức năng chẳng hạn và tin tôi đi, bạn sẽ cảm ơn mình rất nhiều khi viết các hàm xử lý trong các custom operator thay vì là các `python_callable` riêng lẻ.

# Xử lý các phụ thuộc Python phức tạp/có xung đột

Airflow có nhiều Python dependencies và đôi khi chúng xung đột với các dependencies của các task. Để giải quyết vấn đề này, ta sẽ có một số các giải pháp như sau:

### Sử dụng `PythonVirtualenvOperator`

`PythonVirtualenvOperator` cho phép bạn tạo động một `virtualenv` mà chức năng gọi Python của bạn sẽ thực thi. Mỗi task có thể có một `virtualenv` Python độc lập (được tạo động mỗi lần nhiệm vụ được chạy) và có thể chỉ định tập yêu cầu chi tiết cần phải được cài đặt cho nhiệm vụ đó để thực thi.

Operator sẽ chịu trách nhiệm:

- Tạo virtualenv dựa trên môi trường của bạn
- Serializing hàm xử lý và chuyển nó cho trình thông dịch Python của `virtualenv` để thực thi
- Thực thi và lấy kết quả của chức năng gọi và đẩy nó qua Xcom nếu được chỉ định

Các lợi ích của operator là:

- Không cần phải chuẩn bị venv trước. Nó sẽ được tạo động trước khi nhiệm vụ được chạy và bị xóa sau khi hoàn tất.
- Bạn có thể chạy nhiều task với các nhiều bộ `dependencies` khác nhau trên cùng một worker, do đó tài nguyên bộ nhớ được tái sử dụng.

Tuy nhiên, có những giới hạn và chi phí phát sinh bởi operator này:

- Bởi vì hàm xử lý sẽ được serialize trước khi được thực thi nên chúng có thể sẽ có một số thay đổi không đáng có
- Operator thêm chi phí CPU, mạng và thời gian xử lý để chạy mỗi task vì chúng sẽ luôn tạo lại venv cũng cài đặt các thư viện cần có mỗi khi bắt đầu và xóa đi sau khi hoàn tất quá trình thực thi
- Các worker cần truy cập PyPI hoặc các kho lưu trữ riêng để cài đặt các `dependencies`.
- Việc tạo động `virtualenv` có thể dễ bị lỗi tạm thời.

### Sử dụng DockerOperator hoặc Kubernetes Pod Operator

DockerOperator và KubernetesPodOperator sẽ yêu cầu Airflow có quyền truy cập vào một Docker Engine hoặc cụm Kubernetes chạy các task trong các container tùy chỉnh, và điều đó sẽ có một số đặc điểm như sau:

Các lợi ích của việc sử dụng các operator này là:

- Bạn có thể chạy các task với các tập `dependencies` khác nhau của cả Python và cả các `dependencies` cấp hệ thống hoặc thậm chí là các task được viết bằng ngôn ngữ hoàn toàn khác nhau hoặc kiến trúc bộ xử lý khác nhau.
- Môi trường được sử dụng để chạy các nhiệm vụ được tận dụng các tối ưu hóa và sẽ luôn không thay đổi do bản chất của các container.
- Hoàn toàn đảm bảo được tính cô lập giữa các task cũng như các lần chạy khác nhau

Những điểm hạn chế:

- Bạn cần hiểu rõ hơn về cách thức hoạt động của Docker Containers hoặc Kubernetes.
- Khó test ở dưới local, do bạn khó có thể dựng một môi trường Kubernetes tương tự trên production để sử dụng
- Việc thực thi trong các container thường có một chi phí phát sinh, chủ yếu là liên quan đến việc giao tiếp với Docker Engine hoặc cụm Kubernetes. Và bên cạnh đó bạn cũng sẽ cần chuẩn bị trước một image chứa toàn bộ môi trường thực thi cũng như lưu trữ nó trên một registry để nó sẵn sàng được tải về khi cần.

# Tổng kết

Trên đây là một số điểm mình nghĩ rằng nên quan tâm khi sử dụng Airflow, tuy vậy thì Airflow vẫn là một framework cực kỳ đồ sộ và nó còn rất nhiều thứ cần quan tâm để có thể sử dụng nó một cách hiệu quả. Hy vọng rằng những thông tin trên sẽ giúp ích cho các bạn trong quá trình triển khai và sử dụng Airflow. Nếu bạn có bất kỳ thắc mắc hoặc ý kiến đóng góp nào, hãy để lại comment bên dưới nhé. Cảm ơn bạn đã dành thời gian đọc.
