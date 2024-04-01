---
date: '2020-09-11T09:47:25+07:00'
description: "Trong b\xE0i vi\u1EBFt tr\u01B0\u1EDBc vi\u1EBFt v\u1EC1 t\u1ED5ng quan\
  \ v\u1EC1 Tin sinh h\u1ECDc, ch\xFAng ta \u0111\xE3 c\xF3 th\u1EC3 li\u1EC7t k\xEA\
  \ m\u1ED9t s\u1ED1 \u1EE9ng d\u1EE5ng c\u1EE7a c\xE1c ph\u01B0\u01A1ng ph\xE1p c\u1EE7\
  a khoa h\u1ECDc m\xE1y t\xEDnh khi \xE1p d\u1EE5ng cho vi\u1EC7c thu th\u1EADp,\
  \ l\u01B0u tr\u1EEF, t\u1ED5 ch\u1EE9c, ph\xE2n t\xED..."
layout: post
title: "T\xECm hi\u1EC3u v\u1EC1 DNA v\xE0 s\u1EED d\u1EE5ng c\xE1c ph\u01B0\u01A1\
  ng  ph\xE1p h\u1ECDc m\xE1y d\u1EF1 \u0111o\xE1n h\u1ECD gen"

categories:
  - Machine Learning
  - Bioinformatics
---

Trong bài viết trước viết về tổng quan về Tin sinh học, chúng ta đã có thể liệt kê một số ứng dụng của các phương pháp của khoa học máy tính khi áp dụng cho việc thu thập, lưu trữ, tổ chức, phân tích hoặc trực quan hóa dữ liệu Sinh học. Bài viết này sẽ nói về cách thể hiện, cấu trúc dữ liệu cũng như cách xử lý dữ liệu DNA. Cùng với đó, bài viết cũng cung cấp một ví dụ nhỏ về cách phân loại họ gen bằng phương pháp học máy đơn giản.

# DNA là gì

DNA (Deoxyribonucleic Acid) hay còn được gọi với tên ADN (acide désoxyribonucléique) là một phân tử chứa các chỉ dẫn sinh học tạo nên sự đặc trưng của từng loài. DNA, cùng với các thông tin di truyền mà nó chứa, được truyền từ các sinh vật trưởng thành sang con cái của chúng trong quá trình sinh sản.

Bộ gen người có khoảng từ 20.000-25.0000 gen, được tập hợp bên trong các nhiễm sắc thể (NST) nằm trong nhân tế bào. Bộ gen người bao gồm khoảng 3 tỷ phân tử cặp base (bazơ), các cặp DNA của người được cấu tạo nên từ 4 loại nucleotide bao gồm "A", "C", "G", "T".  Tất cả chúng ta đều có bộ gen riêng biệt tuy nhiên vẫn chia sẻ nhiều phần giống nhau.

Đối với dữ liệu DNA, các kĩ thuật học máy có thể được áp dụng ddeer:

- Nắm bắt quan hệ phụ thuộc trong dữ liệu
- Suy luận và phát hiện các giả thuyết sinh học mới

Vậy trong bài viết này, chúng ta sẽ tìm hiểu cách diễn giải cấu trúc DNA và cách các thuật toán học máy có thể được sử dụng để xây dựng mô hình dự đoán trên dữ liệu chuỗi DNA.

> Tham khảo: <https://medium.com/analytics-vidhya/demystify-dna-sequencing-with-machine-learning-and-python-bdbaeb177f56>

## Cách các DNA được thể hiện

Phần lớn các phân tử DNA được cấu tạo từ hai mạch polyme sinh học xoắn đều quanh một trục tưởng tượng tạo thành chuỗi xoắn kép.

