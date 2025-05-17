## Cài đặt

Bước đầu tiên là cài đặt Rust. Chúng ta sẽ tải Rust thông qua `rustup`, một công
cụ dòng lệnh để quản lý các phiên bản Rust và các công cụ liên quan. Bạn sẽ cần
kết nối internet để tải xuống.

> Lưu ý: Nếu bạn không muốn sử dụng `rustup` vì một số lý do, vui lòng xem
> [trang Phương pháp Cài đặt Rust khác][otherinstall] để biết thêm lựa chọn.

Các bước sau đây cài đặt phiên bản ổn định mới nhất của trình biên dịch Rust. Sự
đảm bảo về tính ổn định của Rust đảm bảo rằng tất cả các ví dụ trong sách này
biên dịch được sẽ tiếp tục biên dịch được với các phiên bản Rust mới hơn. Đầu ra
có thể khác nhau một chút giữa các phiên bản vì Rust thường cải thiện thông báo
lỗi và cảnh báo. Nói cách khác, bất kỳ phiên bản Rust ổn định, mới hơn nào mà
bạn cài đặt bằng các bước này sẽ hoạt động như mong đợi với nội dung của cuốn
sách này.

> ### Ký hiệu Dòng lệnh
>
> Trong chương này và xuyên suốt cuốn sách, chúng tôi sẽ hiển thị một số lệnh
> được sử dụng trong terminal. Các dòng mà bạn nên nhập vào terminal đều bắt đầu
> bằng `$`. Bạn không cần phải gõ ký tự `$`; đó là dấu nhắc dòng lệnh được hiển
> thị để chỉ ra điểm bắt đầu của mỗi lệnh. Các dòng không bắt đầu bằng `$`
> thường hiển thị đầu ra của lệnh trước đó. Ngoài ra, các ví dụ dành riêng cho
> PowerShell sẽ sử dụng `>` thay vì `$`.

### Cài đặt `rustup` trên Linux hoặc macOS

Nếu bạn đang sử dụng Linux hoặc macOS, mở terminal và nhập lệnh sau:

