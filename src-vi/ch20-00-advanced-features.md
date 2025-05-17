# Các Tính Năng Nâng Cao

Đến lúc này, bạn đã học được những phần thường được sử dụng nhất của ngôn ngữ
lập trình Rust. Trước khi chúng ta thực hiện một dự án nữa, trong Chương 21,
chúng ta sẽ xem xét một số khía cạnh của ngôn ngữ mà bạn có thể gặp phải thỉnh
thoảng, nhưng có thể không sử dụng hàng ngày. Bạn có thể sử dụng chương này như
một tài liệu tham khảo khi bạn gặp phải những điều chưa biết. Các tính năng được
đề cập ở đây rất hữu ích trong những tình huống rất cụ thể. Mặc dù bạn có thể
không sử dụng chúng thường xuyên, chúng tôi muốn đảm bảo rằng bạn nắm được tất
cả các tính năng mà Rust cung cấp.

Trong chương này, chúng ta sẽ đề cập đến:

- Unsafe Rust: cách từ bỏ một số đảm bảo của Rust và tự chịu trách nhiệm về việc
  duy trì những đảm bảo đó một cách thủ công
- Traits nâng cao: các kiểu liên kết, tham số kiểu mặc định, cú pháp đầy đủ,
  supertraits, và mẫu newtype liên quan đến traits
- Kiểu dữ liệu nâng cao: thêm về mẫu newtype, bí danh kiểu, kiểu never, và các
  kiểu có kích thước động
- Hàm và closure nâng cao: con trỏ hàm và trả về closures
- Macro: các cách để định nghĩa mã nguồn định nghĩa thêm mã nguồn ở thời điểm
  biên dịch

Đây là một tập hợp đa dạng các tính năng của Rust với điều gì đó cho tất cả mọi
người! Hãy bắt đầu!
