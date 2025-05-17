## Packages và Crates

Phần đầu tiên của hệ thống module mà chúng ta sẽ đề cập đến là packages và
crates.

Một _crate_ là đơn vị code nhỏ nhất mà trình biên dịch Rust xem xét tại một thời
điểm. Ngay cả khi bạn chạy `rustc` thay vì `cargo` và truyền vào một tệp mã
nguồn duy nhất (như chúng ta đã làm trong "Viết và Chạy Chương Trình Rust" ở
Chương 1), trình biên dịch xem tệp đó là một crate. Crates có thể chứa modules,
và các modules có thể được định nghĩa trong các tệp khác được biên dịch cùng với
crate, như chúng ta sẽ thấy trong các phần tiếp theo.

Một crate có thể có một trong hai hình thức: binary crate hoặc library crate.
_Binary crates_ là các chương trình bạn có thể biên dịch thành một tệp thực thi
để chạy, như một chương trình dòng lệnh hoặc một máy chủ. Mỗi crate này phải có
một hàm gọi là `main` để xác định điều gì sẽ xảy ra khi chương trình thực thi
chạy. Tất cả các crate chúng ta đã tạo cho đến nay đều là binary crates.

_Library crates_ không có hàm `main`, và chúng không được biên dịch thành tệp
thực thi. Thay vào đó, chúng định nghĩa các chức năng được thiết kế để chia sẻ
giữa nhiều dự án. Ví dụ, crate `rand` mà chúng ta đã sử dụng trong [Chương
2][rand]<!-- ignore --> cung cấp chức năng tạo số ngẫu nhiên. Hầu hết thời gian
khi các lập trình viên Rust nói “crate,“ họ thường muốn nói đến library crate,
và họ sử dụng "crate" thay thế cho khái niệm lập trình chung là “library.“

_Crate root_ là một tệp mã nguồn mà trình biên dịch Rust bắt đầu từ đó và tạo
thành module gốc của crate của bạn (chúng ta sẽ giải thích kỹ về modules trong
["Định nghĩa Modules để Kiểm soát Phạm vi và Quyền riêng
tư"][modules]<!-- ignore -->).

Một _package_ là một gói gồm một hoặc nhiều crates cung cấp một tập hợp chức
năng. Một package chứa một tệp _Cargo.toml_ mô tả cách xây dựng các crates đó.
Cargo thực chất là một package chứa binary crate cho công cụ dòng lệnh mà bạn đã
sử dụng để xây dựng mã của mình. Package Cargo cũng chứa một library crate mà
binary crate phụ thuộc vào. Các dự án khác có thể phụ thuộc vào library crate
Cargo để sử dụng logic tương tự như công cụ dòng lệnh Cargo sử dụng.

Một package có thể chứa nhiều binary crates tùy thích, nhưng nhiều nhất chỉ có
một library crate. Một package phải chứa ít nhất một crate, dù đó là library hay
binary crate.

Hãy đi qua những gì xảy ra khi chúng ta tạo một package. Đầu tiên, chúng ta nhập
lệnh `cargo new my-project`:

```console
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```

Sau khi chúng ta chạy `cargo new my-project`, chúng ta sử dụng `ls` để xem những
gì Cargo tạo ra. Trong thư mục dự án, có một tệp _Cargo.toml_, tạo ra một
package. Có một thư mục _src_ chứa _main.rs_. Mở _Cargo.toml_ trong trình soạn
thảo của bạn và lưu ý rằng không có đề cập đến _src/main.rs_. Cargo tuân theo
quy ước rằng _src/main.rs_ là crate root của một binary crate có cùng tên với
package. Tương tự, Cargo biết rằng nếu thư mục package chứa _src/lib.rs_, thì
package chứa một library crate có cùng tên với package, và _src/lib.rs_ là crate
root. Cargo truyền các tệp crate root cho `rustc` để xây dựng thư viện hoặc
chương trình.

Ở đây, chúng ta có một package chỉ chứa _src/main.rs_, nghĩa là nó chỉ chứa một
binary crate có tên là `my-project`. Nếu một package chứa _src/main.rs_ và
_src/lib.rs_, nó có hai crates: một binary và một library, cả hai đều có cùng
tên với package. Một package có thể có nhiều binary crates bằng cách đặt các tệp
trong thư mục _src/bin_: mỗi tệp sẽ là một binary crate riêng biệt.

[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
