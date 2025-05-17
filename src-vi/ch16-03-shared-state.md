## Đồng Thời với Trạng Thái Được Chia Sẻ

Truyền tin nhắn là một cách tốt để xử lý tính đồng thời, nhưng không phải là
cách duy nhất. Một phương pháp khác là cho phép nhiều thread truy cập cùng một
dữ liệu được chia sẻ. Hãy xem xét phần này từ khẩu hiệu trong tài liệu ngôn ngữ
Go một lần nữa: "Đừng giao tiếp bằng cách chia sẻ bộ nhớ."

Giao tiếp bằng cách chia sẻ bộ nhớ sẽ trông như thế nào? Ngoài ra, tại sao những
người đam mê phương pháp truyền tin nhắn lại khuyên không nên sử dụng việc chia
sẻ bộ nhớ?

Theo một cách nào đó, các kênh trong bất kỳ ngôn ngữ lập trình nào cũng tương tự
như quyền sở hữu đơn bởi vì một khi bạn truyền một giá trị xuống kênh, bạn
không nên sử dụng giá trị đó nữa. Đồng thời với bộ nhớ được chia sẻ giống như
quyền sở hữu đa: nhiều thread có thể truy cập cùng một vị trí bộ nhớ tại cùng
một thời điểm. Như bạn đã thấy trong Chương 15, nơi các con trỏ thông minh làm
cho quyền sở hữu đa trở nên khả thi, quyền sở hữu đa có thể thêm độ phức tạp vì
các chủ sở hữu khác nhau này cần được quản lý. Hệ thống kiểu và quy tắc sở hữu
của Rust hỗ trợ rất nhiều trong việc quản lý này một cách chính xác. Ví dụ, hãy
xem xét mutex, một trong những nguyên thủy đồng thời phổ biến nhất cho bộ nhớ
được chia sẻ.

### Sử Dụng Mutex để Cho Phép Truy Cập Dữ Liệu từ Một Thread tại Một Thời Điểm

_Mutex_ là viết tắt của _mutual exclusion_ (loại trừ tương hỗ), nghĩa là một
mutex chỉ cho phép một thread truy cập một số dữ liệu tại bất kỳ thời điểm nào.
Để truy cập dữ liệu trong một mutex, một thread trước tiên phải báo hiệu rằng nó
muốn truy cập bằng cách yêu cầu có được khóa của mutex. _Khóa_ là một cấu trúc
dữ liệu là một phần của mutex, giúp theo dõi ai hiện đang có quyền truy cập độc
quyền vào dữ liệu. Do đó, mutex được mô tả là _bảo vệ_ dữ liệu mà nó chứa thông
qua hệ thống khóa.

Mutex có tiếng là khó sử dụng vì bạn phải nhớ hai quy tắc:

1. Bạn phải cố gắng có được khóa trước khi sử dụng dữ liệu.
2. Khi bạn đã hoàn thành việc sử dụng dữ liệu mà mutex bảo vệ, bạn phải mở khóa
   dữ liệu để các thread khác có thể có được khóa.

Để có một phép ẩn dụ thực tế cho mutex, hãy tưởng tượng một cuộc thảo luận tại
một hội nghị với chỉ một micro. Trước khi một người tham gia có thể nói chuyện,
họ phải yêu cầu hoặc báo hiệu rằng họ muốn sử dụng micro. Khi họ nhận được
micro, họ có thể nói chuyện trong bao lâu tùy thích và sau đó chuyển micro cho
người tham gia tiếp theo muốn phát biểu. Nếu một người tham gia quên chuyển
micro khi họ đã nói xong, không ai khác có thể nói chuyện. Nếu việc quản lý
micro được chia sẻ bị sai, cuộc thảo luận sẽ không diễn ra như dự kiến!

Việc quản lý mutex có thể rất khó để làm đúng, đó là lý do tại sao rất nhiều
người đam mê sử dụng kênh. Tuy nhiên, nhờ có hệ thống kiểu và quy tắc sở hữu của
Rust, bạn không thể khóa và mở khóa sai.

#### API của `Mutex<T>`

Để làm ví dụ về cách sử dụng mutex, hãy bắt đầu bằng cách sử dụng mutex trong
một ngữ cảnh đơn thread, như trong Listing 16-12.

