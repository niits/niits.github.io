---
date: "2020-12-14T16:17:22+07:00"
description: Microservices hiện nay được đề cập tới trong thế giới phần mềm, công nghệ được kỳ vọng cao và đánh giá như một xu hướng cho tương lai (Open API, service provider, …). Một số ý kiến cho rằng, microservices không có gì mới lạ, chẳng qua nó là SOA (kiến trúc hướng dịch vụ) được đánh bóng, đổi tên mà thôi. Mặc dù việc triển khai microservices dưới dạng multi-repo mang lại nhiều lợi ích tuy nhiên theo một số ý kiển, cách triển khai mã nguồn này có thể gây khó khăn trong một số trường hợp và thay vì triển khai cấu trúc mã nguồn dưới dạng chia thành nhiều repo riêng biệt, họ chọn cách tổ chức chỉ trong một repository duy nhất hay còn gọi là monorepo. Bài viết này sẽ nói về hai cách triển khai mã nguồn này cũng như so sánh các đặc điểm của chúng và cuối cùng là giới thiệu về Nx - một công cụ phát triển được giới thiệu rằng có thể dễ dàng mở rộng thường được sử dụng Monorepos.
layout: post
title: "Monorepos l\xE0 g\xEC v\xE0 t\u1EA1o nhanh m\u1ED9t Monorepos b\u1EB1ng Nx"

categories:
  - Kiến thức cơ bản

tags:
  - Monorepos
---

Microservices hiện nay được đề cập tới trong thế giới phần mềm, công nghệ được kỳ vọng cao và đánh giá như một xu hướng cho tương lai (Open API, service provider, …). Một số ý kiến cho rằng, microservices không có gì mới lạ, chẳng qua nó là SOA (kiến trúc hướng dịch vụ) được đánh bóng, đổi tên mà thôi. Mặc dù việc triển khai microservices dưới dạng multi-repo mang lại nhiều lợi ích tuy nhiên theo một số ý kiển, cách triển khai mã nguồn này có thể gây khó khăn trong một số trường hợp và thay vì triển khai cấu trúc mã nguồn dưới dạng chia thành nhiều repo riêng biệt, họ chọn cách tổ chức chỉ trong một repository duy nhất hay còn gọi là monorepo. Bài viết này sẽ nói về hai cách triển khai mã nguồn này cũng như so sánh các đặc điểm của chúng và cuối cùng là giới thiệu về Nx - một công cụ phát triển được giới thiệu rằng có thể dễ dàng mở rộng thường được sử dụng Monorepos.

# Multiple repositories và Mono repositories

![](images/6ed2d04e-dde1-485a-a10c-7dc6d9a887a8.jpeg)

> Hình ảnh từ bài viết: https://levelup.gitconnected.com/moving-from-multiple-repositories-to-a-lerna-js-mono-repo-faa97aeee35b

Không phải mỗi ứng dụng nào cũng được phát triển dưới dạng microservices. Tuy nhiên, dù có phải tái cấu trúc mã nguồn từ một khối nguyên khối hay không, chúng ta đều sẽ phải lựa chọn cách thức lưu trữ và quản lý mã nguồn theo kiểu multiple repositories (multirepo) hay mono repositories (monorepo). Sơ lược của hai kiểu cách thức lưu trữ và quản lý này như sau:

- Monorepo là kiểu cấu trúc project trong đó tất cả module (hoặc project con) đều nằm trong cùng 1 git repository.
- Multirepo, ngược lại, là kiểu cấu trúc project trong đó mỗi module (hoặc project con) chứa ở những git repository riêng lẻ.

Không thể phủ nhận rằng, việc lưu trữ và quản lý mã nguồn có một số các ưu điểm như sau:

- Phong cách mã nguồn linh hoạt và dễ dàng kiểm thử cho từng repositories
- Phân quyền rõ ràng cho người tham gia phát triển và bảo trì
- Codebase không quá lớn do đó dễ dàng quản lý

Tuy nhiên việc phát triển phần mềm thường hướng đến một ứng dụng tổng thể. Khi đó việc chia nhỏ ứng dụng thành nhiều phần riêng biệt có thể khiến cho việc nhầm lẫn về phạm vi cũng như mục đích và cách hoạt động của các module giữa các team trong toàn bộ tổ chức. Khi đó, các nhóm trở nên phân tán và có thể không biết rõ mục tiêu chính của dự án mà mình tham gia và do đó không bận tâm về những gì người khác đang làm miễn là công việc của họ được hoàn thành. Bên cạnh đó, một số nhận định rằng cách lưu trữ và quản lý này mất khá nhiều thời gian cũng như tài nguyên cùng với đó, trong một số trường hợp các phần của ứng dụng đều sử dụng cùng một ngôn ngữ thì việc có được phong cách mã nguồn linh hoạt không còn quá quan trong. Bởi vậy, họ cho rằng vấn đề này cũng như một số vấn đề kể trên và một số khác chưa kể đến có thể được giải quyết bằng cách sử dụng một phương pháp khác - monorepo.

