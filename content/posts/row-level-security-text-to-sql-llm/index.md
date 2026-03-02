---
title: "Row Level Security + Text-to-SQL LLM"
date: 2026-02-07T22:15:00+07:00

categories:
  - Machine Learning
  - Một vài tip nhỏ

tags:
  - PostgreSQL
  - Row Level Security
  - LLM
  - Text-to-SQL
  - LlamaIndex
  - OpenVINO
---

Cuối tuần nào cũng thấy ai đó khoe "POC text-to-sql LLM trong 1 cuối tuần". Nghe hoành tráng, nhưng mình nghĩ đa phần là **demo nhỏ cho vui** thôi. Mình cũng nghịch thử một notebook kiểu vậy cho biết — và đúng là phần demo chạy ngon lành: người dùng hỏi bằng ngôn ngữ tự nhiên, LLM sinh SQL, hệ thống trả kết quả. Nhưng chưa được bao lâu thì vướng ngay câu chuyện **phân quyền**: nếu LLM sinh query "quá rộng" thì sao? Lúc đó mới thấy **Row Level Security (RLS)** là mảnh ghép rất đáng giá.

Trong bài này mình sẽ dựng một ví dụ end-to-end: Postgres + RLS + LlamaIndex + OpenVINO GenAI. Mục tiêu là bạn có thể copy/paste và chạy lại (reproduce) được.

> À nếu bạn thắc mắc bài viết này có phải viết bởi AI không thì câu trả lời là đúng rồi đấy, bài viết này được viết bởi GPT-5.2-Codex đỉnh cấp pro siêu cấp vũ trụ và được review bởi bố của các LLM — Claude Opus 4.6, còn mình chỉ ngồi bấm phím thôi. :D

## POC text-to-sql → vướng phân quyền

POC thì rất vui: hỏi tự nhiên, có SQL ngay. Nhưng cũng chính vì quá tiện nên rủi ro leak dữ liệu lại cao hơn:

- LLM có thể sinh query "quá rộng" — ví dụ bỏ mất điều kiện `WHERE sale_id = ...`.
- Với ứng dụng multi-tenant, không thể tin tưởng 100% vào prompt hay bộ lọc ở tầng application.

Nói thẳng ra: agent chỉ là "người viết query hộ", còn **phân quyền phải là một boundary thật sự ở tầng hạ tầng**.

## RLS là gì và hoạt động ra sao?

Theo docs của Postgres, RLS (Row Level Security) là cơ chế policy **lọc dữ liệu theo từng dòng**, áp dụng trực tiếp ở mức database — hoạt động song song với hệ thống quyền `GRANT` truyền thống. Dưới đây là vài ý quan trọng:

- Khi bật RLS bằng `ALTER TABLE ... ENABLE ROW LEVEL SECURITY`, **mọi truy cập (SELECT, INSERT, UPDATE, DELETE) đều phải được ít nhất một policy cho phép**. Nếu không có policy nào phù hợp, mặc định sẽ **deny** — tức là user không thấy và không sửa được dòng nào.
- Mỗi policy gồm hai biểu thức tùy chọn:
    - `USING`: quyết định **dòng hiện có nào được nhìn thấy hoặc thao tác** — áp dụng cho SELECT, UPDATE (chọn dòng cần sửa), và DELETE.
    - `WITH CHECK`: quyết định **dòng mới hoặc đã sửa có được ghi vào hay không** — áp dụng cho INSERT và UPDATE (kiểm tra giá trị sau khi sửa). Lưu ý: INSERT không có `USING` (vì không đọc dòng cũ), DELETE không có `WITH CHECK` (vì không tạo dòng mới).
