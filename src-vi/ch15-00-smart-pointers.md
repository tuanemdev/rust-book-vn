# Con trỏ Thông minh

Một _con trỏ_ là một khái niệm chung cho một biến chứa một địa chỉ trong bộ nhớ.
Địa chỉ này tham chiếu tới, hoặc "trỏ tới," một số dữ liệu khác. Loại con trỏ
phổ biến nhất trong Rust là tham chiếu, mà bạn đã học trong Chương 4. Tham chiếu
được biểu thị bằng ký hiệu `&` và mượn giá trị mà chúng trỏ tới. Chúng không có
bất kỳ khả năng đặc biệt nào ngoài việc tham chiếu đến dữ liệu, và chúng không
có chi phí phụ.

_Con trỏ thông minh_, mặt khác, là các cấu trúc dữ liệu hoạt động như một con
trỏ nhưng cũng có thêm siêu dữ liệu và khả năng. Khái niệm về con trỏ thông minh
không phải là duy nhất trong Rust: con trỏ thông minh bắt nguồn từ C++ và cũng
tồn tại trong các ngôn ngữ khác. Rust có nhiều loại con trỏ thông minh khác nhau
được định nghĩa trong thư viện chuẩn cung cấp chức năng vượt xa so với tham
chiếu. Để khám phá khái niệm chung, chúng ta sẽ xem xét một vài ví dụ khác nhau
về con trỏ thông minh, bao gồm một kiểu con trỏ thông minh _đếm tham chiếu_. Con
trỏ này cho phép bạn cho phép dữ liệu có nhiều chủ sở hữu bằng cách theo dõi số
lượng chủ sở hữu và, khi không còn chủ sở hữu nào, dọn dẹp dữ liệu.

Rust, với khái niệm về quyền sở hữu và mượn, có một điểm khác biệt bổ sung giữa
tham chiếu và con trỏ thông minh: trong khi tham chiếu chỉ mượn dữ liệu, trong
nhiều trường hợp con trỏ thông minh _sở hữu_ dữ liệu mà chúng trỏ tới.

Con trỏ thông minh thường được triển khai bằng cách sử dụng structs. Không giống
như một struct thông thường, con trỏ thông minh triển khai các trait `Deref` và
`Drop`. Trait `Deref` cho phép một thể hiện của struct con trỏ thông minh hoạt
động như một tham chiếu để bạn có thể viết mã của mình để làm việc với cả tham
chiếu hoặc con trỏ thông minh. Trait `Drop` cho phép bạn tùy chỉnh mã được chạy
khi một thể hiện của con trỏ thông minh ra khỏi phạm vi. Trong chương này, chúng
ta sẽ thảo luận cả hai trait này và minh họa tại sao chúng quan trọng đối với
con trỏ thông minh.

Do mẫu con trỏ thông minh là một mẫu thiết kế chung được sử dụng thường xuyên
trong Rust, chương này sẽ không đề cập đến mọi con trỏ thông minh hiện có. Nhiều
thư viện có con trỏ thông minh riêng, và bạn thậm chí có thể tự viết. Chúng ta
sẽ đề cập đến những con trỏ thông minh phổ biến nhất trong thư viện chuẩn:

- `Box<T>`, để phân bổ giá trị trên heap
- `Rc<T>`, một kiểu đếm tham chiếu cho phép nhiều quyền sở hữu
- `Ref<T>` và `RefMut<T>`, được truy cập thông qua `RefCell<T>`, một kiểu thực
  thi quy tắc mượn tại thời điểm chạy thay vì thời điểm biên dịch

Ngoài ra, chúng ta sẽ đề cập đến mẫu _tính thay đổi nội bộ_ nơi một kiểu không
thể thay đổi hiển thị một API để thay đổi một giá trị bên trong. Chúng ta cũng
sẽ thảo luận về chu kỳ tham chiếu: cách chúng có thể rò rỉ bộ nhớ và cách ngăn
chặn chúng.

Hãy bắt đầu!