Trong vài năm qua, Babel, Angular, React, Jest và nhiều dự án mã nguồn mở khác đã chuyển sang sử dụng monorepos. Monorepos không chỉ hữu ích cho các dự án nguồn mở. Chúng thậm chí còn phù hợp hơn cho các tổ chức đang phát triển các ứng dụng thực tế. Google, Facebook, Uber, Twitter đều làm được điều đó và họ đều có các công cụ dành cho nhà phát triển dễ dàng tùy chỉnh cung cấp vô vàn chức năng. Không phải tự nhiên, monorepo được sử dụng nhiều đến vậy và sau đây là một số ưu điểm có thể được kể đến:

- Tổ chức liền mạch: Với mono repo, các dự án có thể được tổ chức và nhóm lại với nhau theo bất kỳ cách nào bạn thấy là nhất quán về mặt logic nhất. Sử dụng một repo duy nhất cũng làm giảm chi phí quản lý các phần dependencies.
- Cải thiện văn hóa làm việc chung giữa các nhóm: Với việc áp dụng mono repo, mọi cá nhân trong tổ chức đều nhận thức được các mục tiêu của ứng dụng và điều này làm cho cả tổ chức trở thành một nhóm thống nhất và do đó có thể đóng góp cụ thể hơn cho các mục tiêu và mục tiêu của tổ chức.
- Phối hợp tốt hơn giữa các nhà phát triển: Các nhà phát triển có thể dễ dàng chạy toàn bộ nền tảng trên máy của họ và điều này giúp họ hiểu tất cả các dịch vụ và cách chúng làm việc cùng nhau. Điều này đã khiến các nhà phát triển tìm thấy nhiều lỗi cục bộ hơn trước khi gửi một pull request.
- Tái cấu trúc dễ dàng: Bất kỳ lúc nào chúng ta muốn đổi tên một cái gì đó, việc tái cấu trúc trở nên vô cùng dễ dàng. Việc tái cấu trúc cũng dễ dàng hơn vì mọi thứ đều gọn gàng ở một nơi và dễ hiểu hơn.

Lưu ý rằng việc triển khai dưới dạng mono-repo không hề đồng nhất với kiến trúc nguyên khối (monolith)

![](images/34bae8fa-231f-4955-8309-9f97c3b8392b.png)

Nói tóm lại thì monorepos không chỉ duy trì được hầu hết các ưu điểm khi triển khai dưới dạng microservices mà còn đơn giản hóa việc chia sẻ mã và tái cấu trúc dự án chéo, chúng giảm đáng kể chi phí tạo libs, microservices và microfrontends. Vì vậy, việc áp dụng monorepo thường cho phép triển khai linh hoạt hơn.

# Công cụ Nx

## Giới thiệu chung

> Nx is a set of extensible dev tools for monorepos, which helps you develop like Google, Facebook, and Microsoft.

Qua phần giới thiệu ngắn gọn ta có thể biết qua rằng Nx là một bộ công cụ dùng để tạo các monorepos. Theo như đội ngũ phát triển giới thiệu, Nx được học phát triển dựa trên những kinh nghiệm của họ trong quá trình làm việc ở các tập đoàn lớn như Google, Facebook và Microsoft. Bởi vậy họ kì vọng rằng Nx sẽ alf công cụ đắc lực hỗ trợ các lập trình viên tạo cũng như phát triển các hệ thống một cách dễ dàng với vô vàn các công nghệ frontend cũng như backend đang có hiện nay.

![](images/3122c53e-6f96-43f1-9121-1af6d30dd5d1.png)

Do đặc thù phát triển cho ngôn ngữ Javascript nên hiện nay Nx đang hỗ trợ Angular, React và Node.js. Tuy nhiên có thể trong tương lai không xa Nx có thể hỗ trợ các framework khác như Vue hoặc Svelte, ...

## Sử dụng