- Biểu thức policy được **thêm vào query như một điều kiện lọc ẩn**, đánh giá trên từng dòng. Các điều kiện này thường được áp dụng trước `WHERE` của user query để tránh lộ dữ liệu qua user-defined function. (Ngoại lệ: các hàm hoặc toán tử được đánh dấu `LEAKPROOF` — tức là đảm bảo không rò rỉ thông tin qua side-channel — có thể được query optimizer đẩy lên trước policy.)
- Có thể tạo nhiều policy cho cùng một bảng. Mặc định, policy có kiểu **permissive** và được kết hợp với nhau bằng **OR**; policy kiểu **restrictive** thì kết hợp bằng **AND**.
- Superuser hoặc role có thuộc tính `BYPASSRLS` sẽ bỏ qua RLS. Table owner cũng bỏ qua trừ khi bật thêm `ALTER TABLE ... FORCE ROW LEVEL SECURITY`.

Nói nôm na: dù bạn chạy `SELECT * FROM orders` thì Postgres vẫn tự động ẩn những dòng không thuộc quyền truy cập của bạn.

## Vì sao không thể chỉ “tin” agent tự phân quyền?

Về lý thuyết, LLM/agent có thể được hướng dẫn (qua system prompt hoặc few-shot) để tự thêm điều kiện phân quyền vào SQL. Tuy nhiên, đây chỉ là **soft control** — dựa hoàn toàn vào prompt. Chỉ cần một bug nhỏ ở prompt template, tool routing, hoặc thay đổi model version là query có thể "vượt rào". Trong môi trường production, mình coi cách này là "nice-to-have", không phải "security boundary".

Ngược lại, RLS nằm ở tầng database nên trở thành **hard control**. Dù agent sinh SQL thiếu điều kiện, Postgres vẫn tự động chặn các dòng không được phép.

## Ví dụ ngắn

Giả sử bảng `orders` có cột `sale_id` để phân biệt nhân viên bán hàng. Khi user A hỏi "tổng doanh thu", LLM có thể sinh ra câu `SELECT SUM(amount) FROM orders` — không có `WHERE` nào cả. Nếu không có RLS, query này sẽ tính tổng **toàn bộ dữ liệu**. Nhưng khi bật RLS, Postgres tự động thêm điều kiện ẩn `sale_id = current_setting('app.current_sale_id')` — một hàm đọc biến session (gọi là **GUC** — Grand Unified Configuration) — nên kết quả chỉ gồm các đơn hàng thuộc user A.

## TL;DR kiến trúc

- Dữ liệu nằm trong Postgres.
- Mỗi user có `sale_id` khác nhau.
- RLS policy dùng **GUC** (biến cấu hình session của Postgres, đọc qua `current_setting()`) để filter `sale_id`.
- LLM sinh SQL qua `NLSQLTableQueryEngine` của LlamaIndex.
- Trước mỗi query, hệ thống tự động chạy `SET LOCAL app.current_sale_id = ...` để Postgres biết đang phục vụ user nào.

## 1) Chuẩn bị môi trường

### 1.1 PostgreSQL

Máy local có Postgres là OK. Ví dụ DSN:

```text
postgresql://admin:abcd1234@localhost:5432/agent_info
```

Bạn có thể đổi user/pass tùy ý, miễn đồng bộ trong code.

### 1.2 Python packages

```bash
pip install \
  sqlalchemy psycopg2-binary pydantic \
  llama-index phoenix \
  llama-index-llms-openvino-genai \
  llama-index-embeddings-openvino-genai
```

> Lưu ý: OpenVINO GenAI cần model weights đã export sang định dạng OpenVINO IR. Trong bài mình dùng **Phi-3.5-mini-instruct** (int4) cho LLM và **BGE** cho embedding.

## 2) Tạo DB, role, bảng, seed data

