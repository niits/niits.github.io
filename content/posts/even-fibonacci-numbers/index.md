---
title: Problem 2 - Các số Finobacci chẵn
date: 2020-12-30T23:23:47+07:00
categories:
- Project Euler

excerpt: Mỗi số trong dãy Fibonacci được tạo ra bằng cách cộng hai số liền trước với nhau. Ví dụ như bắt đầu từ 1 và 2, 10 số đầu tiên sẽ là 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, ... Hãy tính tổng các số trong dãy Fibonacci có giá trị chẵn và không vượt quá 4 triệu.

layout: post
---

[Problem 2 - Even Fibonacci numbers](https://projecteuler.net/problem=2 "Problem 2 - Even Fibonacci numbers")

## Đề bài

Mỗi số trong dãy Fibonacci được tạo ra bằng cách cộng hai số liền trước với nhau. Ví dụ như bắt đầu từ 1 và 2, 10 số đầu tiên sẽ là:

1, 2, 3, 5, 8, 13, 21, 34, 55, 89, ...

Hãy tính tổng các số trong dãy Fibonacci có giá trị chẵn và không vượt quá 4 triệu.

## Hướng làm

Tương tự như Problem 1 nếu bạn quá rảnh thì có thể ngồi tính từng số, kiểm tra số mới sinh ra có chẵn hay không rồi cộng vào.

Tuy nhiên chạy đến số 4 triệu chắc chẳng ai rảnh cho lắm nên ta có thể xem xét dãy và thấy rằng các số cần liệt kê là:

2, 8, 34, 144, ....

Các số này dường như có công thức chung là $$F_n = 4 * F{n - 3} + F_{n - 6}$$

Thế nên loại bỏ các số lẻ xen kẽ ta sẽ thu được dãy số mới theo quy luật $$F_n = 4 * F_{n - 3} + F_{n - 6}$$

Tất nhiên là nó sẽ nhanh hơn 1 tý nhưng mọi người vẫn phải chạy 1 cái vòng lặp lâu ơi là lâu nên ta xét tiếp đến cái gọi là [Binet’s formula](https://en.wikipedia.org/wiki/Fibonacci_number#Relation_to_the_golden_ratio).

Khi đó số thứ n trong dãy Fibonacci sẽ được biểu diễn là:

$$F_{n} = {\frac {\varphi ^{n}-\psi ^{n}}{\varphi -\psi }}={\frac {\varphi ^{n}-\psi ^{n}}{\sqrt {5}}}$$

Với

$$\displaystyle \varphi = {\frac {1 + {\sqrt {5}}}{2}}\approx 1.61803 \ldots $$

Được gọi là [Golden ratio](https://en.wikipedia.org/wiki/Golden_ratio) (Tỉ lệ vàng)

Và

$${\displaystyle \psi = {\frac {1-{\sqrt {5}}}{2}}=1-\varphi =-{1 \over \varphi }\approx -0.61803C39887\ldots .}$$

Cái này trên wiki không ghi gì chắc không có tên nên thôi kệ.

Tại vì ở trên ta có thể thấy là các số cần cộng vào đều có chỉ số là bội của 3 thế nên ta có cái công thức sau để tính:

$$\sum_{i=1}^{k}(\frac{\phi^{3i} -\psi^{3i}}{\sqrt{5}}) =\frac{1}{\sqrt{5}}({\sum_{i=1}{k}({\phi^3}^i)} - {\sum_{i=1}{k}({\psi^3}^i)}) =\frac{1}{\sqrt{5}}(\phi^3\frac{1-{\phi^3}^k}{1-\phi^3} - \psi^3\frac{1-{\psi^3}^k}{1-\psi^3})$$

Cái dấu bằng thứ 3 í là áp dụng công thức khai triển $$x^n - 1$$ chắc ai cũng còn nhớ.

Vậy nên với $$F_n$$ cho trước ta có thể tìm số thứ tự cuối n bằng công thức [Fibonacci nghịch đảo](https://en.wikipedia.org/wiki/Fibonacci_number#Recognizing_Fibonacci_numbers).

$$\log_{\phi}(\frac{F_{n}\sqrt{5} + \sqrt{5F_{n}^{2}-4}}{2})$$

Vậy nên sau đây là phần code bằng python:

```python

from math import sqrt, floor, log

phi = (1 + sqrt(5)) / 2
psi = (1 - sqrt(5)) / 2

def reverse_fib(fn):
    return floor(log((fn * sqrt(5) + sqrt(5 * (fn ** 2) - 4)) / 2, phi))

def sum_even(k):
    phi3 = phi ** 3
    psi3 = psi ** 3
    return int((1 / sqrt(5)) * (
        phi3 * ((1 - phi3 ** k) / (1 - phi3)) -
        psi3 * ((1 - psi3 ** k) / (1 - psi3))
    ))

for i in range(4 * (10 ** 6)):
    N = int(input().strip())
    k = reverse_fib(n) // 3
    print(sum_even(k))
```

**Code ăn cắp từ bài viết [Project Euler #2: Even Fibonacci numbers](https://medium.com/@TheZaki/project-euler-2-even-fibonacci-numbers-2219e9438970) của tác giả Oussama Zaki**
