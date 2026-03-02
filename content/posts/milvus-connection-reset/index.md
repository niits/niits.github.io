---
title: "Debug lỗi gRPC connection reset khi dùng Milvus qua Traefik reverse proxy"
date: 2026-03-01
categories: ["Một vài tip nhỏ"]
tags: ["Milvus", "Traefik", "gRPC", "Docker", "Vector Database"]
---

Nếu bạn đang chạy Milvus phía sau Traefik và thỉnh thoảng client cứ văng ra lỗi `Fail connecting to server` một cách bí ẩn thì bài viết này dành cho bạn. Mình đã dành kha khá thời gian để tìm ra nguyên nhân và fix cái lỗi này, nên viết lại đây để lỡ ai đó gặp phải thì đỡ phải đau đầu như mình `(ﾒ﹏ﾒ)`

## Bối cảnh

Mình đang chạy một con **Milvus Standalone v2.6** trên một VM 8GB RAM, dùng để lưu trữ vector embeddings cho một dự án fact-checking. Kiến trúc thì khá cơ bản:

- **Traefik v3** làm reverse proxy, xử lý TLS (Let's Encrypt) cho tất cả các service
- **Milvus Standalone** đứng sau Traefik, expose gRPC qua port 19530
- **etcd** + **MinIO** làm metadata store và object storage cho Milvus
- Tất cả chạy bằng Docker Compose trên cùng một máy

Client PyMilvus từ máy local kết nối đến Milvus thông qua domain `milvus.vm.trungtd.work:19530`, TLS được terminate tại Traefik. Nhìn thì có vẻ đơn giản nhưng cuộc sống đâu có dễ dàng như vậy `=))))`

## Vấn đề gặp phải

Khi chạy script batch insert khoảng 5700 documents vào Milvus (chia thành các batch 100 docs), mọi thứ chạy ngon lành được một lúc rồi đột nhiên văng lỗi:

```
pymilvus.exceptions.MilvusException: <MilvusException: (code=2, message=Fail
connecting to server on milvus.vm.trungtd.work:19530, illegal connection params
or server unavailable)>
```

Điều khó chịu là lỗi này không xảy ra ngay từ đầu. Script chạy được chục batch đầu tiên ngon lành cành đào, rồi tự nhiên fail, rồi lại chạy tiếp được, rồi lại fail. Cứ như đánh bạc vậy, không biết batch nào sẽ crash `(；⌣̀_⌣́)`

Nhìn vào progress bar thì thấy pattern khá rõ:

```
Indexing documents batch...: 100%| 100/100 [00:06<00:00, 15.12it/s]
Indexing documents batch...: 100%| 100/100 [00:06<00:00, 15.09it/s]
Indexing documents batch...:   0%|          0/100 [00:00<?, ?it/s]
```

## Phân tích nguyên nhân

### Bước 1: Kiểm tra Milvus có đang sống không

Đầu tiên là check xem container Milvus có bị crash hay restart không:

```bash
$ docker compose ps
NAME                STATUS                STATE
milvus-standalone   Up 2 days (healthy)   running
traefik             Up 2 days             running
milvus-etcd         Up 2 days (healthy)   running
milvus-minio        Up 2 days (healthy)   running
```

Tất cả đều `healthy`, Up 2 days. Vậy là Milvus không hề bị crash hay restart. Hmm `(¬_¬")`

### Bước 2: Kiểm tra log Milvus

```bash
$ docker logs milvus-standalone 2>&1 | \
    grep -iE "error|OOM|panic|memory" | tail -20
```

Không có bất kỳ dấu hiệu OOM, panic hay error nghiêm trọng nào. Milvus vẫn đang xử lý segment, compaction, audit bình thường. Có một vài warning về `collection schema mismatch` nhưng đó là do client đang dùng schema cũ và Milvus đã tự handle việc đó.

### Bước 3: Kiểm tra log Traefik — Đây mới là thủ phạm

```bash
docker logs traefik 2>&1 | tail -30
```

Và boom, nguyên một bãi chiến trường hiện ra:

```
ERR Error while handling TCP connection error="readfrom tcp
172.18.0.2:37228->172.18.0.5:19530: read tcp 172.18.0.2:19530->1.54.42.162:17571:
read: connection reset by peer"

ERR Error while handling TCP connection error="readfrom tcp
172.18.0.2:33002->172.18.0.5:19530: read tcp 172.18.0.2:19530->1.54.42.162:24417:
read: connection timed out"
```

Hàng loạt lỗi `connection reset by peer` và `connection timed out` trên port 19530. Và pattern rất rõ ràng: cứ mỗi lần lỗi là đi theo nhóm 4 connections (đúng bằng số connections trong connection pool mặc định của PyMilvus) `(╯°□°)╯︵ ┻━┻`

Vậy là rõ rồi. **Milvus hoàn toàn bình thường, thủ phạm chính là Traefik đang ngắt TCP connection giữa chừng.**

## Nguyên nhân gốc rễ

Khi phân tích kỹ hơn, mình xác định được **hai nguyên nhân chính** dẫn đến việc connection bị drop:

### 1. Traefik TCP idle timeout

Traefik mặc định có các timeout cho TCP connections. Khi client hoàn thành một batch insert và chuẩn bị gửi batch tiếp theo, connection rơi vào trạng thái idle. Nếu khoảng thời gian giữa hai batch đủ lớn (do processing, encoding, v.v.), Traefik sẽ tự động đóng connection → batch tiếp theo gửi đến sẽ gặp lỗi `connection reset by peer`.

Điều này giải thích tại sao lỗi không xảy ra ở mọi batch — nó phụ thuộc vào thời gian idle giữa hai batch có vượt quá ngưỡng timeout của Traefik hay không.

### 2. Thiếu gRPC keepalive

gRPC protocol có cơ chế keepalive để duy trì long-lived connections. Tuy nhiên, Milvus server mặc định không gửi keepalive ping đủ thường xuyên. Khi connection đi qua Traefik (một TCP proxy layer ở giữa), nếu không có traffic nào trên connection trong một khoảng thời gian, Traefik sẽ coi connection đó là "chết" và ngắt nó đi.

Nói đơn giản thì cái flow nó như thế này:

```
PyMilvus Client ──TLS──> Traefik ──TCP──> Milvus Server
       │                    │                   │
       │   (idle gap)       │                   │
       │                    │──timeout──> DROP!  │
       │                    │                   │
       │── next batch ──> CONNECTION REFUSED     │
```

## Giải pháp

### Phía Traefik: Tắt TCP timeout cho entrypoint gRPC

Thêm cấu hình disable read/write timeout cho entrypoint `milvus-grpc`:

```yaml
traefik:
  command:
    # ... các config khác ...
    - "--entrypoints.milvus-grpc.address=:19530"
    # Tắt timeout cho gRPC long-lived connections
    - "--entrypoints.milvus-grpc.transport.respondingTimeouts.readTimeout=0s"
    - "--entrypoints.milvus-grpc.transport.respondingTimeouts.writeTimeout=0s"
```

Giá trị `0s` nghĩa là "không bao giờ timeout", phù hợp cho gRPC vì gRPC connections được thiết kế để sống lâu (long-lived) và tự quản lý lifecycle thông qua keepalive.

> **Lưu ý:** Chỉ nên tắt timeout cho entrypoint gRPC chuyên dụng. Đừng tắt timeout cho entrypoint HTTP (`web`/`websecure`) vì sẽ tạo ra lỗ hổng bảo mật (slow loris attack, v.v.)

### Phía Milvus: Bật gRPC keepalive

Thêm các biến môi trường sau vào service Milvus standalone:

```yaml
standalone:
  environment:
    # ... các config khác ...
    # gRPC keepalive - giữ connection sống qua Traefik
    MILVUS_PROXY_GRPC_SERVERKEEPALIVETIME: "60"
    MILVUS_PROXY_GRPC_SERVERKEEPALIVE_TIMEOUT: "20"
    MILVUS_PROXY_GRPC_SERVERKEEPALIVEENFORCEMENT_MINTIME: "10"
    MILVUS_PROXY_GRPC_SERVERKEEPALIVEENFORCEMENT_PERMITWITHOUTSTREAM: "true"
    MILVUS_PROXY_GRPC_SERVERMAXTIMEOUT: "600"
```

Giải thích từng config:

| Biến môi trường | Giá trị | Ý nghĩa |
|---|---|---|
| `SERVERKEEPALIVETIME` | `60` | Server gửi keepalive ping mỗi 60 giây |
| `SERVERKEEPALIVE_TIMEOUT` | `20` | Chờ 20 giây cho keepalive response trước khi coi connection là chết |
| `SERVERKEEPALIVEENFORCEMENT_MINTIME` | `10` | Cho phép client gửi keepalive ping tối thiểu mỗi 10 giây |
| `PERMITWITHOUTSTREAM` | `true` | Cho phép keepalive ngay cả khi không có active RPC stream |
| `SERVERMAXTIMEOUT` | `600` | Tăng max timeout cho mỗi gRPC call lên 10 phút |

Config `PERMITWITHOUTSTREAM: true` là quan trọng nhất trong trường hợp này. Mặc định gRPC server sẽ không gửi keepalive khi không có active stream, nhưng giữa các batch insert thì connection đang idle (không có stream nào), dẫn đến Traefik không nhận được traffic nào và cắt connection.

### Phía client: Đảm bảo dùng TLS

Vì Traefik terminate TLS trên port 19530 (qua `tls.certresolver=letsencrypt`), client **bắt buộc** phải kết nối bằng TLS. Nếu không, TLS handshake sẽ fail ngay → `connection reset by peer`.

```python
from pymilvus import MilvusClient

# Cách 1: Dùng URI scheme https
client = MilvusClient(
    uri="https://milvus.vm.trungtd.work:19530",
    token="user:password"
)

# Cách 2: Dùng connections.connect với secure=True
from pymilvus import connections
connections.connect(
    host="milvus.vm.trungtd.work",
    port="19530",
    secure=True,
    user="root",
    password="your_password",
)
```

## Áp dụng và kiểm tra

Sau khi sửa `docker-compose.yml`, áp dụng thay đổi:

```bash
docker compose up -d
```

Docker Compose sẽ tự detect các container nào có config thay đổi và chỉ recreate những container đó (trong trường hợp này là `traefik` và `milvus-standalone`).

Để verify Traefik đã nhận config mới:

```bash
docker logs traefik 2>&1 | grep "milvus-grpc"
```

Và sau đó chạy lại script batch insert. Nếu mọi thứ ok thì Traefik log sẽ không còn hiện `connection reset by peer` nữa `☆*:.｡.o(≧▽≦)o.｡.:*☆`

## Tổng kết

Bài viết này ghi lại quá trình debug lỗi `Fail connecting to server` khi dùng PyMilvus kết nối đến Milvus qua Traefik reverse proxy. Tóm lại thì:

- **Milvus không hề có lỗi** — server vẫn healthy và xử lý tốt
- **Traefik là thủ phạm** — TCP idle timeout khiến connection bị ngắt giữa các batch
- **Fix bao gồm 3 phần**: tắt TCP timeout cho Traefik gRPC entrypoint, bật gRPC keepalive trên Milvus server, và đảm bảo client dùng TLS

Nếu bạn đang triển khai Milvus (hoặc bất kỳ gRPC service nào) phía sau một reverse proxy như Traefik, Nginx hay HAProxy, thì hãy luôn nhớ rằng gRPC không phải HTTP thông thường — nó cần long-lived connections và keepalive để hoạt động ổn định qua các proxy layer.

Bài viết đến đây là hết, cảm ơn mọi người đã dành thời gian đọc `(b ᵔ▽ᵔ)b`

## Tài liệu tham khảo

- [Traefik TCP Routers](https://doc.traefik.io/traefik/routing/routers/#configuring-tcp-routers)
- [Traefik EntryPoints Transport](https://doc.traefik.io/traefik/routing/entrypoints/#transport)
- [gRPC Keepalive](https://grpc.io/docs/guides/keepalive/)
- [Milvus Configuration](https://milvus.io/docs/configure-docker.md)
- [PyMilvus Documentation](https://milvus.io/api-reference/pymilvus/v2.4.x/About.md)
