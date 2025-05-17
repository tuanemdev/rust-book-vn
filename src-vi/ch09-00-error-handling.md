# Xử Lý Lỗi

Lỗi là điều không thể tránh khỏi trong phần mềm, vì vậy Rust có một số tính năng
để xử lý các tình huống khi có sự cố xảy ra. Trong nhiều trường hợp, Rust yêu
cầu bạn phải thừa nhận khả năng xảy ra lỗi và thực hiện một số hành động trước
khi mã của bạn được biên dịch. Yêu cầu này làm cho chương trình của bạn trở nên
mạnh mẽ hơn bằng cách đảm bảo rằng bạn sẽ phát hiện lỗi và xử lý chúng một cách
thích hợp trước khi triển khai mã của bạn lên môi trường sản phẩm!

Rust phân loại lỗi thành hai danh mục chính: lỗi _có thể khôi phục_ và lỗi
_không thể khôi phục_. Đối với lỗi có thể khôi phục, chẳng hạn như lỗi _không
tìm thấy tệp_, chúng ta rất có thể chỉ muốn báo cáo vấn đề cho người dùng và thử
lại thao tác đó. Lỗi không thể khôi phục luôn là dấu hiệu của các lỗi phần mềm,
chẳng hạn như cố gắng truy cập vào vị trí vượt quá giới hạn của mảng, và vì vậy
chúng ta muốn dừng chương trình ngay lập tức.

Hầu hết các ngôn ngữ không phân biệt giữa hai loại lỗi này và xử lý cả hai theo
cùng một cách, sử dụng các cơ chế như exception. Rust không có exception. Thay
vào đó, nó có kiểu `Result<T, E>` cho lỗi có thể khôi phục và macro `panic!` để
dừng thực thi khi chương trình gặp lỗi không thể khôi phục. Chương này đề cập
đến việc gọi `panic!` trước, sau đó nói về việc trả về giá trị `Result<T, E>`.
Ngoài ra, chúng ta sẽ khám phá các cân nhắc khi quyết định liệu có nên cố gắng
khôi phục từ lỗi hay dừng thực thi.
