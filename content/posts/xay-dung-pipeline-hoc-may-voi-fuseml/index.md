---
date: '2022-05-28T08:29:11+07:00'
description: Vậy là chỉ 2, 3 tháng nữa là lại đến một mùa bảo vệ khóa luận/đồ án. Trên tình thần hỗ trợ các anh chị em bịa thêm một chương trong báo cáo của mình, bài viết này giới thiệu về giải pháp được cung cấp bởi SUSE để xây dựng một pipeline linh hoạt và hiệu suất cao nhằm đáp ứng xây dựng cũng như triển khai các mô hình học máy.

layout: post
title: "X\xE2y d\u1EF1ng pipeline h\u1ECDc m\xE1y v\u1EDBi FuseML"

categories:
  - Một vài tip nhỏ

tags:
  - Machine Learning
  - MLOps
  - DevOps
---

Vậy là chỉ 2, 3 tháng nữa là lại đến một mùa bảo vệ khóa luận/đồ án. Trên tình thần hỗ trợ các anh chị em bịa thêm một chương trong báo cáo của mình, bài viết này giới thiệu về giải pháp được cung cấp bởi SUSE để xây dựng một pipeline linh hoạt và hiệu suất cao nhằm đáp ứng xây dựng cũng như triển khai các mô hình học máy.

Ok nẹt gâu.

# Giới thiệu về FuseML

Việc xây dựng hệ thống Machine Learning hoạt động trên môi trường production thành công đòi hỏi phải diễn giải lại một cách chuyên biệt các phương pháp và văn hóa DevOps truyền thống. MLOps, viết tắt của **M**achine **L**earning **Op**eration**s**, là một ngành kỹ thuật tương đối mới và một tập hợp các phương pháp nhằm cải thiện sự hợp tác và giao tiếp giữa các vai trò và nhóm khác nhau cùng quản lý vòng đời từ đầu đến cuối của các dự án học máy.

