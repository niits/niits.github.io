---
date: 2021-02-17T23:23:47+07:00

title: Machine learning trên môi trường Production
categories:
  - Machine Learning Systems Design

excerpt: Phần này dịch từ Note của so sánh cách triển khai Machine learning khi nghiên cứu và triển khai trên môi trường Production cũng như nêu các vấn đề cần quan tâm khi triển khai các ứng dụng machine learning ở môi trường thực tế.
layout: post
---

## Mục tiêu của Machine learning

Các nghiên cứu cũng như các dự án về Machine learning trong môi trường học thuật nói chung hầu hết đều hướng đến một mục tiêu duy nhất - đề xuất một phương pháp mới nhằm nâng cao hiệu suất để vượt qua các thành tựu của các state-of-the-art trong một bài toán cụ thể mà một số trong các phương pháp đó có thể khiến mô hình trở nên khá phức tạp để có thể sử dụng.

Trong khi đó, trong môi trường doanh nghiệp, có một số sự đánh đổi cần có để có thể triển khai các hệ thống Machine learning trên môi trường Production. Thông thường, các hệ thống này thường giải quyết một tập các bài toán liên quan đến nhau chứ không tập trung chuyên sâu vào khía cạnh nào đó. Chẳng hạn như hệ thống Machine learning phục vụ cho một nền tảng đăng bài sẽ cần giải quyết một số bài toán như sắp xếp các bài viết dựa trên chất lượng, đề xuất các bài viết phù hợp với người dùng, ...Khi đó hệ thống Machine learning của chúng ta là một tập các giải pháp nhằm giải quyết nhiều bài toán khác nhau và mỗi sự thay đổi dù nâng cao hiệu suất của một khía cạnh nhưng lại khiến cả hệ thống có nguy cơ trở nên quá phức tạp để vận hành từ đó ảnh hưởng xấu đến hiệu năng toàn hệ thống.

Bên cạnh đó, việc cải thiện hiệu năng đôi khi không mang lại quá nhiều lợi ích cũng như doanh thu cho mô hình kinh doanh. Chẳng hạn việc tiêu thụ tài nguyên được cải thiện một chút chẳng hạn 2% cũng đủ giúp các tập đoàn lớn tiết kiệm hàng triệu đô la. Tuy nhiên, độ chính xác của một bộ nhận dạng giọng nói tăng từ 92% lên 95% cũng không giúp ích quá nhiều cho việc nâng cao trải nhiệm người dùng mà đôi khi còn gây ra nhiều khó khăn đội ngũ phát triển khi logic của ứng dụng có thể cần được thay đổi và cần thời gian để đảm bảo hệ thống hoạt động ổn định.

## Dữ liệu

Khác với khi nghiên cứu, dữ liệu được sử dụng trong các môi trường thực tế thường không phải là dữ liệu sạch và có định dạng chuẩn mực. Có thể nói, quá trình nghiên cứu tập trung thử nghiệm các phương pháp mới, bởi vậy dữ liệu được sử dụng thường là các bộ dữ liệu uy tín được sử dụng rộng rãi làm thước đo chung để đánh giá hiệu năng cũng như độ chính xác và hiển nhiên chúng cần có chất lượng tốt mới, ổn định nên mới được sử dụng nhiều như vậy.

Trong khi đó, dữ liệu dùng cho các ứng dụng thực tế thường không được như vậy. Nó được thu thập từ nhiều nguồn, chứa nhiều nhiễu do dữ liệu bị sai lệch hoặc bị thiếu do sự cố trong quá trình thu thập và thường không có cấu trúc nhất quán. Hơn thế, dữ liệu mới được cập nhật thường xuyên và nhãn của tập dữ liệu có thể không đồng đều và có thể cần thay đổi ví dụ như thêm, bớt hoặc chia nhỏ nhãn mỗi khi yêu cầu của dự án thay đổi, ngay cả khi mô hình đã được huấn luyện và triển khai. Việc dữ liệu được tạo ra liên tục bởi người dùng, hệ thống và dữ liệu của bên thứ ba khiến cho quá trình huấn luyện mô hình cần có cơ chế đặc thù để giảm thiểu rủi ro, cũng như tính bền bỉ của ứng dụng.

Bên cạnh đó, dữ liệu được sử dụng trong các ứng dụng thực tế cần được đảm bảo tuân theo các quy định chung và tôn trọng quyền riêng tư của người cung cấp.

## Dễ hiểu, để sử dụng, dễ bảo trì

