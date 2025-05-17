## Áp dụng Đồng thời với Async

<!-- Old headings. Do not remove or links may break. -->

<a id="concurrency-with-async"></a>

Trong phần này, chúng ta sẽ áp dụng async vào một số thách thức đồng thời giống
với những gì chúng ta đã giải quyết bằng thread trong chương 16. Vì chúng ta đã
thảo luận về nhiều ý tưởng chính ở đó, trong phần này chúng ta sẽ tập trung vào
những điểm khác biệt giữa thread và future.

Trong nhiều trường hợp, các API để làm việc với đồng thời sử dụng async rất
giống với những API dùng cho thread. Trong các trường hợp khác, chúng lại khá
khác biệt. Ngay cả khi các API _trông_ giống nhau giữa thread và async, chúng
thường có hành vi khác nhau—và gần như luôn có các đặc điểm hiệu suất khác nhau.

<!-- Old headings. Do not remove or links may break. -->

<a id="counting"></a>

### Tạo Task Mới với `spawn_task`

Thao tác đầu tiên chúng ta đã giải quyết trong [Tạo Thread Mới với
Spawn][thread-spawn]<!-- ignore --> là đếm trên hai thread riêng biệt. Hãy làm
điều tương tự bằng async. Crate `trpl` cung cấp một hàm `spawn_task` trông rất
giống với API `thread::spawn`, và một hàm `sleep` là phiên bản async của API
`thread::sleep`. Chúng ta có thể sử dụng chúng cùng nhau để triển khai ví dụ
đếm, như trong Listing 17-6.

<Listing number="17-6" caption="Tạo một task mới để in một thứ trong khi task chính in thứ khác" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-06/src/main.rs:all}}
```

</Listing>

Làm điểm bắt đầu, chúng ta thiết lập hàm `main` với `trpl::run` để hàm cấp cao
nhất của chúng ta có thể là async.

> Lưu ý: Từ điểm này trở đi trong chương, mọi ví dụ sẽ bao gồm mã bao bọc chính
> xác giống nhau với `trpl::run` trong `main`, vì vậy chúng ta thường bỏ qua nó
> giống như chúng ta làm với `main`. Đừng quên bao gồm nó trong mã của bạn!

Sau đó, chúng ta viết hai vòng lặp trong khối đó, mỗi vòng lặp chứa một lệnh gọi
`trpl::sleep`, đợi nửa giây (500 mili giây) trước khi gửi tin nhắn tiếp theo.
Chúng ta đặt một vòng lặp trong phần thân của `trpl::spawn_task` và vòng lặp
khác trong vòng lặp `for` cấp cao nhất. Chúng ta cũng thêm `await` sau các lệnh
gọi `sleep`.

Mã này hoạt động tương tự như triển khai dựa trên thread—bao gồm cả thực tế là
bạn có thể thấy các tin nhắn xuất hiện theo thứ tự khác nhau trong terminal của
bạn khi bạn chạy nó:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
```

Phiên bản này dừng ngay khi vòng lặp `for` trong phần thân của khối async chính
kết thúc, vì task được tạo bởi `spawn_task` bị tắt khi hàm `main` kết thúc. Nếu
bạn muốn nó chạy hết đến khi task hoàn thành, bạn sẽ cần sử dụng một join handle
để đợi task đầu tiên hoàn thành. Với thread, chúng ta đã sử dụng phương thức
`join` để "chặn" cho đến khi thread hoàn tất chạy. Trong Listing 17-7, chúng ta
có thể sử dụng `await` để làm điều tương tự, bởi vì task handle chính nó là một
future. Kiểu `Output` của nó là một `Result`, vì vậy chúng ta cũng unwrap nó sau
khi await nó.