<Listing number="16-12" file-name="src/main.rs" caption="Khám phá API của `Mutex<T>` trong ngữ cảnh đơn thread để đơn giản hóa">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-12/src/main.rs}}
```

</Listing>

Giống như nhiều kiểu, chúng ta tạo một `Mutex<T>` bằng cách sử dụng hàm liên kết
`new`. Để truy cập dữ liệu bên trong mutex, chúng ta sử dụng phương thức `lock`
để có được khóa. Lệnh gọi này sẽ chặn thread hiện tại nên nó không thể làm bất
kỳ công việc nào cho đến khi đến lượt chúng ta có khóa.

Lệnh gọi đến `lock` sẽ thất bại nếu một thread khác đang giữ khóa bị panic.
Trong trường hợp đó, không ai có thể có được khóa, vì vậy chúng ta đã chọn
`unwrap` và có thread này panic nếu chúng ta ở trong tình huống đó.

Sau khi chúng ta đã có được khóa, chúng ta có thể coi giá trị trả về, có tên là
`num` trong trường hợp này, như một tham chiếu có thể thay đổi đến dữ liệu bên
trong. Hệ thống kiểu đảm bảo rằng chúng ta có được khóa trước khi sử dụng giá
trị trong `m`. Kiểu của `m` là `Mutex<i32>`, không phải `i32`, vì vậy chúng ta
_phải_ gọi `lock` để có thể sử dụng giá trị `i32`. Chúng ta không thể quên; hệ
thống kiểu sẽ không cho phép chúng ta truy cập giá trị `i32` bên trong nếu
không.

Lệnh gọi đến `lock` trả về một kiểu được gọi là `MutexGuard`,
được bọc trong một `LockResult` mà chúng ta đã xử lý bằng cách gọi `unwrap`. Kiểu `MutexGuard`
thực hiện `Deref` để trỏ đến dữ liệu bên trong của
chúng ta; kiểu này cũng có một triển khai `Drop` giải phóng khóa một
cách tự động khi `MutexGuard` ra khỏi phạm vi, điều này xảy ra vào cuối phạm vi
bên trong. Kết quả là, chúng ta không có nguy cơ quên giải phóng khóa và chặn
mutex không được sử dụng bởi các thread khác vì việc giải phóng khóa xảy ra tự
động.

Sau khi thả khóa, chúng ta có thể in giá trị mutex và thấy rằng chúng ta đã có
thể thay đổi giá trị `i32` bên trong thành `6`.

#### Chia Sẻ `Mutex<T>` Giữa Nhiều Thread

Bây giờ hãy thử chia sẻ một giá trị giữa nhiều thread bằng cách sử dụng
`Mutex<T>`. Chúng ta sẽ tạo 10 thread và có mỗi thread tăng giá trị bộ đếm lên
1, vì vậy bộ đếm sẽ tăng từ 0 lên 10. Ví dụ trong Listing 16-13 sẽ có lỗi trình
biên dịch, và chúng ta sẽ sử dụng lỗi đó để tìm hiểu thêm về cách sử dụng
`Mutex<T>` và cách Rust giúp chúng ta sử dụng nó đúng cách.

<Listing number="16-13" file-name="src/main.rs" caption="Mười thread, mỗi thread tăng một bộ đếm được bảo vệ bởi `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-13/src/main.rs}}
```

</Listing>

Chúng ta tạo một biến `counter` để chứa một `i32` bên trong một `Mutex<T>`, như
chúng ta đã làm trong Listing 16-12. Tiếp theo, chúng ta tạo 10 thread bằng cách
lặp qua một dãy số. Chúng ta sử dụng `thread::spawn` và cung cấp cho tất cả các
thread cùng một closure: closure di chuyển bộ đếm vào thread, lấy khóa trên
`Mutex<T>` bằng cách gọi phương thức `lock`, và sau đó cộng 1 vào giá trị trong
mutex. Khi một thread hoàn thành việc chạy closure của nó, `num` sẽ ra khỏi phạm
vi và giải phóng khóa để thread khác có thể lấy khóa.

Trong thread chính, chúng ta thu thập tất cả các handle join. Sau đó, như chúng
ta đã làm trong Listing 16-2, chúng ta gọi `join` trên mỗi handle để đảm bảo tất
cả các thread đều hoàn thành. Tại thời điểm đó, thread chính sẽ có được khóa và
in kết quả của chương trình này.

Chúng ta đã gợi ý rằng ví dụ này sẽ không biên dịch. Bây giờ hãy tìm hiểu tại
sao!

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-13/output.txt}}
```