```python
from sqlalchemy import (
    BigInteger,
    Column,
    MetaData,
    Numeric,
    Table,
    Text,
    create_engine,
    text,
)

ADMIN_DSN = "postgresql://admin:abcd1234@localhost:5432/agent_info"
engine = create_engine(ADMIN_DSN)

with engine.begin() as conn:
    conn.execute(text("DROP USER IF EXISTS app_user"))
    conn.execute(text("CREATE USER app_user WITH PASSWORD 'abcd1234'"))

    conn.execute(text("GRANT CONNECT ON DATABASE agent_info TO app_user"))
    conn.execute(text("GRANT USAGE ON SCHEMA public TO app_user"))

metadata = MetaData()

orders = Table(
    "orders",
    metadata,
    Column("order_id", BigInteger, primary_key=True, autoincrement=True),
    Column("sale_id", Text, nullable=False),
    Column("yearmonth", Text, nullable=False),
    Column("amount", Numeric(14, 2), nullable=False),
)

metadata.drop_all(engine)
metadata.create_all(engine)

with engine.begin() as conn:
    conn.execute(
        orders.insert(),
        [
            {"sale_id": "sale_001", "yearmonth": "202401", "amount": 1_000_000},
            {"sale_id": "sale_001", "yearmonth": "202402", "amount": 1_500_000},
            {"sale_id": "sale_002", "yearmonth": "202401", "amount": 2_000_000},
            {"sale_id": "sale_002", "yearmonth": "202402", "amount": 2_500_000},
            {"sale_id": "sale_003", "yearmonth": "202401", "amount": 3_000_000},
        ],
    )
    conn.execute(text("GRANT SELECT ON orders TO app_user"))
```

## 3) Bật RLS + tạo policy theo GUC

Ý tưởng: mỗi session sẽ set biến `app.current_sale_id` (một custom GUC), và policy đọc giá trị biến đó để quyết định dòng nào được trả về.

```python
from sqlalchemy import text

with engine.begin() as conn:
    conn.execute(text("ALTER TABLE orders ENABLE ROW LEVEL SECURITY"))

    conn.execute(
        text(
            """
            DROP POLICY IF EXISTS sale_read_own_orders ON orders;
            """
        )
    )

    conn.execute(
        text(
            """
            CREATE POLICY sale_read_own_orders
            ON orders
            FOR SELECT
            USING (
                sale_id = current_setting('app.current_sale_id', true)::text
            );
            """
        )
    )
```

Test nhanh với user ứng dụng:

```python
from sqlalchemy import create_engine, text

engine_app = create_engine(
    "postgresql+psycopg2://app_user:abcd1234@localhost:5432/agent_info"
)

with engine_app.begin() as conn:
    conn.execute(text("SET LOCAL app.current_sale_id = 'sale_001'"))
    rows = conn.execute(text("SELECT * FROM orders")).fetchall()
    print("sale_001 sees:", rows)

with engine_app.begin() as conn:
    conn.execute(text("SET LOCAL app.current_sale_id = 'sale_002'"))
    rows = conn.execute(text("SELECT * FROM orders")).fetchall()
    print("sale_002 sees:", rows)

with engine_app.begin() as conn:
    rows = conn.execute(text("SELECT * FROM orders")).fetchall()
    print("anonymous sees:", rows)
```

Bạn sẽ thấy mỗi `sale_id` chỉ nhìn được data của mình, còn session không set biến ("anonymous") sẽ trả về **rỗng** vì không match policy nào.

## 4) Vì sao cần class `RowSecurityPolicy`?

Mình muốn hai thứ: **(1)** khai báo policy một lần, dễ đọc, dễ reuse; **(2)** mỗi query tự động chạy `SET LOCAL` mà không phải copy/paste ở nhiều chỗ.

Class `RowSecurityPolicy` giải quyết cả hai:

- **Parse** biểu thức `current_setting(...)::type` trong policy để biết cần set GUC nào, kiểu dữ liệu gì.
- **Sinh ra** `setup_func()` — một callable gắn vào transaction, tự động chạy `SET LOCAL` trước khi thực thi SQL.

Thay vì rải `SET LOCAL` khắp nơi, bạn chỉ cần gọi `policy.get_setup_func({"current_sale_id": "sale_001"})` và truyền kết quả vào database wrapper.

### Code