<Listing number="17-7" caption="Sử dụng `await` với một join handle để chạy một task đến khi hoàn thành" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-07/src/main.rs:handle}}
```

</Listing>

Phiên bản cập nhật này chạy cho đến khi _cả hai_ vòng lặp kết thúc.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the second task!
hi number 1 from the first task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Cho đến nay, có vẻ như async và thread cho chúng ta cùng một kết quả cơ bản, chỉ
với cú pháp khác nhau: sử dụng `await` thay vì gọi `join` trên join handle, và
await các lệnh gọi `sleep`.

Sự khác biệt lớn hơn là chúng ta không cần phải tạo một thread hệ điều hành khác
để làm điều này. Thực tế, chúng ta thậm chí không cần phải tạo một task ở đây.
Bởi vì các khối async được biên dịch thành các future ẩn danh, chúng ta có thể
đặt mỗi vòng lặp trong một khối async và có runtime chạy cả hai đến khi hoàn
thành bằng cách sử dụng hàm `trpl::join`.

Trong phần [Chờ Tất cả Các Thread Hoàn thành Bằng Cách Sử dụng `join`
Handles][join-handles]<!-- ignore -->, chúng ta đã thấy cách sử dụng phương thức
`join` trên kiểu `JoinHandle` được trả về khi bạn gọi `std::thread::spawn`. Hàm
`trpl::join` tương tự, nhưng dành cho future. Khi bạn đưa cho nó hai future, nó
tạo ra một future mới duy nhất mà output của nó là một tuple chứa output của mỗi
future bạn đã truyền vào khi cả hai đều hoàn thành. Vì vậy, trong Listing 17-8,
chúng ta sử dụng `trpl::join` để đợi cả `fut1` và `fut2` hoàn thành. Chúng ta
_không_ await `fut1` và `fut2` mà thay vào đó là future mới được tạo ra bởi
`trpl::join`. Chúng ta bỏ qua output, vì nó chỉ là một tuple chứa hai giá trị
đơn vị.

<Listing number="17-8" caption="Sử dụng `trpl::join` để await hai future ẩn danh" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-08/src/main.rs:join}}
```

</Listing>

Khi chúng ta chạy điều này, chúng ta thấy cả hai future chạy đến khi hoàn thành:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the first task!
hi number 1 from the second task!
hi number 2 from the first task!
hi number 2 from the second task!
hi number 3 from the first task!
hi number 3 from the second task!
hi number 4 from the first task!
hi number 4 from the second task!
hi number 5 from the first task!
hi number 6 from the first task!
hi number 7 from the first task!
hi number 8 from the first task!
hi number 9 from the first task!
```

Bây giờ, bạn sẽ thấy thứ tự chính xác giống nhau mỗi lần, điều này rất khác với
những gì chúng ta đã thấy với thread. Đó là bởi vì hàm `trpl::join` là _công
bằng_, nghĩa là nó kiểm tra mỗi future thường xuyên như nhau, luân phiên giữa
chúng, và không bao giờ để một future chạy trước nếu future khác đã sẵn sàng.
Với thread, hệ điều hành quyết định thread nào được kiểm tra và cho phép nó chạy
bao lâu. Với async Rust, runtime quyết định task nào cần kiểm tra. (Trong thực
tế, chi tiết trở nên phức tạp vì một runtime async có thể sử dụng thread hệ điều
hành bên dưới như một phần của cách nó quản lý đồng thời, vì vậy việc đảm bảo
tính công bằng có thể là nhiều việc hơn cho một runtime—nhưng nó vẫn có thể thực
hiện được!) Các runtime không phải đảm bảo sự công bằng cho bất kỳ thao tác nào,
và chúng thường cung cấp các API khác nhau để cho phép bạn chọn liệu bạn có muốn
sự công bằng hay không.

Hãy thử một số biến thể này khi await các future và xem chúng làm gì:

- Xóa bỏ khối async xung quanh một trong hai hoặc cả hai vòng lặp.
- Await mỗi khối async ngay sau khi định nghĩa nó.
- Chỉ bọc vòng lặp đầu tiên trong một khối async, và await future kết quả sau
  phần thân của vòng lặp thứ hai.

Để thử thách hơn, hãy xem liệu bạn có thể tìm ra output sẽ như thế nào trong mỗi
trường hợp _trước_ khi chạy mã!

<!-- Old headings. Do not remove or links may break. -->

<a id="message-passing"></a>

### Đếm trên Hai Task Sử dụng Truyền Thông Điệp

Chia sẻ dữ liệu giữa các future cũng sẽ quen thuộc: chúng ta sẽ sử dụng truyền
thông điệp một lần nữa, nhưng lần này với các phiên bản async của các kiểu và
hàm. Chúng ta sẽ đi theo một con đường hơi khác với những gì chúng ta đã làm
trong [Sử dụng Truyền Thông Điệp để Chuyển Dữ liệu Giữa Các
Thread][message-passing-threads]<!-- ignore --> để minh họa một số điểm khác
biệt chính giữa đồng thời dựa trên thread và dựa trên future. Trong Listing
17-9, chúng ta sẽ bắt đầu với chỉ một khối async duy nhất—_không_ tạo một task
riêng biệt như chúng ta đã tạo một thread riêng biệt.

<Listing number="17-9" caption="Tạo một kênh async và gán hai nửa cho `tx` và `rx`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-09/src/main.rs:channel}}
```

