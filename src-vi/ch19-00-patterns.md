# Mẫu và Khớp mẫu

_Mẫu_ (pattern) là một cú pháp đặc biệt trong Rust để so khớp với cấu trúc của
các kiểu dữ liệu, từ đơn giản đến phức tạp. Sử dụng mẫu kết hợp với biểu thức
`match` và các cấu trúc khác cung cấp cho bạn nhiều kiểm soát hơn đối với luồng
điều khiển chương trình. Một mẫu bao gồm một số kết hợp của các thành phần sau:

- Các giá trị nghĩa đen (Literals)
- Phân rã mảng, enum, struct, hoặc tuple
- Biến
- Ký tự đại diện (Wildcards)
- Vị trí giữ chỗ (Placeholders)

Một số ví dụ về mẫu bao gồm `x`, `(a, 3)`, và `Some(Color::Red)`. Trong các ngữ
cảnh mà mẫu có giá trị, những thành phần này mô tả hình dạng của dữ liệu. Chương
trình của chúng ta sau đó so khớp giá trị với mẫu để xác định liệu nó có đúng
hình dạng dữ liệu để tiếp tục chạy một phần mã cụ thể không.

Để sử dụng một mẫu, chúng ta so sánh nó với một giá trị nào đó. Nếu mẫu khớp với
giá trị, chúng ta sử dụng các phần của giá trị trong mã của mình. Hãy nhớ lại
biểu thức `match` trong Chương 6 đã sử dụng mẫu, chẳng hạn như ví dụ về máy phân
loại tiền xu. Nếu giá trị phù hợp với hình dạng của mẫu, chúng ta có thể sử dụng
các phần được đặt tên. Nếu nó không phù hợp, mã liên kết với mẫu sẽ không chạy.

Chương này là tài liệu tham khảo về tất cả các vấn đề liên quan đến mẫu. Chúng
ta sẽ đề cập đến những nơi hợp lệ để sử dụng mẫu, sự khác biệt giữa mẫu có thể
bác bỏ và không thể bác bỏ, và các loại cú pháp mẫu khác nhau mà bạn có thể
thấy. Đến cuối chương, bạn sẽ biết cách sử dụng mẫu để biểu đạt nhiều khái niệm
một cách rõ ràng.
