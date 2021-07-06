---
title: Từ Monolith đến Microservices
categories:
- LFS158x - Introduction to Kubernetes

excerpt: Phần này giải thích nguyên khối là gì, thảo luận về những thách thức của kiến trúc nguyên khối trong nền tảng đám mây cùng với đó là giải thích khái niệm microservices cũng như nói về lợi thế của microservices trên đám mây và cuối cùng là mô tả cách thức chuyển đổi từ kiến trúc nguyên khối thành microservices.
layout: post
---

Hầu hết các công ty mới ngày nay đều triển khai quy trình kinh doanh của họ trên đám mây. Có thể nói rằng, công ty khởi nghiệp và doanh nghiệp mới hơn, những người đã sớm nhận ra hướng đi của công nghệ, đã phát triển các ứng dụng của họ cho đám mây. Tuy nhiên, Không phải tất cả các công ty đều may mắn như vậy. Một số đã xây dựng thành công từ nhiều thập kỷ trước trên nền tảng công nghệ cũ - các ứng dụng nguyên khối với tất cả các thành phần được kết hợp chặt chẽ và gần như không thể tách rời, một cơn ác mộng để quản lý và triển khai trên phần cứng siêu đắt đỏ. Việc chuyển đổi sang các ứng dụng sang "cloud-based" đôi khi không hề dễ dàng một chút nào nhưng sẽ cái thiện đáng kể hiệu năng ứng dụng của chúng ta, ít nhất là khi ta thực hiện đúng.

## Kiến trúc nguyên khối lỗi thời

Hầu hết các doanh nghiệp tin rằng nền tảng đám mây sẽ là ngôi nhà mới cho những ứng dụng đã lỗi thời, nhưng không phải tất cả các ứng dụng cũ đều phù hợp với nền tảng đám mấy, ít nhất là chưa.

Chuyển một ứng dụng lên đám mây có dễ dàng như đi dạo trên bãi biển, nhặt những viên sỏi vào túi và dễ dàng mang chúng đến bất cứ nơi nào cần thiết. Tuy nhiên một tảng đá 1000 tấn sẽ không dễ dàng chút nào để có thể mang đi. Tảng đá này đại diện cho ứng dụng nguyên khối - các lớp tính năng và logic dư thừa được lắng đọng được dịch thành hàng nghìn dòng mã, được viết bằng một ngôn ngữ lập trình duy nhất, không quá hiện đại, dựa trên các nguyên tắc và mẫu kiến trúc phần mềm lỗi thời.

Theo thời gian, các tính năng và cải tiến mới được bổ sung vào độ phức tạp của mã, khiến việc phát triển trở nên khó khăn hơn - thời gian tải, biên dịch và xây dựng tăng lên với mỗi bản cập nhật mới. Tuy nhiên, nó lại khiến việc quản trị trở nên khá dễ dàng vì ứng dụng đang chạy trên một máy chủ duy nhất, lý tưởng là Máy ảo hoặc Máy tính lớn (Mainframe).

Một hệ thống nguyên khối mang đến một trải nghiệm khá đắt đỏ trong phần cứng. Là một phần mềm lớn, đơn lẻ liên tục phát triển, nó phải chạy trên một hệ thống duy nhất phải đáp ứng các yêu cầu về máy tính, bộ nhớ, lưu trữ và mạng. Phần cứng có dung lượng như vậy vừa phức tạp vừa rất đắt tiền.

Vì toàn bộ ứng dụng nguyên khối chạy như một tiến trình duy nhất, nên việc mở rộng các tính năng riêng lẻ của nguyên khối là gần như không thể. Nó có trong nội bộ một số lượng kết nối và hoạt động được mã hóa cứng. Tuy nhiên, việc mở rộng quy mô toàn bộ ứng dụng có thể đạt được bằng cách triển khai thủ công một phiên bản mới của nguyên khối trên một máy chủ khác, thường nằm sau một thiết bị cân bằng tải - một giải pháp đắt tiền khác.

Trong quá trình nâng cấp, các khoảng thời gian ngừng hoạt động để phục vụ các bản vá lỗi hoặc migrations của ứng dụng nguyên khối là không thể tránh khỏi và các cửa sổ bảo trì phải được lên kế hoạch trước vì sự gián đoạn trong dịch vụ dự kiến sẽ ảnh hưởng đến khách hàng. Mặc dù có các giải pháp của bên thứ ba để giảm thiểu thời gian ngừng hoạt động cho khách hàng bằng cách thiết lập các ứng dụng nguyên khối trong một cấu hình chủ động / thụ động có sẵn cao, nhưng chúng đưa ra những thách thức mới cho các kỹ sư hệ thống để giữ tất cả các hệ thống ở cùng một mức vá lỗi và có thể đưa ra chi phí cấp phép mới có thể có.

## Kiến trúc Microservice hiện đại

