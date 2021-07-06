---
title: Bộ điều phối container
categories:
- LFS158x - Introduction to Kubernetes

excerpt: Phần này giải thích khái niệm điều phối container nêu những lợi ích của việc sử dụng chúng cũng như nêu các tùy chọn điều phối container khác nhau và các tùy chọn triển khai điều phối container khác nhau.
layout: post
---

Với container images, chúng ta giới hạn mã ứng dụng, runtimes của nó và tất cả các phần phụ thuộc của nó ở định dạng được xác định trước. Và, với các runtimes của containers như runC, containerd hoặc cri-o, chúng ta có thể sử dụng các images đóng gói sẵn đó để tạo một hoặc nhiều containers. Tất cả các runtimes này đều hoạt động tốt khi chạy các containers trên một máy chủ duy nhất. Tuy nhiên, trên thực tế, chúng ta muốn có một giải pháp có khả năng mở rộng và chịu được lỗi, có thể đạt được bằng cách tạo một container / bộ điều khiển / đơn vị quản lý duy nhất, sau khi kết nối nhiều nút với nhau. Một container/ bộ điều khiển / đơn vị quản lý này thường được gọi là bộ điều phối container.

Trong chương này, chúng ta sẽ khám phá lý do tại sao chúng ta nên sử dụng bộ điều phối containers, các cách triển khai khác nhau của bộ điều phối containers và nơi triển khai chúng.

## Các container là gì

Trước khi đi sâu vào tìm hiểu bộ điều phối container, hãy cùng nhau ôn lại khái niệm của container.

Các containers là phướng pháp lấy ứng dụng làm trung tâm, để có thể phân phối các ứng dụng hiệu năng cao, dễ dàng thu phóng đến mỗi hệ sinh thái mà bạn lựa chọn. Container là lựa chọn phù hợp nhất để phân phối các microservices bằng cách cung cấp các môi trường ảo di động, được cô lập để các ứng dụng chạy mà không bị các ứng dụng đang chạy khác can thiệp.

{:refdef: style="text-align: center;"}
![Container deployment](/assets/img/container_deployment.png)
{: refdef}

Microservices là các ứng dụng nhỏ gọn được viết bằng vô vàn các ngôn ngữ hiện đại với các dependencies, thư viện và môi trường ảo cụ thể. Để chắc chắn rằng các ứng dụng này có mọi thứ chúng muốn để có thể hoạt động đúng với mong đợi, chúng thường được đóng gói cùng với các dependencies.

Các containers đóng gói các microservices và các dependencies của chúng nhưng lại không chạy chúng. Containers chạy các container images.

Một container image "bó" một ứng dụng với môi trường runtime, thư viện, dependencies và nó đại diện cho nguồn của một vùng chứa được triển khai để cung cấp một môi trường thực thi biệt lập cho ứng dụng.
Container có thể được triển khai từ một image cụ thể lên một số nền tảng, chẳng hạn máy trạm, máy ảo, cloud công cộng, ....

## Bộ điều phối container là gì

Ở môi trường dev (môi trường phát triển), chạy các containers trên một máy chủ để phát triển cũng như kiểm thử các ứng dụng có thể là một lựa chọn. Tuy nhiên, để có thể chuyển sang môi trường Quality Assurance (QA) và Production (Prod), nó sẽ không còn là một lựa chọn tốt vì những ứng dụng và dịch vụ này cần có những yêu cầu cụ thể:

- Khả năng chịu lỗi
- Thu phóng dựa trên nhu cầu
- Tối ưu sử dụng tài nguyên
- Auto-discovery để có thể tự động phát hiện và giao tiếp với nhau
- Khả năng tiếp cận từ thế giới bên ngoài
- Khả năng cập nhật cũng như phục hồi phiên bản cũ liền mạch với lượng downtime không đáng kể.

Bộ điều phối container là các công cụ nhóm các hệ thống lại với nhau thành các cụm nơi mà quá trình triển khai cũng như việc quản lý các container được tự động ở quy mô (???) trong khi đáp ứng các yêu cầu nêu trên.

## Một số bộ điều phối container

