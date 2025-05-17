## Phụ lục D: Công cụ Phát triển Hữu ích

Trong phụ lục này, chúng ta nói về một số công cụ phát triển hữu ích mà dự án
Rust cung cấp. Chúng ta sẽ xem xét định dạng tự động, cách nhanh chóng để áp
dụng các sửa lỗi cảnh báo, công cụ kiểm tra và tích hợp với các IDE.

### Định dạng Tự động với `rustfmt`

Công cụ `rustfmt` định dạng lại mã của bạn theo phong cách mã của cộng đồng.
Nhiều dự án hợp tác sử dụng `rustfmt` để ngăn chặn các tranh cãi về phong cách
nào để sử dụng khi viết Rust: Mọi người đều định dạng mã của họ bằng công cụ
này.

Cài đặt Rust bao gồm `rustfmt` theo mặc định, vì vậy bạn nên đã có sẵn các
chương trình `rustfmt` và `cargo-fmt` trên hệ thống của mình. Hai lệnh này tương
tự như `rustc` và `cargo` ở chỗ `rustfmt` cho phép kiểm soát chi tiết hơn và
`cargo-fmt` hiểu các quy ước của dự án sử dụng Cargo. Để định dạng bất kỳ dự án
Cargo nào, hãy nhập như sau:

```console
$ cargo fmt
```

Chạy lệnh này sẽ định dạng lại tất cả mã Rust trong crate hiện tại. Điều này chỉ
nên thay đổi phong cách mã, không phải ngữ nghĩa mã. Để biết thêm thông tin về
`rustfmt`, hãy xem [tài liệu của nó][rustfmt].

[rustfmt]: https://github.com/rust-lang/rustfmt

### Sửa Mã của Bạn với `rustfix`

Công cụ `rustfix` được bao gồm trong cài đặt Rust và có thể tự động sửa các cảnh
báo của trình biên dịch có cách rõ ràng để sửa vấn đề mà có thể là điều bạn
muốn. Bạn có thể đã thấy các cảnh báo của trình biên dịch trước đây. Ví dụ, hãy
xem xét mã này:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
fn main() {
    let mut x = 42;
    println!("{x}");
}
```

Ở đây, chúng ta đang định nghĩa biến `x` là có thể thay đổi, nhưng chúng ta
không bao giờ thực sự thay đổi nó. Rust cảnh báo chúng ta về điều đó:

```console
$ cargo build
   Compiling myprogram v0.1.0 (file:///projects/myprogram)
warning: variable does not need to be mutable
 --> src/main.rs:2:9
  |
2 |     let mut x = 0;
  |         ----^
  |         |
  |         help: remove this `mut`
  |
  = note: `#[warn(unused_mut)]` on by default
```

Cảnh báo gợi ý rằng chúng ta nên xóa từ khóa `mut`. Chúng ta có thể tự động áp
dụng gợi ý đó bằng cách sử dụng công cụ `rustfix` bằng cách chạy lệnh
`cargo fix`:

```console
$ cargo fix
    Checking myprogram v0.1.0 (file:///projects/myprogram)
      Fixing src/main.rs (1 fix)
    Finished dev [unoptimized + debuginfo] target(s) in 0.59s
```

Khi chúng ta xem lại _src/main.rs_, chúng ta sẽ thấy rằng `cargo fix` đã thay
đổi mã:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
fn main() {
    let x = 42;
    println!("{x}");
}
```

Biến `x` giờ đây không thể thay đổi, và cảnh báo không còn xuất hiện nữa.

Bạn cũng có thể sử dụng lệnh `cargo fix` để chuyển đổi mã của mình giữa các
phiên bản Rust khác nhau. Các phiên bản được đề cập trong [Phụ lục
E][editions]<!-- ignore -->.

### Kiểm tra Nhiều hơn với Clippy

Công cụ Clippy là một bộ sưu tập các công cụ kiểm tra để phân tích mã của bạn để
bạn có thể phát hiện các lỗi thông thường và cải thiện mã Rust của mình. Clippy
được bao gồm trong cài đặt Rust tiêu chuẩn.

Để chạy kiểm tra của Clippy trên bất kỳ dự án Cargo nào, hãy nhập như sau:

```console
$ cargo clippy
```

Ví dụ, giả sử bạn viết một chương trình sử dụng một giá trị xấp xỉ của một hằng
số toán học, như pi, như chương trình này:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = 3.1415;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Chạy `cargo clippy` trên dự án này sẽ dẫn đến lỗi này:

```text
error: approximate value of `f{32, 64}::consts::PI` found
 --> src/main.rs:2:13
  |
2 |     let x = 3.1415;
  |             ^^^^^^
  |
  = note: `#[deny(clippy::approx_constant)]` on by default
  = help: consider using the constant directly
  = help: for further information visit https://rust-lang.github.io/rust-clippy/master/index.html#approx_constant
```

Lỗi này cho bạn biết rằng Rust đã có một hằng số `PI` chính xác hơn được định
nghĩa sẵn, và chương trình của bạn sẽ chính xác hơn nếu bạn sử dụng hằng số đó
thay vì. Sau đó bạn sẽ thay đổi mã của mình để sử dụng hằng số `PI`.

Mã sau đây không gây ra bất kỳ lỗi hoặc cảnh báo nào từ Clippy:

<Listing file-name="src/main.rs">

```rust
fn main() {
    let x = std::f64::consts::PI;
    let r = 8.0;
    println!("the area of the circle is {}", x * r * r);
}
```

</Listing>

Để biết thêm thông tin về Clippy, hãy xem [tài liệu của nó][clippy].

[clippy]: https://github.com/rust-lang/rust-clippy

### Tích hợp IDE bằng `rust-analyzer`

Để giúp tích hợp IDE, cộng đồng Rust khuyến nghị sử dụng
[`rust-analyzer`][rust-analyzer]<!-- ignore -->. Công cụ này là một bộ các tiện
ích tập trung vào trình biên dịch, nói [Giao thức Máy chủ Ngôn ngữ][lsp]<!--
ignore -->, đây là một đặc tả cho các IDE và ngôn ngữ lập trình để giao tiếp với
nhau. Các máy khách khác nhau có thể sử dụng `rust-analyzer`, chẳng hạn như [plug-in
Rust analyzer cho Visual Studio Code][vscode].

Truy cập trang chủ của dự án `rust-analyzer` [trang
chủ][rust-analyzer]<!-- ignore --> để biết hướng dẫn cài đặt, sau đó cài đặt hỗ
trợ máy chủ ngôn ngữ trong IDE cụ thể của bạn. IDE của bạn sẽ có được các khả
năng như tự động hoàn thành, chuyển đến định nghĩa và lỗi nội tuyến.

[rustfmt]: https://github.com/rust-lang/rustfmt
[editions]: appendix-05-editions.md
[clippy]: https://github.com/rust-lang/rust-clippy
[rust-analyzer]: https://rust-analyzer.github.io
[lsp]: http://langserver.org/
[vscode]:
  https://marketplace.visualstudio.com/items?itemName=rust-lang.rust-analyzer
