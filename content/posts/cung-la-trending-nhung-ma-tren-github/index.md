---
date: "2020-11-20T09:36:00+07:00"
description: GitHub có lẽ là một khái niệm quá quen thuộc đối với các lập trình viên, bởi nó là một trang web được các developer vô cùng yêu thích, với tần suất truy cập rất lớn và thường xuyên. Là một trong những kho mã nguồn mở lớn nhất hiện nay, hiển nhiên mà nói, GitHub có vô vàn dự án mã nguồn thú vị trên đó. Bởi vậy nên bài viết này giới thiệu về Github Trending - một tính năng độc đáo của GitHub giúp chúng ta nhanh chóng khám phá các dự án mới nổi hoặc nổi từ lâu rồi và giờ vẫn thế cũng như giới thiệu một số dự án được quan tâm bởi đông đảo người dùng GitHub trong thời gian vừa qua.
layout: post
title: "C\u0169ng l\xE0 Trending, nh\u01B0ng m\xE0 tr\xEAn GitHub"

categories:
  - Một vài tip nhỏ
  - Cách tìm kiếm tài liệu

tags:
  - GitHub
---

GitHub có lẽ là một khái niệm quá quen thuộc đối với các lập trình viên, bởi nó là một trang web được các developer vô cùng yêu thích, với tần suất truy cập rất lớn và thường xuyên. Là một trong những kho mã nguồn mở lớn nhất hiện nay, hiển nhiên mà nói, GitHub có vô vàn dự án mã nguồn thú vị trên đó. Bởi vậy nên bài viết này giới thiệu về Github Trending - một tính năng độc đáo của GitHub giúp chúng ta nhanh chóng khám phá các dự án mới nổi hoặc nổi từ lâu rồi và giờ vẫn thế cũng như giới thiệu một số dự án được quan tâm bởi đông đảo người dùng GitHub trong thời gian vừa qua.

## GitHub Trending và cách GitHub tính điểm reputation

