# Quản lý Dự án Ngày càng Lớn với Packages, Crates, và Modules

Khi bạn viết các chương trình lớn, việc tổ chức mã của bạn sẽ ngày càng trở nên
quan trọng. Bằng cách nhóm các chức năng liên quan và tách riêng mã với các tính
năng riêng biệt, bạn sẽ làm rõ nơi tìm mã thực hiện một tính năng cụ thể và nơi
để thay đổi cách một tính năng hoạt động.

Các chương trình chúng ta đã viết cho đến nay đều nằm trong một module trong một
tệp. Khi một dự án phát triển, bạn nên tổ chức mã bằng cách chia nó thành nhiều
module và sau đó là nhiều tệp. Một package có thể chứa nhiều binary crate và tùy
chọn một library crate. Khi một package phát triển, bạn có thể trích xuất các
phần thành các crate riêng biệt trở thành các phụ thuộc bên ngoài. Chương này đề
cập đến tất cả các kỹ thuật này. Đối với các dự án rất lớn bao gồm một tập hợp
các package có liên quan phát triển cùng nhau, Cargo cung cấp _workspaces_, mà
chúng ta sẽ đề cập trong ["Cargo Workspaces"][workspaces]<!-- ignore --> ở
Chương 14.

Chúng ta cũng sẽ thảo luận về việc đóng gói chi tiết triển khai, điều này cho
phép bạn tái sử dụng mã ở mức cao hơn: khi bạn đã triển khai một thao tác, mã
khác có thể gọi mã của bạn thông qua giao diện công khai mà không cần biết cách
triển khai hoạt động như thế nào. Cách bạn viết mã định nghĩa những phần nào là
công khai cho mã khác sử dụng và những phần nào là chi tiết triển khai riêng tư
mà bạn có quyền thay đổi. Đây là một cách khác để hạn chế lượng chi tiết mà bạn
phải giữ trong đầu.

Một khái niệm liên quan là phạm vi: ngữ cảnh lồng nhau mà mã được viết có một
tập hợp các tên được định nghĩa là "trong phạm vi." Khi đọc, viết và biên dịch
mã, các lập trình viên và trình biên dịch cần biết liệu một tên cụ thể tại một
vị trí cụ thể đề cập đến một biến, hàm, struct, enum, module, hằng số, hoặc mục
khác và mục đó có ý nghĩa gì. Bạn có thể tạo phạm vi và thay đổi những tên nào
đang ở trong hoặc ngoài phạm vi. Bạn không thể có hai mục với cùng một tên trong
cùng một phạm vi; các công cụ có sẵn để giải quyết xung đột tên.

Rust có một số tính năng cho phép bạn quản lý tổ chức mã của mình, bao gồm những
chi tiết nào được hiển thị, những chi tiết nào là riêng tư, và những tên nào nằm
trong mỗi phạm vi trong chương trình của bạn. Những tính năng này, đôi khi được
gọi chung là _hệ thống module_, bao gồm:

- **Packages**: Một tính năng của Cargo cho phép bạn xây dựng, kiểm tra và chia
  sẻ crates
- **Crates**: Một cây các module tạo ra một thư viện hoặc một tệp thực thi
- **Modules và use**: Cho phép bạn kiểm soát tổ chức, phạm vi và quyền riêng tư
  của các đường dẫn
- **Paths**: Một cách để đặt tên cho một mục, chẳng hạn như struct, hàm hoặc
  module

Trong chương này, chúng ta sẽ đề cập đến tất cả các tính năng này, thảo luận về
cách chúng tương tác, và giải thích cách sử dụng chúng để quản lý phạm vi. Đến
cuối chương, bạn sẽ có hiểu biết vững chắc về hệ thống module và có thể làm việc
với các phạm vi như một chuyên gia!

[workspaces]: ch14-03-cargo-workspaces.html