Thông báo lỗi nói rằng giá trị `counter` đã được di chuyển trong lần lặp trước
của vòng lặp. Rust đang nói với chúng ta rằng chúng ta không thể di chuyển quyền
sở hữu của `counter` vào nhiều thread. Hãy sửa lỗi trình biên dịch bằng
phương pháp sở hữu đa mà chúng ta đã thảo luận trong Chương 15.

#### Sở Hữu Đa với Nhiều Thread

Trong Chương 15, chúng ta đã cung cấp một giá trị cho nhiều chủ sở hữu bằng cách sử
dụng con trỏ thông minh `Rc<T>` để tạo một giá trị được đếm tham chiếu. Hãy làm
tương tự ở đây và xem điều gì xảy ra. Chúng ta sẽ bọc `Mutex<T>` trong `Rc<T>`
trong Listing 16-14 và sao chép `Rc<T>` trước khi di chuyển quyền sở hữu vào
thread.

<Listing number="16-14" file-name="src/main.rs" caption="Cố gắng sử dụng `Rc<T>` để cho phép nhiều thread sở hữu `Mutex<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-14/src/main.rs}}
```

</Listing>

Một lần nữa, chúng ta biên dịch và nhận được... các lỗi khác nhau! Trình biên
dịch đang dạy chúng ta rất nhiều.

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-14/output.txt}}
```

Wow, thông báo lỗi đó rất dài dòng! Đây là phần quan trọng cần tập trung:
`` `Rc<Mutex<i32>>` không thể được gửi giữa các thread một cách an toàn ``.
Trình biên dịch cũng đang nói với chúng ta lý do tại sao:
`` đặc tính `Send` không được triển khai cho `Rc<Mutex<i32>>` ``. Chúng ta sẽ
nói về `Send` trong phần tiếp theo: đó là một trong những đặc tính đảm bảo rằng
các kiểu mà chúng ta sử dụng với thread được thiết kế để sử dụng trong các tình
huống đồng thời.

Thật không may, `Rc<T>` không an toàn để chia sẻ giữa các thread. Khi `Rc<T>`
quản lý số lượng tham chiếu, nó cộng vào số lượng cho mỗi lệnh gọi `clone` và
trừ từ số lượng khi mỗi bản sao được loại bỏ. Nhưng nó không sử dụng bất kỳ
nguyên thủy đồng thời nào để đảm bảo rằng các thay đổi đối với số lượng không
thể bị gián đoạn bởi một thread khác. Điều này có thể dẫn đến số lượng sai - các
lỗi tinh vi có thể dẫn đến rò rỉ bộ nhớ hoặc một giá trị bị loại bỏ trước khi
chúng ta hoàn thành với nó. Những gì chúng ta cần là một kiểu giống hệt như
`Rc<T>`, nhưng thực hiện thay đổi số lượng tham chiếu theo cách an toàn với
thread.

#### Đếm Tham Chiếu Nguyên Tử với `Arc<T>`

May mắn thay, `Arc<T>` _là_ một kiểu giống như `Rc<T>` mà an toàn để sử dụng
trong các tình huống đồng thời. Chữ _a_ là viết tắt của _atomic_ (nguyên tử), có
nghĩa là nó là một kiểu _đếm tham chiếu nguyên tử_. Nguyên tử là một loại nguyên
thủy đồng thời bổ sung mà chúng ta sẽ không đề cập chi tiết ở đây: xem tài liệu
thư viện chuẩn cho [`std::sync::atomic`][atomic]<!-- ignore --> để biết thêm chi
tiết. Tại thời điểm này, bạn chỉ cần biết rằng nguyên tử hoạt động giống như các
kiểu nguyên thủy nhưng an toàn để chia sẻ giữa các thread.

Bạn có thể thắc mắc tại sao tất cả các kiểu nguyên thủy không phải là nguyên tử
và tại sao các kiểu thư viện chuẩn không được triển khai để sử dụng `Arc<T>`
theo mặc định. Lý do là vì an toàn thread đi kèm với một mức phạt hiệu suất mà
bạn chỉ muốn trả khi thực sự cần. Nếu bạn chỉ thực hiện các thao tác trên các
giá trị trong một thread duy nhất, mã của bạn có thể chạy nhanh hơn nếu nó không
phải thực thi các bảo đảm mà nguyên tử cung cấp.

