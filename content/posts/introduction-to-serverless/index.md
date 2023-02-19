---
title: "Efficientnet"
date: 2022-03-17T23:23:47+07:00

title: Tổng quan về Serverless
categories:
  - "LFS157x - Introduction to Serverless on Kubernetes"

layout: post
---
Trong chương này, chúng ta sẽ khám phá thuật ngữ Serverless nghĩa là gì và điều gì tạo nên một chức năng so với một dịch vụ web thông thường. Chúng tôi cũng sẽ đối chiếu hai quan điểm về ý nghĩa của một ứng dụng không có máy chủ.

Đến cuối chương này, bạn sẽ có thể:

- Cung cấp một định nghĩa về máy chủ và FaaS (Function as a Service).
- Thảo luận về sự khác biệt giữa các functions sử dụng trong điện toán đám mây và Kubernetes.

## Serverless là gì?

Serverless đã trở thành một trong những thuật ngữ gây chia rẽ nhất của điện toán đám mây, với hai trường phái suy nghĩ về điều gì làm cho phần mềm trở thành “Serverless”. Cũng có một số thảo luận về sự khác biệt giữa Phương thức như một Dịch vụ (Functions as a Service - FaaS) và Serverless.

Để bắt đầu, Serverless là một thuật ngữ trừu tượng và nó không được hiểu theo nghĩa đen, mặc dù nhiều người làm như vậy. Giống như cách mà Cloud Computing không đặt máy chủ trên mây, Serverless không liên quan đến việc thực thi mã mà không có máy chủ. Nó chỉ đơn thuần đề cập đến trải nghiệm mà người dùng hoặc khách hàng có và thể hiện sự liên tục về mức độ cần thiết của một người để làm việc với phần cứng và cơ sở hạ tầng.

![serverless-axis](serverless-axis.png)

Khi chúng ta di chuyển dọc theo trục, chúng ta thấy rằng cùng với mối quan tâm giảm xuống đối với cơ sở hạ tầng, thì khối lượng công việc cũng giảm và quy mô của nó giảm xuống. Do đó, nếu Serverless là một cách tiếp cận và một mẫu kiến ​​trúc thì Functions và FaaS là ​​một tập hợp con của nó, cung cấp một cách cụ thể để áp dụng công nghệ và lý tưởng.

## Đặc trưng of Functions

Khi được so sánh với kiến trúc nguyên khối, các functions thường có xu hướng:

- Cho phép nhà phát triển tập trung vào mã, thay vì cơ sở hạ tầng và tạo tác triển khai.
- Tạo các tạo tác nhỏ hơn và ít dòng mã hơn, vì chúng có ít trách nhiệm hơn (thậm chí có thể chỉ một).
- Chúng có thể là hướng sự kiện (được kích hoạt bởi cơ sở dữ liệu, cửa hàng đối tượng, pub-sub) hoặc được triển khai dưới dạng điểm cuối REST và được truy cập qua HTTP.
- Chúng dễ quản lý vì chúng không dựa vào bộ nhớ cơ bản.
- Và chúng là đẳng cấu - nghĩa là mỗi chức năng phải tuân theo một giao diện môi trường chạy nhất định.

## Nền tảng Serverless

Nền tảng không máy chủ chịu trách nhiệm cho việc tự động mở rộng quy mô, có hai dạng:

- Thu phóng các functions nhằm tiết kiệm chi phí và giảm tải cho hệ thống khi các chức năng không hoạt động. Không phải tất cả các nền tảng đều hỗ trợ thu nhỏ về 0 (scale to zero) và việc mở rộng quy mô trở lại từ không bản sao có xu hướng liên quan đến hình phạt độ trễ được gọi là “khởi động lạnh” khi mã được triển khai lại hoặc khởi tạo lại.

- Mở rộng các functions theo tỷ lệ khi nhu cầu trên một điểm cuối cụ thể tăng lên.

Với định nghĩa Serverless và FaaS như trên, chúng ta hãy quan sát kĩ hơn hai quan điểm từ cộng đồng.

Các nhà cung cấp đám mây sở hữu và vận hành các sản phẩm SaaS không máy chủ sẽ khiến bạn tin rằng một nền tảng chỉ là Serverless nếu:

- Nhà phát triển chỉ cần viết các khối mã nhỏ được gọi là hàm
- Khách hàng được lập hóa đơn dựa trên mức tiêu thụ (mili giây, không tính thời gian nhàn rỗi).
- Nhà phát triển hoặc nhà điều hành đều không được có bất kỳ quyền truy cập hoặc kiến thức nào về các máy chủ mà mã đang chạy trên đó.
- Mã có thể mở rộng quy mô để thực thi song song lớn.
- Có rất ít hoặc không cần phải di chuyển mã giữa các đám mây.

Bên cạnh đó, một số khác, chẳng hạn như các thành viên của Serverless Working Group trong Cloud Native Computing Foundation (CNCF) tin rằng, mặc dù quan điểm này phổ biến, nhưng nó không phải là sự phân biệt và rằng serverless là một cách tiếp cận triết học hơn là một tập hợp các ràng buộc theo nghĩa đen:

- Nhà phát triển chỉ viết các khối mã nhỏ được gọi là functions hoặc microservices.
- Serverless là một trải nghiệm dành cho nhà phát triển, chứ không phải là một tập hợp các ràng buộc theo nghĩa đen.
- Mối quan tâm của nhà phát triển đối với máy chủ được giảm bớt, nếu không được loại bỏ hoàn toàn, trong khi nhà điều hành vẫn có thể phải gộp các tài nguyên lại với nhau.
- Mã có thể mở rộng quy mô song song lớn bằng cách sử dụng các tài nguyên hiện có hoặc bằng cách cung cấp các tài nguyên mới trên đám mây, đúng lúc.
- Việc lập hóa đơn theo giây không phải là mối quan tâm đối với nhà phát triển.
- Có thể di chuyển mã giữa các hệ thống hoặc đám mây khác nhau nếu được yêu cầu.

Ở một mức độ nào đó, cuộc thảo luận chuyển từ *“Serverless có phải là một tập hợp các quy tắc cứng và nhanh”, sang "hệ thống Serverless trông như thế nào? Trải nghiệm như thế nào đối với người điều hành và đối với nhà phát triển?"* Điểm chung từ hai khía cạnh là các nhà phát triển nên quan tâm ít hơn đến cơ sở hạ tầng và viết các khối mã nhỏ để dễ triển khai.

Trong CNCF, có nhiều dự án được xây dựng để chạy trên Kubernetes, với một số dự án cung cấp kiểu trải nghiệm kiểu Serverless. Khách hàng sử dụng các nền tảng này vì chúng có thể di động và có thể chạy trong trung tâm dữ liệu của khách hàng hoặc đám mây đã chọn mà không cần thay đổi ứng dụng.
