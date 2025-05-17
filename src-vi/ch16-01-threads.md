## Sử Dụng Luồng để Chạy Mã Đồng Thời

Trong hầu hết các hệ điều hành hiện đại, mã của một chương trình đã thực thi
được chạy trong một _quy trình_ (process), và hệ điều hành sẽ quản lý nhiều quy
trình cùng một lúc. Trong một chương trình, bạn cũng có thể có các phần độc lập
chạy đồng thời. Các tính năng chạy các phần độc lập này được gọi là _luồng_
(threads). Ví dụ, một máy chủ web có thể có nhiều luồng để nó có thể phản hồi
nhiều hơn một yêu cầu cùng một lúc.

Việc chia nhỏ tính toán trong chương trình của bạn thành nhiều luồng để chạy
nhiều tác vụ cùng một lúc có thể cải thiện hiệu suất, nhưng nó cũng làm tăng độ
phức tạp. Bởi vì các luồng có thể chạy đồng thời, không có đảm bảo cố hữu về thứ
tự mà các phần mã của bạn trên các luồng khác nhau sẽ chạy. Điều này có thể dẫn
đến các vấn đề, chẳng hạn như:

- Điều kiện tranh đua (Race conditions), trong đó các luồng đang truy cập dữ
  liệu hoặc tài nguyên theo thứ tự không nhất quán
- Bế tắc (Deadlocks), trong đó hai luồng đang chờ đợi nhau, ngăn cản cả hai
  luồng tiếp tục
- Lỗi chỉ xảy ra trong một số tình huống nhất định và khó tái tạo và sửa chữa
  một cách đáng tin cậy

Rust cố gắng giảm thiểu các tác động tiêu cực của việc sử dụng luồng, nhưng lập
trình trong bối cảnh đa luồng vẫn cần suy nghĩ cẩn thận và yêu cầu cấu trúc mã
khác với cấu trúc trong các chương trình chạy trong một luồng duy nhất.

Các ngôn ngữ lập trình triển khai luồng theo một vài cách khác nhau, và nhiều hệ
điều hành cung cấp một API mà ngôn ngữ lập trình có thể gọi để tạo luồng mới.
Thư viện chuẩn của Rust sử dụng mô hình _1:1_ cho việc triển khai luồng, theo đó
một chương trình sử dụng một luồng hệ điều hành cho mỗi một luồng ngôn ngữ. Có
các crate triển khai các mô hình luồng khác có sự đánh đổi khác nhau so với mô
hình 1:1. (Hệ thống async của Rust, mà chúng ta sẽ thấy trong chương tiếp theo,
cung cấp một cách tiếp cận khác cho đồng thời.)

### Tạo Luồng Mới với `spawn`

Để tạo một luồng mới, chúng ta gọi hàm `thread::spawn` và truyền cho nó một
closure (chúng ta đã nói về closure trong Chương 13) chứa mã mà chúng ta muốn
chạy trong luồng mới. Ví dụ trong Listing 16-1 in một số văn bản từ luồng chính
và văn bản khác từ một luồng mới.

<Listing number="16-1" file-name="src/main.rs" caption="Tạo một luồng mới để in một thứ trong khi luồng chính in một thứ khác">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-01/src/main.rs}}
```

</Listing>

Lưu ý rằng khi luồng chính của một chương trình Rust hoàn thành, tất cả các
luồng được tạo ra đều bị tắt, cho dù chúng đã hoàn thành chạy hay chưa. Đầu ra
từ chương trình này có thể hơi khác nhau mỗi lần, nhưng nó sẽ trông giống như
sau:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 1 from the spawned thread!
hi number 2 from the main thread!
hi number 2 from the spawned thread!
hi number 3 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the main thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
```

Các lệnh gọi đến `thread::sleep` buộc một luồng dừng thực thi trong một khoảng
thời gian ngắn, cho phép một luồng khác chạy. Các luồng có thể sẽ lần lượt thay
phiên nhau, nhưng điều đó không được đảm bảo: nó phụ thuộc vào cách hệ điều hành
của bạn lên lịch các luồng. Trong lần chạy này, luồng chính in trước, mặc dù câu
lệnh in từ luồng được tạo ra xuất hiện đầu tiên trong mã. Và mặc dù chúng ta bảo
luồng được tạo ra in cho đến khi `i` là `9`, nó chỉ đạt đến `5` trước khi luồng
chính tắt.

Nếu bạn chạy mã này và chỉ thấy đầu ra từ luồng chính, hoặc không thấy bất kỳ sự
chồng chéo nào, hãy thử tăng số lượng trong các phạm vi để tạo thêm cơ hội cho
hệ điều hành chuyển đổi giữa các luồng.

### Chờ Tất Cả Các Luồng Hoàn Thành Bằng Cách Sử Dụng `join` Handles