</Listing>

Ở đây, chúng ta sử dụng `trpl::channel`, một phiên bản async của API kênh đa-nhà
sản xuất, đơn-người tiêu dùng mà chúng ta đã sử dụng với thread trong Chương 16.
Phiên bản async của API chỉ hơi khác so với phiên bản dựa trên thread: nó sử
dụng một receiver `rx` có thể thay đổi thay vì bất biến, và phương thức `recv`
của nó tạo ra một future mà chúng ta cần await thay vì trực tiếp tạo ra giá trị.
Bây giờ chúng ta có thể gửi tin nhắn từ sender đến receiver. Lưu ý rằng chúng ta
không phải tạo một thread riêng biệt hoặc thậm chí một task; chúng ta chỉ cần
await lệnh gọi `rx.recv`.

Phương thức đồng bộ `Receiver::recv` trong `std::mpsc::channel` chặn cho đến khi
nó nhận được tin nhắn. Phương thức `trpl::Receiver::recv` không làm vậy, vì nó
là async. Thay vì chặn, nó giao quyền kiểm soát lại cho runtime cho đến khi hoặc
một tin nhắn được nhận hoặc phía gửi của kênh đóng lại. Ngược lại, chúng ta
không await lệnh gọi `send`, vì nó không chặn. Nó không cần phải làm vậy, vì
kênh mà chúng ta đang gửi vào là không giới hạn.

> Lưu ý: Vì tất cả mã async này chạy trong một khối async trong một lệnh gọi
> `trpl::run`, mọi thứ trong đó có thể tránh việc chặn. Tuy nhiên, mã _bên
> ngoài_ nó sẽ chặn khi hàm `run` trả về. Đó là toàn bộ mục đích của hàm
> `trpl::run`: nó cho phép bạn _chọn_ nơi để chặn đối với một số mã async, và do
> đó là nơi để chuyển đổi giữa mã đồng bộ và async. Trong hầu hết các runtime
> async, `run` thực sự được đặt tên là `block_on` chính vì lý do này.

Lưu ý hai điều về ví dụ này. Đầu tiên, tin nhắn sẽ đến ngay lập tức. Thứ hai,
mặc dù chúng ta sử dụng một future ở đây, nhưng chưa có sự đồng thời. Mọi thứ
trong listing xảy ra tuần tự, giống như khi không có future nào tham gia.

