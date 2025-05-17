## Viết Thông Báo Lỗi vào Standard Error Thay vì Standard Output

Hiện tại, chúng ta đang viết tất cả đầu ra của mình vào terminal bằng cách sử
dụng macro `println!`. Trong hầu hết các terminal, có hai loại đầu ra: _standard
output_ (`stdout`) cho thông tin chung và _standard error_ (`stderr`) cho thông
báo lỗi. Sự phân biệt này cho phép người dùng lựa chọn chuyển hướng đầu ra thành
công của chương trình vào một file nhưng vẫn in thông báo lỗi ra màn hình.

Macro `println!` chỉ có khả năng in vào standard output, vì vậy chúng ta phải sử
dụng một cách khác để in vào standard error.

### Kiểm tra Nơi Lỗi Được Ghi

Đầu tiên, hãy quan sát cách nội dung được in bởi `minigrep` hiện đang được ghi
vào standard output, bao gồm cả các thông báo lỗi mà chúng ta muốn viết vào
standard error thay thế. Chúng ta sẽ làm điều đó bằng cách chuyển hướng luồng
standard output vào một file trong khi chủ ý gây ra lỗi. Chúng ta sẽ không
chuyển hướng luồng standard error, vì vậy bất kỳ nội dung nào được gửi đến
standard error sẽ tiếp tục hiển thị trên màn hình.

Các chương trình dòng lệnh được mong đợi sẽ gửi thông báo lỗi đến standard error
stream để chúng ta vẫn có thể thấy thông báo lỗi trên màn hình ngay cả khi chúng
ta chuyển hướng standard output stream vào một tệp. Chương trình của chúng ta
hiện không hoạt động tốt: chúng ta sắp thấy rằng nó lưu đầu ra thông báo lỗi vào
một file thay vì màn hình!

Để chứng minh hành vi này, chúng ta sẽ chạy chương trình với `>` và đường dẫn
file, _output.txt_, nơi chúng ta muốn chuyển hướng luồng standard output đến.
Chúng ta sẽ không truyền bất kỳ đối số nào, điều đó sẽ gây ra lỗi:

```console
$ cargo run > output.txt
```

Cú pháp `>` nói với shell rằng hãy ghi nội dung của standard output vào
_output.txt_ thay vì màn hình. Chúng ta không thấy thông báo lỗi mà chúng ta đã
mong đợi in trên màn hình, vì vậy điều đó có nghĩa là nó hẳn đã xuất hiện trong
file. Đây là nội dung của _output.txt_:

```text
Problem parsing arguments: not enough arguments
```

Đúng vậy, thông báo lỗi của chúng ta đang được in vào standard output. Sẽ hữu
ích hơn nhiều nếu các thông báo lỗi như thế này được in ra standard error để chỉ
dữ liệu từ một lần chạy thành công mới xuất hiện trong file. Chúng ta sẽ thay
đổi điều đó.

### In Lỗi vào Standard Error

Chúng ta sẽ sử dụng mã trong Listing 12-24 để thay đổi cách thông báo lỗi được
in. Vì việc cấu trúc lại mã mà chúng ta đã làm trước đó trong chương này, tất cả
mã in thông báo lỗi nằm trong một hàm, `main`. Thư viện chuẩn cung cấp macro
`eprintln!` để in vào standard error stream, vì vậy hãy thay đổi hai nơi chúng
ta đã gọi `println!` để in lỗi bằng cách sử dụng `eprintln!` thay thế.

<Listing number="12-24" file-name="src/main.rs" caption="Viết thông báo lỗi vào standard error thay vì standard output bằng cách sử dụng `eprintln!`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-24/src/main.rs:here}}
```

</Listing>

Bây giờ hãy chạy chương trình một lần nữa theo cùng một cách, không có đối số
nào và chuyển hướng standard output với `>`:

```console
$ cargo run > output.txt
Problem parsing arguments: not enough arguments
```

Bây giờ chúng ta thấy lỗi trên màn hình và _output.txt_ không chứa gì cả, đó là
hành vi mà chúng ta mong đợi từ các chương trình dòng lệnh.

Hãy chạy chương trình một lần nữa với các đối số không gây ra lỗi nhưng vẫn
chuyển hướng standard output vào một file, như sau:

```console
$ cargo run -- to poem.txt > output.txt
```

Chúng ta sẽ không thấy bất kỳ đầu ra nào trên terminal, và _output.txt_ sẽ chứa
các kết quả của chúng ta:

<span class="filename">Filename: output.txt</span>

```text
Are you nobody, too?
How dreary to be somebody!
```

Điều này chứng minh rằng chúng ta hiện đang sử dụng standard output cho đầu ra
thành công và standard error cho đầu ra lỗi một cách thích hợp.

## Tóm tắt

Chương này đã nhắc lại một số khái niệm chính mà bạn đã học cho đến nay và đã đề
cập đến cách thực hiện các hoạt động I/O thông thường trong Rust. Bằng cách sử
dụng đối số dòng lệnh, files, biến môi trường, và macro `eprintln!` để in lỗi,
bạn đã sẵn sàng để viết các ứng dụng dòng lệnh. Kết hợp với các khái niệm trong
các chương trước, mã của bạn sẽ được tổ chức tốt, lưu trữ dữ liệu hiệu quả trong
các cấu trúc dữ liệu phù hợp, xử lý lỗi một cách tốt và được kiểm tra kỹ lưỡng.

Tiếp theo, chúng ta sẽ khám phá một số tính năng của Rust bị ảnh hưởng bởi các
ngôn ngữ lập trình hàm: closures và iterators.