```python
import re
from dataclasses import dataclass
from functools import partial
from typing import Any, Callable, Dict, List, Optional, Tuple, Type

from pydantic import BaseModel, Field, model_validator
from sqlalchemy.engine.base import Connection
from sqlalchemy.sql.elements import TextClause
from sqlalchemy import text


@dataclass(frozen=True)
class _GUCRef:
    guc_name: str
    param_name: str
    pg_type: str
    python_type: Type
    raw_expression: str


class RowSecurityPolicy(BaseModel):
    name: str
    table: str
    commands: List[str] = Field(default_factory=lambda: ["ALL"])
    roles: List[str] = Field(default_factory=lambda: ["PUBLIC"])
    using: str | None = None
    with_check: str | None = None
    permissive: bool | None = None

    guc_refs: List[_GUCRef] = Field(default_factory=list)

    _CURRENT_SETTING_RE = re.compile(
        r"""
        current_setting
        \(
            \s*'(?P<guc>[a-zA-Z0-9_.]+)'\s*
            (?:,\s*true\s*)?
        \)
        \s*::\s*(?P<cast>[a-zA-Z0-9_]+)
        """,
        re.VERBOSE | re.IGNORECASE,
    )

    _PG_TO_PY_TYPE: Dict[str, Type] = {
        "int": int,
        "integer": int,
        "bigint": int,
        "smallint": int,
        "uuid": str,
        "text": str,
        "varchar": str,
        "boolean": bool,
        "bool": bool,
    }

    @model_validator(mode="after")
    def _extract_gucs(self) -> "RowSecurityPolicy":
        refs: Dict[str, _GUCRef] = {}

        def scan(expr: str):
            for m in self._CURRENT_SETTING_RE.finditer(expr):
                guc = m.group("guc")
                cast = m.group("cast").lower()

                if cast not in self._PG_TO_PY_TYPE:
                    raise ValueError(
                        f"Unsupported cast type '{cast}' for current_setting('{guc}')"
                    )

                refs[guc] = _GUCRef(
                    guc_name=guc,
                    param_name=guc.split(".")[-1],
                    pg_type=cast,
                    python_type=self._PG_TO_PY_TYPE[cast],
                    raw_expression=m.group(0),
                )

        if self.using:
            scan(self.using)

        if self.with_check:
            scan(self.with_check)

        self.guc_refs = list(refs.values())
        return self

    def get_setup_func(
        self,
        params: Dict[str, Any],
    ) -> Callable[[Connection], None]:
        args = []

        for ref in self.guc_refs:
            name = ref.param_name

            if name not in params:
                raise KeyError(
                    f"Missing GUC parameter '{name}' for policy '{self.name}'"
                )

            value = params[name]

            if value is None:
                raise ValueError(f"GUC parameter '{name}' cannot be NULL")

            if not isinstance(value, ref.python_type):
                raise TypeError(
                    f"GUC parameter '{name}' must be "
                    f"{ref.python_type.__name__}, "
                    f"got {type(value).__name__}"
                )

            args.append(
                (
                    text(f"SET LOCAL {ref.guc_name} = :{ref.param_name}"),
                    {name: value},
                )
            )

        def set_transaction_variable(
            conn: Connection, args: list[tuple[TextClause, dict]]
        ):
            for text_clause, param_dict in args:
                conn.execute(text_clause, param_dict)

        return partial(set_transaction_variable, args=args)
```

Khởi tạo policy object:

```python
policy = RowSecurityPolicy(
    name="sale_read_own_orders",
    table="orders",
    commands=["SELECT"],
    using="""
        sale_id = current_setting('app.current_sale_id', true)::text
    """,
)
```

## 5) SQLDatabase có RLS (tự động setup GUC mỗi query)

Class `RowLevelSecuritySQLDatabase` kế thừa `SQLDatabase` của LlamaIndex và override phương thức `run_sql`. Trước khi thực thi bất kỳ câu SQL nào (do LLM sinh ra), nó sẽ gọi `setup_funcs` để chạy `SET LOCAL` — đảm bảo RLS policy luôn được áp dụng đúng user.

