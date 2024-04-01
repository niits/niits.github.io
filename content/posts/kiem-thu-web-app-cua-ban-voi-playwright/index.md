---
date: '2020-10-14T13:14:40+07:00'
description: Như chúng ta đã biết, các headles browsers có khả năng thực thi mã JavaScript nhanh và mô phỏng việc nhấp hoặc di chuột vào các phần tử trên trang trên các thiết bị khác nhau. Các headles browsers cũng cực kỳ hữu ích khi kiểm tra hoạt động mạng, bắt chước các hành vi của người dùng khi sử dụng và tạo các requests. Chúng thường nhanh hơn các trình duyệt thực vì ta không cần khởi động GUI của trình duyệt, vậy nên ta có thể bỏ qua thời gian trình duyệt thực cần để tải CSS và JavaScript và hiển thị HTML. Bởi vậy, chúng thường được sử dụng trong quá trình kiểm thử tự đông nhằm đảm bảo mọi thứ hoạt động như dự định trước khi mã nguồn được triển khai lên môi trường production. Có nhiều giải pháp để viết các ca kiểm thử sử dụng headless browser và trong bài viết này, chúng ta sẽ cùng nhau tìm hiều về thư viện Playwright.

layout: post
title: "Ki\u1EC3m th\u1EED Web App c\u1EE7a b\u1EA1n v\u1EDBi Playwright"

categories:
  - Một vài tip nhỏ

tags:
  - Playwright
  - Automation Testing
  - Headless Browser
  - Node.js
---

Như chúng ta đã biết, các headless browsers có khả năng thực thi mã JavaScript nhanh và mô phỏng việc nhấp hoặc di chuột vào các phần tử trên trang trên các thiết bị khác nhau. Các headles browsers cũng cực kỳ hữu ích khi kiểm tra hoạt động mạng, bắt chước các hành vi của người dùng khi sử dụng và tạo các requests. Chúng thường nhanh hơn các trình duyệt thực vì ta không cần khởi động GUI của trình duyệt, vậy nên ta có thể bỏ qua thời gian trình duyệt thực cần để tải CSS và JavaScript và hiển thị HTML. Bởi vậy, chúng thường được sử dụng trong quá trình kiểm thử tự đông nhằm đảm bảo mọi thứ hoạt động như dự định trước khi mã nguồn được triển khai lên môi trường production. Có nhiều giải pháp để viết các ca kiểm thử sử dụng headless browser và trong bài viết này, chúng ta sẽ cùng nhau tìm hiều về thư viện Playwright.

# Playwright là gì?

