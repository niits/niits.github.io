---
date: '2021-02-19T10:00:04+07:00'
description: Thông thường không phải lúc nào cột đơ cũng đẻ ra code tối ưu. Lường trước được điều đó, các trình biên dịch mã luôn có bước tối ưu mã trung gian nhằm loại bỏ những phần mã không cần thiết nhằm tối ưu khả năng sử dụng tài nguyên mà vẫn đảm bảo tính đúng đắn của chương trình. Bài viết này trình bày cơ bản về quá trình này cũng như cung cấp một số ví dụ về cách mã LLVM IR được tối ưu bằng các phương pháp chẳng hạn như constant folding, inlining functions, loop unrolling, ...

layout: post
title: "Minh h\u1ECDa qu\xE1 tr\xECnh t\u1ED1i \u01B0u m\xE3 trung gian v\u1EDBi LLVM"

categories:
  - Kiến thức cơ bản

tags:
    - LLVM
    - Compiler
    - Optimization
---

Thông thường không phải lúc nào cột đơ cũng đẻ ra code tối ưu. Lường trước được điều đó, các trình biên dịch mã luôn có bước tối ưu mã trung gian nhằm loại bỏ những phần mã không cần thiết nhằm tối ưu khả năng sử dụng tài nguyên mà vẫn đảm bảo tính đúng đắn của chương trình. Bài viết này trình bày cơ bản về quá trình này cũng như cung cấp một số ví dụ về cách mã LLVM IR được tối ưu bằng các phương pháp chẳng hạn như constant folding, inlining functions, loop unrolling, ...

# Tối ưu mã trung gian
Về cơ bản thì tối ưu mã trung gian là một kỹ thuật chuyển đổi chương trình, cố gắng cải thiện mã trung gian bằng cách làm cho nó tiêu thụ ít tài nguyên hơn (tức là CPU, Bộ nhớ) để mã máy chạy nhanh hơn.

![](images/ab5c5aa5-60a8-437d-8b5d-a75646b1cb38.png)

> Hình ảnh từ <https://www.guru99.com/compiler-design-phases-of-compiler.html>

Sau khi mã nguồn được phân tích qua các bước ở phía trước và được sử dụng để tạo mã trung gian ở bước Intermediate Code Generator như trong biểu đồ trên, bước tối ưu tiếp theo sẽ nhận đầu vào là các đoạn mã độc lập với nền tảng nhưng có cấu trúc tương tự như mã máy như Three-address code hoặc LLVM IR, ... được gọi chung là intermediate code hay mã trung gian nhằm tối ưu phần mã này.

Một quá trình tối ưu mã tốt cần có các đặc điểm sau:

- Việc tối ưu hóa phải chính xác, nó không được thay đổi ý nghĩa của chương trình theo bất kỳ cách nào.
- Tối ưu hóa sẽ làm tăng tốc độ và hiệu suất của chương trình.
- Quá trình tối ưu hóa không được làm ảnh hưởng xấu đến quá trình biên dịch tổng thể chẳng hạn như tốn quá nhiều thời gian và tài nguyên chỉ để tối ưu một đoạn mã mà không cải thiện quá nhiều quá trình sử dụng tài nguyên khi thực thi.

# Một số kĩ thuật được sử dụng để tối ưu mã trung gian
Các kĩ thuật tối ưu có thể được phân loại thành hai loại:

- Các phương pháp tối ưu độc lập với thiết bị: Giai đoạn tối ưu hóa mã này cố gắng cải thiện mã trung gian để có được mã đích tốt hơn làm đầu ra. Phần mã trung gian được chuyển đổi ở đây không liên quan đến bất kỳ thanh ghi CPU hoặc vị trí bộ nhớ tuyệt đối nào.

- Các phương pháp tối ưu dựa trên đặc điểm của thiết bị: Tối ưu hóa phụ thuộc vào máy được thực hiện sau khi mã đích đã được tạo
 và khi mã được chuyển đổi theo kiến trúc máy đích. Qúa trình tối ưu này liên quan đến các thanh ghi CPU và có thể có tham chiếu bộ nhớ tuyệt đối hơn là tham chiếu tương đối. Các trình tối ưu hóa phụ thuộc vào đặc trưng của thiết bị nỗ lực để tận dụng tối đa hệ thống phân cấp bộ nhớ.

Trong bài viết này một số phương pháp không phụ thuộc vào đặc trưng của thiết bị sẽ được minh họa chi tiết như sau:

## Constant folding
Gấp liên tục (constant folding hay constant propagation) là những tối ưu hóa trình biên dịch có liên quan được nhiều trình biên dịch hiện đại sử dụng. Đây  là quá trình nhận biết và tính toán các biểu thức không đổi tại thời điểm biên dịch thay vì tính toán chúng tại runtime. 

Để hình dung rõ hơn ta xét ví dụ sau:

![](images/37d535a5-d405-41a9-a1c1-c1ffc7c34203.png)

Trong trường hợp này biểu thức `28 / x + 2` có giá trị không đổi nên ta có thể tạo tối ưu mã trung gian được tạo ra từ đoạn mã và tối ưu chúng bằng cách sử dụng:

```bash
clang -S -emit-llvm constant_folding.c -o - | sed s/optnone// | opt -S -mem2reg  -constprop
```

Khi đó kết quả thu được có phần mã chính như sau:
![](images/173f7fe3-267e-4d9c-98f3-accd8d57e09d.png)

Có thể quan sát như hình trên, biết tương ứng cho biến `x` ở của mã nguồn không được tạo và biểu thức  `28 / x + 2` được thay thế bằng giá trị bằng 4



## Dead code elimination
Tiếp theo đó loại bỏ mã chết (dead code elimination) là một phương pháp tối ưu hóa để loại bỏ mã thường được gọi là mã chết, bao gồm mã không bao giờ có thể được thực thi (mã không thể truy cập) và mã chỉ ảnh hưởng đến các biến chết (được ghi vào, nhưng không bao giờ được đọc lại), tức là, không liên quan đến chương trình.

Việc loại bỏ các phần mã không ảnh hưởng đến kết quả của chương trình giúp kích thước chương trình được thu nhỏ lại và nó cho phép chương trình tránh được việc thực hiện các công việc không liên quan từ đó giảm thời gian thực thi và tiết kiểm được tài nguyên hệ thống.

![](images/87daa965-b0ce-4698-9c24-ea67d9285c39.png)

Xét ví dụ sau ta có thể thấy rằng, biến `b` không được sử dụng sau khi gán cũng như các dòng code sau khi trả về giá trị của `c` không được thực thi. Chúng chính là các phần mã chết như đề cập trên và sẽ được loại bỏ trong quá trình tối ưu. Loại bỏ các đoạn mã này bằng cách sử dụng câu lệnh sau:

```bash
clang -S -emit-llvm dead_code_elimination.c -o - | sed s/optnone// | opt -S  -o opted.ll -dce
```

Ta thu được mã LLVM IR mới như sau:

![](images/eb204bc4-c250-40d8-bf38-316499d76d06.png)

Có thể thấy đoạn mã IR trên chỉ tính toán giá trị của c và trả về. Các phần mã chết ở trên đã được loại bỏ.

## Canonicalize induction variables
Chuẩn hóa biến cảm ứng (canonicalize induction variables) là phương pháp phân tích và biến đổi các  induction variables (và các phép tính bắt nguồn từ chúng) thành các dạng đơn giản hơn phù hợp cho các phép phân tích và biến đổi tiếp theo. Xét ví dụ sau ta có:

![](images/a66abe6b-2529-4c1b-8927-5a38d80c122d.png)

Có thể thấy rằng điều kiện `i*i < 4` có thể thay thế bằng điều kiên đơn giản hơn là `i < 2` là đó là điều phương pháp chuẩn hóa biến cảm ứng thực hiện khi ta sử dụng câu lệnh ở phần trên trên với flag  `-indvars`. Kết quả thu được như sau:

![](images/dcc09562-c732-44dd-acb3-67157872cbef.png)

Các bạn không đọc được cái thứ LLVM IR nhìn chả khác gì hợp ngữ kia? Tôi cũng thế. Tuy nhiên nhìn vào đoạn mã kia ta cũng đoán được đoạn gán `%exitcond` được dùng để định nghĩa điều kiện dừng của cái gì đó. Cùng với việc `icmp` là predicate cho một phép so sánh số nguyên và `ne` được sử dụng để kiểm 2 hai giá trị không bằng nhau chúng ta có thể đoán được đây chính là điều kiện `i < 2` được sử dụng làm điều kiện dừng của vòng lặp. Vậy quả thực phép so sánh `i*i < 4` có thể thay thế bằng một phép đơn giản hơn để tối ưu cho quá trình thực thi sau này.

# Tổng kết

Bài viết này trình bày cơ bản về quá trình tối ưu mã trung gian cũng như cung cấp một số ví dụ về một vài phương pháp đơn giản được sử dụng phục vụ cho quá trình này. Có thể thấy rằng, ba phương pháp được nêu trên chỉ là các phương pháp đơn giản và việc áp dụng riêng lẻ chỉ thu được các tối ưu cục bộ và khó có thể cải thiện một cách đáng kể cho hiệu năng của toàn bộ một chương trình. Bởi vậy các phương pháp này cùng vô vàn các phương pháp khác thường được sử dụng kết hợp với nhau chẳng hạn như sử dụng copy propagation trước khi khử mã chết, ... và lặp lại nhiều lần cho đến khi thu được mã tối ưu toàn cục. Bài viết đến đây là hết cảm ơn các bạn đã dành thời gian đọc.