Đá cuội, trái ngược với tảng đá 1000 tấn, dễ xử lý hơn nhiều. Chúng được chia tách từ nguyên khối, tách rời khỏi nhau, trở thành các thành phần rời rạc, mỗi thành phần được mô tả bởi một tập hợp các đặc điểm cụ thể. Sau khi tập hợp tất cả lại với nhau, các viên sỏi tạo nên trọng lượng của toàn bộ tảng đá. Những viên sỏi này đại diện cho các microservices được ghép nối lỏng lẻo, mỗi microservices thực hiện một chức năng nghiệp vụ cụ thể. Tất cả các chức năng được nhóm lại với nhau tạo thành chức năng tổng thể của ứng dụng nguyên khối ban đầu. Sỏi dễ dàng lựa chọn và nhóm lại với nhau dựa trên màu sắc, kích thước, hình dạng và yêu cầu nỗ lực tối thiểu để di dời khi cần thiết. Hãy thử di dời tảng đá 1000 tấn bằng cách này đi, thật dễ dàng.

Microservices có thể được triển khai riêng lẻ trên các máy chủ riêng biệt được cung cấp với ít tài nguyên hơn - chỉ những gì được yêu cầu bởi từng dịch vụ và bản thân hệ thống máy chủ, giúp giảm chi phí tài nguyên.

Kiến trúc dựa trên microservices phù hợp với các nguyên tắc Kiến trúc hướng sự kiện và Kiến trúc hướng dịch vụ (SOA), trong đó các ứng dụng phức tạp bao gồm các quy trình độc lập nhỏ giao tiếp với nhau thông qua các API qua mạng. API cho phép truy cập bởi các dịch vụ nội bộ khác của cùng một ứng dụng hoặc các dịch vụ và ứng dụng bên ngoài, bên thứ ba.

Mỗi microservice được phát triển và viết bằng ngôn ngữ lập trình hiện đại, được chọn lọc để phù hợp nhất với loại dịch vụ và chức năng kinh doanh của nó. Điều này cung cấp rất nhiều tính linh hoạt khi kết hợp microservices với phần cứng cụ thể khi được yêu cầu, cho phép triển khai trên phần cứng tiết kiệm hơn.

Mặc dù bản chất phân tán của microservices làm tăng thêm sự phức tạp cho kiến trúc, nhưng một trong những lợi ích lớn nhất của microservices là khả năng mở rộng. Với ứng dụng tổng thể trở thành mô-đun, mỗi microservice có thể được mở rộng quy mô riêng lẻ, theo cách thủ công hoặc tự động thông qua tính năng tự động chia tỷ lệ theo nhu cầu.

Quy trình nâng cấp và vá lỗi liền mạch là những lợi ích khác của kiến trúc microservices. Hầu như không có thời gian chết và không có gián đoạn dịch vụ đối với khách hàng vì các bản nâng cấp được triển khai liền mạch - một dịch vụ tại một thời điểm, thay vì phải biên dịch lại, xây dựng lại và khởi động lại toàn bộ ứng dụng nguyên khối. Do đó, các doanh nghiệp có thể phát triển và tung ra các tính năng và bản cập nhật mới nhanh hơn rất nhiều có các nhóm riêng biệt tập trung vào các tính năng riêng biệt, do đó hiệu quả hơn và tiết kiệm chi phí hơn.

## Tái cấu trúc

Các doanh nghiệp mới hơn, hiện đại hơn sở hữu kiến thức và công nghệ để xây dựng các ứng dụng gốc đám mây hỗ trợ hoạt động kinh doanh của họ.

Thật không may, đó không phải là trường hợp của các doanh nghiệp đã thành lập chạy trên các ứng dụng nguyên khối kế thừa. Một số đã cố gắng chạy ứng dụng nguyên khối dưới dạng microservices và so với điều người ta mong đợi, nó lại hoạt động không tốt. Bài học rút ra là một ứng dụng đa quy trình kích thước nguyên khối không thể chạy như một microservice bởi vậy nên bước tất yếu tiếp theo trong quá trình chuyển đổi từ nguyên khối sang microservices cần là tái cấu trúc. Tuy nhiên, việc chuyển một ứng dụng có tuổi đời hàng thập kỷ lên đám mây thông qua tái cấu trúc đặt ra những thách thức nghiêm trọng và doanh nghiệp phải đối mặt với tình thế tiến thoái lưỡng nan về phương pháp tái cấu trúc: hoặc tái cấu trúc gia tăng hoặc đập đi xây lại từ đầu.

Đập đi xây lại từ đầu thì đúng là không có gì để nói, chúng ta sẽ nói đến phương pháp tái cấu trúc gia tăng Phương pháp này đảm bảo rằng các tính năng mới được phát triển và triển khai dưới dạng các dịch vụ vi mô hiện đại có thể giao tiếp với khối nguyên khối thông qua các API mà không cần gắn vào mã của khối nguyên khối. Trong khi đó, các tính năng được tái cấu trúc từ nguyên khối sẽ dần biến mất trong khi tất cả hoặc hầu hết chức năng của nó được hiện đại hóa thành microservices. Cách tiếp cận gia tăng này cung cấp sự chuyển đổi dần dần từ kiến trúc nguyên khối kế thừa sang kiến trúc microservices hiện đại và cho phép chuyển theo từng giai đoạn của các tính năng ứng dụng lên nền tảng đám mây.

