## Đồng Thời Mở Rộng với Các Đặc Tính `Send` và `Sync`

<!-- Old link, do not remove -->

<a id="extensible-concurrency-with-the-sync-and-send-traits"></a>

Điều thú vị là, hầu hết các tính năng đồng thời mà chúng ta đã nói đến trong
chương này đều là một phần của thư viện chuẩn, không phải ngôn ngữ. Lựa chọn của
bạn để xử lý tính đồng thời không bị giới hạn ở ngôn ngữ hoặc thư viện chuẩn;
bạn có thể viết các tính năng đồng thời của riêng mình hoặc sử dụng những tính
năng được viết bởi người khác.

Tuy nhiên, trong số các khái niệm đồng thời quan trọng được nhúng vào ngôn ngữ
thay vì thư viện chuẩn là các đặc tính `std::marker` là `Send` và `Sync`.

### Cho Phép Chuyển Giao Quyền Sở Hữu Giữa Các Thread với `Send`

Đặc tính đánh dấu `Send` chỉ ra rằng quyền sở hữu của các giá trị thuộc kiểu
triển khai `Send` có thể được chuyển giao giữa các thread. Hầu hết mọi kiểu Rust
triển khai `Send`, nhưng có một số ngoại lệ, bao gồm `Rc<T>`: kiểu này không thể
triển khai `Send` vì nếu bạn sao chép một giá trị `Rc<T>` và cố gắng chuyển
quyền sở hữu của bản sao đó sang một thread khác, cả hai thread có thể cập nhật
số lượng tham chiếu cùng một lúc. Vì lý do này, `Rc<T>` được triển khai để sử
dụng trong các tình huống đơn thread, nơi bạn không muốn trả giá cho hiệu suất
an toàn thread.

Do đó, hệ thống kiểu và ràng buộc đặc tính của Rust đảm bảo rằng bạn không bao
giờ vô tình gửi một giá trị `Rc<T>` qua các thread một cách không an toàn. Khi
chúng ta cố gắng làm điều này trong Listing 16-14, chúng ta đã nhận được lỗi
`` the trait `Send` is not implemented for `Rc<Mutex<i32>>` ``. Khi chúng ta
chuyển sang `Arc<T>`, vốn triển khai `Send`, mã đã biên dịch thành công.

Bất kỳ kiểu nào được tạo thành hoàn toàn từ các kiểu `Send` cũng tự động được
đánh dấu là `Send`. Hầu hết tất cả các kiểu nguyên thủy triển khai `Send`, ngoại
trừ con trỏ thô, thứ mà chúng ta sẽ thảo luận trong Chương 20.

### Cho Phép Truy Cập từ Nhiều Thread với `Sync`

Đặc tính đánh dấu `Sync` chỉ ra rằng kiểu triển khai `Sync` an toàn để được tham
chiếu từ nhiều thread. Nói cách khác, bất kỳ kiểu `T` nào triển khai `Sync` nếu
`&T` (một tham chiếu không thay đổi đến `T`) triển khai `Send`, nghĩa là tham
chiếu có thể được gửi an toàn đến một thread khác. Tương tự như `Send`, tất cả
các kiểu nguyên thủy đều triển khai `Sync`, và các kiểu được tạo thành hoàn toàn
từ các kiểu triển khai `Sync` cũng triển khai `Sync`.

Con trỏ thông minh `Rc<T>` cũng không triển khai `Sync` vì cùng những lý do mà
nó không triển khai `Send`. Kiểu `RefCell<T>` (mà chúng ta đã nói đến trong
Chương 15) và họ các kiểu `Cell<T>` liên quan không triển khai `Sync`. Việc
triển khai kiểm tra mượn mà `RefCell<T>` thực hiện trong thời gian chạy không an
toàn cho thread. Con trỏ thông minh `Mutex<T>` triển khai `Sync` và có thể được
sử dụng để chia sẻ quyền truy cập với nhiều thread, như bạn đã thấy trong ["Chia
Sẻ `Mutex<T>` Giữa Nhiều
Thread"][sharing-a-mutext-between-multiple-threads]<!-- ignore -->.

### Triển Khai `Send` và `Sync` Thủ Công Là Không An Toàn

Vì các kiểu được tạo thành hoàn toàn từ các kiểu khác triển khai các đặc tính
`Send` và `Sync` cũng tự động triển khai `Send` và `Sync`, chúng ta không cần
phải triển khai những đặc tính này thủ công. Là các đặc tính đánh dấu, chúng
thậm chí không có bất kỳ phương thức nào để triển khai. Chúng chỉ hữu ích để
thực thi các bất biến liên quan đến tính đồng thời.

Việc triển khai thủ công các đặc tính này liên quan đến việc triển khai mã Rust
không an toàn. Chúng ta sẽ nói về việc sử dụng mã Rust không an toàn trong
Chương 20; hiện tại, thông tin quan trọng là việc xây dựng các kiểu đồng thời
mới không được tạo từ các phần `Send` và `Sync` đòi hỏi suy nghĩ cẩn thận để duy
trì các đảm bảo an toàn. ["The Rustonomicon"][nomicon] có thêm thông tin về các
đảm bảo này và cách để duy trì chúng.

## Tóm Tắt

Đây không phải là lần cuối cùng bạn thấy tính đồng thời trong cuốn sách này:
chương tiếp theo tập trung vào lập trình bất đồng bộ, và dự án trong Chương 21
sẽ sử dụng các khái niệm trong chương này trong một tình huống thực tế hơn so
với các ví dụ nhỏ được thảo luận ở đây.

Như đã đề cập trước đó, vì rất ít phần trong cách Rust xử lý tính đồng thời là
một phần của ngôn ngữ, nhiều giải pháp đồng thời được triển khai dưới dạng các
crate. Những crate này phát triển nhanh hơn so với thư viện chuẩn, vì vậy hãy
nhớ tìm kiếm online cho các crate hiện đại, tiên tiến để sử dụng trong các tình
huống đa luồng.

Thư viện chuẩn của Rust cung cấp các kênh để truyền tin nhắn và các kiểu con trỏ
thông minh, như `Mutex<T>` và `Arc<T>`, an toàn để sử dụng trong các ngữ cảnh
đồng thời. Hệ thống kiểu và bộ kiểm tra mượn đảm bảo rằng mã sử dụng các giải
pháp này sẽ không gặp phải các cuộc đua dữ liệu hoặc tham chiếu không hợp lệ.
Khi bạn đã biên dịch được mã của mình, bạn có thể yên tâm rằng nó sẽ chạy tốt
trên nhiều thread mà không gặp phải những lỗi khó theo dõi thường thấy trong các
ngôn ngữ khác. Lập trình đồng thời không còn là một khái niệm đáng sợ nữa: hãy
tiến tới và làm cho chương trình của bạn chạy đồng thời, không sợ hãi!

[sharing-a-mutext-between-multiple-threads]:
  ch16-03-shared-state.html#sharing-a-mutext-between-multiple-threads
[nomicon]: ../nomicon/index.html
