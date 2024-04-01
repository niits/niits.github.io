---
date: '2020-10-30T09:23:09+07:00'
description: Tích hợp liên tục (CI - Continuous Integration) là phương pháp mà các team Agile sử dụng để đảm bảo code của toàn dự án luôn build được, luôn chạy đúng bằng cách tự động tiến hành kiểm thử truy hồi để phát hiện lỗi nhanh nhất có thể. Tích hợp CI vào dự án của mình là lựa chọn của hầu hết các nhóm phát triển phần mềm do hầu như tất cả các developer đều thấy rằng phương pháp tiếp cận này giúp giảm bớt vấn đề về tích hợp hơn và cho phép phát triển phần mềm gắn kết nhanh hơn. Bài viết này giới thiệu về `Lighthouse CI` - tập các cộng cụ được sử dụg trong việc chạy phân tích hiệu suất trên quy trình phát triển cho một ứng dụng web và cách tích hợp nó với CircleCI.
layout: post
title: "Ki\u1EC3m th\u1EED hi\u1EC7u n\u0103ng \u1EE9ng d\u1EE5ng web v\u1EDBi Lighthouse\
  \ CI"

---

Tích hợp liên tục (CI - Continuous Integration) là phương pháp mà các team Agile sử dụng để đảm bảo code của toàn dự án luôn build được, luôn chạy đúng bằng cách tự động tiến hành kiểm thử truy hồi để phát hiện lỗi nhanh nhất có thể. Tích hợp CI vào dự án của mình là lựa chọn của hầu hết các nhóm phát triển phần mềm do hầu như tất cả các developer đều thấy rằng phương pháp tiếp cận này giúp giảm bớt vấn đề về tích hợp hơn và cho phép phát triển phần mềm gắn kết nhanh hơn. Bài viết này giới thiệu về `Lighthouse CI` - tập các cộng cụ được sử dụg trong việc chạy phân tích hiệu suất trên quy trình phát triển cho một ứng dụng web và cách tích hợp nó với CircleCI.

# Lighthouse CI là gì

> Lighthouse CI is a suite of tools that make continuously running, saving, retrieving, and asserting against Lighthouse results as easy as possible.
> 
> <https://github.com/GoogleChrome/lighthouse-ci>

Từ phần giới thiệu ở mục Overview, chúng ta có thể hiểu rằng `Lighthouse CI` là một bộ công cụ giúp chạy liên tục, lưu, truy xuất và xác nhận kết quả `Lighthouse` dễ dàng nhất có thể. Đây là dự án mã nguồn mở được đội ngũ đến từ Google phát triển đã được release qua 42 phiên bản ( phiên bản hiện tại là 0.6.0)


![](images/213a055e-f677-4e6c-acd1-3bf82b99e55e.png)

Nói đến `Lighthouse` thì đây là công cụ mã nguồn mở tự động của Google dùng để phân tích, đo lường và cải thiện chất lượng website. Lighthouse sẽ giúp bạn kiểm tra toàn diện các tiêu chí cần thiết để web hoạt động hiệu quả, bao gồm: hiệu suất, khả năng truy cập, progressive web app, khả năng SEO,… và rất nhiều mặt khác nữa. Có thể nói Lighthouse là công cụ toàn diện nhất của Google hiện nay để giúp người dùng đánh giá trang web một cách chính xác nhất. Ngoài khả năng phân tích bao quát, công cụ này cũng đưa ra  các lời khuyên hữu ích để bạn nâng cao chất lượng website.

Để có thể hình dung được kết quả trả về của `Lighthouse CI` chúng ta có thể quan sát kết quả trả về khi sử dụng `Lighthouse` ở Chrome DevTools trước tiên. ~~Audits panel của Chrome DevTools hiện đang được cấp quyền áp dụng Google Lighthouse cho website.~~ Để tạo báo cáo, chúng ta cần thực hiện các bước sau trên trình duyệt Google Chrome.

- Truy cập website bạn muốn kiểm tra bằng trình duyệt Google Chrome.
- Nhấn F12 hoặc bấm tổ hợp phím Ctrl – Shift – I để mở Developer Tools. Sau đó chọn tab `Lighthouse`.
- Đánh dấu tick vào các mục bạn muốn phân tích ở mục Categories (nên chọn tất cả các mục để được báo cáo hoàn chỉnh) cũng như chọn loại Device.
- Click vào nút Generate report. Sau một khoảng thời gian ngắn, `Lighthouse` sẽ tự động hiển thị kết quả phân tích của trang.

Giả sử đối với trang với trang Followings của Viblo, sau khi chạy, `Lighthouse` trả về cho chúng ta kết quả như sau:

![](images/a331bbf4-ccb9-41d1-aff4-5976cd02ab56.png)

