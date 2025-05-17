# Viết Kiểm Thử Tự Động

Trong bài luận năm 1972 "The Humble Programmer", Edsger W. Dijkstra đã nói rằng
"kiểm thử chương trình có thể là một cách rất hiệu quả để phát hiện lỗi, nhưng
nó hoàn toàn không đủ để chứng minh sự vắng mặt của chúng." Điều đó không có
nghĩa là chúng ta không nên cố gắng kiểm thử nhiều nhất có thể!

Tính đúng đắn trong chương trình của chúng ta là mức độ mà code của chúng ta
thực hiện đúng những gì chúng ta muốn. Rust được thiết kế với sự quan tâm lớn
đến tính đúng đắn của chương trình, nhưng tính đúng đắn là phức tạp và không dễ
để chứng minh. Hệ thống kiểu của Rust gánh vác phần lớn gánh nặng này, nhưng hệ
thống kiểu không thể bắt được tất cả. Do đó, Rust bao gồm hỗ trợ để viết các bài
kiểm thử phần mềm tự động.

Giả sử chúng ta viết một hàm `add_two` cộng thêm 2 vào bất kỳ số nào được truyền
vào nó. Chữ ký của hàm này chấp nhận một số nguyên làm tham số và trả về một số
nguyên làm kết quả. Khi chúng ta triển khai và biên dịch hàm đó, Rust thực hiện
tất cả các kiểm tra kiểu và kiểm tra mượn mà bạn đã học cho đến nay để đảm bảo
rằng, chẳng hạn, chúng ta không truyền giá trị `String` hoặc tham chiếu không
hợp lệ cho hàm này. Nhưng Rust _không thể_ kiểm tra rằng hàm này sẽ thực hiện
chính xác những gì chúng ta dự định, đó là trả về tham số cộng thêm 2 thay vì,
ví dụ, tham số cộng thêm 10 hoặc tham số trừ đi 50! Đó là lúc các bài kiểm thử
phát huy tác dụng.

Chúng ta có thể viết các bài kiểm thử khẳng định, ví dụ, khi chúng ta truyền `3`
vào hàm `add_two`, giá trị trả về là `5`. Chúng ta có thể chạy các bài kiểm thử
này bất cứ khi nào chúng ta thay đổi code của mình để đảm bảo rằng bất kỳ hành
vi đúng đắn hiện có nào cũng không bị thay đổi.

Kiểm thử là một kỹ năng phức tạp: mặc dù chúng ta không thể đề cập đến mọi chi
tiết về cách viết các bài kiểm thử tốt trong một chương, nhưng trong chương này
chúng ta sẽ thảo luận về cơ chế của các tiện ích kiểm thử của Rust. Chúng ta sẽ
nói về các chú thích và macro có sẵn cho bạn khi viết các bài kiểm thử, hành vi
mặc định và các tùy chọn được cung cấp để chạy các bài kiểm thử của bạn, và cách
tổ chức các bài kiểm thử thành kiểm thử đơn vị và kiểm thử tích hợp.