Khi một doanh nghiệp đã chọn con đường tái cấu trúc, sẽ có những thứ khác cần được cân nhắc trong quá trình này. Các thành phần nghiệp vụ nào cần tách khỏi nguyên khối để trở thành các dịch vụ vi mô phân tán, cách tách cơ sở dữ liệu khỏi ứng dụng để tách độ phức tạp của dữ liệu khỏi logic ứng dụng và cách kiểm tra các vi dịch vụ mới và các phụ thuộc của chúng, chỉ là một vài quyết định của một doanh nghiệp phải đối mặt trong quá trình tái cấu trúc.

Giai đoạn tái cấu trúc từ từ chuyển đổi ứng dụng nguyên khối thành một ứng dụng dựa trên đám mây tận dụng tối đa các tính năng của đám mây, bằng cách lập trình bằng các ngôn ngữ lập trình mới và áp dụng các mẫu kiến trúc hiện đại. Thông qua tái cấu trúc, một ứng dụng nguyên khối kế thừa nhận được cơ hội thứ hai trong cuộc sống - tồn tại như một hệ thống mô-đun được điều chỉnh để tích hợp hoàn toàn với các dịch vụ và công cụ tự động hóa đám mây có nhịp độ nhanh hiện nay.

## Thách thức

Con đường tái cấu trúc từ ứng dụng nguyên khối thành microservices có không suôn sẻ và không phải không có thách thức. Không phải tất cả các ứng dụng nguyên khối đều là ứng cử viên hoàn hảo để tái cấu trúc, trong khi một số thậm chí có thể không "sống sót" trong giai đoạn hiện đại hóa như vậy. Khi quyết định xem ứng dụng nguyên khối có phải là một ứng cử viên khả thi để tái cấu trúc hay không, có nhiều vấn đề có thể xảy ra cần xem xét.

Khi xem xét một hệ thống dựa trên Mainframe lỗi thời, được viết bằng các ngôn ngữ lập trình cũ hơn - Cobol hoặc Assembler, có thể tiết kiệm hơn nếu xây dựng lại nó từ đầu như một ứng dụng gốc đám mây. Một ứng dụng lỗi thời được thiết kế kém cần được thiết kế lại và xây dựng lại từ đầu theo các mẫu kiến trúc hiện đại cho microservices và thậm chí cả các containers. Các ứng dụng kết hợp chặt chẽ với kho dữ liệu cũng là những ứng cử viên kém cho việc tái cấu trúc.

Khi khối nguyên khối sống sót qua giai đoạn tái cấu trúc, thách thức tiếp theo là thiết kế cơ chế hoặc tìm các công cụ phù hợp để giữ cho tất cả các mô-đun tách rời tồn tại nhằm đảm bảo khả năng phục hồi của ứng dụng nói chung.

Chọn môi trường runtimes có thể là một thách thức khác. Nếu triển khai nhiều mô-đun trên một máy chủ vật lý hoặc ảo, rất có thể các thư viện và môi trường thời gian chạy khác nhau có thể xung đột với nhau gây ra lỗi và hỏng hóc. Điều này buộc việc triển khai các mô-đun đơn lẻ trên mỗi máy chủ để tách biệt các phần phụ thuộc của chúng - không phải là một cách quản lý tài nguyên tiết kiệm và không có sự phân tách thực sự giữa các thư viện và thời gian chạy, vì mỗi máy chủ cũng có một Hệ điều hành cơ bản chạy với các thư viện của nó, do đó tiêu tốn tài nguyên máy chủ - đôi khi hệ điều hành tiêu thụ nhiều tài nguyên hơn chính mô-đun ứng dụng.

Cuối cùng thì các containers của ứng dụng cũng ra đời, cung cấp môi trường thời gian chạy nhẹ được đóng gói cho các mô-đun ứng dụng. Các containers hứa hẹn cung cấp môi trường phần mềm nhất quán cho các nhà phát triển, người thử nghiệm, trong suốt quá trình từ lúc phát triển đến triển khai lên môi trường Production. Sự hỗ trợ rộng rãi containers chứa đảm bảo tính di động của ứng dụng từ máy thực sangmMáy ảo, nhưng lần này với nhiều ứng dụng được triển khai trên cùng một máy chủ, mỗi ứng dụng chạy trong môi trường thực thi riêng biệt với nhau, do đó tránh được xung đột, lỗi và thất bại. Các tính năng khác của môi trường ứng dụng container là tính năng máy chủ cao hơn, khả năng mở rộng mô-đun riêng lẻ, tính linh hoạt, khả năng tương tác và tích hợp dễ dàng với các công cụ tự động hóa.

## Tổng kết

Túm cái váy lại thì khóa học nhập môn về Kubernetes mang đến cho chúng ta những kiến thức cơ bản về kiến trúc microservices và Kubernetes để rồi khi dùng thì ta sẽ biết dùng sao cho đúng.
