## Xin chào, Cargo!

Cargo là hệ thống build và quản lý gói của Rust. Hầu hết các Rustacean (lập
trình viên Rust) đều sử dụng công cụ này để quản lý các dự án Rust vì Cargo xử
lý nhiều tác vụ cho bạn, chẳng hạn như xây dựng code, tải xuống các thư viện mà
code của bạn phụ thuộc và build các thư viện đó. (Chúng tôi gọi các thư viện mà
code của bạn cần là _dependencies_ - các phụ thuộc.)

Các chương trình Rust đơn giản nhất, như cái chúng ta đã viết cho đến giờ, không
có bất kỳ phụ thuộc nào. Nếu chúng ta xây dựng dự án "Hello, world!" với Cargo,
nó sẽ chỉ sử dụng một phần của Cargo cho việc xử lý build code của bạn. Khi bạn
viết các chương trình Rust phức tạp hơn, bạn sẽ thêm các phụ thuộc, và nếu bạn
bắt đầu một dự án bằng Cargo, việc thêm phụ thuộc sẽ dễ dàng hơn nhiều.

Vì phần lớn các dự án Rust sử dụng Cargo, phần còn lại của cuốn sách này giả
định rằng bạn cũng đang sử dụng Cargo. Cargo được cài đặt cùng với Rust nếu bạn
đã sử dụng trình cài đặt chính thức được thảo luận trong phần ["Cài
đặt"][installation]<!-- ignore -->. Nếu bạn đã cài đặt Rust thông qua một số
phương tiện khác, hãy kiểm tra xem Cargo đã được cài đặt chưa bằng cách nhập
lệnh sau vào terminal của bạn:

```console
$ cargo --version
```

Nếu bạn thấy một số phiên bản, bạn đã có nó! Nếu bạn thấy một lỗi, chẳng hạn như
`command not found`, hãy xem tài liệu cho phương thức cài đặt của bạn để xác
định cách cài đặt Cargo riêng biệt.

### Tạo một dự án với Cargo

Hãy tạo một dự án mới sử dụng Cargo và xem nó khác với dự án "Hello, world!" ban
đầu của chúng ta như thế nào. Quay lại thư mục _projects_ của bạn (hoặc bất cứ
nơi nào bạn đã quyết định lưu trữ code). Sau đó, trên bất kỳ hệ điều hành nào,
chạy lệnh sau:

```console
$ cargo new hello_cargo
$ cd hello_cargo
```

Lệnh đầu tiên tạo một thư mục và dự án mới có tên _hello_cargo_. Chúng ta đã đặt
tên dự án là _hello_cargo_, và Cargo tạo các tệp của nó trong một thư mục cùng
tên.

Đi vào thư mục _hello_cargo_ và liệt kê các tệp. Bạn sẽ thấy rằng Cargo đã tạo
hai tệp và một thư mục cho chúng ta: một tệp _Cargo.toml_ và một thư mục _src_
với một tệp _main.rs_ bên trong.

Nó cũng đã khởi tạo một repository Git mới cùng với tệp _.gitignore_. Các tệp
Git sẽ không được tạo nếu bạn chạy `cargo new` trong một Git repository đã tồn
tại; bạn có thể ghi đè hành vi này bằng cách sử dụng `cargo new --vcs=git`.

> Lưu ý: Git là một hệ thống quản lý phiên bản phổ biến. Bạn có thể thay đổi
> `cargo new` để sử dụng một hệ thống quản lý phiên bản khác hoặc không sử dụng
> hệ thống quản lý phiên bản nào bằng cách sử dụng cờ `--vcs`. Chạy
> `cargo new --help` để xem các tùy chọn có sẵn.

Mở _Cargo.toml_ trong trình soạn thảo văn bản bạn chọn. Nó sẽ trông giống với mã
trong Listing 1-2.

<Listing number="1-2" file-name="Cargo.toml" caption="Nội dung của *Cargo.toml* được tạo bởi `cargo new`">

```toml
[package]
name = "hello_cargo"
version = "0.1.0"
edition = "2024"

[dependencies]
```

</Listing>

Tệp này có định dạng [_TOML_][toml]<!-- ignore --> (_Tom's Obvious, Minimal
Language_), đây là định dạng cấu hình của Cargo.

Dòng đầu tiên, `[package]`, là một tiêu đề phần cho biết rằng các câu lệnh tiếp
theo đang cấu hình một gói. Khi chúng ta thêm thông tin vào tệp này, chúng ta sẽ
thêm các phần khác.

Ba dòng tiếp theo thiết lập thông tin cấu hình mà Cargo cần để biên dịch chương
trình của bạn: tên, phiên bản và phiên bản của Rust để sử dụng. Chúng ta sẽ nói
về khóa `edition` trong [Phụ lục E][appendix-e]<!-- ignore -->.

Dòng cuối cùng, `[dependencies]`, là phần bắt đầu cho bạn liệt kê bất kỳ phụ
thuộc nào của dự án. Trong Rust, các gói mã được gọi là _crates_. Chúng ta sẽ
không cần bất kỳ crate nào khác cho dự án này, nhưng chúng ta sẽ cần trong dự án
đầu tiên ở Chương 2, vì vậy chúng ta sẽ sử dụng phần phụ thuộc này sau.

Bây giờ mở _src/main.rs_ và xem qua:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
fn main() {
    println!("Hello, world!");
}
```

Cargo đã tạo một chương trình "Hello, world!" cho bạn, giống như cái mà chúng ta
đã viết trong Listing 1-1! Cho đến nay, sự khác biệt giữa dự án của chúng ta và
dự án mà Cargo đã tạo là Cargo đặt mã trong thư mục _src_, và chúng ta có một
tệp cấu hình _Cargo.toml_ ở thư mục cấp cao nhất.

Cargo mong muốn các tệp mã nguồn của bạn nằm trong thư mục _src_. Thư mục dự án
cấp cao nhất chỉ dành cho các tệp README, thông tin giấy phép, các tệp cấu hình
và bất kỳ thứ gì khác không liên quan đến mã của bạn. Sử dụng Cargo giúp bạn tổ
chức các dự án. Có một vị trí cho mọi thứ, và mọi thứ đều ở đúng vị trí.

Nếu bạn đã bắt đầu một dự án không sử dụng Cargo, như chúng ta đã làm với dự án
"Hello, world!", bạn có thể chuyển đổi nó thành một dự án sử dụng Cargo. Di
chuyển mã dự án vào thư mục _src_ và tạo một tệp _Cargo.toml_ thích hợp. Một
cách dễ dàng để có được tệp _Cargo.toml_ đó là chạy `cargo init`, nó sẽ tạo tự
động cho bạn.

### Xây dựng và chạy một dự án Cargo

Bây giờ hãy xem có gì khác khi chúng ta xây dựng và chạy chương trình "Hello,
world!" với Cargo! Từ thư mục _hello_cargo_ của bạn, xây dựng dự án của bạn bằng
cách nhập lệnh sau:

```console
$ cargo build
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 2.85 secs
```

Lệnh này tạo một tệp thực thi trong _target/debug/hello_cargo_ (hoặc
_target\debug\hello_cargo.exe_ trên Windows) thay vì trong thư mục hiện tại của
bạn. Vì bản dựng mặc định là bản dựng debug, Cargo đặt tệp nhị phân vào một thư
mục có tên _debug_. Bạn có thể chạy tệp thực thi bằng lệnh này:

```console
$ ./target/debug/hello_cargo # hoặc .\target\debug\hello_cargo.exe trên Windows
Hello, world!
```

Nếu mọi việc suôn sẻ, `Hello, world!` sẽ được in ra terminal. Chạy `cargo build`
lần đầu tiên cũng khiến Cargo tạo một tệp mới ở cấp cao nhất: _Cargo.lock_. Tệp
này theo dõi các phiên bản chính xác của các phụ thuộc trong dự án của bạn. Dự
án này không có phụ thuộc, vì vậy tệp hơi thưa thớt. Bạn sẽ không bao giờ cần
thay đổi tệp này theo cách thủ công; Cargo quản lý nội dung của nó cho bạn.

Chúng ta vừa xây dựng một dự án với `cargo build` và chạy nó với
`./target/debug/hello_cargo`, nhưng chúng ta cũng có thể sử dụng `cargo run` để
biên dịch mã và sau đó chạy tệp thực thi kết quả tất cả trong một lệnh:

```console
$ cargo run
    Finished dev [unoptimized + debuginfo] target(s) in 0.0 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Sử dụng `cargo run` thuận tiện hơn việc phải nhớ chạy `cargo build` và sau đó sử
dụng toàn bộ đường dẫn đến tệp nhị phân, vì vậy hầu hết các nhà phát triển sử
dụng `cargo run`.

Lưu ý rằng lần này chúng ta không thấy đầu ra cho biết rằng Cargo đang biên dịch
`hello_cargo`. Cargo nhận ra rằng các tệp không thay đổi, vì vậy nó đã không xây
dựng lại mà chỉ chạy tệp nhị phân. Nếu bạn đã sửa đổi mã nguồn, Cargo sẽ xây
dựng lại dự án trước khi chạy nó, và bạn sẽ thấy đầu ra này:

```console
$ cargo run
   Compiling hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.33 secs
     Running `target/debug/hello_cargo`
Hello, world!
```

Cargo cũng cung cấp một lệnh có tên là `cargo check`. Lệnh này nhanh chóng kiểm
tra mã của bạn để đảm bảo nó biên dịch được nhưng không tạo ra tệp thực thi:

```console
$ cargo check
   Checking hello_cargo v0.1.0 (file:///projects/hello_cargo)
    Finished dev [unoptimized + debuginfo] target(s) in 0.32 secs
```

Tại sao bạn lại không muốn có một tệp thực thi? Thông thường, `cargo check`
nhanh hơn nhiều so với `cargo build` vì nó bỏ qua bước tạo ra tệp thực thi. Nếu
bạn đang liên tục kiểm tra công việc của mình trong khi viết mã, việc sử dụng
`cargo check` sẽ tăng tốc quá trình cho bạn biết liệu dự án của bạn vẫn đang
biên dịch hay không! Như vậy, nhiều Rustacean chạy `cargo check` định kỳ khi họ
viết chương trình của họ để đảm bảo nó biên dịch. Sau đó, họ chạy `cargo build`
khi họ sẵn sàng sử dụng tệp thực thi.

Hãy tổng kết những gì chúng ta đã học được cho đến nay về Cargo:

- Chúng ta có thể tạo một dự án bằng `cargo new`.
- Chúng ta có thể xây dựng một dự án bằng `cargo build`.
- Chúng ta có thể xây dựng và chạy một dự án trong một bước bằng `cargo run`.
- Chúng ta có thể xây dựng một dự án mà không tạo ra tệp nhị phân để kiểm tra
  lỗi bằng `cargo check`.
- Thay vì lưu kết quả của bản dựng trong cùng thư mục với mã của chúng ta, Cargo
  lưu trữ nó trong thư mục _target/debug_.

Một lợi ích bổ sung của việc sử dụng Cargo là các lệnh giống nhau bất kể bạn
đang làm việc trên hệ điều hành nào. Vì vậy, tại thời điểm này, chúng tôi sẽ
không cung cấp hướng dẫn cụ thể cho Linux và macOS so với Windows nữa.

### Build cho release (phát hành)

Khi dự án của bạn cuối cùng đã sẵn sàng để phát hành, bạn có thể sử dụng
`cargo build --release` để biên dịch nó với các tối ưu hóa. Lệnh này sẽ tạo một
tệp thực thi trong _target/release_ thay vì _target/debug_. Các tối ưu hóa làm
cho mã Rust của bạn chạy nhanh hơn, nhưng việc bật chúng làm kéo dài thời gian
biên dịch chương trình. Đây là lý do tại sao có hai hồ sơ khác nhau: một cho
phát triển, khi bạn muốn xây dựng lại nhanh chóng và thường xuyên, và một khác
để xây dựng chương trình cuối cùng mà bạn sẽ giao cho người dùng, sẽ không được
xây dựng lại nhiều lần và sẽ chạy nhanh nhất có thể. Nếu bạn đang đo hiệu suất
thời gian chạy của mã, hãy chắc chắn chạy `cargo build --release` và đo hiệu
suất với tệp thực thi trong _target/release_.

<!-- Old headings. Do not remove or links may break. -->
<a id="cargo-as-convention"></a>

### Tận dụng các quy ước của Cargo

Với các dự án đơn giản, Cargo không cung cấp nhiều giá trị hơn so với chỉ sử
dụng `rustc`, nhưng nó sẽ chứng minh giá trị của nó khi chương trình của bạn trở
nên phức tạp hơn. Khi chương trình phát triển thành nhiều tệp hoặc cần một phụ
thuộc, thì sẽ dễ dàng hơn nhiều để Cargo phối hợp việc xây dựng.

Mặc dù dự án `hello_cargo` đơn giản, nhưng bây giờ nó sử dụng nhiều công cụ thực
tế mà bạn sẽ sử dụng trong phần còn lại của sự nghiệp Rust của mình. Trên thực
tế, để làm việc trên bất kỳ dự án hiện có nào, bạn có thể sử dụng các lệnh sau
để kiểm tra mã sử dụng Git, thay đổi thành thư mục của dự án đó, và xây dựng:

```console
$ git clone example.org/someproject
$ cd someproject
$ cargo build
```

Để biết thêm thông tin về Cargo, hãy xem [tài liệu của nó][cargo].

## Tóm tắt

Bạn đã có một khởi đầu tuyệt vời cho hành trình Rust của mình! Trong chương này,
bạn đã học cách:

- Cài đặt phiên bản Rust ổn định mới nhất sử dụng `rustup`.
- Cập nhật lên phiên bản Rust mới hơn.
- Mở tài liệu đã cài đặt cục bộ.
- Viết và chạy chương trình "Hello, world!" sử dụng `rustc` trực tiếp.
- Tạo và chạy một dự án mới sử dụng các quy ước của Cargo.

Đây là thời điểm tuyệt vời để xây dựng một chương trình đáng kể hơn để làm quen
với việc đọc và viết mã Rust. Vì vậy, trong Chương 2, chúng ta sẽ xây dựng một
chương trình trò chơi đoán số. Nếu bạn thích bắt đầu bằng cách học cách các khái
niệm lập trình phổ biến hoạt động trong Rust, hãy xem Chương 3 và sau đó quay
lại Chương 2.

[installation]: ch01-01-installation.html#installation
[toml]: https://toml.io
[appendix-e]: appendix-05-editions.html
[cargo]: https://doc.rust-lang.org/cargo/