Hãy giải quyết phần đầu tiên bằng cách gửi một loạt tin nhắn và ngủ giữa chúng,
như trong Listing 17-10.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-10" caption="Gửi và nhận nhiều tin nhắn qua kênh async và ngủ với `await` giữa mỗi tin nhắn" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-10/src/main.rs:many-messages}}
```

</Listing>

Ngoài việc gửi tin nhắn, chúng ta cần phải nhận chúng. Trong trường hợp này, vì
chúng ta biết có bao nhiêu tin nhắn đến, chúng ta có thể làm điều đó thủ công
bằng cách gọi `rx.recv().await` bốn lần. Tuy nhiên, trong thế giới thực, chúng
ta thường đang đợi một số lượng tin nhắn _không xác định_, vì vậy chúng ta cần
tiếp tục đợi cho đến khi chúng ta xác định rằng không còn tin nhắn nào nữa.

Trong Listing 16-10, chúng ta đã sử dụng một vòng lặp `for` để xử lý tất cả các
mục nhận được từ một kênh đồng bộ. Tuy nhiên, Rust chưa có cách để viết một vòng
lặp `for` trên một loạt các mục _bất đồng bộ_, vì vậy chúng ta cần sử dụng một
vòng lặp mà chúng ta chưa thấy trước đây: vòng lặp điều kiện `while let`. Đây là
phiên bản vòng lặp của cấu trúc `if let` mà chúng ta đã thấy trong phần [Kiểm
soát Luồng Ngắn gọn với `if let` và `let...else`][if-let]<!-- ignore -->. Vòng lặp
sẽ tiếp tục thực thi miễn là mẫu mà nó chỉ định tiếp tục khớp với giá trị.

Lệnh gọi `rx.recv` tạo ra một future, mà chúng ta await. Runtime sẽ tạm dừng
future cho đến khi nó sẵn sàng. Khi một tin nhắn đến, future sẽ giải quyết thành
`Some(message)` nhiều lần khi tin nhắn đến. Khi kênh đóng lại, bất kể _bất kỳ_
tin nhắn nào đã đến, future sẽ thay vào đó giải quyết thành `None` để chỉ ra
rằng không còn giá trị nào nữa và do đó chúng ta nên dừng polling—nghĩa là, dừng
awaiting.

Vòng lặp `while let` kết hợp tất cả những điều này. Nếu kết quả của việc gọi
`rx.recv().await` là `Some(message)`, chúng ta có quyền truy cập vào tin nhắn và
chúng ta có thể sử dụng nó trong phần thân vòng lặp, giống như chúng ta có thể
làm với `if let`. Nếu kết quả là `None`, vòng lặp kết thúc. Mỗi khi vòng lặp
hoàn thành, nó chạm vào điểm await một lần nữa, vì vậy runtime tạm dừng nó một
lần nữa cho đến khi một tin nhắn khác đến.

Mã bây giờ thành công gửi và nhận tất cả các tin nhắn. Đáng tiếc, vẫn còn một
vài vấn đề. Một là, các tin nhắn không đến ở các khoảng thời gian nửa giây.
Chúng đến cùng một lúc, 2 giây (2.000 mili giây) sau khi chúng ta bắt đầu chương
trình. Vấn đề khác là chương trình này không bao giờ thoát! Thay vào đó, nó đợi
mãi mãi cho các tin nhắn mới. Bạn sẽ cần phải tắt nó bằng
<span class="keystroke">ctrl-c</span>.

Hãy bắt đầu bằng cách kiểm tra lý do tại sao các tin nhắn đến cùng một lúc sau
khi trì hoãn đầy đủ, thay vì đến với độ trễ giữa mỗi tin. Trong một khối async
nhất định, thứ tự mà các từ khóa `await` xuất hiện trong mã cũng là thứ tự mà
chúng được thực thi khi chương trình chạy.

Chỉ có một khối async trong Listing 17-10, vì vậy mọi thứ trong đó chạy tuyến
tính. Vẫn chưa có sự đồng thời. Tất cả các lệnh gọi `tx.send` xảy ra, xen kẽ với
tất cả các lệnh gọi `trpl::sleep` và các điểm await liên quan. Chỉ sau đó vòng
lặp `while let` mới đi qua bất kỳ điểm `await` nào trong các lệnh gọi `recv`.

Để có được hành vi mà chúng ta muốn, nơi độ trễ sleep xảy ra giữa mỗi tin nhắn,
chúng ta cần đặt các thao tác `tx` và `rx` trong các khối async riêng của chúng,
như trong Listing 17-11. Sau đó, runtime có thể thực thi từng khối riêng biệt
bằng cách sử dụng `trpl::join`, giống như trong ví dụ đếm. Một lần nữa, chúng ta
await kết quả của việc gọi `trpl::join`, không phải các future riêng lẻ. Nếu
chúng ta await các future riêng lẻ một cách tuần tự, chúng ta sẽ chỉ quay lại
một luồng tuần tự—chính xác những gì chúng ta đang cố gắng _không_ làm.

<!-- We cannot test this one because it never stops! -->

<Listing number="17-11" caption="Tách `send` và `recv` vào các khối `async` riêng của chúng và await các future cho các khối đó" file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch17-async-await/listing-17-11/src/main.rs:futures}}
```

</Listing>

Với mã đã cập nhật trong Listing 17-11, các tin nhắn được in ở khoảng thời gian
500-mili giây, thay vì tất cả cùng một lúc sau 2 giây.

Tuy nhiên, chương trình vẫn không bao giờ thoát, vì cách mà vòng lặp `while let`
tương tác với `trpl::join`:

- Future được trả về từ `trpl::join` chỉ hoàn thành khi _cả hai_ future được
  truyền vào nó đã hoàn thành.
- Future `tx` hoàn thành khi nó hoàn tất việc ngủ sau khi gửi tin nhắn cuối cùng
  trong `vals`.
- Future `rx` sẽ không hoàn thành cho đến khi vòng lặp `while let` kết thúc.
- Vòng lặp `while let` sẽ không kết thúc cho đến khi await `rx.recv` tạo ra
  `None`.
- Await `rx.recv` sẽ chỉ trả về `None` khi đầu kia của kênh được đóng lại.
- Kênh sẽ chỉ đóng khi chúng ta gọi `rx.close` hoặc khi phía gửi, `tx`, bị drop.
- Chúng ta không gọi `rx.close` ở bất kỳ đâu, và `tx` sẽ không bị drop cho đến
  khi khối async ngoài cùng được truyền vào `trpl::run` kết thúc.