```python
from typing import Dict, Tuple
from sqlalchemy import Engine
from sqlalchemy.exc import OperationalError, ProgrammingError
from sqlalchemy.sql.elements import TextClause
from llama_index.core import SQLDatabase


class RowLevelSecuritySQLDatabase(SQLDatabase):
    def __init__(
        self,
        engine: Engine,
        schema: str | None = None,
        metadata: MetaData | None = None,
        ignore_tables: List[str] | None = None,
        include_tables: List[str] | None = None,
        sample_rows_in_table_info: int = 3,
        indexes_in_table_info: bool = False,
        custom_table_info: dict | None = None,
        view_support: bool = False,
        max_string_length: int = 300,
        setup_funcs: Optional[list[Callable[[Connection], None]]] = None,
    ):
        super().__init__(
            engine,
            schema,
            metadata,
            ignore_tables,
            include_tables,
            sample_rows_in_table_info,
            indexes_in_table_info,
            custom_table_info,
            view_support,
            max_string_length,
        )
        self.setup_funcs = setup_funcs

    def run_sql(self, command: str) -> Tuple[str, Dict]:
        with self._engine.begin() as connection:
            if self.setup_funcs is not None:
                for setup_func in self.setup_funcs:
                    setup_func(connection)
            try:
                if self._schema:
                    command = command.replace("FROM ", f"FROM {self._schema}.")
                    command = command.replace("JOIN ", f"JOIN {self._schema}.")
                cursor = connection.execute(text(command))
            except (ProgrammingError, OperationalError) as exc:
                raise NotImplementedError(
                    f"Statement {command!r} is invalid SQL.\nError: {exc.orig}"
                ) from exc
            if cursor.returns_rows:
                result = cursor.fetchall()
                truncated_results = []
                for row in result:
                    truncated_row = tuple(
                        self.truncate_word(column, length=self._max_string_length)
                        for column in row
                    )
                    truncated_results.append(truncated_row)
                return str(truncated_results), {
                    "result": truncated_results,
                    "col_keys": list(cursor.keys()),
                }
        return "", {}
```

Test nhanh:

```python
engine_agent = create_engine(
    "postgresql+psycopg2://app_user:abcd1234@localhost:5432/agent_info"
)

sql_database = RowLevelSecuritySQLDatabase(
    engine_agent,
    include_tables=["orders"],
    setup_funcs=[policy.get_setup_func({"current_sale_id": "sale_001"})],
)
print(sql_database.run_sql("SELECT * FROM orders"))
```

## 6) Text-to-SQL với LLM (OpenVINO + LlamaIndex)

```python
import phoenix as px
import llama_index
from llama_index.core.query_engine import NLSQLTableQueryEngine
from llama_index.embeddings.openvino_genai import OpenVINOGENAIEmbedding
from llama_index.llms.openvino_genai import OpenVINOGenAILLM

px.launch_app()
llama_index.core.set_global_handler("arize_phoenix")

llm = OpenVINOGenAILLM(
    model_path="./weights/OpenVINO/Phi-3.5-mini-instruct-int4-ov/",
    device="CPU",
)

ov_embed_model = OpenVINOGENAIEmbedding(
    model_path="./weights/OpenVINO/bge_ov",
    device="CPU",
)

query_engine = NLSQLTableQueryEngine(
    llm=llm,
    embed_model=ov_embed_model,
    sql_database=sql_database,
    tables=["orders"],
)

query_str = "Sum total amount"
response = query_engine.query(query_str)
print(response.response)
```

Kết quả trả về chỉ tính trên dữ liệu của `sale_001`, vì `setup_funcs` đã set `current_sale_id = 'sale_001'` trước khi chạy SQL. Nếu đổi sang `sale_002`, con số sẽ khác hoàn toàn — đúng như kỳ vọng.

> `phoenix` (Arize Phoenix) là tool observability cho LLM. Dòng `set_global_handler("arize_phoenix")` giúp bạn trace được prompt, SQL sinh ra, và kết quả — rất tiện khi debug.

## Một vài lưu ý nhỏ

