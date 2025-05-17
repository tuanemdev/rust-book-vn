# Tính năng Ngôn ngữ Hàm: Iterators và Closures

Thiết kế của Rust đã lấy cảm hứng từ nhiều ngôn ngữ và kỹ thuật hiện có, và một
ảnh hưởng đáng kể là _lập trình hàm_. Lập trình theo phong cách hàm thường bao
gồm việc sử dụng hàm như là giá trị bằng cách truyền chúng vào đối số, trả về
chúng từ các hàm khác, gán chúng cho biến để thực thi sau này, và nhiều điều
tương tự.

Trong chương này, chúng ta sẽ không tranh luận về vấn đề lập trình hàm là gì
hoặc không phải là gì, mà thay vào đó sẽ thảo luận về một số tính năng của Rust
tương tự với các tính năng trong nhiều ngôn ngữ thường được gọi là hàm.

Cụ thể hơn, chúng ta sẽ đề cập đến:

- _Closures_, một cấu trúc giống hàm mà bạn có thể lưu trữ trong một biến
- _Iterators_, một cách để xử lý một chuỗi các phần tử
- Cách sử dụng closures và iterators để cải thiện dự án I/O trong Chương 12
- Hiệu suất của closures và iterators (tiết lộ: chúng nhanh hơn bạn nghĩ!)

Chúng ta đã đề cập đến một số tính năng Rust khác, như pattern matching và
enums, cũng bị ảnh hưởng bởi phong cách lập trình hàm. Vì việc thành thạo
closures và iterators là một phần quan trọng của việc viết mã Rust thông thường,
nhanh chóng, chúng ta sẽ dành toàn bộ chương này cho chúng.