Có thể thấy rằng kết quả trả về của `Lighthouse` bao gồm cả điểm số lẫn thông tin chi tiết vậy nên có thể nói kết quả này khá đầy đủ và trực quan. Các mục mà `Lighthouse` kiểm tra bao gồm như sau:

- `Performance`: Điểm hiệu suất là trung bình có trọng số của các điểm số. Đương nhiên, các chỉ số có trọng số cao hơn có ảnh hưởng lớn hơn đến điểm Hiệu suất tổng thể của bạn. Điểm số liệu không hiển thị trong báo cáo nhưng được tính toán chi tiết. Các metrics đo lường cùng với trọng số của nó là:
  - First Contentful Paint - 15%: đo lường thời gian trình duyệt mất để hiển thị phần nội dung DOM đầu tiên sau khi người dùng điều hướng đến trang của bạn.
  - Speed Index - 15%: đo lường mức độ nhanh chóng mà nội dung được hiển thị trực quan trong quá trình tải trang. Đầu tiên, Lighthouse quay video tải trang trong trình duyệt và tính toán tiến trình hình ảnh giữa các khung hình. Sau đó, Lighthouse sử dụng mô-đun Speedline Node.js để tạo điểm Chỉ số tốc độ.
  - Largest Contentful Paint - 25%: báo cáo thời gian hiển thị của hình ảnh hoặc khối văn bản lớn nhất có thể nhìn thấy trong chế độ xem.
  - Time to Interactive - 15%: đo thời gian một trang trở nên hoàn toàn có thể tương tác.
  - Total Blocking Time - 25%: đo tổng thời gian mà một trang bị chặn phản hồi với thông tin nhập của người dùng, chẳng hạn như nhấp chuột, nhấn vào màn hình hoặc nhấn bàn phím
  - Cumulative Layout Shift - 5%: đo lường tổng của tất cả các điểm thay đổi bố cục riêng lẻ cho mọi thay đổi bố cục không mong muốn xảy ra trong toàn bộ thời gian tồn tại của trang.
- `Accessibility`: tính điểm điểm trung bình có trọng số của tất cả các lần kiểm tra khả năng tiếp cận theo các tiêu chí được liệt kê tại <https://web.dev/accessibility-scoring/>
- `Best Practices`: tính toán dựa trên cái tiêu chí có sẵn mà 'Lighthouse' khuyến khích sử dụng để có thể cải thiện tình trạng mã tổng thể của ứng dụng web <https://web.dev/lighthouse-best-practices/>
- `SEO`: tính toán điểm số dựa trên khả năng đảm bảo rằng trang của bạn được tối ưu hóa cho xếp hạng kết quả của công cụ tìm kiếm. Các tiêu chí của mục này được liệt kê ở <https://web.dev/lighthouse-seo/>
- `Progressive Web App`: Tính toán điểm số  bằng cách kiểm tra xác thực các khía cạnh của Progressive Web App. Tương tự các mục trên, các tiêu chí được cung cấp đầy đủ tại <https://web.dev/lighthouse-pwa/>

Trong suốt báo cáo,`Lighthouse` không chỉ nêu ra  vấn đề của website mà còn cung cấp một số gợi ý để ta có thể nâng cao chất lượng website chẳng hạn như sau:

![](images/c0889779-3f60-41f1-b056-549b1bedebb3.png)

# Tích hợp Lighthouse CI với CircleCi

Mặc dù được cung cấp báo cáo đầy đủ về website của mình nhưng việc phải thực hiện thủ công trên từng trang tốn rất nhiều thời gian và công sức cũng như khó có thể kết hợp với hệ thống CI mà dự án sử dụng. Để giải quyết vấn đề đó `Lighthouse CI` có thể được sử dụng để tự động kiểm tra và tạo báo cáo. Phần sau đây sẽ hướng dẫn tích hợp Lighthouse CI với CircleCi.

## Chuẩn bị project

Để tiết kiệm thời gian chúng ta sẽ sử dụng project có sẵn để tích hợp Lighthouse CI với Github Action. Truy cập fork repository <https://github.com/nuxt-company/demo-blog-nuxt-content> này và clone về cài các dependencies bằng `yarn` và chạy thử bằng `yarn dev` ta có thể xem thành quả bằng cách truy cập <http://localhost:3000/>

![](images/95c98c6e-2af2-461c-883a-3071bf11d805.png)

## Setup với CircleCI

Đầu tiên, ở thư mục gốc của project chúng ta chạy lệnh sau để thêm cli của Lighthouse
```bash
yarn add -D @lhci/cli
```

Tiếp đó thêm script vào `package.json`
```javascript:package.json
{
  "scripts": {
      ...
       "lhci": "lhci autorun",
      ...
  }
}
```