- Luôn dùng `SET LOCAL` (thay vì `SET`) để đảm bảo biến GUC chỉ tồn tại trong transaction hiện tại — tránh lẫn context giữa các request.
- Nếu LLM sinh SQL có `JOIN` nhiều bảng, nhớ bật RLS cho **từng bảng** liên quan, không chỉ bảng chính.
- Nếu muốn chống leak schema (LLM có thể suy luận cấu trúc DB từ error message), hãy giới hạn `include_tables` và cân nhắc redact `table_info`.
- Trong production, **không nên hard-code DSN** trong source code. Dùng biến môi trường hoặc secret manager.

## Hệ thống lớn làm RLS kiểu gì?

Ở mức khái quát, các nền tảng dữ liệu enterprise thường gắn RLS với hệ thống **identity** (SSO, IAM, IdP) và dùng thông tin user/role để apply policy ở tầng dữ liệu. Cách triển khai cụ thể tùy nền tảng, nhưng thường rơi vào hai hướng chính:

- **User principal / role membership**: policy filter trực tiếp dựa trên role hoặc username của session hiện tại.
- **Tenant mapping**: với kiến trúc multi-tenant, hệ thống map user → `tenant_id` (qua bảng mapping hoặc function), rồi policy filter dữ liệu theo `tenant_id`.

Ý chính: **xác thực ở identity provider, phân quyền ở data engine**. Agent/LLM chỉ là "người viết query hộ", không phải nơi quyết định ai được xem gì.

Nếu muốn tự xây, bạn có thể dựng một lớp data repository để mapping user → tenant rồi set context (GUC) trước mỗi query — tương tự cách mình làm ở trên. Không có công cụ nào fit mọi bài toán, nên phần này phụ thuộc kiến trúc và ràng buộc của từng team.

## Ngoài RLS thì còn phân quyền gì?

Trong các hệ thống data lớn, RLS chỉ là một lớp trong mô hình phân quyền nhiều tầng. Thường còn có:

- **Role-based access control (RBAC)**: phân quyền theo role — ai được query, ai được write, ai được admin. Đây là nền tảng cơ bản nhất.
- **Column-level security (CLS)**: ẩn hoặc giới hạn quyền đọc một số cột nhạy cảm (PII, lương, số tài khoản).
- **Object-level permission**: quyền truy cập theo từng object cụ thể — schema, table, view, function.
- **Data masking / Redaction**: che một phần dữ liệu khi trả về (ví dụ chỉ hiện 4 số cuối của số điện thoại).
- **Audit / Logging**: ghi log mọi truy cập để phục vụ truy vết (forensics) và tuân thủ (compliance).

Tùy hệ thống, các lớp này có thể nằm ở data engine, data catalog, API gateway, hoặc kết hợp nhiều nơi.

## Kết luận

Với text-to-sql, phân quyền **không thể để agent tự lo** — đó là soft control, không đủ tin cậy. RLS là lớp bảo vệ cứng, đặt đúng chỗ ở tầng database. Kết hợp thêm một lớp mapping user/tenant và audit logging, bạn sẽ có hệ thống vừa dễ dùng (user hỏi tự nhiên), vừa an toàn (DB tự chặn data trái phép).

Bài viết đến đây là hết. Cảm ơn mọi người đã dành thời gian đọc!

## Tài liệu tham khảo

- https://www.postgresql.org/docs/current/ddl-rowsecurity.html
- https://www.postgresql.org/docs/current/sql-createpolicy.html
- https://www.postgresql.org/docs/current/functions-admin.html#FUNCTIONS-ADMIN-SET
- https://docs.llamaindex.ai/
- https://docs.llamaindex.ai/en/stable/api_reference/query_engine/nl_sql_table_query_engine/
- https://docs.llamaindex.ai/en/stable/api_reference/llms/openvino_genai/
- https://docs.llamaindex.ai/en/stable/api_reference/embeddings/openvino_genai/
- https://www.intel.com/content/www/us/en/developer/tools/openvino-toolkit/overview.html