Do đặc thù của môi trường nghiên cứu, khả năng có thể được diễn giải một cách tường minh của mô hình không quá quan trọng bởi các nhà nghiên cứu sẵn sàng bỏ ra rất nhiều thời gian để hiểu rõ ràng cách thức hoạt động của một phương pháp nào đó. Còn đối với giai đoạn triển khai thành ứng dụng thực tế, các mô hình này cần được hiểu ở mức độ nào đó một cách dễ dàng bởi các bên triển khai và nhất là người dùng cũng như phía quản lý mô hình kinh doanh, những người hầu như có ít hiểu biết về lĩnh vực này để họ tin tưởng vào mô hình được cung cấp và từ đó có thể nhận biết và báo cáo mỗi khi có sự cố xảy ra. Việc được báo cáo các sự cố sẽ giúp quá trình gỡ lỗi và cải thiện mô hình - vốn là quá trình quyết định sự sống còn của ứng dụng - có thể phát huy được tác dụng.

## Ứng dụng sử dụng Machine learning và ứng dụng truyền thống

Vì ML là một phần của kỹ thuật phần mềm và phần mềm đã được sử dụng thành công trong sản xuất trong hơn nửa thế kỷ, một số người có thể thắc mắc tại sao ta không áp dụng các phương pháp hay nhất đã thử nghiệm trong kỹ thuật phần mềm và áp dụng chúng cho ML. Tuy nhiên do đặc thù của các ứng dụng dựa trên Machine learning, khi mà dữ liệu liên tục được thay đổi và ảnh hưởng trực tiếp vào hiệu năng của ứng dụng chứ không được tách rời như các ứng dụng phần mềm truyền thống, ta luôn có những thách thức riêng dành cho các loại ứng dụng này.

Trong các ứng dụng truyền thống, ta chỉ cần tập trung vào thử nghiệm cài đặt mã theo từng phiên bản thì với ML, ta cũng phải thử nghiệm và huấn luyện mô hình dựa trên các phiên bản dữ liệu của mình. Vấn đề được đặt ra là làm thế nào để biết một mẫu dữ liệu tốt hay xấu cho hệ thống sẵn có bởi không phải tất cả các mẫu dữ liệu đều có đóng góp tới việc cải thiện hiệu năng như nhau. Bên cạnh đó, việc sử dụng dữ liệu có sẵn một cách bừa bãi có thể ảnh hưởng đến hiệu suất của mô hình có sẵn và thậm chí khiến mô hình dễ bị tấn công nhiễm độc dữ liệu. Những vấn đề được liệt kê dưới đây được cho là những vấn đề mà các kỹ sư Machine learning thường gặp phải khi phát triển và triển khai ứng dụng của mình:

- Kiểm tra dữ liệu: Đảm bảo tính đúng đắn và đánh giá tính hữu ích của bộ dữ liệu
- Kiểm soát phiên bản dữ liệu và mô hình: Lập chỉ mục phiên bản cho dữ liệu và mô hình, đảm bảo khả năng so sánh giữa hai phiên bản nhằm rút ra được những thay đổi nào được áp dụng và điều đó đóng góp như thế nào vào việc cải thiện chất lượng ứng dụng.
- Gắn nhãn dữ liệu: Đảm bảo có phương pháp hiệu quả để nhanh chóng gắn nhãn dữ liệu mới hoặc gắn nhãn lại dữ liệu hiện có cho mô hình mới
- Tích hợp CI/CD: Đảm bảo quá trình thử nghiệm diễn ra nhanh chóng và đảm bảo được mô hình đạt được mục tiêu kì vọng
- Triển khai: Đảm bảo có phương pháp hiệu quả để đóng gói và triển khai một mô hình mới hoặc thay thế một mô hình hiện có
- Nén mô hình: Đảm bảo có phương pháp để giảm kích cỡ một mô hình ML để phù hợp với các thiết bị tiêu dùng?
- Tối ưu hóa quá trình Inference: Đảm bảo có phương pháp hiệu quả để tăng tốc thời gian suy luận cho các mô hình từ đó giảm thiểu tối đa độ trễ
- Thiết bị cạnh: Phần cứng được thiết kế để chạy các thuật toán ML nhanh và với giá thành và chi phí vận hành nhỏ
- Quyền riêng tư: Cần có quy trình để sử dụng dữ liệu người dùng để đào tạo mô hình hiệu quả trong khi vẫn bảo vệ quyền riêng tư của họ