GitHub là hệ thống cung cấp dịch vụ lưu trữ để phát triển phần mềm và kiểm soát phiên bản bằng Git. Hệ thống này cung cấp chức năng kiểm soát phiên bản phân tán và quản lý mã nguồn của Git, cùng với các tính năng riêng của nó và một số tính năng cộng tác như theo dõi lỗi, yêu cầu tính năng, quản lý tác vụ, tích hợp liên tục và wiki cho mọi dự án. ( <https://en.wikipedia.org/wiki/GitHub>)

Bên cạnh là kho lưu trữ mã nguồn, GitHub hoạt động giống như một mạng xã hội cho lập trình viên, nó cung cấp các chức năng mạng xã hội như feed, theo dõi, wiki (sử dụng phần mềm Gollum Wiki) và đồ thị mạng xã hội để hiển thị cách các nhà phát triển làm việc trên kho lưu trữ. Bởi vậy nên, GitHub cung cấp cho chúng ta công cụ `Explore` nhằm dễ dàng khám phá các dự án mã nguồn dựa trên nhiều tiêu chí khác nhau mà một trong số đó là xu hướng trong một khoảng thời gian xác định mà có thể là theo ngày, theo tuần và theo tháng.

![](images/0ce05bce-0f30-4101-8db4-2b5ae3eead9c.png)

Vậy để có được danh sách như trên, GitHub dựa trên tiêu chí nào để lọc ra những dự án đang được quan tâm nhất trong hàng triệu dự án đang công bố mã nguồn trên nền tảng của nó. Tiêu chí đầu tiên có thể dễ dàng nhận ra nhất là tổng số `stars` (thứ tương đương với lượng like trên Facebook hoặc up vote trên Reddit) trong khoảng thời gian đã chọn. Tuy nhiên vẫn có một số repo nhận được số `stars` tương đương nhưng lại không được đề cập trong danh sách này. Vậy nên, tổng số `stars` không phải là tiêu chí duy nhất được áp dụng mà bên cạnh nó có thể là số lượt được xem, số lượt fork, và quan trọng nhất là chất lượng cũng như ý tưởng và lợi ích mà nó mang đến cho cộng đồng. Dù sao đi nữa những tiêu chí trên cũng là phỏng đoán vì phía GitHub cũng không công bố tiêu chuẩn của họ và tất nhiên, mặc dù khuyến khích việc khám phá cũng đóng góp cho vô vàn dự án mã nguồn mở có trên hệ thống của mình, ban vận hành hệ thống GitHub không mong muốn nơi này trở thành chỗ chỉ để mọi người đua trending. :smiley_cat:

![](images/b43dd4e3-3d6d-4d18-ba5c-6864a0bb8b74.png)

## Một số dự án mã nguồn mở được chú ý tại thời điểm hiện tại

Dựa vào GitHub Trending, chúng ta dễ dàng thấy được những repo được quan tâm bởi đông đảo người dùng GitHub trong thời gian vừa qua. Phần sau đây liệt kê một số dự án mà mình cảm thấy thú vi được liệt kê trong thời điểm hiện tại. Để có thể tìm hiểu thêm các repo khác cũng như cập nhật danh sách trending hiện tại, mọi người có thể xem tại <https://github.com/trending>

![](images/0ad3c14c-70f7-4f45-a02c-2194cd0791ab.png)

### servo/servo

`servo` là thư viện cung cấp một công cụ web độc lập, được cấu trúc dưới dạng module, có thể nhúng, cho phép các nhà phát triển cung cấp nội dung và ứng dụng bằng các tiêu chuẩn web. Servo được viết bằng Rust và chia sẻ mã với Mozilla Firefox và hệ sinh thái rộng lớn của Rust.
![](images/20e17263-5516-456a-a26a-715c75b8cd8a.png)

Được khởi tạo từ năm 2014, lượng star của `servo` khá tăng khá ổn định trong 6 năm qua và hiện nay đã đạt mức 18,000 stars. Bên cạnh đó, repo này có 43,325 lượt commits và nhận được xấp xỉ 2,500 lần lượt fork từ cộng đồng. Có thể nói rằng Rust là một trong những ngôn ngữ mạnh mẽ trong thời điểm hiện nay, và tất nhiên các thư viện của nó cũng sẽ nhận được sự quan tâm của cộng đồng.
![](images/f2a6d22c-9e2a-4763-b18f-b61cf75f344c.png)

### streamich/react-use

React là thư viện JavaScript phổ biến nhất để xây dựng giao diện người dùng. Hệ sinh thái khổng lồ xung quanh thư viện này gồm vô vàn những thư viện phụ trợ mạng mẽ mà `react-use` là một trong số đó. Dựa trên phần tự giới thiệu của thư viện này, chúng ta có thể hiểu rằng nó là một `tập hợp những React Hooks thiết yếu` _(nguyên văn: Collection of essential React Hooks.)_

![](images/64efa422-b3c1-4839-9e16-a3afe02826aa.png)
Chỉ mới được khởi tạo năm 2019, tuy nhiên thư viện này phát triển với tốc độ đáng kinh ngạc khi đạt được xấp xỉ 16,900 stars chỉ với 2 năm cũng như có gần 1,200 lượt fork từ cộng đồng.

![](images/ebe8f948-7de9-44c6-b036-1a9b6df62f8d.png)

### beautifulinteractions/beautiful-react-diagrams

Có thể nói rằng React đang phát triển không ngừng và áp đảo các thư viện/framework của JavaScript được dùng để xây dựng giao diện người dùng khác khi mà có đến 3 thư viện phụ trợ nằm trong top 6 những repo được quan tâm nhiều nhất trong thời điểm hiện tại mà một trong số đó là `beautiful-react-diagrams`. Thư viện này giúp chúng ta có thể tạo, chỉnh sửa cũng như tương tác với một số loại biểu đồ một cách dễ dàng.

![](images/5e030815-1e38-437d-a952-021c249706ae.gif)

Chỉ mới được khởi tạo từ tháng 7 năm này, lượng stars của dự án này tăng đột biến trong tháng 11 này, có thể do tính năng của thư viện vào thời điểm đó đã hoàn thiện và thu hút được sự chú ý của đông đảo cộng đồng. Mặc dù vẫn còn khá ít tính năng, tuy nhiên việc tăng mạnh số stars từ 200 lên xấp xỉ 1,400 trong chưa đầy 1 tháng cho chúng ta thấy rằng đây là thư viện vô cùng tiềm năng và có thể phát triển mạnh hơn nữa trong thời gian tới.
![](images/9e32636a-8418-47bf-8683-b10d500f2ed0.png)

### y1ndan/genshin-impact-helper

Giới thiệu qua thì Genshin Impact là tựa game cực kì nối tiếng được phát hành trong thời gian gần đây. Trong ngày đầu phát hành, Genshin Impact đã có hơn 110.000 người xem cùng lúc trên Twitch. Phiên bản trên iOS của trò chơi ngay lập tức leo lên hạng 2 ở App Store Trung Quốc, chỉ sau TikTok. Theo Qimai Data, chỉ riêng phiên bản iOS đã đem về doanh thu khoảng 1,84 triệu USD ở ngày đầu phát hành cho nhà phát triển.

Là một game theo kiểu Gacha ~~hút máu~~, nôm na là quay tướng may mắn trúng thưởng, Genshin có phần thưởng hàng ngày cho việc đăng nhập, mặc dù bèo bọt nhưng mà có còn hơn không. Từ vấn đề trên, mạnh thường quân `y1ndan` đã cung cấp giải pháp đăng nhập tự động thay vì phải đăng nhập bằng cơm để ta có thể tiết kiệm vài phút hàng ngày mà vẫn có thể có cơ hội nhận 6W + Mora và sách kinh nghiệm màu tím tím.

![](images/114878d5-671c-4dd8-9344-905643de659b.png)

Do chỉ mới được khởi tạo dự án vào đầu tháng 11 cũng như chỉ phục vụ mục đích đăng nhập tự động Genshin Impact nên repo này có lượng stars khá khiêm tốn khi chỉ có 494 stars cho đến thời điểm hiện tại. Tuy nhiên có thể do cần fork để sử dụng chung với GitHub Workflows nên số lượt fork của repo này rất cao, xấp xỉ 1,900 lượt, nhiều hơn cả `react-use`. Có thể do lý do này mà `genshin-impact-helper` được liệt kê trong danh sách Trending :sweat_smile:
![](images/1415c0aa-d2d4-407b-96c0-130ab0062765.png)

> Không hiểu sao mình vào link mà tác giả cung cấp thì lại không đăng nhập được :cry: có thể là do cách này chỉ áp dụng cho máy chủ nội địa. Thôi thì hàng ngày vào đăng nhập vậy.

### trekhleb/javascript-algorithms

Dự án cuối cùng mà mình đề cập là `javascript-algorithms` - dự án cung cấp mã cài đặt của các thuật toán và cấu trúc dữ liệu trong JavaScript cùng với giải thích và liên kết đến tài liệu tham khảo. Cài đặt hầu hết các thuật toán cũng như các kiểu cấu trúc dữ liệu phổ biến một cách tường mình dễ đọc, dễ hiểu cùng với giải thích ngắn gọn rõ ràng là hai yếu tố chính khiến dự án này là tài liệu tốt để mọi người có thể ôn luyện về cấu trúc dữ liệu và giải thuật.

![](images/3ad0b39f-841a-4448-8812-205f535bb05c.png)

Được khởi tạo từ đầu năm 2019, lượng star của `servo` khá tăng khá ổn định trong những năm qua và hiện nay đã đạt mức 80,200 stars. Bên cạnh đó, repo này có gần 3,500 lượt watch và nhận được xấp xỉ 14,500 lần lượt fork từ cộng đồng. Có thể nói, chất lượng mã nguồn cao và được giải thích tường minh rõ ràng cùng với việc có chủ đề là thuật toán nói chung, vốn là thứ mà bất kì ai muốn học lập trình đều phải tìm hiểu qua và dễ tiếp cận là hai yếu tố chính giúp dự án mã nguồn này được chú ý nhiều như vậy.
![](images/722059c1-ed79-47dc-80b3-3f301a31d781.png)

## Tổng kết

Bài viết này giới thiệu về Github Trending - một tính năng độc đáo của GitHub giúp chúng ta nhanh chóng khám phá các dự án đang là xu hướng trên GitHub cũng như giới thiệu một số dự án được quan tâm bởi đông đảo người dùng GitHub trong thời gian vừa qua.Việc tham khảo mã nguồn cũng như đóng góp cho các dự án mã nguồn mở giúp ích chúng ta rất nhiều trong việc cải thiện coding skill của bản thân. Bài viết đến đây là hết, cảm ơn mọi người đã dành thời gian đọc.

# Tài liệu tham khảo

- <https://github.com/trending>
- <https://github.community/t/how-github-detect-trending-repositories/824/8>
- <https://en.wikipedia.org/wiki/GitHub>
