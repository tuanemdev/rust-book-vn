<!-- Old link, do not remove -->

<a id="installing-binaries-from-cratesio-with-cargo-install"></a>

## Cài đặt Tệp Nhị phân với `cargo install`

Lệnh `cargo install` cho phép bạn cài đặt và sử dụng các crate nhị phân một cách
cục bộ. Điều này không nhằm mục đích thay thế các gói hệ thống; nó được thiết kế
để cung cấp một cách thuận tiện cho các nhà phát triển Rust cài đặt các công cụ
mà người khác đã chia sẻ trên [crates.io](https://crates.io/)<!-- ignore -->.
Lưu ý rằng bạn chỉ có thể cài đặt các gói có mục tiêu nhị phân. Một _mục tiêu
nhị phân_ là chương trình có thể chạy được tạo ra nếu crate có tệp _src/main.rs_
hoặc một tệp khác được chỉ định là tệp nhị phân, khác với mục tiêu thư viện
không thể chạy độc lập nhưng phù hợp để bao gồm trong các chương trình khác.
Thông thường, các crate có thông tin trong tệp _README_ về việc liệu một crate
là thư viện, có mục tiêu nhị phân, hoặc cả hai.

Tất cả các tệp nhị phân được cài đặt với `cargo install` được lưu trữ trong thư
mục _bin_ của thư mục gốc cài đặt. Nếu bạn đã cài đặt Rust bằng _rustup.rs_ và
không có bất kỳ cấu hình tùy chỉnh nào, thư mục này sẽ là _$HOME/.cargo/bin_.
Đảm bảo rằng thư mục đó nằm trong `$PATH` của bạn để có thể chạy các chương
trình bạn đã cài đặt với `cargo install`.

Ví dụ, trong Chương 12 chúng ta đã đề cập rằng có một triển khai Rust của công
cụ `grep` gọi là `ripgrep` để tìm kiếm tệp. Để cài đặt `ripgrep`, chúng ta có
thể chạy như sau:

<!-- manual-regeneration
cargo install something you don't have, copy relevant output below
-->

```console
$ cargo install ripgrep
    Updating crates.io index
  Downloaded ripgrep v14.1.1
  Downloaded 1 crate (213.6 KB) in 0.40s
  Installing ripgrep v14.1.1
--snip--
   Compiling grep v0.3.2
    Finished `release` profile [optimized + debuginfo] target(s) in 6.73s
  Installing ~/.cargo/bin/rg
   Installed package `ripgrep v14.1.1` (executable `rg`)
```

Dòng áp chót của đầu ra hiển thị vị trí và tên của tệp nhị phân đã cài đặt,
trong trường hợp của `ripgrep` là `rg`. Miễn là thư mục cài đặt nằm trong
`$PATH` của bạn, như đã đề cập trước đó, bạn có thể chạy `rg --help` và bắt đầu
sử dụng một công cụ nhanh hơn, "Rust hơn" để tìm kiếm tệp!
