---
date: '2021-05-31T11:04:17+07:00'
description: "Kubernetes l\xE0 g\xEC"
layout: post
title: "T\u1ED5ng quan v\u1EC1 ki\u1EBFn tr\xFAc c\u1EE7a Kubernetes"

categories:
  - Kiến thức cơ bản

tags:
  - Kubernetes
---

# Kubernetes là gì

Dựa vào thông tin từ trang web của Kubernetes,

> "Kubernetes is an open-source system for automating deployment, scaling, and management of containerized applications."

Tạm dịch là : "Kubernetes là một hệ thống mã nguồn mở dành cho việc tự động triển khai, thu phóng cũng như quản lý các ứng dụng đã được contanier hoas"

Từ Kubernetes bắt nguồn từ κυβερνήτης trong tiếng Hi Lạp, có nghĩ là người lái tàu. Với sự liên tưởng trong tâm trí của mình, ta có thể nghĩ Kubernetes giống như người thuyền trưởng trên một con tàu chứa đầy containers. Kubernetes đôi khi được viết tắt là k8s (đọc là Kate's), bởi có 8 kí tự giữa k và s. Nó lấy cảm hứng từ hệ thống Borg của Google, một bộ điều phối container và workload cho quá trình vận hành toàn cầu trong hơn một thập kỷ công ty này. Nó là một dự án mã nguồn mở được viết bằng ngôn ngữ Go và được phân phối dưới giấy phép  Apache License, phiên bản 2.0.

Bên cạnh vô vàn các tính năng mà Kubernetes hoàn toàn hỗ trợ, Kubernetes còn mang đến cho chúng ta sự di động và mềm dẻo. Nó có thể được triển khai trên rất nhiều môi trường, từ máy ảo cục bộ hoặc từ xa, máy chủ vật lý một người thuê, (bare metal) hoặc trên những môi trường cài đặt lai/multi-cloud. Nó hỗ trợ và được hỗ trợ bởi rất công cụ mã nguồn mở từ bên thứ ba nằm nâng cao khả năng và cung cấp những trải nhiệm tốt với lượng tính năng khổng lồ cho người sử dụng.

Kiến trúc của Kubernetes mô đun hóa và dễ dàng lắp đặt, kết nối. Nó không chỉ sắp xếp các ứng dụng loại microservices được phân tách theo mô-đun, mà cả kiến trúc của nó cũng tuân theo các mẫu microservices được tách rời. Chức năng của Kubernetes có thể được mở rộng bằng cách viết tài nguyên tùy chỉnh, toán tử, API tùy chỉnh, quy tắc lập lịch hoặc plugin.

# Kiến trúc của Kubernetes

Ở một mức vô cùng cao, Kubernetes có các thành phần chính như sau:

- Một **nút master** hoặc nhiều hơn, đây là một bộ phận của **control plane**
- Một **nút worker** hoặc nhiền hơn

![](https://courses.edx.org/assets/courseware/v1/51120ad23b216a6946e3c4ebef2106bf/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/arch-1.19-components-of-kubernetes.svg)

## Nút master

**Nút master** cung cấp môi trường cho **control plane** chịu trách nhiệm quản lý trạng thái của các cụm Kubernetes và nó là đầu nào đằng sau mọi quá trình vận hành trong cụm. Thành phần control plane là các tác tưr với vai trò vô cùng đặc thù trong bộ quản lý cụm. Để có thể giao tiếp với cụm Kubernetes, người sử dụng gửi các requests đến control plane thông qua các công cụ Command Line Interface (CLI), một Dashboard dưới dạng giao diện người dùng web (Web UI) , hoặc các API (Application Programming Interface).

Việc giữ control plane chạy bằng mọi giá là tối quan trọng. Việc control plane ngừng hoạt động có thể dẫn đến downtime, thứ mà gây ra việc các dịch vụ gián đoạn và không thể cung cấp cho người dùng và có thể làm kinh doanh thua lỗ. Để đảm bảo cho khả năng chịu lỗi của control plane, các bản sao của nút master được thêm vào cụm, được cấu hình ở chế độ HA (High-Availability, độ khả dụng cao). Trong khi chỉ một trong nút master được chuyên biệt để quản lý chủ động cụm, các thành phần control plane giữ trạng thái đồng bộ giữa các bản sao của nút master. Chính loại cấu hình này đã cung cấp khả năng tự phục hồi cho control plane của cụm, thứ có thể cho phép nút master đang ở trạng thái active xảy ra lỗi ở mức độ nào đó.

Để duy trình trạng thái của cụm, tất cả các dữ liệu về cấu hình của cụm đều được lưu trong [etcd](https://etcd.io/). **etcd** là một bộ lưu trữ dạng key-value phân tán, thứ chỉ giữ các dữ liệu liên quan đến trạng thái của cụm chứ không bao gồm dữ liệu workload của client. etcd có thể được cấu hình trên nút master ([stacked topology](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/#stacked-etcd-topology)) hoặc trên một host được chuyên biệt ([external topology](https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/ha-topology/#external-etcd-topology)) để có thể giảm thiểu khả nằng mất mát dữ liệu được lưu trữ bằng cách tách riêng chúng khỏi các tác tử control plane khác.

Với ở dạng stacked topology, các bản sao HA của nút master đảm bảo khả năng tự phục hồi của dữ liệu lưu trữ trong etcd. Tuy nhiên, đây không giống như trường hợp dùng etcd ở dạng external topology, khi mà host của etcd bắt buộc phải được tạo các nhân bản và tách rời để phục vụ cho chế đồ HA, cấu hình mà dẫn đến nhu cầu về các phần cứng bổ sung.

Một nút master có thể có những thành phần control plane như sau:

### API Server (máy chủ API)

Tất cả các tác vụ quản trị đều được phối hợp triển khai bởi **kube-apiserver**, thành phần control plane trung tâm được chạy trên nút master. AOI Server tiếp nhận các yếu cầu RESTful từ người dùng, tác tử hoạc các tác tử từ phía bên ngoài, sau đó xác nhận và xử lý chúng. Trong khi xử lý, API Server đọc trạng thái hiện tại của cụm Kubernetes từ bộ lưu trữ dữ liệu etcd và sau khi thực thi xong yêu cầu, trạng thái kết quả của cụm Kubernetes được lưu vào bộ lưu trữ dữ liệu dạng key-value phân tán để đảm bảo tính bền bỉ.  API Server là thành phần master plane duy nhất có thể giao tiếp với bộ lưu trữ dữ liệu etcd. bao gồm cả đọc và ghi các thông tin trạng thái của cụm Kubernetes - hoạt động như một giao diện trung gian cho bất kì một tác tử control plane khác truy cập đến trạng thái của cụm.

API Server có thể cấu hình và tùy biến một cách dễ dàng. Nó có thể mở rộng quy mô theo chiều ngang, nhưng nó cũng hỗ trợ việc bổ sung API Server phụ tùy chỉnh, một cấu hình biến API Server chính thành proxy cho tất cả API Server tùy chỉnh, thứ cấp và định tuyến tất cả các lệnh gọi RESTful đến chúng dựa trên các quy tắc được xác định tùy chỉnh.

### Scheduler (Bộ lập lịch)

Vai trò của **kube-scheduler** là giao các các đối tượng workload mới, chẳng hạn như các pod cho các nút. Trong suốt quá trình lập lịch, các quyết định sẽ được tạo dựa trên trạng thái hiện tại của cụm Kubernetes và yêu cầu của các đối tượng mới. Bộ lập lịch thu thập từ bộ lưu trữ dữ liệu, thông qua API server, dữ liệu sử dụng tài nguyên của từng nút worker trong cụm. Bộ lập lịch cũng có thể nhận từ API Server các requirements của các đối tượng mới, thứ là một phần của dữ liệu cấu hình của chúng. Những requirements này có thể chứa các rằng buộc được người dùng hoặc các tác tử cài đặt, chẳng hạn như công việc lập lịch trên một nút đã được gán nhán với căp key/value **disk==ssd**. Bộ lập lịch cũng nhận các yêu cầu về Chất lượng Dịch vụ (QoS) của tài khoản, data locality, affinity, anti-affinity, taints, toleration, cluster topology, ... Một khi tất cả các dữ liệu của cụm đã khả dụng, thuật toán lập lịch lọc các nút với các thuộc từ (predicates) để cô lập  các nút ứng cử viên có thể đảm nhiệm công việc, những nút được đánh giá để có thể chọn một nút mà thỏa mãn tất cả các yêu cầu cho việc phục vụ một khối lượng công việc mới. Kết quả của quá trình đưa ra quyết định này là liên lạc lại với API Server, thứ mà sau đó ủy sẽ thác việc triển khai khối lượng công việc với các tác tử control plane khác.

Bộ lập lịch có khả năng cấu hình và tùy chỉnh cực kì cao dựa trên các scheduling policies, plugins, and profiles. Tiếp đó, các bộ lập lịch tùy chỉnh bổ sung có thể cũng được hỗ trợ, do đó các dữ liệu cấu hình (objects' configuration data) nên bao gồm tên của bộ lập lịch tùy chỉnh mong muốn để có những quyết định lập lịch phù hợp với cài đặt của chúng. Nếu không có bất kỳ dữ liệu nào được cung cấp, bộ lập lịch mặc định sẽ được sử dụng.

Một bộ lập lịch thường cực kì quan trọng và phức tạp trong một cụm Kubernetes có nhiều nút. Tuy nhiên trong các cụm chỉ có duy nhất một nút, chẳng hạn như cụm được sử dụng làm ví dụ trong khóa học này, công việc của bộ lập lịch về cơ bản khá đơn giản.

### Controller Managers (Trình quản lý các controller)

**controller managers** là các thành phần của control plane trên nút master, thứ mà chạy các controller để điều tiết trạng thái của cụm Kubernetes. Các controller luôn trong qúa trình liên tục lặp lại việc quan sát để có thể chạy và so sánh trạng thái đã được miêu tả (được cung cấp bởi các objects' configuration data) và trạng thái hiện tại của nó (thu thập từ dữ liệu của `etcd` lưu trữ thông qua API server) TRong trường hợp xảy ra sự nhầm lẫn nào đó, hành động sửa lỗi được thực hiện trong cụm cho đến khi trạng thái hiện tại đúng với trạng thái được mô tả.

**kube-controller-manager** chạy các controllers chịu trách nhiệm phản ứng khi các nút không khả dụng, để đảm bảo số pod như mong đợi, để tạo endpoints, service accounts, và API access tokens.

**cloud-controller-manager** chạy các controllers chịu trách nhiệm tương tác với các hệ sinh thái cơ bản của bên cung cấp dịch vụ đám mây khi có nút trở nên không khả dụng cùng với đó là quản lý các container dữ liệu được cung cấp bởi các dịch vụ đám mấy và quản lý quá trình cân bằng tải và điều hướng.

### Data Store (Kho dữ liệu)

**etcd** là kho dữ liệu dạng key/value phân tán và có có tính nhất quán cao được sử dụng để đảm bảo tính bền bỉ của trạng thái của cụm Kubernetes. Dữ liệu mới được ghi vào kho lưu trữ chỉ bằng cách thêm vào cuối nó bởi vậy nên dữ liệu sẽ không bao giờ bị thay đổi trong đây. Dữ liệu lỗi thời được nén định kỳ để giảm thiểu dung lượng của kho dữ liệu.

Trong tất cả các thành phần của control plane, chỉ có API Server là có khả năng giao tiếp với kho dữ liệu `etcd`

etcd's CLI management tool - **etcdctl**, provides backup, snapshot, and restore capabilities which come in handy especially for a single etcd instance Kubernetes cluster - common in Development and learning environments. However, in Stage and Production environments, it is extremely important to replicate the data stores in HA mode, for cluster configuration data resiliency.

Công cụ quản lý dưới dạng CLI cuar etcd - **etcdctl** cung cấp khả năng backup, snapshot, and restore, những thứ đặc biệt tiện dụng đối với một cụm Kubernetes có một etcd duy nhất - thường thấy trong trong môi trường Phát triển. Tuy nhiên, trong môi trường Staging và Production, điều cực kỳ quan trọng là phải sao chép các kho dữ liệu ở chế độ HA, để có khả năng phục hồi dữ liệu cấu hình cụm.

Một số công cụ khởi động cụm Kubernetes, chẳng hạn như **kubeadm**, mặc định sẽ cung cấp nút master với etcd dạng stacked, bơi ,à kho dữ liệu chạy song song và chia sẻ các tài nguyên với các thành phần control plane khác trên nút master đó.

![](https://courses.edx.org/assets/courseware/v1/7aed6c75efbc84567cbf1b7d61ec150a/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/kubeadm-ha-topology-stacked-etcd.svg)

Để cách ly kho dữ liệu khỏi các thành phần control plane, quy trình khởi động có thể được cấu hình cho cấu trúc liên kết etcd bên ngoài (external), nơi lưu trữ dữ liệu được cung cấp trên một máy chủ riêng biệt chuyên dụng, do đó giảm nguy cơ xảy ra lỗi etcd.

![](https://courses.edx.org/assets/courseware/v1/9abe43636a43eecfed5dfd3478ddae45/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/kubeadm-ha-topology-external-etcd.svg)

Cả cách cấu hình etcd dạng stacked hoặc external đều hỗ trợ cấu hình HA. etcd dựa trên [thuật toán Raft Consensus](https://web.stanford.edu/~ouster/cgi-bin/papers/raft-atc14) điều này cho phép một tập hợp các máy hoạt động như một Group thống nhất có thể tồn tại sau những sự cố của một số thành viên của nó. Tại bất kỳ thời điểm nào, một trong các nút trong Group sẽ là nút chính (master) và phần còn lại sẽ là nút theo dõi (follower). etcd xử lý khéo léo các cuộc bầu cử chính và có thể chịu được lỗi của nút, bao gồm cả lỗi của nút chính. Bất kỳ nút nào cũng có thể được coi là nút chính.

![](https://courses.edx.org/assets/courseware/v1/24010f6f4fb57dc844242ece997a67c4/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/Master_and_Followers.png)

etcd được viết bằng ngôn ngữ lập trình Go. Trong Kubernetes, bên cạnh việc lưu trữ trang thái cụm, etcd cũng được sử dụng để lưu trữ đặc tả các dữ liệu cấu hình chẳng hạn như subnets, ConfigMaps, Secrets,...

Ngoài ra, nút master có thể có:

- Container Runtime
- Node Agent
- Proxy.

## Nút worker

**worker node** cung cấp môi trường chạy cho các ứng dụng client. Trong vô vàng các microservices đã được container hóa, các ứng dụng được cô lập trong các Pods, được điều khiển bằng các tác tử của control plane của cụm chạy trên nút master. Pods được lập lịch trên nút worker, nơi chúng có thể tìm thấy các tài nguyên để tính toán và lưu trữ để chạy và được kết nối để có thể giao tiếp với nhau cũng như với thế giới bên ngoài. Một Pod là đơn vị lập lịch nhỏ nhất của Kubernetes. Nó là một tập hợp dựa trên logic của một hoặc nhiều container đã được lập lịch cùng với nhau và tập hợp đó có để được khởi chạy, dừng hoắc tái lập lịch như một đơn vị riêng biệt của công việc.

Mặt khác, trên một cụm Kubernetes có nhiều worker, lưu lượng mạng giữa các client users và các ứng dụng được triển khai trên các Pods được xử lý trực tiếp bởi các nút worker và nó không được điều hướng thông qua nút master.

Một nút worker có các thành phần như sau:

### Container Runtime (môi trường chạy của container)

Mặc dù Kubernetes được thiết kế như một engine có khả năng điều phối container, nó lại không có khả năng để có thể xử lý trực tiếp các container. Để quản lý vòng đời của một container, Kubernetes cần có **container runtime** trên nuts mà một Pod và các container của nó được lập lịch. Kubernetes hỗ trợ các môi trường runtime sau:

- [Docker](https://www.docker.com/) - container runtime phổ biến nhất được sử dụng với Kubernetes (nhưng k8s đã ngừng hỗ trợ tại thời điểm gần đây)
- [CRI-O](https://cri-o.io/) - container runtime cho Kubernetes, nó cũng hỗ trợ Docker image registries
- [containerd](https://containerd.io/) - container runtime đơn giản và di động cung cấp sự tự động đáng kể
- [frakti](https://github.com/kubernetes/frakti#frakti) - container runtime dựa trên hypervisor cho Kubernetes

### Tác tử nút - kubelet

**kubelet** là tác tử chạy trên mỗi nút và giao tiếp với các thành phần của control plane từ nút master. Nó nhận thông tin định nghĩa Pod, chủ yếu từ API Server và tương tác với container runtime ở trên nút để chạy các container liên quan đến Pod đó. Nó cũng theo dõi tình trạng và tài nguyên của các Pod đang chạy các containers.

kubelet kết nối với các container runtime thông qua interface dựa trên plugin - [Container Runtime Interface (CRI)](https://github.com/kubernetes/community/blob/master/contributors/devel/sig-node/container-runtime-interface.md). CRI bao gồm bộ đệm protocol, gRPC API, thư viện cũng như các thông số kỹ thuật và công cụ bổ sung hiện đang được phát triển. Để kết nối với container runtime có thể hoán đổi cho nhau, kubelet sử dụng ứng dụng **shim** để cung cấp một lớp trừu tượng rõ ràng giữa kubelet và tcontainer runtime

![](https://courses.edx.org/assets/courseware/v1/ab209f7c32ceb17ed43dcf6b66056cea/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/CRI.png)

Như hình trên, kubelet hoạt động như một grpc client kết nối với shim của CRI như một grpc server để thực hiện các thao tác container và image. CRI cài đặt hai dịch vụ: **ImageService** và **RuntimeService**. **ImageService** chịu trách nhiệm cho các thao tác liên quan đến image trong khi **RuntimeService** chịu trách nhiệm cho các thao tác liên quan đến Pod và container.

Các container runtime đẫ từng thường được hard-coded bên trong kubelet, nhưng từ khi CRI được giới thiệu, Kubernetes trở nên mềm dẻo hơn khi có thể sử dụng nhiều container runtimes khác nhau mà không cần phải dịch lại. Bất cứ container runtime nào mà được cài đặt CRI đều có thể được Kubernetes sử dụng để quản lý Pods, containers và các contianer images.

### kubelet - CRI shims

Shim là các cài đặt CRI hoặc các interface cho từng loại container runtume được hỗ trợ bởi Kubernetes. Dưới đây là một số ví dụ của CRI shim:

- **dockershim**

Với dockershim, các container được tạo bằng Docker được cài đặt trên các nút công nhân. Trong nội bộ, Docker sử dụng containerd để tạo và quản lý các container:

![](https://courses.edx.org/assets/courseware/v1/aa11f8d767939eb27a989d12423e5ae6/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/dockershim.png)

- **cri-containerd**

Với cri-containerd, chúng ta có thể trực tiếp sử dụng containerd để tạo và quản lý các container:

![](https://courses.edx.org/assets/courseware/v1/4d76490e58857edcf3a9c335f46fdcb9/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/cri-containerd.png)

- **CRI-O**

CRI-O cho phép sử dụng nếu có bất kỳ runtime nào tương thích với Open Container Initiative (OCI) với Kubernetes. Vào thời điểm khóa học này được tạo, CRI-O đã hỗ trợ runC và Clear Containers dưới dạng container runtime. Tuy nhiên, về nguyên tắc, bất kỳ runtime nào tuân theo OCI đều có thể được sử dụng cùng CRI-O.

![](https://courses.edx.org/assets/courseware/v1/8213cc7fbbc3b2a0f4af7f926c35ad68/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/crio.png)

- **frakti**

frakti cho phép triển khai CRI thông qua ảo hóa phần cứng, nhằm đạt được mức độ bảo mật và cách ly cao hơn so với các container cấp hệ điều hành Linux truyền thống dựa trên Group và không gian tên. CRI frakti shim nhằm mục đích cho phép kubelet tương tác với Kata Container:

![](https://courses.edx.org/assets/courseware/v1/9e71fcc2a9b585d0a3513db3ed2a1a45/asset-v1:LinuxFoundationX+LFS158x+3T2020+type@asset+block/frakti.png)

### Proxy - kube-proxy

**kube-proxy** là tác nhân mạng chạy trên mỗi nút chịu trách nhiệm cập nhật và bảo trì động của các luật điều phối mạng trên nút đó. Nó trừu tượng hóa chi tiết của quá trình hoạt động mạng của Pod và chuyển tiếp các kết nối đến Pods.

The kube-proxy is responsible for TCP, UDP, and SCTP stream forwarding or round-robin forwarding across a set of Pod backends, and it implements forwarding rules defined by users through Service API objects.

kube-proxy chịu trách nhiệm cho việc chuyển tiếp dòng TCP, UDP và SCTP hoặc xoay vòng chuyển tiếp qua một tập các Pod backends và nó cũng cài đặt các luật chuyển tiếp được định nghĩa bởi người dùng thông qua các đối tượng Server API.

# Tổng kết
Bài viết này giới thiệu sơ lược về Kubernetes cũng như trình bày tổng quan kiến trúc của nó. Có thể thấy rằng nếu muốn sử dụng k8s một cách hiệu quả, ta cần hiểu khá rõ về các khái niệm của nền tảng nguồn mở này. Nội dung bài viết này tham khảo phần lớn ở khóa học [Introduction to Kubernetes](https://www.edx.org/course/introduction-to-kubernetes)  trên edX, để tìm hiểu rõ hơn, mọi người có thể đăng ký khóa học này. ài viết đến đây là kết thúc cảm ơn mọi người đã giành thời gian đọc.