Mã trong Listing 16-1 không chỉ dừng luồng được tạo ra sớm hơn dự kiến trong hầu
hết thời gian do luồng chính kết thúc, mà còn vì không có đảm bảo về thứ tự chạy
của các luồng, chúng ta cũng không thể đảm bảo rằng luồng được tạo ra sẽ có cơ
hội chạy!

Chúng ta có thể khắc phục vấn đề luồng được tạo ra không chạy hoặc việc nó kết
thúc sớm bằng cách lưu giá trị trả về của `thread::spawn` trong một biến. Kiểu
trả về của `thread::spawn` là `JoinHandle<T>`. Một `JoinHandle<T>` là một giá
trị được sở hữu mà, khi chúng ta gọi phương thức `join` trên nó, sẽ chờ cho
luồng của nó kết thúc. Listing 16-2 hiển thị cách sử dụng `JoinHandle<T>` của
luồng mà chúng ta đã tạo trong Listing 16-1 và cách gọi `join` để đảm bảo luồng
được tạo ra hoàn thành trước khi `main` thoát.

<Listing number="16-2" file-name="src/main.rs" caption="Lưu một `JoinHandle<T>` từ `thread::spawn` để đảm bảo luồng được chạy đến khi hoàn thành">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-02/src/main.rs}}
```

</Listing>

Gọi `join` trên handle sẽ chặn luồng hiện đang chạy cho đến khi luồng được đại
diện bởi handle kết thúc. _Chặn_ (Blocking) một luồng có nghĩa là luồng đó bị
ngăn không cho thực hiện công việc hoặc thoát. Bởi vì chúng ta đã đặt lệnh gọi
đến `join` sau vòng lặp `for` của luồng chính, việc chạy Listing 16-2 sẽ tạo ra
đầu ra tương tự như sau:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 1 from the spawned thread!
hi number 3 from the main thread!
hi number 2 from the spawned thread!
hi number 4 from the main thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
```

Hai luồng tiếp tục thay phiên nhau, nhưng luồng chính chờ đợi vì lời gọi đến
`handle.join()` và không kết thúc cho đến khi luồng được tạo ra hoàn thành.

Nhưng hãy xem điều gì xảy ra khi chúng ta thay vào đó di chuyển `handle.join()`
trước vòng lặp `for` trong `main`, như thế này:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/no-listing-01-join-too-early/src/main.rs}}
```

</Listing>

Luồng chính sẽ chờ luồng được tạo ra hoàn thành và sau đó chạy vòng lặp `for`
của nó, vì vậy đầu ra sẽ không còn xen kẽ nữa, như hiển thị ở đây:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
hi number 1 from the spawned thread!
hi number 2 from the spawned thread!
hi number 3 from the spawned thread!
hi number 4 from the spawned thread!
hi number 5 from the spawned thread!
hi number 6 from the spawned thread!
hi number 7 from the spawned thread!
hi number 8 from the spawned thread!
hi number 9 from the spawned thread!
hi number 1 from the main thread!
hi number 2 from the main thread!
hi number 3 from the main thread!
hi number 4 from the main thread!
```

Các chi tiết nhỏ, chẳng hạn như nơi `join` được gọi, có thể ảnh hưởng đến việc
các luồng của bạn có chạy cùng lúc hay không.

### Sử Dụng Closure `move` với Luồng

Chúng ta thường sử dụng từ khóa `move` với closure được truyền cho
`thread::spawn` bởi vì closure sau đó sẽ lấy quyền sở hữu các giá trị mà nó sử
dụng từ môi trường, do đó chuyển quyền sở hữu của các giá trị đó từ một luồng
này sang luồng khác. Trong ["Capturing References or Moving
Ownership"][capture]<!-- ignore --> trong Chương 13, chúng ta đã thảo luận về
`move` trong bối cảnh closure. Bây giờ chúng ta sẽ tập trung nhiều hơn vào sự
tương tác giữa `move` và `thread::spawn`.

Lưu ý trong Listing 16-1 rằng closure mà chúng ta truyền cho `thread::spawn`
không nhận tham số nào: chúng ta không sử dụng bất kỳ dữ liệu nào từ luồng chính
trong mã của luồng được tạo ra. Để sử dụng dữ liệu từ luồng chính trong luồng
được tạo ra, closure của luồng được tạo ra phải bắt các giá trị mà nó cần.
Listing 16-3 hiển thị một nỗ lực tạo một vectơ trong luồng chính và sử dụng nó
trong luồng được tạo ra. Tuy nhiên, điều này chưa hoạt động được, như bạn sẽ
thấy trong một lúc nữa.

<Listing number="16-3" file-name="src/main.rs" caption="Cố gắng sử dụng một vectơ được tạo bởi luồng chính trong một luồng khác">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-03/src/main.rs}}
```

</Listing>

Closure sử dụng `v`, vì vậy nó sẽ bắt `v` và làm cho nó trở thành một phần của
môi trường của closure. Bởi vì `thread::spawn` chạy closure này trong một luồng
mới, chúng ta nên có thể truy cập `v` bên trong luồng mới đó. Nhưng khi chúng ta
biên dịch ví dụ này, chúng ta nhận được lỗi sau:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-03/output.txt}}
```

