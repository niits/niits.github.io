---
title: Kubernetes
categories:
- LFS158x - Introduction to Kubernetes

excerpt: Phần này giải thích khái niệm Kubernetes, trình bày lý do nên sử dụng Kubernetes và các tính năng của nó. Cùng với đó, phần này cũng nói về quá trình phát triển của Kubernetes từ Borg và làm rõ vai trò của Cloud Native Computing Foundation.
layout: post
---

Trong chương này, chúng ta sẽ làm rõ Kubernetes cùng các tính năng và lý do bạn nên sử dụng nó. Chúng ta sẽ khám phá lịch sử phát triển của Kubernetes từ Borg, trình quản lý khối lượng công việc phân tán rất riêng của Google.

We will also learn about the Cloud Native Computing Foundation (CNCF), which currently hosts the Kubernetes project, along with other popular cloud-native projects, such as Prometheus, Fluentd, cri-o, containerd, Helm, Envoy, and Contour, just to name a few.

Chúng ta cũng sẽ học về Cloud Native Computing Foundation (CNCF), tổ chức ddanxng quản lý dự án Kubernetes bên cạnh rất nhiều dự án cloud-native khác như Prometheus, Fluentd, cri-o, containerd, Helm, Envoy, và Contour là một trong số đó.

## Kubernetes là gì

Dựa vào thông tin từ trang web của Kubernetes,

> "Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications."

Tạm dịch là : "Kubernetes là một hệ thống mã nguồn mở dành cho việc tự động triển khai, thu phóng cũng như quản lý các ứng dụng đã được contanier hoas"

Từ Kubernetes bắt nguồn từ κυβερνήτης trong tiếng Hi Lạp, có nghĩ là người lái tàu. Với sự liên tưởng trong tâm trí của mình, ta có thể nghĩ Kubernetes giống như người thuyền trưởng trên một con tàu chứa đầy containers.

