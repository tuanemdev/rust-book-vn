## Mở rộng Cargo với Lệnh Tùy chỉnh

Cargo được thiết kế để bạn có thể mở rộng nó với các lệnh phụ mới mà không cần
phải sửa đổi nó. Nếu một tệp nhị phân trong `$PATH` của bạn có tên là
`cargo-something`, bạn có thể chạy nó như thể nó là một lệnh phụ của Cargo bằng
cách chạy `cargo something`. Các lệnh tùy chỉnh như thế này cũng được liệt kê
khi bạn chạy `cargo --list`. Khả năng sử dụng `cargo install` để cài đặt các
phần mở rộng và sau đó chạy chúng giống như các công cụ Cargo tích hợp là một
lợi ích cực kỳ thuận tiện từ thiết kế của Cargo!

## Tóm tắt

Chia sẻ mã với Cargo và [crates.io](https://crates.io/)<!-- ignore --> là một
phần làm cho hệ sinh thái Rust hữu ích cho nhiều tác vụ khác nhau. Thư viện
chuẩn của Rust nhỏ gọn và ổn định, nhưng các crate dễ dàng được chia sẻ, sử dụng
và cải tiến theo một tiến độ khác với ngôn ngữ. Đừng ngần ngại chia sẻ mã hữu
ích cho bạn trên [crates.io](https://crates.io/)<!-- ignore -->; rất có thể nó
cũng sẽ hữu ích cho người khác!