Rust _suy luận_ cách bắt `v`, và bởi vì `println!` chỉ cần một tham chiếu đến
`v`, closure cố gắng mượn `v`. Tuy nhiên, có một vấn đề: Rust không thể biết
luồng được tạo ra sẽ chạy trong bao lâu, vì vậy nó không biết có phải tham chiếu
đến `v` sẽ luôn hợp lệ hay không.

Listing 16-4 cung cấp một kịch bản mà có khả năng cao hơn có một tham chiếu đến
`v` mà sẽ không hợp lệ.

<Listing number="16-4" file-name="src/main.rs" caption="Một luồng với một closure cố gắng bắt một tham chiếu đến `v` từ một luồng chính mà làm rơi `v`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-04/src/main.rs}}
```

</Listing>

Nếu Rust cho phép chúng ta chạy mã này, có khả năng rằng luồng được tạo ra sẽ
ngay lập tức bị đặt vào nền mà không chạy chút nào. Luồng được tạo ra có một
tham chiếu đến `v` bên trong, nhưng luồng chính ngay lập tức làm rơi `v`, sử
dụng hàm `drop` mà chúng ta đã thảo luận trong Chương 15. Sau đó, khi luồng được
tạo ra bắt đầu thực thi, `v` không còn hợp lệ nữa, vì vậy một tham chiếu đến nó
cũng không hợp lệ. Ôi không!

Để sửa lỗi trình biên dịch trong Listing 16-3, chúng ta có thể sử dụng lời
khuyên từ thông báo lỗi:

<!-- manual-regeneration
after automatic regeneration, look at listings/ch16-fearless-concurrency/listing-16-03/output.txt and copy the relevant part
-->

```text
help: to force the closure to take ownership of `v` (and any other referenced variables), use the `move` keyword
  |
6 |     let handle = thread::spawn(move || {
  |                                ++++
```

Bằng cách thêm từ khóa `move` trước closure, chúng ta buộc closure lấy quyền sở
hữu các giá trị mà nó đang sử dụng thay vì để Rust suy luận rằng nó nên mượn các
giá trị. Sửa đổi Listing 16-3 như trong Listing 16-5 sẽ biên dịch và chạy như
chúng ta dự định.

<Listing number="16-5" file-name="src/main.rs" caption="Sử dụng từ khóa `move` để buộc một closure lấy quyền sở hữu các giá trị mà nó sử dụng">

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-05/src/main.rs}}
```

</Listing>

Chúng ta có thể bị cám dỗ để thử cùng một điều để sửa mã trong Listing 16-4 nơi
luồng chính gọi `drop` bằng cách sử dụng một closure `move`. Tuy nhiên, cách sửa
này sẽ không hoạt động vì những gì Listing 16-4 đang cố gắng làm bị cấm vì lý do
khác. Nếu chúng ta thêm `move` vào closure, chúng ta sẽ di chuyển `v` vào môi
trường của closure, và chúng ta không thể gọi `drop` trên nó trong luồng chính
nữa. Chúng ta sẽ nhận được lỗi trình biên dịch này thay thế:

```console
{{#include ../listings/ch16-fearless-concurrency/output-only-01-move-drop/output.txt}}
```

Các quy tắc sở hữu của Rust đã cứu chúng ta một lần nữa! Chúng ta nhận được lỗi
từ mã trong Listing 16-3 vì Rust đang bảo thủ và chỉ mượn `v` cho luồng, điều đó
có nghĩa là luồng chính có thể theo lý thuyết làm mất hiệu lực tham chiếu của
luồng được tạo ra. Bằng cách nói với Rust để chuyển quyền sở hữu của `v` cho
luồng được tạo ra, chúng ta đang đảm bảo với Rust rằng luồng chính sẽ không sử
dụng `v` nữa. Nếu chúng ta thay đổi Listing 16-4 theo cùng một cách, chúng ta sẽ
vi phạm các quy tắc sở hữu khi chúng ta cố gắng sử dụng `v` trong luồng chính.
Từ khóa `move` ghi đè mặc định bảo thủ của Rust về việc mượn; nó không cho phép
chúng ta vi phạm các quy tắc sở hữu.

Bây giờ chúng ta đã đề cập những gì là luồng và các phương thức được cung cấp
bởi API luồng, hãy xem một số tình huống mà chúng ta có thể sử dụng luồng.

[capture]: ch13-01-closures.html#capturing-references-or-moving-ownership