>> Playwright is a Node.js library to automate [Chromium](https://www.chromium.org/Home), [Firefox](https://www.mozilla.org/en-US/firefox/new/) and [WebKit](https://webkit.org/) with a single API. Playwright is built to enable cross-browser web automation that is **ever-green**, **capable**, **reliable** and **fast**.

Dựa vào phần giới thiệu `Playwright`, ta có thể hiểu qua rằng nó là một thư viện Node.js để tự động hóa Chromium, Firefox và WebKit bằng một API duy nhất. Thư viện `Playwright` được xây dựng để cho phép tự động hóa web trên nhiều trình duyệt, luôn được cập nhật các phiên bản mới nhất, ổn định, hiệu quả, đáng tin cậy và nhanh chóng. `Playwright` được viết bởi đội ngũ lập trình viên đến từ Microsoft, là dữ án mã nguồn mở, mã nguồn của nó được cung cấp trên Github tại [https://github.com/microsoft/playwright](https://github.com/microsoft/playwright)
![](images/361b0e83-9a79-4dbe-bf22-4a81352696e9.png)
`Playwright` được xây dựng để tự động hóa hầu hết các tính năng trình duyệt web, những thứ ngày càng phổ biến rộng rãi được sử dụng bởi Single Page Apps và Progressive Web Apps. Với `Playwright` cung cấp cho chúng ta nhiều tiện ích như sau:
- Thực hiện các kịch bản kiểm thử kéo dài bao gồm nhiều trang, tên miền và iframe
- Tự động đợi các phần tử sẵn sàng trước khi thực hiện các hành động (như nhấp chuột, điền form, ...)
- Chặn hoạt động mạng đối với các yêu cầu mạng bắt chước và giả mạo
- Mô phỏng thiết bị di động, vị trí địa lý, và sự quản lý quyền
- Hỗ trợ các thành phần web thông qua shadow-piercing selectors
- Sự kiện đầu vào native cho chuột và bàn phím
- Tải lên và tải xuống tệp

# Bắt đầu làm quen với Playwright
Giả sử bạn đã cài đặt Node bởi vì `Playwright` là một thư viện của Node mà :v (Nếu không, không sao, hãy truy cập trang web chính thức của Node.js và tải xuống và cài đặt nó.)\

Tiếp theo, hãy tạo một thư mục cho project của bạn và khởi tạo một project Node mới bên trong thư mục bằng câu lệnh sau:
```bash
mkdir playright-example && cd playright-example && npm init -y
```
Sau khi thiết lập xong project, chúng ta có thể bắt đầu viết các headless tests. Để làm được điều đó, chúng ta cần cài đặt thư viện `Playwright` và tạo một tệp có tên là `index.js` nơi chúng tôi viết mã cho quá trình kiểm thử.
```bash
touch index.js && npm i --save playwright
```
Câu lệnh trên cài đặt Playwright và các tệp nhị phân cho Chromium, Firefox và WebKit. Sau khi cài đặt, bạn có thể import thư viện `Playwright`.

# Viết các headless tests
Để đơn giản, đối với ca kiểm thử đầu tiên, chúng ta sẽ truy cập whatsmyuseragent.org và chụp lại ảnh màn hình của trang. Nếu các bạn chưa biết thì whatsmyuseragent.org là một trang web hiển thị phiên bản tác nhân người dùng và dữ liệu, cũng như địa chỉ IP của bạn.
![](images/b5281ce0-fe00-4053-b6da-14d76db584ab.png)
Để làm được điều đó, chúng ta cần import các engine của các trình duyệt từ playwright chẳng hạn như `firefox`. Sau đó bằng cách gọi phương thức `launch` để tạo một đối tượng `browser` sau đó tạo một `page`, thứ mà có thể hiểu tương tự như một tab khi chúng ta duyệt web bằng cách sử dụng phương thức `newPage` cuối cùng là dùng `goto` và `screenshot` để đi đến trang whatsmyuseragent.org và chụp màn hình. Mã nguồn cụ thể cho file `index.js` có thể tham khảo như sau:

```javascript
const {
  webkit,
  firefox,
  chromium
} = require('playwright');

const simulate = async (engine) => {

  const browser = await engine.launch();
  const page = await browser.newPage();

  await page.goto('http://whatsmyuseragent.org/');
  await page.screenshot({ path: `example-${engine._initializer.name}.png` });

  await browser.close();
};

simulate(firefox)
simulate(webkit)
simulate(chromium)
```
Sau khi thực thi đoạn mã trong file `index.js`, chúng ta thu được 3 file ảnh mới được tạo có tên lần lượt là `example-chromium.png`, `example-firefox.png`, `example-webkit.png` Ba file ảnh này chính là ảnh chụp màn hình được tạo khi phương thức `screenshot` được gọi.
| example-chromium  | example-firefox.png | example-webkit.png |
| -------- | -------- | -------- |
| ![](images/97e1dfcb-4239-4195-b731-ba060cbb36e5.png)| ![](images/929fc0fa-401c-4858-a026-9b291d0f6a43.png)|  ![](images/21dbb058-dbe0-4991-93e3-29f39a98b3c4.png)|

# Sử dụng Playwright với Mocha

Một điều quan trọng cần lưu ý về Playwright, giống như nhiều công cụ kiểm tra tự động khác, nó được thiết kế để tự động hóa tương tác với trình duyệt, nhưng bạn phải sử dụng công cụ xác nhận để tự động hóa quá trình kiểm thử. Và trong bài viết này chúng ta sẽ dùng thử thư viện Mocha cùng với Playwright. Giới thiệu qua thì Mocha là một trình chạy test Javascript / NodeJS phổ biến, rất tốt cho thử nghiệm E2E. Thư viện này hoạt động rất tốt khi kết hợp với Playwright. Bạn có thể tạo các ca kiểm thử, sử dụng các công cụ xác nhận và tạo các báo cáo kiểm thử.

Bây giờ ta có thể tạo một kịch bản kiểm thử cơ bản kiểm thử trang web <http://todomvc.com/examples/react/#/> Thông thường, ta sẽ tạo các tệp này trong một thư mục test, với tên tệp kết thúc bằng .spec.js:
```javascript
const playwright = require('playwright')
const chai = require('chai')
const expect = chai.expect
const BASE_URL = 'http://todomvc.com/examples/react/#/'

let page, browser, context

describe('TO DO APP TESTS - PLAYWRIGHT', () => {

    beforeEach(async () => {
        browser = await playwright['chromium'].launch({ headless: false })
        context = await browser.newContext()
        page = await context.newPage(BASE_URL)
    })

    afterEach(async function() {
        await page.screenshot({ path: `${this.currentTest.title.replace(/\s+/g, '_')}.png` })
        await browser.close()
    })

    it('List is loaded empty', async() => {
        const sel = 'ul.todo-list li'
        const list = await page.$$(sel)
        expect(list.length).to.equal(0)
    })

    it('Adds a new todo in empty list', async() => {
        await page.waitForSelector('input')
        const element = await page.$('input')
        await element.type('Practice microsoft playwright')
        await element.press('Enter')

        // check list of ToDo
        const sel = 'ul.todo-list li'
        await page.waitForSelector(sel)
        const list = await page.$$(sel)
        expect(list.length).to.equal(1)
        expect(await page.$eval(sel, node => node.innerText)).to.be.equal('Practice microsoft playwright')
    })
})
```
Ví dụ này sẽ sử dụng Playwright để kết nối với trình duyệt Chrome. Nó sẽ mở  <http://todomvc.com/examples/react/#/> và thực hiện hai ca kiểm thử được định nghĩa ở trên. Để chạy thử chúng ta dùng lệnh sau:

```bash
 mocha # hoặc node_modules/mocha/bin/mocha
```
Kết quả thu được như hình sau, test case thứ có lỗi có thể do bị timeout. Mocha hỗ trợ chúng ta thay đổi giá trị mặc định bằng cách thêm `--timeout` và giá trị thời gian khi gọi mocha để chạy test.

# Sử dụng với GitHub Action
Chúng ta đều biết tự ra đời của Test automation và Continuous Integration/Continuous Development(CI-CD) là giải pháp được tạo ra để đảm bảo quá phát triển phần mềm hoạt động một cách liên tục, mượt mà trong khi vẫn đảm bảo chất lượng sản phẩm là một yêu cầu cấp thiết mà GitHub Action là một trong số đó. Để có thể sử dụng `Playwright` cùng với hệ thống này, chúng ta cần thực hiện một số bước như sau:

- Thêm use: `microsoft/playwright-github-action@v1` vào định nghĩa GitHub workflow của GitHub trước khi chạy test.
```yaml
on:
  push:
    branches:
    - master

jobs:
  e2e-tests:
    runs-on: ubuntu-latest # or macos-latest, windows-latest

    steps:
      - uses: actions/checkout@v2

      - uses: actions/setup-node@v1

      - uses: microsoft/playwright-github-action@v1

      - name: Install dependencies and run tests
        run: npm install && npm test
 ```
- Sau đó kết hợp action này với `Upload Artifact action` để upload test artifacts (như screenshots hoặc logs).

```yaml
steps:
- uses: microsoft/playwright-github-action@v1

- name: Install dependencies and run tests
  run: npm install && npm test

- uses: actions/upload-artifact@v2
  with:
    name: test-artifacts
    path: path/to/artifacts
```
# Tổng kết 
Mỗi công cụ trình duyệt đều có các quy tắc riêng để hiển thị HTML và CSS trên màn hình. Vậy nên các thư viện kiểm thử trên trình duyệt tự động có thể vô cùng hữu ích khi chúng ta muốn đảm bảo bố cục ứng dụng hoạt động đúng như những gì ta mong muốn trên tất cả các thiết bị và trình duyệt khác nhau mà `playwright` là một trong số đó.

Bài viết này giới thiệu sơ qua về thư viện `playwright` và cách thư viện này hoạt động, Để có thể tìm hiểu rõ hơn về thư viện này, cũng như tìm hiểu về điểm khác nhau giữa nó và `puppeteer` cũng như cách nó hoạt động với các hệ thống CI/CD, mọi người có thể đọc thêm tại đường dẫn bên dưới. Bài viết đến đây là hết, cảm ơn mọi người đã giành thời gian đọc.
# Tài liệu tham khảo
- https://github.com/microsoft/playwright
- https://testguild.com/what-is-playwright/