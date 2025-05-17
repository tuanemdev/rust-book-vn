# Các Bộ Sưu Tập Thông Dụng

Thư viện chuẩn của Rust bao gồm một số cấu trúc dữ liệu rất hữu ích được gọi là
_các bộ sưu tập (collections)_. Hầu hết các kiểu dữ liệu khác đại diện cho một
giá trị cụ thể, nhưng các bộ sưu tập có thể chứa nhiều giá trị. Không giống như
các kiểu mảng và bộ (tuple) tích hợp sẵn, dữ liệu mà các bộ sưu tập này trỏ đến
được lưu trữ trên heap, có nghĩa là lượng dữ liệu không cần phải biết tại thời
điểm biên dịch và có thể tăng hoặc giảm khi chương trình chạy. Mỗi loại bộ sưu
tập có khả năng và chi phí khác nhau, và việc chọn một bộ sưu tập phù hợp cho
tình huống hiện tại của bạn là một kỹ năng mà bạn sẽ phát triển theo thời gian.
Trong chương này, chúng ta sẽ thảo luận về ba bộ sưu tập được sử dụng rất thường
xuyên trong các chương trình Rust:

- Một _vector_ cho phép bạn lưu trữ một số lượng biến đổi các giá trị cạnh nhau.
- Một _chuỗi (string)_ là một bộ sưu tập các ký tự. Chúng ta đã đề cập đến kiểu
  `String` trước đây, nhưng trong chương này chúng ta sẽ nói về nó một cách chi
  tiết.
- Một _bảng băm (hash map)_ cho phép bạn liên kết một giá trị với một khóa cụ
  thể. Đây là một triển khai cụ thể của cấu trúc dữ liệu tổng quát hơn được gọi
  là _map_.

Để tìm hiểu về các loại bộ sưu tập khác được cung cấp bởi thư viện chuẩn, hãy
xem [tài liệu][collections].

Chúng ta sẽ thảo luận về cách tạo và cập nhật vector, chuỗi, và bảng băm, cũng
như những điều gì làm cho mỗi loại đặc biệt.

[collections]: ../std/collections/index.html