Kubernetes đôi khi được viết tắt là k8s (đọc là Kate's), bởi có 8 kí tự giữa k và s.

Kubernetes lấy cảm hứng từ hệ thống Borg của Google, một bộ điều phối container và workload cho quá trình vận hành toàn cầu trong hơn một thập kỷ công ty này. Nó là một dự án mã nguồn mở được viết bằng ngôn ngữ Go và được phân phối dưới giấy phép  Apache License, phiên bản 2.0.

Kubernetes được khởi dộng bởi Google và khi phiên bản 1.0 của nó được phát hành vào tháng 7 năm 2015, Google đã quyên góp nó cho Cloud Native Computing Foundation (CNCF).

Phiên bản mới của Kubernetes được phát hành mỗi 3 tháng. Phiển bản ổn định tính đến thời điểm khóa học được tạo là 1.19 (tháng 8 năm 2020).

## Từ Borg đến Kubernetes

Dựa vào phần giới thiệu của bài báo Borg của Google xuất bản năm 2015,

> "Google's Borg system is a cluster manager that runs hundreds of thousands of jobs, from many thousands of different applications, across a number of clusters each with up to tens of thousands of machines".

Tạm dịch: Hệ thống Borg của Google là một trình quản lý những cụng chạy hàng trăm của hàng ngàn jobs từ hàng ngàn ứng dụng khác nhau, xuyên suốt số cụm mà môi cụm có thể có tới hàng tấn của hàng ngàn machines.

Trong hơn một thập kỉ, Borg đã là vũ khí bí mật của Google, chạy vô vàn workloads được container hóa trên toàn cầu trên môi trường production. Các dịch vụ mà chúng ta sử dụng từ Google, chẳng ạn như Gmail, Drive, Map, Docs, ... tất cả chúng đều được cung cấp bằng cách sử dụng Borg.

Một số tác giả đầu tiên của Kubernetes đã từng là nhân viên của Google, những người đã từng sử dụng Borg và phát triển nó trong quá khứ. Họ đã tập trung toàn bộ những kiến trức và kinh nghiệm quý giá trong suốt quá trình thiết kế Kubernetes. Một trong những tính năng/đặc trưng của Kubernetes có thể được tình thấy từ Borg hoặc những bài học từ nó có thể kể đến là:

- API servers
- Pods
- IP-per-Pod
- Services
- Labels.

Chúng ta sẽ cùng nhau làm rõ tất cả chúng hơn nữa trong khóa học này.

## Tính năng của Kubernetes, phần 1

Kubernetes mang đến một khối tính năng đồ sộ dành cho một bộ điều phối container. Một trong những tính năng mà nó hoàn toàn hỗ trợ có thể kể đến là:

- Automatic bin packing (Tự động đóng gói nhị phân): Kubernetes tự động lập lịch cho các containers dựa trên rài nguyên mà chúng cần và ràng buộc, để có thể tối đa hóa tính hữu ích mà không bị mất tính khả dụng.
- Self-healing (Tự sửa chữa): Kubernetes tự động thay thế và tự tái lập lịch containers từ những nút bị lỗi. Nó cưỡng chế tắt và khởi động lại các containers không phản hồi để kiểm tra tình trạng dựa trên các quy định/chính sách có sẵn. Nó cũng ngăn cản các gói lưu lượng từ bộ định tuyến đến những container không phản hồi.
- Horizontal scaling (Thu phóng theo chiều ngang): Với Kubernetes các ứng dụng được thu phóng thủ công hoặc tự động dựa trên tính khả năng sử dụng CPU hoặc các hệ đo lường tùy chỉnh khác.
- Service discovery and Load balancing (Khám phá dịch vụ và cân bằng tải): Các containers nhận các địa chỉ IP của chúng từ Kubernetes trong khi nó được gán một tên riêng biệt trên Domain Name System (DNS) để tập hợp thành một tập hợp các vùng chứa để hỗ trợ các yêu cầu cân bằng tải trên các containers của tập hợp đó.

## Tại sao nên dùng Kubernetes

Bên cạnh vô vàn các tính năng mà Kubernetes hoàn toàn hỗ trợ, Kubernetes còn mang đến cho chúng ta sự di động và mềm dẻo. Nó có thể được triển khai trên rất nhiều môi trường, từ máy ảo cục bộ hoặc từ xa, máy chủ vật lý một người thuê, (bare metal) hoặc trên những môi trường cài đặt công khai/bí mât/lai/multi-cloud. Nó hỗ trợ và được hộ trợ bởi rât công cụ mã nguồn mử từ bên thứ ba nằm nâng cao khả năng và cung cấp những trải nhiệm tốt với lượng tính năng khổng lồ cho người sử dụng.

Kiến trúc của Kubernetes mô đun hóa và dễ dàng lắp đặt, kết nnoois. Nó không chỉ sắp xếp các ứng dụng loại microservices được phân tách theo mô-đun, mà cả kiến trúc của nó cũng tuân theo các mẫu microservices được tách rời. Chức năng của Kubernetes có thể được mở rộng bằng cách viết tài nguyên tùy chỉnh, toán tử, API tùy chỉnh, quy tắc lập lịch hoặc plugin.

Để có được một dự án mã nguồn mở thành công, một cộng đồng lớn mạnh là yếu tố rất quan trọng để có được mã nguồn tốt. Kubernetes được hỗ trợ bởi một cộng đồng đang phát triên mạnh trên toàn thế giới, Nó có hơn 2.800 người tham gia đóng góp, những người, trong suốt thời gian qua đã đẩy lên gần 94.000 lượt commits. Họ đã hợp thành các nhóm ở các thành phố và đất nước khác nhau, những nhóm mà thường xuyên gặp gỡ và trao đổi về Kubernetes và hệ sinh thái của nó. Trong đó có những nhóm đặc biệt quan tâm đến dự án này gọi là SIGs (Special Interest Groups) thường tập trung vào những chủ đề đặc biệt như thu phóng, máy chủ vật lý, mạng máy tính, ... Chúng ta sẽ tìm hiểu thêm về họ ở chương cuối, Kubernetes Communities.

## Khách hàng của Kubernetes

Chỉ vài năm trở lại đây từ ngày Kubernetes được giới thiệu, rất nhiều các doanh nghiệp với kích cỡ đa dạng đã triển khai các workloads của họ bằng cách sử dụng Kubernetes. Nó là giải pháp được sử dụng dành cho quản lý workloads trong ngân hàng, giáo dục, tài chính, đầu tư, trò chơi điện thử, công nghệ thông tin, truyền thông và trực tuyến, bán lẻ trực tuyến, chia sẻ xe, viễn thông và rất nhiều ngành công nghiệp khác. Có rất nhiều tình huống sử dụng và những trường hợp thành công trên trang web của Kubernetes:

- BlaBlaCar
- BlackRock
- Box
- eBay
- Haufe Group
- Huawei
- IBM
- ING
- Nokia
- Pearson
- Wikimedia
- Và nhiều hơn thế nữa.

## Cloud Native Computing Foundation (CNCF)

The Cloud Native Computing Foundation (CNCF) là một trong các dự án được quản lý bởi Linux Foundation. CNCF hướng tới đẩy nhanh sự áp dụng các công nghệ như containers, microservices và ứng dụng thuần nền tảng đám mây (cloud-native applications).

![CNCF logo](https://www.cncf.io/wp-content/uploads/2020/08/cncf-thumbnail-default.svg)

CNCF triển khai vô vang các dự án và con số này sẽ tăng lên trong tương lai. CNCF cung cấp tài nguyên cho mỗi dự án nhưng tai một thời điểm bất kì, mội dự án sẽ được vận hàng đọc lập dưới hệ thống quản trị đã xác định trước và những người bảo trước đó. Dự án bên trong CNCF được chia loại dựa trên trạng thái đạt được: Sandbox, Incubating, and Graduated. Tại thời điểm khóa học được tạo hàng chục dự án đã đạt được trạng thái Graduated và số còn lại, nhiều hơn đang ở Incubating và Sandbox.

Các dự án được đánh giá Graduated bao gồm:

- Kubernetes dành cho điều phối containers
- Prometheus dành cho việc giám sát
- Envoy dành cho lưới dịch vụ
- CoreDNS dành cho việc khám phá dịch vụ
- containerd dành cho các môi trường runtime của container
- Fluentd dành cho logging
- Harbor dành cho registry
- Helm dành cho quản lý gói
- Vitess dành cho việc lưu trữ thuần nền tảng đám mây
- Jaeger dành cho việc theo dõi phân tán
- TUF dành cho việc cập nhập phần mềm
- TiKV dành cho việc lưu trữ dạng khóa/giá trị

Các dự án đang ở mức Incubating bao gồm:

- CRI-O dành cho các môi trường runtime của container
- Linkerd dành cho lưới dịch vụ
- Contour dành cho vấn đề xâm nhập
- etcd dành cho việc lưu trữ dạng khóa/giá trị
- gRPC dành cho remote procedure call (RPC)
- CNI dành cho các API mạng máy tính
- Rook dành cho việc lưu trữ thuần nền tảng đám mây
- Notary dành cho bảo mật
- NATS dành cho messaging
- OpenTracing dành cho quản lý phân tán
- Open Policy Agent dành cho policy
- Và nhiều hơn thế nữa.

Có rất nhiều dự án Sandbox trong CNCF hướng tới các phương pháp, giám sát, định danh, scripting, serverless, nodeless, edge, mong đợi đạt được trạng thái Incubating và có thể Graduated. Trong khi nhiều dự án đang hoạt động đang chuẩn bị cất cánh, những dự án khác đang được lưu trữ khi chúng trở nên ít hoạt động hơn. Dự án được lưu trữ đầu tiên là môi trường runtime rkt.

Các dự án trong CNCF bao gồm toàn bộ vòng đời của một ứng dụng thuần nền tảng đám mây, từ việc thực thi nó bằng cách sử dụng container runtime, cho đến giám sát và ghi nhật ký của nó. Điều này rất quan trọng để đáp ứng các mục tiêu của CNCF.

## CNCF và Kubernetes

Với Kubernetes, Cloud Native Computing Foundation đóng vai trò:

- Cung cấp một ngôi nhà trung lập cho nhãn hiệu Kubernetes và quản lý cách hoạt động của nó sao cho phù hợp
- Cung cấp giấy phép quét mã lõi và mã nhà cung cấp
- Cung cấp hướng dẫn pháp lý về các vấn đề bằng sáng chế và bản quyền
- Tạo chương trình học, đào tạo và chứng nhận mã nguồn mở cho cả quản trị viên Kubernetes (CKA, certification for both Kubernetes administrators) và nhà phát triển ứng dụng (CKAD, certification application developers)
- Quản lý một nhóm làm việc tuân thủ phần mềm
- Tích cực tiếp thị Kubernetes
- Hỗ trợ các hoạt động đặc biệt
- Tài trợ cho các hội nghị và sự kiện gặp mặt.
