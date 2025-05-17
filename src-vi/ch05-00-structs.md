# Sử dụng Structs để Cấu trúc Dữ liệu Liên quan

Một _struct_, hay _structure_, là một kiểu dữ liệu tùy chỉnh cho phép bạn đóng
gói và đặt tên cho nhiều giá trị liên quan tạo thành một nhóm có ý nghĩa. Nếu
bạn đã quen với ngôn ngữ hướng đối tượng, _struct_ giống như các thuộc tính dữ
liệu của một đối tượng. Trong chương này, chúng ta sẽ so sánh và đối chiếu tuple
với struct để xây dựng trên những gì bạn đã biết và chứng minh khi nào struct là
cách tốt hơn để nhóm dữ liệu.

Chúng ta sẽ trình bày cách định nghĩa và khởi tạo struct. Chúng ta sẽ thảo luận
về cách định nghĩa các hàm liên kết, đặc biệt là loại hàm liên kết được gọi là
_methods_, để chỉ định hành vi liên quan đến một kiểu struct. Struct và enums
(được thảo luận trong Chương 6) là các khối xây dựng để tạo ra các kiểu mới
trong miền chương trình của bạn để tận dụng tối đa khả năng kiểm tra kiểu tại
thời điểm biên dịch của Rust.