Hãy quay lại ví dụ của chúng ta: `Arc<T>` và `Rc<T>` có cùng một API, vì vậy
chúng ta sửa chương trình của mình bằng cách thay đổi dòng `use`, lệnh gọi đến
`new`, và lệnh gọi đến `clone`. Mã trong Listing 16-15 cuối cùng sẽ biên dịch và
chạy.

<Listing number="16-15" file-name="src/main.rs" caption="Sử dụng `Arc<T>` để bọc `Mutex<T>` để có thể chia sẻ quyền sở hữu giữa nhiều thread">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-15/src/main.rs}}
```

</Listing>

Mã này sẽ in ra như sau:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Result: 10
```

Chúng ta đã làm được! Chúng ta đã đếm từ 0 đến 10, điều này có vẻ không quá ấn
tượng, nhưng nó đã dạy chúng ta rất nhiều về `Mutex<T>` và an toàn thread. Bạn
cũng có thể sử dụng cấu trúc chương trình này để thực hiện các thao tác phức tạp
hơn chỉ là tăng bộ đếm. Sử dụng chiến lược này, bạn có thể chia một phép tính
thành các phần độc lập, chia các phần đó giữa các thread, và sau đó sử dụng
`Mutex<T>` để có mỗi thread cập nhật kết quả cuối cùng với phần của nó.

Lưu ý rằng nếu bạn đang thực hiện các phép toán số đơn giản, có các kiểu đơn
giản hơn `Mutex<T>` được cung cấp bởi [mô-đun `std::sync::atomic` của thư viện
chuẩn][atomic]<!-- ignore -->. Các kiểu này cung cấp truy cập an toàn, đồng
thời, nguyên tử vào các kiểu nguyên thủy. Chúng ta đã chọn sử dụng `Mutex<T>`
với một kiểu nguyên thủy cho ví dụ này để chúng ta có thể tập trung vào cách
`Mutex<T>` hoạt động.

### Sự Tương Đồng Giữa `RefCell<T>`/`Rc<T>` và `Mutex<T>`/`Arc<T>`

Bạn có thể đã nhận thấy rằng `counter` không thể thay đổi nhưng chúng ta có thể
nhận được một tham chiếu có thể thay đổi đến giá trị bên trong nó; điều này có
nghĩa là `Mutex<T>` cung cấp khả năng thay đổi nội bộ, giống như họ `Cell` làm.
Theo cách tương tự, chúng ta đã sử dụng `RefCell<T>` trong Chương 15 để cho phép
chúng ta thay đổi nội dung bên trong `Rc<T>`, chúng ta sử dụng `Mutex<T>` để
thay đổi nội dung bên trong `Arc<T>`.

Một chi tiết khác cần lưu ý là Rust không thể bảo vệ bạn khỏi tất cả các loại
lỗi logic khi bạn sử dụng `Mutex<T>`. Nhớ lại từ Chương 15 rằng việc sử dụng
`Rc<T>` đi kèm với nguy cơ tạo ra các chu kỳ tham chiếu, trong đó hai giá trị
`Rc<T>` tham chiếu đến nhau, gây ra rò rỉ bộ nhớ. Tương tự, `Mutex<T>` đi kèm
với nguy cơ tạo ra _deadlocks_ (bế tắc). Những điều này xảy ra khi một thao tác
cần khóa hai tài nguyên và hai thread mỗi cái đã có được một trong các khóa,
khiến chúng đợi nhau mãi mãi. Nếu bạn quan tâm đến deadlocks, hãy thử tạo một
chương trình Rust có deadlock; sau đó nghiên cứu các chiến lược giảm thiểu
deadlock cho mutex trong bất kỳ ngôn ngữ nào và thử triển khai chúng trong Rust.
Tài liệu API của thư viện chuẩn cho `Mutex<T>` và `MutexGuard` cung cấp thông
tin hữu ích.

Chúng ta sẽ kết thúc chương này bằng cách nói về các đặc tính `Send` và `Sync`
và cách chúng ta có thể sử dụng chúng với các kiểu tùy chỉnh.

[atomic]: ../std/sync/atomic/index.html