```console
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

Lệnh này tải xuống một script và bắt đầu cài đặt công cụ `rustup`, công cụ này
sẽ cài đặt phiên bản ổn định mới nhất của Rust. Bạn có thể được yêu cầu nhập mật
khẩu. Nếu cài đặt thành công, dòng sau sẽ xuất hiện:

```text
Rust is installed now. Great!
```

Bạn cũng sẽ cần một _linker_, là một chương trình mà Rust sử dụng để kết hợp các
đầu ra đã biên dịch thành một tệp. Có khả năng bạn đã có sẵn một cái. Nếu bạn
gặp lỗi linker, bạn nên cài đặt một trình biên dịch C, thường sẽ bao gồm một
linker. Một trình biên dịch C cũng hữu ích vì một số gói Rust phổ biến phụ thuộc
vào mã C và sẽ cần một trình biên dịch C.

Trên macOS, bạn có thể có một trình biên dịch C bằng cách chạy:

```console
$ xcode-select --install
```

Người dùng Linux nên cài đặt GCC hoặc Clang, theo tài liệu của bản phân phối của
họ. Ví dụ, nếu bạn sử dụng Ubuntu, bạn có thể cài đặt gói `build-essential`.

### Cài đặt `rustup` trên Windows

Trên Windows, truy cập
[https://www.rust-lang.org/tools/install][install]<!-- ignore --> và làm theo
hướng dẫn để cài đặt Rust. Tại một số điểm trong quá trình cài đặt, bạn sẽ được
yêu cầu cài đặt Visual Studio. Điều này cung cấp một linker và các thư viện gốc
cần thiết để biên dịch chương trình. Nếu bạn cần thêm trợ giúp với bước này, xem
[https://rust-lang.github.io/rustup/installation/windows-msvc.html][msvc]<!-- ignore -->

Phần còn lại của cuốn sách này sử dụng các lệnh hoạt động trong cả _cmd.exe_ và
PowerShell. Nếu có sự khác biệt cụ thể, chúng tôi sẽ giải thích nên sử dụng cái
nào.

### Khắc phục sự cố

Để kiểm tra xem bạn đã cài đặt Rust đúng cách chưa, mở shell và nhập dòng này:

```console
$ rustc --version
```

Bạn sẽ thấy số phiên bản, commit hash và ngày commit cho phiên bản ổn định mới
nhất đã được phát hành, theo định dạng sau:

```text
rustc x.y.z (abcabcabc yyyy-mm-dd)
```

Nếu bạn thấy thông tin này, bạn đã cài đặt Rust thành công! Nếu bạn không thấy
thông tin này, kiểm tra xem Rust có trong biến hệ thống `%PATH%` của bạn như
sau.

Trong Windows CMD, sử dụng:

```console
> echo %PATH%
```

Trong PowerShell, sử dụng:

```powershell
> echo $env:Path
```

Trong Linux và macOS, sử dụng:

```console
$ echo $PATH
```

Nếu tất cả đều đúng và Rust vẫn không hoạt động, có một số nơi bạn có thể nhận
trợ giúp. Tìm hiểu cách liên hệ với các Rustaceans khác (một biệt danh ngớ ngẩn
mà chúng tôi tự gọi mình) trên [trang cộng đồng][community].

### Cập nhật và gỡ cài đặt

Khi Rust đã được cài đặt qua `rustup`, việc cập nhật lên phiên bản mới phát hành
rất dễ dàng. Từ shell của bạn, chạy script cập nhật sau:

```console
$ rustup update
```

Để gỡ cài đặt Rust và `rustup`, chạy script gỡ cài đặt sau từ shell của bạn:

```console
$ rustup self uninstall
```

<!-- Old headings. Do not remove or links may break. -->
<a id="local-documentation"></a>

### Đọc tài liệu cục bộ

Việc cài đặt Rust cũng bao gồm một bản sao cục bộ của tài liệu để bạn có thể đọc
nó khi không có kết nối internet. Chạy `rustup doc` để mở tài liệu cục bộ trong
trình duyệt của bạn.

Bất cứ khi nào một kiểu dữ liệu hoặc hàm được cung cấp bởi thư viện tiêu chuẩn
và bạn không chắc chắn nó làm gì hoặc cách sử dụng nó, hãy sử dụng tài liệu giao
diện lập trình ứng dụng (API) để tìm hiểu!

<!-- Old headings. Do not remove or links may break. -->
<a id="text-editors-and-integrated-development-environments"></a>

### Sử dụng Trình soạn thảo văn bản và IDEs

Cuốn sách này không đưa ra giả định về công cụ bạn sử dụng để viết mã Rust. Hầu
hết các trình soạn thảo văn bản đều có thể hoàn thành công việc! Tuy nhiên,
nhiều trình soạn thảo văn bản và môi trường phát triển tích hợp (IDEs) có hỗ trợ
tích hợp cho Rust. Bạn luôn có thể tìm thấy danh sách khá cập nhật của nhiều
trình soạn thảo và IDE trên [trang công cụ][tools] trên trang web Rust.

### Làm việc ngoại tuyến với cuốn sách này

Trong một số ví dụ, chúng tôi sẽ sử dụng các gói Rust ngoài thư viện tiêu chuẩn.
Để làm việc qua các ví dụ đó, bạn sẽ cần có kết nối internet hoặc đã tải xuống
các phụ thuộc đó trước. Để tải xuống các phụ thuộc trước, bạn có thể chạy các
lệnh sau. (Chúng tôi sẽ giải thích `cargo` là gì và từng lệnh này làm gì chi
tiết sau.)

```console
$ cargo new get-dependencies
$ cd get-dependencies
$ cargo add rand@0.8.5 trpl@0.2.0
```

Điều này sẽ lưu trữ các gói đã tải xuống để bạn không cần phải tải xuống chúng
sau này. Sau khi bạn đã chạy lệnh này, bạn không cần giữ thư mục
`get-dependencies`. Nếu bạn đã chạy lệnh này, bạn có thể sử dụng cờ `--offline`
với tất cả các lệnh `cargo` trong phần còn lại của cuốn sách để sử dụng các
phiên bản đã lưu trữ này thay vì cố gắng sử dụng mạng.

[otherinstall]:
  https://forge.rust-lang.org/infra/other-installation-methods.html
[install]: https://www.rust-lang.org/tools/install
[msvc]: https://rust-lang.github.io/rustup/installation/windows-msvc.html
[community]: https://www.rust-lang.org/community
[tools]: https://www.rust-lang.org/tools