Tiếp theo, tạo tệp cấu hình cho `Lighthouse CI`. Tạo tệp có tên `.lighthouserc.json` ngay dưới thư mục gốc của dự án và viết nội dung sau.
```javascript:.lighthouserc.json
module.exports = {
  ci: {
    collect: {
      numberOfRuns: 1,
      startServerCommand: 'yarn start',
      url: [
        'http://localhost:3000/',
        'http://localhost:3000/blog/my-third-blog-post',
        'http://localhost:3000/blog/tag/web_development'
      ]
    },
    assert: {
      preset: 'lighthouse:no-pwa',
      assertions: {
        'offscreen-images': 'off',
        'uses-webp-images': 'off',
        'uses-http2': 'off'
      }
    },
    upload: {
      target: 'temporary-public-storage'
    }
  }
}
```

Trường url là nơi mà chúng ta nói cho `Lighthouse CI` cần kiểm tra các trang nào.

Tiếp đến là cài đặt Lighthouse CI App tại <https://github.com/apps/lighthouse-ci> cho dự án của bạn trên Github. Sau khi cài xong ta sẽ thu được một `LHCI_GITHUB_APP_TOKEN`. Token này sẽ được sử dụng để thêm vào `Environment Variables` bên CircleCI
![](images/6ed8a00b-9e16-473b-971b-bc75b227c399.png)

Bước tiếp theo là kết nối dự án của bạn với CircleCI. CircleCI yêu cầu config để có thể thực hiện quá trình CI vậy nên ta tạo file `config.yml` có nội dung như sau để sử dungj:

```yaml:.circleci/config.yml
version: 2.1

commands:
  restore_npm_cache:
    description: "Restore npm cache"
    steps:
      - restore_cache:
          keys:
            - v1-dependencies-{{ checksum "package.json" }}
            - v1-dependencies-
  save_npm_cache:
    description: "Save npm cache"
    steps:
      - save_cache:
          paths:
            - node_modules
          key: v1-dependencies-{{ checksum "package.json" }}

jobs:
  lighthouse:
    docker:
      - image: circleci/node:12.3-browsers
    working_directory: ~/repo
    steps:
      - checkout
      - restore_npm_cache
      - run: yarn install
      - save_npm_cache
      - run: yarn build
      - run:
          name: lhci autorun
          command: yarn lhci || echo "LHCI failed!"

workflows:
  version: 2
  build:
    jobs:
      - lighthouse
```

Bước cuối cùng là điền `LHCI_GITHUB_APP_TOKEN` vào `Environment Variables` cho project của bạn ở CircleCI bằng cách chọn `Projects Setting` > `Environment Variables`

![](images/2770671c-ee21-449b-aae2-9992abdd9eb0.png)

Voilà. Đến bước này, chúng ta đã tích hợp `Lighthouse CI` xong CircleCi. Tạo một commit bất kì push lên. Khi đó `Lighthouse CI` sẽ chạy và ta sẽ thu được kết quả như sau:

![](images/8a4cd41b-a95c-42f8-ba68-1fdc9ef3eaa4.png)

Sau khi chạy xong, `Lighthouse CI` gửi các báo cáo về Github Action và hiện thị danh sách ở mỗi commit.

![](images/b67f9795-3a95-4709-b591-1a6db7724729.png)

Chúng ta có thể xem chi tiết từng báo cáo bằng cách click vào `Detail` Có thể thấy rằng báo cáo được tạo ra có đầy đủ các phần giống như khi chúng ta thực hiện trên Google DevTools

![](images/c4dea3fc-51f5-49e4-8dc1-a70f8432cfbd.png)


# Tổng kết
Có thể thấy rằng `Lighthouse CI` là một công cụ hữu ích vì nó cho phép bạn kiểm tra một cách tự động hiệu năng cũng như một số tiêu chí khác trước khi merge để tìm bất kỳ mã nào có thể ảnh hưởng đến chất lượng trang web của bạn.  Lighthouse CI cũng có sẵn trong các CI khác, vì vậy nó có thể được sử dụng trong Travis CI và GitHub Actions.  Bài viết này giới thiệu về `Lighthouse CI` - tập các cộng cụ được sử dụg trong việc chạy phân tích hiệu suất trên quy trình phát triển cho một ứng dụng web và cách tích hợp nó với CircleCi. Bài viết đến đây là hết, cảm ơn mọi người đã giành thời gian đọc.

# Tài liệu tham khảo
- <https://github.com/GoogleChrome/lighthouse-ci>
- <https://wiki.matbao.net/google-lighthouse-la-gi-toan-tap-cach-su-dung-google-lighthouse/>
- <https://qiita.com/Tsuyoshi84/items/6a7b546a4ed1cdc11c81#%E6%9C%80%E5%BE%8C%E3%81%AB>