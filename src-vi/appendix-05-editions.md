## Phụ lục E - Các Phiên bản

Trong Chương 1, bạn đã thấy rằng `cargo new` thêm một số siêu dữ liệu vào file
_Cargo.toml_ của bạn về phiên bản. Phụ lục này nói về ý nghĩa của điều đó!

Ngôn ngữ và trình biên dịch Rust có chu kỳ phát hành sáu tuần một lần, có nghĩa
là người dùng nhận được một dòng liên tục các tính năng mới. Các ngôn ngữ lập
trình khác phát hành những thay đổi lớn hơn ít thường xuyên hơn; Rust phát hành
các bản cập nhật nhỏ hơn thường xuyên hơn. Sau một thời gian, tất cả những thay
đổi nhỏ này cộng lại. Nhưng từ phiên bản này sang phiên bản khác, có thể khó để
nhìn lại và nói, "Wow, giữa Rust 1.10 và Rust 1.31, Rust đã thay đổi rất nhiều!"

Cứ khoảng ba năm một lần, đội ngũ Rust tạo ra một _phiên bản_ Rust mới. Mỗi
phiên bản tập hợp các tính năng đã ra mắt thành một gói rõ ràng với tài liệu và
công cụ được cập nhật đầy đủ. Các phiên bản mới được phát hành như một phần của
quy trình phát hành sáu tuần một lần thông thường.

Các phiên bản phục vụ những mục đích khác nhau cho những người khác nhau:

- Đối với người dùng Rust tích cực, một phiên bản mới tập hợp các thay đổi tăng
  dần thành một gói dễ hiểu.
- Đối với người không phải người dùng, một phiên bản mới báo hiệu rằng một số
  cải tiến lớn đã xuất hiện, điều này có thể khiến Rust đáng để xem xét lại.
- Đối với những người phát triển Rust, một phiên bản mới cung cấp một điểm tụ
  họp cho dự án nói chung.

Tại thời điểm viết bài này, có bốn phiên bản Rust: Rust 2015, Rust 2018, Rust
2021, và Rust 2024. Cuốn sách này được viết bằng các cách dùng của phiên bản
Rust 2024.

Khóa `edition` trong _Cargo.toml_ chỉ ra phiên bản nào trình biên dịch nên sử
dụng cho mã của bạn. Nếu khóa không tồn tại, Rust sử dụng `2015` làm giá trị
phiên bản vì lý do tương thích ngược.

Mỗi dự án có thể chọn một phiên bản khác với phiên bản mặc định 2015. Các phiên
bản có thể chứa các thay đổi không tương thích, chẳng hạn như bao gồm một từ
khóa mới xung đột với các định danh trong mã. Tuy nhiên, trừ khi bạn chọn những
thay đổi đó, mã của bạn sẽ tiếp tục được biên dịch ngay cả khi bạn nâng cấp
phiên bản trình biên dịch Rust mà bạn sử dụng.