Nx sử dụng khái niệm `workspace` để chỉ một ứng dụng toàn thể được triển khai monorepos. Bởi vậy khi bắt đầu sử dụng chúng ta tạo một workspace mới bằng `npx` như sau:

```bash
npx create-nx-workspace
```

Ngoài việc sử dụng `npx` chúng ta có thể dùng `npm init` bằng `npm init nx-workspace` hoặc `yarn create` với `yarn create nx-workspace`. Ngay sau khi chạy lệnh trên, Nx sẽ hỏi chúng ta một số thứ chẳng hạn như muốn tạo workspace theo mẫu sẵn có nào hay không, ... như sau:

![](images/b08a0b55-015a-496f-8cd0-39ca3bcf9b25.png)

Sau khi chọn mẫu sẵn có phù hợp với nhu cầu của mình, Nx sẽ tạo cho chúng ta workspace mới. Ở đây mình chọn wokspace theo mẫu react để thử dùng Nx CLI - công cụ được Nx cung cấp để quản lý workspace mà mọi người có thể cài đặt bằng cách dùng câu lệnh `npm install -g nx`
Để có thể chạy ứng dụng ở local, chúng ta có thể dùng lệnh `npx nx serve demo`

## Thêm các plugin cho workspace của bạn

Nx là một nền tảng mở với các plugin cho nhiều công cụ và framework hiện đại. Bởi vậy ta có thể dùng câu lệnh ` npx nx list` để tìm hiểu xem Nx có hỗ trợ sẵn các framework bạn muốn sử dụng hay không

![](images/a39f6692-e430-4f7a-b4f0-0724373d2178.png)
Kết quả cho thấy rằng không chỉ bản thân Nx hỗ trợ rất nhiều các framework mà nó còn có các plugin được cộng đồng làm để có thể sử dụng các framework khác mà Nx vẫn chưa có plugin chính thức. Thông thường mỗi ứng dụng thường đi kèm với 'backend' vậy nên trong ví dụ này chúng ta sẽ sử dụng express bằng chạy `npm install --save-dev @nrwl/express` và sử dụng câu lệnh sau:

```bash
npx nx g @nrwl/express:app api --frontendProject=demo
```

Ngay sau đó, Nx CLI sẽ tạo cho chúng ta thư mục `api` và kết quả như sau:

![](images/b0e2cb83-ec6f-477e-b40f-c870542126a2.png)

Có thể thấy rằng quá trình trên ngoài việc tạo thư mục api còn chỉnh sửa nội dung một số file như file workspace.json. Mở file workspace.json ra, ta có thể thấy rằng một số mục đã được thêm vào mà một trong số đó là `proxyConfig` trỏ đến file `proxy.conf.json`. Có thể thấy rằng mục này lưu thông cấu hình proxy cho phần mã frontend có thể giao tiếp với backend.

## Quản lý các plugin với Dep graph

Thông thường một ứng dụng chỉ có một phần frontend và một phần backend hoặc thêm một số phần phụ trợ như e2e testing. Tuy nhiên đội ngũ phát triển hướng người sử dụng tới việc chia nhỏ ứng dụng của mình thành các module trong một workspace (có thể dưới dạng micro frontend và micoservices) cũng như tạo các thư viện và chia sẻ mã giữa các module nhằm tối ưu hóa mã nguồn, tránh việc lặp lại. Bởi vậy khi đó số lượng module sẽ rất lớn khiến việc quản lý thủ công không còn hiệu quả. Bởi vậy, Nx CLI cung cấp Dep Graph hay Dependencies Graph nhằm thể hiện một cách trực quan quan hệ giữa các module. Sau khi sử dụng câu lệnh sau `npx nx dep-graph` ta có thể nhìn thấy một biểu đồ dependencies được thể hiện trên trình duyệt như sau:

![](images/c7191465-9a3f-4263-8351-9d1a2ce720a3.png)

# Tổng kết

Bài viết này nói về hai cách triển khai mã nguồn này cũng như so sánh các đặc điểm của chúng và cuối cùng là giới thiệu về Nx - một công cụ phát triển thường được sử dụng Monorepos. Có thể thấy rằng việc tạo khung cho một ứng dụng web hoàn chỉnh thường mất khá nhiều thời gian và công sức. Bởi vậy một số công cụ như Nx được tạo ra nhằm mục đích đơn giản hóa quá trình đó cũng như việc quản lý ứng dụng sau này và cả việc cấu trúc theo các phương pháp tối ưu hiện nay, Bài viết đến đây là kết thúc cảm ơn mọi người đã dành thời gian đọc.
