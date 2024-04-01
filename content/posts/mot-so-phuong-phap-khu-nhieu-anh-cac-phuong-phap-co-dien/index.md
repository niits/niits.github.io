---
date: "2021-07-18T14:33:54+07:00"
description: Là một tác vụ cơ bản của xử lý ảnh và thị giác máy, khử nhiễu đóng vai trò khá quan trọng nhằm cải thiện chất lượng đầu rvào trước khi xử lý ở các bước tiếp theo. Trong bài viết này ta sẽ cùng tìm hiểu về các phương pháp khử nhiễu phổ biến thường được gọi là các phương pháp cổ điển và từ đó có cơ sở để so sánh với các phương pháp sử dụng học sâu ở bài tiếp theo (nếu mình có viết)
layout: post
title:
  "M\u1ED9t s\u1ED1 ph\u01B0\u01A1ng ph\xE1p kh\u1EED nhi\u1EC5u \u1EA3nh: C\xE1\
  c ph\u01B0\u01A1ng ph\xE1p c\u1ED5 \u0111i\u1EC3n"

categories:
  - Kiến thức cơ bản

tags:
  - Image Processing
  - Computer Vision
  - Noise Reduction
  - Image Denoising
---

Là một tác vụ cơ bản của xử lý ảnh và thị giác máy, khử nhiễu đóng vai trò khá quan trọng nhằm cải thiện chất lượng đầu rvào trước khi xử lý ở các bước tiếp theo. Trong bài viết này ta sẽ cùng tìm hiểu về các phương pháp khử nhiễu phổ biến thường được gọi là các phương pháp cổ điển và từ đó có cơ sở để so sánh với các phương pháp sử dụng học sâu ở bài tiếp theo (nếu mình có viết)

# Nhiễu và khử nhiễu

## Khái niệm về nhiễu

Do các hạn chế vật lý vốn có của các thiết bị ghi khác nhau, hình ảnh có xu hướng bị nhiễu ngẫu nhiên trong quá trình thu nhận hình ảnh. Nhiễu có thể hiểu là hiện tượng méo tín hiệu cơ bản gây cản trở quá trình quan sát hình ảnh và trích xuất thông tin. Với sự gia tăng mạnh mẽ của việc tạo ra hình ảnh kỹ thuật số thường được chụp trong điều kiện không khí/ánh sáng kém, các phương pháp khôi phục hình ảnh đã trở thành công cụ không thể thiếu trong kỷ nguyên phân tích có sự hỗ trợ của máy tính.

![](images/717f5817-8408-429c-8328-81b71478ec27.jpg)

> Ví dụ về nhiễu trong ảnh được lấy từ trang https://stackoverflow.com/questions/54089393/noise-removal-from-rgb-image-matlab

Bên cạnh hai lý do trên, nhiễu xuất hiện trong ảnh bởi nhiều nguyên nhân như do sự thay đổi độ nhạy của đầu dò, do sự biến đổi của môi trường, do chính bản thân chất liệu sinh ra, do sai số lượng tử hóa hay sai số truyền,... Tất cả các nguyên nhân gây ra nhiễu ở trên đã sinh ra nhiễu được phân thành hai loại chính như sau:

- **Nhiễu Gauss**: Nhiễu này có được do bản chất rời rạc của bức xạ (hệ thống ghi ảnh bằng cách đếm các photon (lượng tử ánh sáng). Mỗi pixel trong ảnh nhiễu là tổng giá trị pixel đúng và pixel ngẫu nhiên .
- **Nhiễu muối – tiêu (Salt & Pepper noise)**: Nhiễu này sinh ra do xảy ra sai số trong quá trình truyền dữ liệu. Những pixel đơn được đặt luân phiên mang giá trị bằng 0 hay giá trị cực đại tạo ra hình chấm dạng muối tiêu trên ảnh.
- **Nhiễu Shot hay nhiễu Poisson**: Nhiễu này sinh ra do trong quá trình thu nhận, số lượng lớn hạt photon đã tập trung vào một điểm và chúng đã tạo ra nhiễu tại điểm đó. Nhiễu được đặc trưng bởi hàm mật độ phân bố xác suất Poisson, nên được gọi là nhiễu Poisson
- **Nhiễu Speckle hay nhiễu đốm**: Là loại nhiễu phát sinh do ảnh hưởng của điều kiện môi trường lên cảm biến hình ảnh trong quá trình thu nhận hình ảnh. Nhiễu lốm đốm hầu như được phát hiện trong trường hợp ảnh y tế, ảnh Radar hoạt động và ảnh Radar khẩu độ tổng hợp (SAR).

## Các phương pháp khử nhiễu

### Mô hình hóa vấn đề

Có nhiều nguồn nhiễu trong hình ảnh và những nhiễu này đến từ nhiều khía cạnh khác nhau như thu nhận, truyền và nén hình ảnh. Về mặt công thức hóa một ảnh nhiễu $v(x)$ được tạo thể hiện là tổng của ảnh gốc không chứa nhiễu $u(x)$ và hàm lỗi $\eta(x)$ như sau:

$$v(x) = u(x) + \eta(x)$$

Mục đích hướng đến của các phương pháp khử nhiễu là giảm nhiễu trong hình ảnh tự nhiên đồng thời giảm thiểu việc mất các tính năng gốc và cải thiện độ nhiễu tín hiệu (SNR). Để giải quyết vấn đề này, khá nhiều phương pháp khử nhiễu đã được đề xuất và sử dụng. Về cơ bản chúng có thể chia thành nhiều nhóm chính và hai trong số đó là các phương pháp khử nhiễu cổ điển (được trình bày ở bài viết này) và các phương pháp dựa trên các mô hình học sâu, cụ thể là CNN ()

### Các phương pháp khử nhiễu cổ điển

Phương pháp cổ điển dựa trên miền không gian loại bỏ nhiễu bằng cách tính toán giá trị xám của mỗi pixel dựa trên mối tương quan giữa các pixel / mảng ảnh trong ảnh gốc. Nói chung, các phương pháp miền không gian có thể được chia thành hai loại: lọc miền không gian (Spatial domain filtering) và các phương pháp khử biến thiên (Variational denoising methods)

#### Các phương pháp lọc miền không gian

Vì bộ lọc là một phương tiện chính để xử lý hình ảnh, một số lượng lớn các bộ lọc không gian đã được áp dụng để làm giảm hình ảnh, có thể được phân loại thêm thành hai loại như sau:

- Lọc tuyến tính: các bộ lọc tuyến tính thường được sử dụng để loại bỏ nhiễu trong miền không gian, nhưng chúng không giữ được kết cấu hình ảnh
  - Bộ lọc trung bình đã được áp dụng để giảm nhiễu Gaussian, tuy nhiên, nó có thể làm mịn ảnh quá mức với nhiễu cao.
  - Bộ lọc Wiener đã được sử dụng nhằm khắc phục nhược điểm của lọc trung bình, nhưng nó cũng có thể dễ dàng làm mờ các cạnh.
- Lọc phi tuyến tính.
  - Bộ lọc trung vị và bộ lọc trung vị có trọng số có thể được loại bỏ nhiễu mà không cần bất kỳ phương pháp nhận dạng nào.
  - bộ lọc Bilateral được sử dụng rộng rãi để làm nhiễu hình ảnh do bản chất là bộ lọc làm mịn không tuyến tính, bảo toàn cạnh và giảm nhiễu. Giá trị cường độ của mỗi pixel được thay thế bằng giá trị cường độ trung bình có trọng số từ các pixel lân cận. Tuy vậy bộ lọc này có nhược điểm rằng việc thực hiện brute-force của nó mất khá nhiều thời gian $O(Nr^2)$, cực kỳ cao khi bán kính kernel lớn.

Nhìn chung, các bộ lọc không gian loại bỏ nhiễu ở một mức độ hợp lý nhưng với chi phí là làm mờ hình ảnh, do đó làm mất các cạnh sắc nét.

#### Các phương pháp khử nhiễu biến thiên

Các phương pháp làm nhiễu hiện có sử dụng giá trị gốc của hình ảnh và tối thiểu hóa hàm năng lượng E để tính toán ảnh đã khử nhiễu $\hat{x}$. Đầu tiên, ta nhận được một hàm E từ một hình ảnh nhiễu y, và sau đó một số thấp tương ứng với hình ảnh không bị nhiễu thông qua quy trình ánh xạ. sẽ được tính toán. Sau đó, ta có thể xác định một ảnh đã được khử nhiễu $\hat{x}$ bằng cách tối thiểu hóa hàm E:

$$
\hat{x} ∈ arg  min_x E(x)
$$

Từ quan điểm Bayes, ước tính xác suất MAP của $x$ là

$$
\hat{x} = argmax_x P(x|y) = argmax_x \frac{P(y|x)P(x)}{P(y)}
$$

Và có thể được xây dựng tương đương như:

$$
\hat{x} =  argmax_x (logP(y|x) + log P(x))
$$

trong đó số hạng đầu tiên $P (y|x)$ là một hàm likelihood của x, và số hạng thứ hai $P(x)$ đại diện cho hình ảnh trước đó. Trong trường hợp của AWGN, hàm mục tiêu thường có thể được xây dựng dưới dạng như sau:

$$
\hat{x} = arg min \frac{1}{2} ||y- x||_2^2 + λRx
$$

Trong đó $||y- x||_2^2$ là thuật ngữ biểu thị sự khác biệt giữa hình ảnh gốc và hình ảnh bị nhiễu. $R(x) = ‐log P(x)$ biểu thị thuật ngữ chính quy và and là tham số chính quy. Đối với các phương pháp làm giảm biến phân, điểm quuan trọng để giải quyết là tìm một hình ảnh phù hợp trước ($R(x)$). Các mô hình thành công trước đây bao gồm priors gradient, priors tự tương tự không cục bộ (NSS), priors thưa thớt và priors cấp thấp. Phần tiếp theo sau đây trình bày một phương pháp có tên là Total variation regularization (Tổng biến đổi chính quy)

#### Total variation regularization

Bắt đầu với chính quy Tikhonov, những lợi thế của chính quy không bậc hai (non-quadratic regularizations) đã được nghiên cứu trong một thời gian dài. Mặc dù phương pháp Tikohonov là phương pháp đơn giản nhất trong đó $R(x)$ được thu nhỏ với tiêu chuẩn L2, nhưng nó làm quá mịn các chi tiết hình ảnh. Để giải quyết vấn đề này, các phương pháp dựa trên khuếch tán dị hướng đã được sử dụng để bảo toàn chi tiết hình ảnh, tuy nhiên, các cạnh vẫn bị mờ.

Trong khi đó, để giải quyết vấn đề "smoothness", `total variation regularization` đã được đề xuất. Đây là nghiên cứu có ảnh hưởng nhất trong lĩnh vực làm giảm hình ảnh. Việc điều chỉnh TV dựa trên thực tế thống kê rằng hình ảnh tự nhiên mịn cục bộ và cường độ điểm ảnh thay đổi dần dần ở hầu hết các vùng. Nó được định nghĩa như sau:

$$
R_{TV}(x) = ||∇x||_1
$$

Với $∇x$ là gradient của x khi đó ta có

$$
R_{NSS}(x) = \sum_{x_i∈x} ||x_i - NLM(x_i)||_2^2 =  \sum_{x_i∈x} ||x_i - w_i^Tκ_i||_2^2
$$

trong đó $κ_i$ và $$w_i biểu thị vectơ cột

Hiện tại, hầu hết các nghiên cứu về biến dạng hình ảnh đã chuyển từ các phương pháp cục bộ sang phương pháp không cục bộ. Ví dụ, các phần mở rộng của các phương pháp phi địa phương để chính quy hóa TV đã được đề xuất trong [Adaptive texturepreserving denoising method using gradient histogram and nonlocal selfsimilarity priors](https://ieeexplore.ieee.org/document/8515226) hay [ Nonlocal image denoising usingedge-based similarity metric and adaptive parameter selection](https://doi.org/10.1007/s11432-017-9207-9)

Trên cơ sở xem xét giá trị tương ứng của các phương pháp TV và NLM, một phương pháp điều chỉnh thích ứng của NLM (R-NL) đã được đề xuất để kết hợp NLM với chính quy TV. Kết quả cho thấy sự kết hợp của hai mô hình này đã thành công trong việc loại bỏ nhiễu. Tuy nhiên, thông tin cấu trúc không được bảo quản tốt bằng các phương pháp này, điều này làm giảm chất lượng hình ảnh trực quan. Hơn nữa, các phần mở rộng và cải tiến nổi bật hơn nữa của các phương pháp NSS dựa trên việc tìm hiểu khả năng xuất hiện của các bản vá hình ảnh và khai thác thuộc tính lowrank bằng cách sử dụng tối thiểu hóa định mức kernel có trọng số (WNNM).

# Tổng kết

Về tổng quan, bài viết này trình bày về khử nhiễu, vốn là tác vụ đóng vai trò khá quan trọng nhằm cải thiện chất lượng đầu vào trước khi xử lý ở các bước tiếp theo. Xuyên suốt bài viết, ta đã cùng tìm hiểu về các phương pháp khử nhiễu phổ biến thường được gọi là các phương pháp cổ điển và từ đó có cơ sở để so sánh với các phương pháp sử dụng học sâu ở bài tiếp theo (nếu mình có viết). Bài viết đến đây là kết thúc, cảm ơn mọi người đã dành thời gian đọc.