Được cung cấp bởi [`SUSE`](https://www.suse.com/), [`FuseML`](https://fuseml.github.io/docs/v0.3/) được xây dựng để trở thành `một bộ điều phối MLOps được cung cấp bởi một khuôn khổ linh hoạt được thiết kế cho các hoạt động nhất quán và một bộ sưu tập phong phú các công thức tích hợp phản ánh các trường hợp sử dụng trong thế giới thực giúp bạn giảm nợ kỹ thuật và tránh bị vendor lock-in.` Các nguyên tắc mà `FuseML` tuân theo bao gồm:

- **Tính linh hoạt**: tạo và quản lý quy trình làm việc MLOps động kết nối các công cụ AI / ML khác nhau trên nhiều miền cơ sở hạ tầng

- **Khả năng mở rộng**: tận dụng tập hợp các cơ chế mở rộng và trừu tượng của FuseML để thêm hỗ trợ cho các công cụ AI / ML

- **Khả năng tương thích**: xây dựng các quy trình làm việc MLOps phức tạp cho các dự án của bạn từ các khối xây dựng có thể kết hợp thực hiện một loạt các chức năng học máy

- **Cộng tác** : sử dụng các công thức tích hợp công cụ và tự động hóa MLOps được tạo ra với sự cộng tác của tất cả các vai trò trong nhóm AI / ML - Nhà khoa học dữ liệu, Kỹ sư dữ liệu và Kỹ sư DevOps

- **GitOps cho Machine learning**: áp dụng các nguyên tắc chính mà GitOps dựa trên, chẳng hạn như lập phiên bản và mô hình khai báo cho các khái niệm cụ thể cho máy học - ví dụ: mô hình, bộ dữ liệu, quy trình làm việc - để cung cấp các tính năng như lập phiên bản tạo tác đầu cuối và dòng , theo dõi, khả năng tái tạo và tự động hóa phản ứng

![](https://fuseml.github.io/docs/v0.3/fuseml-mlops-anim-fade.gif)

Với phiên bản hiện tại là `v0.3`, khả năng FuseML sẽ còn đập đi xây lại khá nhiều. Tuy vậy việc xây dựng một công cụ như vậy khá phù hợp với xu hướng phát triển hiện tại và có thể nói đây là quyết định rất đúng đắn của đội ngũ đến từ SUSE (để mình có tool free mà dùng :v ) Thông tin quảng cáo về framework này có thể đọc tại <https://www.suse.com/c/rancher_blog/accelerating-machine-learning-with-mlops-and-fuseml-part-one/>

# Cài đặt FuseML

## Đăng ký tài khoản DigitalOcean

Giới thiệu qua qua thì DigitalOcean là nhà cung cấp dịch vụ máy chủ ảo khá mới trên thị trường. Các dịch vụ Cloud Server VPS của họ dựa trên nền tảng điện toán đám mây giúp triển khai và mở rộng ứng dụng chạy đồng thời trên nhiều máy tính với các tính năng tối ưu từ đám mây (Cloud). Sẽ chẳng có gì để nói nếu như DigitalOcean có cho người dùng $100 để vọc thử dịch vụ của họ (nếu tạo tài khoản mới và thêm thẻ credit/debit) trong hai tháng. Đây cũng là thời gian khá dài, đủ để các bạn có thể múa qua mùa khóa luận :v:

![](images/5b7e7ce3-44aa-4a46-b6a7-3fbff47545d8.png)

## Tạo Kubernetes cluster

![](images/b75964ca-a209-4201-b544-3c7928b23422.png)

Tiếp đó vào phần `Kubernetes`  để tạo cụm với cấu hình tùy chọn. Mặc định DigitalOcean sẽ chỉ cho ta chọn tối đa 3 node trong cụm. Nếu cần thêm mọi người có thể gửi yêu cầu tăng số lượng node cho họ.

![](images/f7f64a0a-b92d-49a6-a2d7-1cb8b07e417f.png)

Sau khi tạo, ta sẽ được điều hướng đến trang tổng quan về cụm Kubernetes vừa tạo. Ở đây, chúng ta có thể tải về file config để giao tiếp với cluster cũng như sửa đổi một số cài đặt các thứ. Quá trình khởi tạo cụm này sẽ khá là lâu, chúng ta sẽ biết được quá trình này đã hoàn tất khi thanh thể hiện tiến trình đạt được 100%

![](images/5c8235cf-2eaf-4262-944e-748777c70431.png)

Tiếp đó DigitalOcean có hỗ trợ chúng ta cài một số  thành phần phổ biến bên tab `Add-ons`. Ở đây mình định cài thêm `Cert-Manager` do sau này có thể dùng đến nhưng mà đọc docs của `FuseML` thì có rồi nên hoi :v.

![](images/ead56702-3275-47a6-8c85-57226e6cf6fd.png)

## Cài đặt FuseML

Với tinh thần support user tận răng, FuseML cung cấp cho chúng ta bộ installer mà để dựng cả framework này ta chỉ cần dùng đúng một câu lệnh là `
curl -sfL https://raw.githubusercontent.com/fuseml/fuseml/main/install.sh | sh -`. Tất nhiên là cái này dùng trên Linux thôi còn cài trên Windows chắc lằng nhằng hơn :v:

Tiếp đó, ta cài đặt các thành phần cần thiết bằng câu lệnh `fuseml-installer install`. Kết quả thu được sẽ trông khá giống với phần ở trong trang [docs](https://fuseml.github.io/docs/v0.3/quickstart/#3-install-fuseml) và ta sẽ thấy rằng một tá các dịch vụ ở địa chỉ dạng `http://*.159.89.210.216.nip.io` trong đó một số thành phần đáng chú ý được cài đặt bao gồm:

- **Workloads**: Các thành phần phục vụ cho việc chạy các workflow (mình đoán thế :v )
- **Gitea**: một dịch vụ Git tự chạy, chạy ở <http://gitea.159.89.210.216.nip.io>
- **Registry**:  Registry cho cái gì đó, chắc là các container image
- **Tekton**: Một opensource để tạo các hệ thống CI/CD, chạy tại <http://tekton.159.89.210.216.nip.io>
- **FuseML core**: thành phần chính của FuseML, chạy tại <http://fuseml-core.159.89.210.216.nip.io>

Chắc là sau này FuseML sẽ cho phép người dùng tích hợp với các dịch vụ khác như kiểu Github hoặc các dịch vụ CI/CD chẳng hạn. Tuy nhiên đấy là việc cả các phiên bản sau, giờ thì có gì dùng đấy đừng có đòi hỏi. À nếu bạn thắc mắc là sao không thấy có cái gì liên quan đến TLS ở đây thì xin chia buồn là issue này vẫn nằm vật vờ trong backlog, chi tiết có thể xem tại [Enable TLS for exposed fuseml services #119](https://github.com/fuseml/fuseml/issues/119).

## Tạo workflow

### Cấu trúc mainfest của một workflow

Quy trình công việc FuseML là các quy trình tự động hóa được xây dựng từ các bước riêng lẻ, có thể tái sử dụng, được kết nối với nhau để tạo thành một đường ống. Chúng là một cách để tự động hóa quá trình tạo và triển khai các mô hình ML. Chạy quy trình làm việc dẫn đến việc khởi chạy một số công việc và dịch vụ được tạo và thực thi theo đúng thứ tự để tạo ra kết quả đầu ra mong muốn. Mỗi bước trong quy trình làm việc được biểu thị bằng hình ảnh vùng chứa triển khai một chức năng cụ thể trong vòng đời MLOps.

FuseML workflow được định cấu hình bằng cú pháp YAML khai báo như ví dụ dưới đây:

```yaml
name: mlflow-e2e
description: cái gì gì đó
inputs:
  - ...
outputs:
  - ...
steps:
  - ...
```

Bỏ qua `name` và `description`, thành phần đầu tiên của phần cấu hình trên là `inputs`, bao gồm các thông tin được truyền vào trong quá trình chạy của workflow, ví dụ như sau:

```yaml
inputs:
  - name: mlflow-codeset
    description: an MLFlow compatible codeset
    type: codeset
  - name: predictor
    description: type of predictor engine
    type: string
    default: auto
```

Tiếp đến là thành phần `outputs`, trả về thông tin mà như ví dụ dưới đây là địa chỉ phục vụ cho việc gửi các yêu cầu đoán nhận:

```yaml
outputs:
  - name: prediction-url
    description: "The URL where the exposed prediction service endpoint can be contacted to run predictions."
    type: string
```

Cuối cùng là thành phần `steps` định nghĩa các bước cần được thực hiện:

```yaml
steps:
  - name: builder
    image: ghcr.io/fuseml/mlflow-builder:v0.3.0
    inputs:
      - name: mlflow-codeset
        codeset:
          name: "{{ inputs.mlflow-codeset }}"
          path: /project
    outputs:
      - name: image
- name: trainer
    ...

```

### FuseML workflow để triển khai mô hình bằng cách sử dụng KServe

OK lý thuyết thế là đủ rồi. Ví dụ này cho thấy cách FuseML có thể được sử dụng để tự động hóa quy trình học máy từ đầu đến cuối bằng cách sử dụng kết hợp các công cụ khác nhau. Trong trường hợp này, ta sẽ sử dụng một mô hình ML scikit-learning đang được đào tạo bằng cách sử dụng [MLflow](https://mlflow.org/) và sau đó được phục vụ với [KServe](https://kserve.github.io/website/0.8/).

#### Cài đặt các công cụ ML

Chạy ví dụ này yêu cầu MLflow và KServe phải được cài đặt trong cùng một cụm với FuseML, vậy nên ta cài đặt các thành phần bổ sung thông qua vieejv sử dụng câu lệnh sau:

```bash
fuseml-installer extensions --add mlflow,kserve
```

#### Chuẩn bị biến môi trường

Lưu ý rằng FuseML sẽ được địa chỉ của server từ biến `FUSEML_SERVER_URL` nhằm tương tác với các dịch vụ đang chạy. Bởi vậy ta đặt biến môi trường này bằng câu lệnh như sau:

```bash
export FUSEML_SERVER_URL=http://$(kubectl get VirtualService -n fuseml-core fuseml-core -o jsonpath="{.spec.hosts[0]}")
```

#### Đẩy code lên Gitea

Do FuseML chưa hỗ trợ các dịch vụ git khác nên ta push code cho quá trình training và đóng gói lên Gitea thông qua hai buớc như sau:

- Clone code mẫu về:

```bash
git clone --depth 1 -b release-0.3 https://github.com/fuseml/examples.git
cd examples
```

- Đăng ký codeset lên Gitea

```bash
fuseml codeset register --name "mlflow-test" --project "mlflow-project-01" codesets/mlflow/sklearn
```

Kết quả thu được là ta có source code đã được push lên tại <http://gitea.159.89.210.216.nip.io/mlflow-project-01/mlflow-test.git> Trong trường hợp bạn vào trang này mà không thấy gì thì lý do là do hết tiền nên tôi hủy cụm k8s đi rồi :(

#### Tạo workflow

FuseML workflow sẽ được tạo bao gồm trong kho lưu trữ ví dụ đại diện cho ML pipeline từ đầu đến cuối hoàn chỉnh "tương thích" với bất kỳ bộ mã nào có chứa MLProject. Nó bao gồm tất cả các bước cần thiết để đào tạo một mô hình với MLflow, lưu mô hình và sau đó tạo dịch vụ dự đoán KServe cho nó. Ở đây ta dùng luôn code mẫu tại <https://github.com/fuseml/examples/blob/main/workflows/mlflow-e2e.yaml>. Workflow kì vọng sẽ được tạo thông qua câu lệnh:

```bash
fuseml workflow create workflows/mlflow-e2e.yaml
```

Sau khi thấy workflow đã được tạo tại <http://tekton.159.89.210.216.nip.io/#/namespaces/fuseml-workloads/pipelines/mlflow-e2e>, bước tiếp theo cần được thực hiện sẽ là gán workflow cho codeset đã tạo ở trên thông qua câu lệnh:

```bash
fuseml workflow assign --name mlflow-e2e --codeset-name mlflow-test --codeset-project mlflow-project-01
```

Kết quả thu được là ta có một `PipelineRun` có thể kiểm tra trạng thái tại <http://tekton.159.89.210.216.nip.io/#/namespaces/fuseml-workloads/pipelineruns/fuseml-mlflow-project-01-mlflow-test-7sfx4?pipelineTask=clone> Trong trường hợp bạn thích dùng terminal thì ta có thể kiểm tra trạng thái thông qua câu lệnh `fuseml workflow list-runs --name mlflow-e2e`, kết quả thu được sẽ trông như sau:

```bash
+--------------------------------------------+------------+--------------+----------+---------+
| NAME                                       | WORKFLOW   | STARTED      | DURATION | STATUS  |
+--------------------------------------------+------------+--------------+----------+---------+
| fuseml-mlflow-project-01-mlflow-test-7sfx4 | mlflow-e2e | 1 minute ago | ---      | Running |
+--------------------------------------------+------------+--------------+----------+---------+
```

Khi quá trình chạy thành công, giá trị trạng thái sẽ thay đổi thành Succeeded trong CLI và ta thu được một số thứ như sau:

- Các artifact được push lên model registry, có thể xem tại [http://mlflow.159.89.210.216.nip.io/](http://mlflow.159.89.210.216.nip.io/#/experiments/1/runs/e5bf83475e2946298c81fb678609ae05)
- Một inference service mới được tạo và đăng ký dưới dạng ứng dụng FuseML có URL là <http://mlflow-project-01-mlflow-test-mlflow-e2e.fuseml-workloads.159.89.210.216.nip.io/v2/models/mlflow-project-01-mlflow-test-mlflow-e2e/infer>

#### Sử dụng  ứng dụng web để kiểm tra mô hình đã triển khai

Tất nhiên là gửi yêu cầu bằng cURL hay Postman thì  cũng không uy tín cho lắm thế nên ta có thể sử dụng một ứng dụng đơn giản được code cho ví dụ này bằng cách sử dụng streamlit. Để làm được điều đó ta triển khai ứng dụng vào cụm kubernetes của bạn bằng cách chạy lệnh sau:

```bash
kubectl apply -f webapps/winery/service.yaml
```

Chạy lệnh sau để kiểm tra trạng thái triển khai ứng dụng:

```bash
kubectl get ksvc -n fuseml-workloads winery
```

Tại một thời điểm nào đó, nó sẽ đạt đến `READY` trạng thái và một URL được cung cấp để truy cập ứng dụng mà trong phần thực hiện của tôi nó sẽ là <http://winery.fuseml-workloads.159.89.210.216.nip.io/>. Giao diện của web app này sẽ giống dưới đây còn mọi người tự vào URL kia để test thử (nếu nó còn sống) nhé.

![](images/089c96f9-3e43-408b-9aa9-8dbad38f7888.png)

# Tổng kết

Bài viết này giới thiệu về `FuseML`, một framework điều phối mã nguồn mở cho MLOps. Thông qua phần giới thiệu ở trên, ta cũng có thể thấy rằng framework này vẫn còn thiếu khá nhiều thứ để thực sự đáp ứng được các yêu cầu phát sinh trong quá trình xây dựng một dịch vụ ML. Tuy vậy, việc được phát triển bài bản cũng như hậu thuẫn bởi SUSE (có thể nổi tiếng với Rancher và các sản phẩm khác), ta có thể kì vọng đây sẽ là bộ giải pháp hữu ích và có thể sử dụng hiệu quả trong tương lai. Bài viết của mình đến đây là kết thúc, cảm ơn mọi người đã dành thời gian đọc.