Với việc các doanh nghiệp chứa các ứng dụng của họ và chuyển chúng lên đám mây, nhu cầu ngày càng tăng về các giải pháp điều phối vùng chứa. Mặc dù có nhiều giải pháp khả dụng, nhưng một số chỉ là bản phân phối lại của các công cụ điều phối vùng chứa được thiết lập tốt, được bổ sung thêm các tính năng và đôi khi có những hạn chế nhất định về tính linh hoạt.

Mặc dù không đầy đủ, danh sách dưới đây cung cấp một số công cụ và dịch vụ điều phối vùng chứa khác nhau hiện có:

- Amazon Elastic Container Service
- Azure Container Instances
- Azure Service Fabric
- Kubernetes
- Marathon
- Nomad
- Docker Swarm

## Tại sao cần dùng các bộ điều phối container

Mặc dù chúng ta có thể bảo trì thủ công một cặp container hoặc viết các đoạn script để quản lý vòng đời hàng chục containers, bộ điều phối khiến mọi thứ trở nên dễ dàng hơn để người vận hành đặc biệt khi họ cần quản lý hàng trăm, thậm chí là hàng ngàn containers chạy trên một hệ sinh thái toàn cục.

Hầu hết các bộ điều phối container đều có thể:

- Nhóm các hosts thành cụm
- Lập lịch cho các containers để chạy trên các hosts trong một cụm dựa trên tính khả dụng của tài nguyên.
- Cho phép các containers trong một cụm có thể giao tiếp với nhau bất kể máy chủ mà chúng được triển khai trong cụm.
- Ràng buộc container với các tài nguyên lưu trữ
- Nhóm những container tương tự nhau và ràng buộc chúng với các cấu trúc cân bằng tải để đơn giản hóa truy cập đến các ứng dụng đã container hóa bằng cách tạo ra các mức trừu tượng giữa các container và người dùng.
- Quản lý và tối ưu sử dụng tài nguyên
- Cho phép cài đặt các chính sách để có thể bảo mật truy cập đến các ứng dụng đang chạy bên trong các containers.

Với những tính năng không chỉ có thể cấu hình mà còn linh hoạt, các bộ điều phối container là lựa chọn hiển nhiên khi nó xuất hiện để quản lý các ứng dụng đã được container hóa dựa trên tỉ lệ. Trong khóa học LFS158x, Kubernetes sẽ được sử dụng.

## Các bộ điều phối container được triển khai ở đâu

Hầu hết các bộ điều phối container đều được triển khai trên các hệ sinh thái dựa trên lựa chọn của bạn, trên các máy thực, máy ảo, tại chỗ trên cloud công cộng và cloud lai. Kubernetes, như một ví dụ, có thể được triển khai trên máy trạm, cùng với hoặc không cần một hypervisor địa phương như Oracle VirtualBox, bên trong trung tâm dữ liệu của công ty, trong nền tảng đám mây trên instance của AWS Elastic Compute Cloud (EC2), Google Compute Engine (GCE) VMs, DigitalOcean Droplets, OpenStack, ...

Có các giải pháp dạng chìa khóa trao tay cho phép cài đặt các cụm Kubernetes, chỉ với một số lệnh, trên cơ sở hạ tầng Cloud-as-a-Services, chẳng hạn như GCE, AWS EC2, Docker Enterprise, IBM Cloud, Rancher, VMware Tanzu và đa -các giải pháp đám mây thông qua IBM Cloud Private hoặc StackPointCloud.

Cuối cùng nhưng không kém phần quan trọng, đó là điều phối vùng chứa được quản lý dưới dạng Services, cụ thể hơn là giải pháp Dịch vụ Kubernetes được quản lý, được cung cấp và lưu trữ bởi các nhà cung cấp đám mây lớn, chẳng hạn như Amazon Elastic Kubernetes Service (Amazon EKS), Azure Dịch vụ Kubernetes (AKS), DigitalOcean Kubernetes, Google Kubernetes Engine (GKE), IBM Cloud Kubernetes Service, Oracle Container Engine cho Kubernetes hoặc VMware Tanzu Kubernetes Grid.
