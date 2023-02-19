---
title: Problem 1 - Bội số của 3 và 5
date: 2020-12-30T23:23:47+07:00

categories:
- Project Euler

excerpt: Nếu chúng ta liệt kê tất cả các số tự nhiên dưới 10 là bội số của 3 hoặc 5, chúng ta sẽ nhận được 3, 5, 6 và 9. Tổng của các bội số này là 23. Đề bài yêu cầu tìm tổng của tất cả các bội số của 3 hoặc 5 dưới 1000.

layout: post
---

[Problem 1 - Multiples of 3 and 5](https://projecteuler.net/problem=1 "Problem 1 - Multiples of 3 and 5")

## Đề bài

Nếu chúng ta liệt kê tất cả các số tự nhiên dưới 10 là bội số của 3 hoặc 5, chúng ta sẽ nhận được 3, 5, 6 và 9. Tổng của các bội số này là 23.

Tìm tổng của tất cả các bội số của 3 hoặc 5 dưới 1000.

## Hướng làm

Có thể thấy rằng có một cách đơn giản rằng ta có thể duyệt tất cả các số từ 1 đến 1000 kiểm tra chúng là bội của 3 và 5 không, nếu có thì cộng chúng lại với nhau.

```python
result = 0
for i in range (1000):
    if i % 3 == 0 or i % 5 == 0:
        result = result + i
```

Tuy nhiên bài này trong Project Euler nên việc làm hùng hục như trâu húc mả vậy là không chấp nhận được.

Để ý một chút ta có thể thấy tổng các số nhỏ hơn 1000 và là bội số của 3 là:

$$
3 + 6 + 9 + 12 + ... + 999 = 3 * (1 + 2 + 3+ ... + 333)
$$

Tương tự với 5 là:

$$
5 + 10 + 15 + ... +  995 = 5 * (1 + 2 + 3+ ... + 199)
$$

Mà ta lại biết rằng:

$$
1 + 2 + 3 + 4 + ... + N = N * ( N + 1 ) / 2
$$

Vậy nên ta có cách có vẻ có não hơn như sau:

```python
def sum_divisble_by(n, p):
    return n * (p / n) * ((p / n) + 1) /2

result = sum_divisble_by(3, 999) + sum_divisble_by(5, 999) - sum_divisble_by(15, 999)
```

Có xuất hiện việc trừ đi ```python sum_divisble_by(15, 999)``` là do các bội số của 15 sẽ được tính 2 lần.