- Khối không thể kết thúc vì nó bị chặn trên `trpl::join` đang hoàn thành, điều
  này đưa chúng ta trở lại đầu danh sách này.

Chúng ta có thể đóng `rx` thủ công bằng cách gọi `rx.close` ở đâu đó, nhưng điều
đó không có nhiều ý nghĩa. Việc dừng lại sau khi xử lý một số lượng tin nhắn tùy
ý sẽ làm cho chương trình tắt, nhưng chúng ta có thể bỏ lỡ các tin nhắn. Chúng
ta cần một cách khác để đảm bảo rằng `tx` bị drop _trước_ khi kết thúc hàm.

Hiện tại, khối async nơi chúng ta gửi tin nhắn chỉ mượn `tx` vì gửi tin nhắn
không yêu cầu quyền sở hữu, nhưng nếu chúng ta có thể di chuyển `tx` vào khối
async đó, nó sẽ bị drop khi khối đó kết thúc. Trong phần Chương 13 [Nắm bắt Tham
chiếu hoặc Di chuyển Quyền sở hữu][capture-or-move]<!-- ignore -->, bạn đã học
cách sử dụng từ khóa `move` với closure, và, như đã thảo luận trong phần Chương
16 [Sử dụng Closure `move` với Thread][move-threads]<!-- ignore
-->, chúng ta thường cần di chuyển dữ liệu vào closure khi làm việc với thread. Cùng
động lực cơ bản áp dụng cho các khối async, vì vậy từ khóa `move` hoạt động với các
khối async giống như với closure.

Trong Listing 17-12, chúng ta thay đổi khối được sử dụng để gửi tin nhắn từ
`async` thành `async move`. Khi chúng ta chạy _phiên bản_ mã này, nó tắt một
cách thanh lịch sau khi tin nhắn cuối cùng được gửi và nhận.

<Listing number="17-12" caption="Bản sửa đổi của mã từ Listing 17-11 sẽ tắt đúng cách khi hoàn thành" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-12/src/main.rs:with-move}}
```

</Listing>

Kênh async này cũng là một kênh đa-nhà sản xuất, vì vậy chúng ta có thể gọi
`clone` trên `tx` nếu chúng ta muốn gửi tin nhắn từ nhiều future, như trong
Listing 17-13.

<Listing number="17-13" caption="Sử dụng nhiều nhà sản xuất với các khối async" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-13/src/main.rs:here}}
```

</Listing>

Đầu tiên, chúng ta sao chép `tx`, tạo ra `tx1` bên ngoài khối async đầu tiên.
Chúng ta di chuyển `tx1` vào khối đó giống như chúng ta đã làm trước đây với
`tx`. Sau đó, sau này, chúng ta di chuyển `tx` ban đầu vào một khối async _mới_,
nơi chúng ta gửi thêm tin nhắn với độ trễ hơi chậm hơn. Chúng ta tình cờ đặt
khối async mới này sau khối async để nhận tin nhắn, nhưng nó cũng có thể đi
trước nó. Chìa khóa là thứ tự mà các future được await, không phải thứ tự chúng
được tạo ra.

Cả hai khối async để gửi tin nhắn cần phải là các khối `async move` để cả `tx`
và `tx1` đều bị drop khi các khối đó kết thúc. Nếu không, chúng ta sẽ quay trở
lại vòng lặp vô hạn mà chúng ta đã bắt đầu. Cuối cùng, chúng ta chuyển từ
`trpl::join` sang `trpl::join3` để xử lý future bổ sung.

Bây giờ chúng ta thấy tất cả các tin nhắn từ cả hai future gửi, và vì các future
gửi sử dụng các độ trễ hơi khác nhau sau khi gửi, các tin nhắn cũng được nhận ở
những khoảng thời gian khác nhau đó.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'the'
received 'messages'
received 'future'
received 'for'
received 'you'
```

Đây là một khởi đầu tốt, nhưng nó giới hạn chúng ta ở chỉ một vài future: hai
với `join`, hoặc ba với `join3`. Hãy xem chúng ta có thể làm việc với nhiều
future hơn như thế nào.

[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[join-handles]:
  ch16-01-threads.html#waiting-for-all-threads-to-finish-using-join-handles
[message-passing-threads]: ch16-02-message-passing.html
[if-let]: ch06-03-if-let.html
[capture-or-move]:
  ch13-01-closures.html#capturing-references-or-moving-ownership
[move-threads]: ch16-01-threads.html#using-move-closures-with-threads