![](images/333eea06-e76c-49c3-8daf-32764f48b659.jpg)
> Hình minh họa từ trang [https://biologydictionary.net/double-helix/](https://biologydictionary.net/double-helix/)

Hai mạch DNA này được gọi là các polynucleotide vì thành phần của chúng bao gồm các đơn phân nucleotide. Mỗi nucleotide được cấu tạo từ một trong bốn loại nucleobase chứa nitơ—hoặc là cytosine (C), guanine (G), adenine (A), hay thymine (T)—liên kết với đường deoxyribose và một nhóm phosphat. Các nucleotide liên kết với nhau thành một mạch DNA bằng liên kết cộng hóa trị giữa phân tử đường của nucleotide với nhóm phosphat của nucleotide tiếp theo, tạo thành "khung xương sống" đường-phosphat luân phiên vững chắc.

Thứ tự hoặc trình tự của các nucleobase xác định những chỉ dẫn sinh học nào được chứa trong một chuỗi DNA. Ví dụ như một vùng cụ thể trên nhiễm sắc thể thứ 15 có vai trò quan trọng trong việc quy định màu mắt.

## Cách Python xử lý các dữ liệu DNA

Chúng ta đều biết rằng, Python có rất nhiều thư viện hỗ trợ cho việc xử lý cũng như trực quan hóa dữ liệu mà một phần trong số đó là dữ liệu sinh học. Hai trong số các thư viện có thể kể đến là `Biopython` và `squiggle`

- **Biopython** là một tập hợp các module python cung cấp các chức năng để xử lý các hoạt động của chuỗi DNA, RNA và protein như bổ sung ngược lại chuỗi DNA, tìm các mô-đun trong chuỗi protein, v.v. Nó cung cấp rất nhiều trình phân tích cú pháp để đọc tất cả các cơ sở dữ liệu di truyền chính như GenBank , SwissPort, FASTA, v.v.,

- **Squiggle** là một công cụ phần mềm tự động tạo ra các biểu diễn đồ họa hai chiều trên nền web của chuỗi DNA thô. Squiggle có cài đặt một số thuật toán trực quan hóa trình tự được công bố trước đó và giới thiệu các phương pháp trực quan hóa mới được thiết kế để tối đa hóa khả năng sử dụng của người dùng.

Để cài đặt hai thư viện trên, bắt đầu với môi trường Jupyter Notebook chúng ta sử dụng các câu lệnh sau:

```python
!pip install biopython
!pip install Squiggle
```

Dữ liệu về gen thường được lưu ở một số định dạng và một trong số đó là FASTA. Khi mở một tệp FASTA, chúng ta có thể thấy nội dung sẽ tương tự như sau:

```
>HSBGPG Human gene for bone gla protein (BGP)
GGCAGATTCCCCCTAGACCCGCCCGCACCATGGTCAGGCATGCCCCTCCTCATCGCTGGGCACAGCCCAGAGGGT
ATAAACAGTGCTGGAGGCTGGCGGGGCAGGCCAGCTGAGTCCTGAGCAGCAGCCCAGCGCAGCCACCGAGACACC
ATGAGAGCCCTCACACTCCTCGCCCTATTGGCCCTGGCCGCACTTTGCATCGCTGGCCAGGCAGGTGAGTGCCCC
CACCTCCCCTCAGGCCGCATTGCAGTGGGGGCTGAGAGGAGGAAGCACCATGGCCCACCTCTTCTCACCCCTTTG
......
```

> Để xem toàn bộ nội dung tệp ví dụ trên mọi người có thể xem tại [http://www.cbs.dtu.dk/services/NetGene2/fasta.php](http://www.cbs.dtu.dk/services/NetGene2/fasta.php). Toàn bộ nội dung của tệp FASTA là phần in đậm ở trang web trên. Sao chép phần đó và lưu lại với định dạng .fa như file text thông thường chẳng hạn như example.fa, chúng có thể sử dụng file đó làm dữ liệu cho ví dụ với `Squiggle`.

Để sử dụng tốt matplotlib với Jupyter Notebook chúng ta sử dụng câu lệnh magic muôn thủa

```python
%matplotlib inline
```

Tiếp đó, với thư viện Biopython chúng ta có thể đọc được thông tin của file dữ liệu bằng các câu lệnh sau:

```python
from Bio import SeqIO

for sequence in SeqIO.parse('./example.fa', 'fasta'):
    print(sequence.id)
    print(sequence.seq)
    print(len(sequence))
```

Kết quả thu được như sau:

```bash
HSBGPG
GGCAGATTCCCCCTAGACCCGCCCGCACCATGGTCAGGCATGCCCCTCCTCATCGCTGGGCACAGCCCAGAGGGTATAAACAGTGCTGGAGGCTGGCGGGGCAGGCCAGCTGAGTCCTGAGCAGCAGCCCAGCGCAGCCACCGAGACACCATGAGAGCCCTCACACTCCTCGCCCTATTGGCCCTGGCCGCACTTTGCATCGCTGGCCAGGCAGGTGAGTGCCCCCACCTCCCCTCAGGCCGCATTGCAGTGGGGGCTGAGAGGAGGAAGCACCATGGCCCACCTCTTCTCACCCCTTTGGCTGGCAGTCCCTTTGCAGTCTAACCACCTTGTTGCAGGCTCAATCCATTTGCCCCAGCTCTGCCCTTGCAGAGGGAGAGGAGGGAAGAGCAAGCTGCCCGAGACGCAGGGGAAGGAGGATGAGGGCCCTGGGGATGAGCTGGGGTGAACCAGGCTCCCTTTCCTTTGCAGGTGCGAAGCCCAGCGGTGCAGAGTCCAGCAAAGGTGCAGGTATGAGGATGGACCTGATGGGTTCCTGGACCCTCCCCTCTCACCCTGGTCCCTCAGTCTCATTCCCCCACTCCTGCCACCTCCTGTCTGGCCATCAGGAAGGCCAGCCTGCTCCCCACCTGATCCTCCCAAACCCAGAGCCACCTGATGCCTGCCCCTCTGCTCCACAGCCTTTGTGTCCAAGCAGGAGGGCAGCGAGGTAGTGAAGAGACCCAGGCGCTACCTGTATCAATGGCTGGGGTGAGAGAAAAGGCAGAGCTGGGCCAAGGCCCTGCCTCTCCGGGATGGTCTGTGGGGGAGCTGCAGCAGGGAGTGGCCTCTCTGGGTTGTGGTGGGGGTACAGGCAGCCTGCCCTGGTGGGCACCCTGGAGCCCCATGTGTAGGGAGAGGAGGGATGGGCATTTTGCACGGGGGCTGATGCCACCACGTCGGGTGTCTCAGAGCCCCAGTCCCCTACCCGGATCCCCTGGAGCCCAGGAGGGAGGTGTGTGAGCTCAATCCGGACTGTGACGAGTTGGCTGACCACATCGGCTTTCAGGAGGCCTATCGGCGCTTCTACGGCCCGGTCTAGGGTGTCGCTCTGCTGGCCTGGCCGGCAACCCCAGTTCTGCTCCTCTCCAGGCACCCTTCTTTCCTCTTCCCCTTGCCCTTGCCCTGACCTCCCAGCCCTATGGATGTGGGGTCCCCATCATCCCAGCTGCTCCCAAATAAACTCCAGAAG
1231
HSGLTH1
CCACTGCACTCACCGCACCCGGCCAATTTTTGTGTTTTTAGTAGAGACTAAATACCATATAGTGAACACCTAAGACGGGGGGCCTTGGATCCAGGGCGATTCAGAGGGCCCCGGTCGGAGCTGTCGGAGATTGAGCGCGCGCGGTCCCGGGATCTCCGACGAGGCCCTGGACCCCCGGGCGGCGAAGCTGCGGCGCGGCGCCCCCTGGAGGCCGCGGGACCCCTGGCCGGTCCGCGCAGGCGCAGCGGGGTCGCAGGGCGCGGCGGGTTCCAGCGCGGGGATGGCGCTGTCCGCGGAGGACCGGGCGCTGGTGCGCGCCCTGTGGAAGAAGCTGGGCAGCAACGTCGGCGTCTACACGACAGAGGCCCTGGAAAGGTGCGGCAGGCTGGGCGCCCCCGCCCCCAGGGGCCCTCCCTCCCCAAGCCCCCCGGACGCGCCTCACCCACGTTCCTCTCGCAGGACCTTCCTGGCTTTCCCCGCCACGAAGACCTACTTCTCCCACCTGGACCTGAGCCCCGGCTCCTCACAAGTCAGAGCCCACGGCCAGAAGGTGGCGGACGCGCTGAGCCTCGCCGTGGAGCGCCTGGACGACCTACCCCACGCGCTGTCCGCGCTGAGCCACCTGCACGCGTGCCAGCTGCGAGTGGACCCGGCCAGCTTCCAGGTGAGCGGCTGCCGTGCTGGGCCCCTGTCCCCGGGAGGGCCCCGGCGGGGTGGGTGCGGGGGGCGTGCGGGGCGGGTGCAGGCGAGTGAGCCTTGAGCGCTCGCCGCAGCTCCTGGGCCACTGCCTGCTGGTAACCCTCGCCCGGCACTACCCCGGAGACTTCAGCCCCGCGCTGCAGGCGTCGCTGGACAAGTTCCTGAGCCACGTTATCTCGGCGCTGGTTTCCGAGTACCGCTGAACTGTGGGTGGGTGGCCGCGGGATCCCCAGGCGACCTTCCCCGTGTTTGAGTAAAGCCTCTCCCAGGAGCAGCCTTCTTGCCGTGCTCTCTCGAGGTCAGGACGCGAGAGGAAGGCGC
1020
```

Tất nhiên bên cạnh việc chỉ in ra như trên chúng ta có thể sử dụng thư viện `Squiggle` đã nhắc đến ở trên để trực quan hóa dữ liệu.
Sử dụng câu lệnh đưới đây, `Squiggle` sẽ mở một trang web trực quan hóa dữ liệu mà trên đó chúng ta có thể di chuyển, phóng to, và tương tác với dữ liệu.

```python
!squiggle ./example.fa
```

![](images/5ce3f92a-b63f-47be-8849-1863b3f3badc.png)

> Ngoài ra còn nhiều tùy chọn của `Squiggle` mà chúng ta có thể tìm hiểu thêm tại trang [document](https://squiggle.readthedocs.io/en/latest/https://squiggle.readthedocs.io/en/latest/) của thư viện này.

# Dự đoán họ gen dựa trên các phương pháp học máy

Họ gen là một tập hợp của một số gen giống nhau, được hình thành bằng cách nhân đôi của một gen ban đầu, nhìn chung có chức năng sinh hóa tương tự. Các gen được phân loại thành các họ dựa trên trình tự nucleotide hoặc protein được chia sẻ.

Biết được trình tự của protein được mã hóa bởi một gen có thể cho phép các nhà nghiên cứu áp dụng các phương pháp tìm kiếm sự tương đồng giữa các trình tự protein cung cấp nhiều thông tin hơn là sự tương đồng hoặc khác biệt giữa các trình tự DNA.

> Nội dụng trên tham khảo ở [7.15A: Gene Families](https://bio.libretexts.org/Bookshelves/Microbiology/Book%3A_Microbiology_(Boundless)/7%3A_Microbial_Genetics/7.15%3A_Genome_Evolution/7.15A%3A_Gene_Families)
>
## Chuẩn bị dữ liệu

Trong ví dụ này, chúng ta sẽ sử dụng dữ liệu về DNA của người, tinh tinh và loài chó, ví dụ này được thực hiện dựa trên repo của tác giả [Nagesh Singh Chauhan](https://github.com/nageshsinghc4) vậy nên dữ liệu được sử dụn của ba loài trên cũng có sẵn trong repo [DNA-Sequence-Machine-learning](https://github.com/nageshsinghc4/DNA-Sequence-Machine-learning).
Dữ liệu bao gồm các nhóm gen được gán nhãn như sau:

| Gene Family                 | Class label |
|-----------------------------|-------------|
| G protein coupled receptors | 0           |
| Tyrosine kinase             | 1           |
| Tyrosine phosphatase        | 2           |
| Synthetase                  | 3           |
| Synthase                    | 4           |
| Ion channel                 | 5           |
| Transcription factor        | 6           |

Sau khi tải về các dữ liệu trên, đầu tiên như thường lệ, chúng ta import các thư viện cần sử dungj:

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt

from functools import partial
```

Tiếp theo đó bằng thư viện `pandas` chúng ta đọc dữ liệu đã tải về như sau:

```python
humandf = pd.read_table('data/human_data.txt')
chimpdf = pd.read_table('data/chimp_data.txt')
dogdf = pd.read_table('data/dog_data.txt')
```

## Chuyển đổi dữ liệu chuỗi DNA thành ma trận đặc trưng bằng k-mer

Kĩ thuật `k-mer counting` có thể hiểu là chuỗi thành các "word" có độ dài k trùng lấp nhau. Trong ví dụ này, ta sẽ sử dụng k = 6. Hàm sau được sử dụng để áp dụng kĩ thuật k-mer:

```python
def Kmers_funct(seq, size=4):
    return [seq[x:x+size].lower() for x in range(len(seq) - size + 1)]
 ```

 Tiếp đến ta có hàm `sentencize` để tạo các "câu" k-mer

 ```python
 def sentencize(df):
    '''Generate a df into kmers sentences and target vectors'''
    seq_list = df['sequence'].tolist()
    y_gt = df['class'].tolist()

    # apply Kmer-counting to seq-list
    kmers_list = map(partial(Kmers_funct, size=6), seq_list) #k-mer words of length 6
    sentences = [' '.join(seq) for seq in kmers_list]
    return sentences, y_gt
 ```

 Từ đó ta thu được `sents_human` và `y_human` qua câu lệnh sau:

 ```python
 sents_human, y_human = sentencize(humandf)
 ```

 Tiếp tục vectorize bằng cách sử dụng BOW cũng như chia tập train và test:

 ```python
 tf = CountVectorizer(ngram_range=(4,4)) # 4-gram
x_human = tf.fit_transform(sents_human)

from sklearn.model_selection import train_test_split

x_train, x_test, y_train, y_test = train_test_split(x_human, y_human, test_size=0.2, random_state=42)
 ```

## Phân lớp

### Phân lớp bằng Multinomial Naive Bayes

 Khởi tạo mô hình và huấn luyện bằng câu lệnh sau:

 ```python
 from sklearn.naive_bayes import MultinomialNB

#create a model
clf_bayes = MultinomialNB(alpha=0.1)

#train on data
clf_bayes.fit(x_train, y_train)
 ```

 Sau đó sử dụng mô hình vừa huấn luyện để đoán trên tập test, ta thu được `y_pred`

 ```python
 y_pred = clf_bayes.predict(x_test)
 ```

 Từ kết quả đó, chúng ta có thể đánh giá bằng các phương pháp phổ biến hiện nay như sau:

```python
#look at model performance: confusion matrix, accuracy, precision, recall, f1 score

from sklearn.metrics import accuracy_score, f1_score, precision_score, recall_score

print("Confusion matrix for predictions on human test DNA sequence\n")
print(pd.crosstab(pd.Series(y_test, name='Actual'), pd.Series(y_pred, name='Predicted')))

def calculate_metrics(y_test, y_predicted):
    accuracy = accuracy_score(y_test, y_predicted)
    precision = precision_score(y_test, y_predicted, average='weighted')
    recall = recall_score(y_test, y_predicted, average='weighted')
    f1 = f1_score(y_test, y_predicted, average='weighted')
    print("accuracy = %.3f \nprecision = %.3f \nrecall = %.3f \nf1 = %.3f" % (accuracy, precision, recall, f1))

calculate_metrics(y_test, y_pred)
```

Từ đó ta thu được kết quả như sau

```bash
Confusion matrix for predictions on human test DNA sequence

Predicted   0    1   2    3    4   5    6
Actual
0          99    0   0    0    1   0    2
1           0  104   0    0    0   0    2
2           0    0  78    0    0   0    0
3           0    0   0  124    0   0    1
4           1    0   0    0  143   0    5
5           0    0   0    0    0  51    0
6           1    0   0    1    0   0  263
accuracy = 0.984
precision = 0.984
recall = 0.984
f1 = 0.984
```

### Máy vector tựa

 Đoạn mã sau được sử dụng để huấn luyện cũng như đánh giá kết quả của mô hình

 ```python
 from sklearn import svm

clf_svm = svm.SVC(C=100, gamma=0.001, kernel='rbf')

#train
clf_svm.fit(x_train, y_train)

#predict and eval
calculate_metrics(y_test, clf_svm.predict(x_test))
 ```

Kết quả thu được như sau:

```bash
accuracy = 0.893
precision = 0.921
recall = 0.893
f1 = 0.896
```

### Dự đoán từ dữ liệu trình tự DNA của các loài khác

Tương tự với khi xử lý dữ liệu gen người, chúng ta tiến hành trên dữ liệu DNA của các loài khác như sau:

```python
sents_chimp, y_chimp = sentencize(chimpdf)
sents_dog, y_dog = sentencize(dogdf)

x_chimp = tf.transform(sents_chimp)
x_dog = tf.transform(sents_dog)
```

Tiếp theo đó định nghĩa hàm `evaluate_classifier` như sau:

```python
def evaluate_classifier(model):
    #predict
    y_pred_chimp = model.predict(x_chimp)
    y_pred_dog = model.predict(x_dog)
    #eval
    print("Performance on Chimpanzee test DNA")
    calculate_metrics(y_chimp, y_pred_chimp)

    print("----\nPerformance on Dog test DNA")
    calculate_metrics(y_dog, y_pred_dog)
```

Từ đó sử dụng hàm `evaluate_classifier` với hai mô hình được huấn luyện với dữ liệu DNA của người chúng ta thu được:

- Sử dụng mô hình Multinomial Naive Bayes
  - Dự đoán trên dữ liệu DNA của tinh tinh
    - accuracy = 0.993
    - precision = 0.994
    - recall = 0.993
    - f1 = 0.993
  - Dự đoán trên dữ liệu DNA của loài chó
    - accuracy = 0.926
    - precision = 0.934
    - recall = 0.926
    - f1 = 0.925
- Sử dụng mô hình máy vector tựa:
  - Dự đoán trên dữ liệu DNA của tinh tinh
    - accuracy = 0.968
    - precision = 0.971
    - recall = 0.968
    - f1 = 0.969
  - Dự đoán trên dữ liệu DNA của loài chó
    - accuracy = 0.493
    - precision = 0.803
    - recall = 0.493
    - f1 = 0.436

Từ kết quả trên có thể thấy rằng mô hình SVM chỉ rõ ra rằng loài người và tinh tinh có quan hệ gần hơn rất nhiều so với loài chó khi mà kết quả đáng giá khi dùng mô hình đoán nhận với dữ liệu DNA loài chó thấp hơn rất nhiều khi dùng đoán nhận DNA của tinh tinh.

# Kết luận

Tin sinh học - ngành nghiên cứu tích hợp dữ liệu sinh học thông lượng cao và mô hình thống kê thông qua tính toán chuyên sâu, đã thu hút được sự quan tâm lớn trong thời gian gần đây và giải trình tự DNA là một trong những vấn đề cốt lõi của nó. Lượng lớn thông tin thu được từ việc giải trình tự đã giúp chúng ta hiểu sâu hơn và kiến thức cơ bản về sinh vật. Bài viết trên trình bày sơ qua những hiểu biết cơ bản về DNA, giới thiệu một số thư viện được sử dụng để trực quan hóa cũng như xử lý dữ liệu và cuối cùng là một ví dụ nhỏ ứng dụng  các phương pháp của khoa học máy tính mà cụ thể hơn là các phương pháp học máy để xử lý và trích rút một số hiểu biết về dữ liệu chuỗi DNA của một số loài.
