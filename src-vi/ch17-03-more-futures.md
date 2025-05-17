<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### Nhường Quyền Kiểm Soát cho Runtime

Nhớ lại từ phần ["Our First Async Program"][async-program]<!-- ignore --> rằng
tại mỗi điểm await, Rust cho phép runtime có cơ hội tạm dừng tác vụ và chuyển
sang một tác vụ khác nếu future đang được await chưa sẵn sàng. Điều ngược lại
cũng đúng: Rust _chỉ_ tạm dừng các async block và trả quyền kiểm soát lại cho
runtime tại một điểm await. Mọi thứ giữa các điểm await là đồng bộ.

Điều đó có nghĩa là nếu bạn thực hiện một loạt công việc trong một async block
mà không có điểm await, future đó sẽ chặn bất kỳ future nào khác thực hiện tiến
trình. Đôi khi bạn có thể nghe điều này được gọi là một future _đói_ các future
khác. Trong một số trường hợp, điều đó có thể không phải là vấn đề lớn. Tuy
nhiên, nếu bạn đang thực hiện một số loại thiết lập đắt tiền hoặc công việc chạy
dài, hoặc nếu bạn có một future sẽ tiếp tục thực hiện một số tác vụ cụ thể vô
thời hạn, bạn sẽ cần suy nghĩ về khi nào và ở đâu để bàn giao quyền kiểm soát
cho runtime.

Hãy mô phỏng một hoạt động chạy dài để minh họa vấn đề đói, sau đó khám phá cách
giải quyết nó. Listing 17-14 giới thiệu một hàm `slow`.

<Listing number="17-14" caption="Sử dụng `thread::sleep` để mô phỏng các hoạt động chậm" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

Đây chắc chắn là một cải tiến so với việc chuyển đổi giữa `join` và `join3` và
`join4` và v.v.! Tuy nhiên, ngay cả dạng macro này cũng chỉ hoạt động khi chúng
ta biết số lượng future trước. Trong Rust thực tế, việc đưa các future vào một
tập hợp và sau đó đợi một số hoặc tất cả các future hoàn thành là một mẫu phổ
biến.

Để kiểm tra tất cả các future trong một tập hợp, chúng ta cần lặp qua và kết hợp
_tất cả_ chúng. Hàm `trpl::join_all` chấp nhận bất kỳ kiểu nào thực thi trait
`Iterator`, mà bạn đã học trong [The Iterator Trait and the `next`
Method][iterator-trait]<!-- ignore --> Chương 13, vì vậy nó có vẻ như đúng là
thứ chúng ta cần. Hãy thử đặt các future của chúng ta vào một vector và thay thế
`join!` bằng `join_all` như trong Listing 17-15.

<Listing  number="17-15" caption="Lưu trữ các future ẩn danh trong một vector và gọi `join_all`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:here}}
```

</Listing>

Thật không may, mã này không biên dịch được. Thay vào đó, chúng ta nhận được lỗi
này:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo build
copy just the compiler error
-->

```text
error[E0308]: mismatched types
  --> src/main.rs:45:37
   |
10 |         let tx1_fut = async move {
   |                       ---------- the expected `async` block
...
24 |         let rx_fut = async {
   |                      ----- the found `async` block
...
45 |         let futures = vec![tx1_fut, rx_fut, tx_fut];
   |                                     ^^^^^^ expected `async` block, found a different `async` block
   |
   = note: expected `async` block `{async block@src/main.rs:10:23: 10:33}`
              found `async` block `{async block@src/main.rs:24:22: 24:27}`
   = note: no two async blocks, even if identical, have the same type
   = help: consider pinning your async block and casting it to a trait object
```

Điều này có thể gây ngạc nhiên. Rốt cuộc, không có async block nào trả về bất cứ
thứ gì, nên mỗi block tạo ra một `Future<Output = ()>`. Hãy nhớ rằng `Future` là
một trait, tuy nhiên, và trình biên dịch tạo ra một enum duy nhất cho mỗi async
block. Bạn không thể đặt hai struct viết tay khác nhau trong một `Vec`, và quy
tắc tương tự áp dụng cho các enum khác nhau được tạo ra bởi trình biên dịch.

Để làm cho điều này hoạt động, chúng ta cần sử dụng _trait objects_, giống như
chúng ta đã làm trong ["Returning Errors from the run
function"][dyn]<!-- ignore --> ở Chương 12. (Chúng ta sẽ đề cập chi tiết về các
trait object trong Chương 18.) Sử dụng trait object cho phép chúng ta xem xét
mỗi future ẩn danh được tạo ra bởi các kiểu này là cùng một kiểu, bởi vì tất cả
chúng đều thực thi trait `Future`.

> Lưu ý: Trong [Using an Enum to Store Multiple Values][enum-alt]<!-- ignore -->
> ở Chương 8, chúng ta đã thảo luận về một cách khác để bao gồm nhiều kiểu trong
> một `Vec`: sử dụng một enum để đại diện cho mỗi kiểu có thể xuất hiện trong
> vector. Chúng ta không thể làm điều đó ở đây. Một mặt, chúng ta không có cách
> để đặt tên cho các kiểu khác nhau, bởi vì chúng là ẩn danh. Mặt khác, lý do
> chúng ta chọn vector và `join_all` ngay từ đầu là để có thể làm việc với một
> tập hợp động của các future mà chúng ta chỉ quan tâm rằng chúng có cùng kiểu
> đầu ra.

Chúng ta bắt đầu bằng cách bọc mỗi future trong `vec!` trong một `Box::new`, như
trong Listing 17-16.

<Listing number="17-16" caption="Sử dụng `Box::new` để căn chỉnh các kiểu của future trong một `Vec`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

Thật không may, mã này vẫn không biên dịch. Thực tế, chúng ta nhận được cùng lỗi
cơ bản mà chúng ta đã nhận trước đó cho cả hai lệnh gọi `Box::new` thứ hai và
thứ ba, cũng như các lỗi mới đề cập đến trait `Unpin`. Chúng ta sẽ quay lại các
lỗi `Unpin` trong giây lát. Trước tiên, hãy sửa các lỗi kiểu trên các lệnh gọi
`Box::new` bằng cách chú thích rõ ràng kiểu của biến `futures` (xem Listing
17-17).

<Listing number="17-17" caption="Sửa các lỗi không khớp kiểu còn lại bằng cách sử dụng khai báo kiểu rõ ràng" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:here}}
```

</Listing>

Khai báo kiểu này hơi phức tạp, vì vậy hãy cùng đi qua nó:

1. Kiểu bên trong cùng là future. Chúng ta lưu ý rõ ràng rằng đầu ra của future
   là kiểu đơn vị `()` bằng cách viết `Future<Output = ()>`.
2. Sau đó, chúng ta chú thích trait với `dyn` để đánh dấu nó là động.
3. Toàn bộ tham chiếu trait được bọc trong một `Box`.
4. Cuối cùng, chúng ta nêu rõ rằng `futures` là một `Vec` chứa những mục này.

Điều đó đã tạo ra một sự khác biệt lớn. Bây giờ khi chúng ta chạy trình biên
dịch, chúng ta chỉ nhận được các lỗi đề cập đến `Unpin`. Mặc dù có ba lỗi, nhưng
nội dung của chúng rất giống nhau.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-17
cargo build
# copy *only* the errors
# fix the paths
-->

```text
error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
   --> src/main.rs:49:24
    |
49  |         trpl::join_all(futures).await;
    |         -------------- ^^^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
    |         |
    |         required by a bound introduced by this call
    |
    = note: consider using the `pin!` macro
            consider using `Box::pin` if you need to access the pinned value outside of the current scope
    = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `join_all`
   --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:105:14
    |
102 | pub fn join_all<I>(iter: I) -> JoinAll<I::Item>
    |        -------- required by a bound in this function
...
105 |     I::Item: Future,
    |              ^^^^^^ required by this bound in `join_all`

error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:49:9
   |
49 |         trpl::join_all(futures).await;
   |         ^^^^^^^^^^^^^^^^^^^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

error[E0277]: `dyn Future<Output = ()>` cannot be unpinned
  --> src/main.rs:49:33
   |
49 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `dyn Future<Output = ()>`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<dyn Future<Output = ()>>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`

For more information about this error, try `rustc --explain E0277`.
error: could not compile `async_await` (bin "async_await") due to 3 previous errors
```

Đó là _rất nhiều_ thông tin để tiêu hóa, vì vậy hãy phân tích nó. Phần đầu tiên
của thông báo cho chúng ta biết rằng async block đầu tiên
(`src/main.rs:8:23: 20:10`) không thực thi trait `Unpin` và đề xuất sử dụng
`pin!` hoặc `Box::pin` để giải quyết vấn đề. Về sau trong chương, chúng ta sẽ đi
sâu vào một vài chi tiết khác về `Pin` và `Unpin`. Tuy nhiên, hiện tại, chúng ta
có thể làm theo lời khuyên của trình biên dịch để giải quyết vấn đề. Trong
Listing 17-18, chúng ta bắt đầu bằng cách import `Pin` từ `std::pin`. Tiếp theo,
chúng ta cập nhật chú thích kiểu cho `futures`, với một `Pin` bao bọc mỗi `Box`.
Cuối cùng, chúng ta sử dụng `Box::pin` để cố định các future.

<Listing number="17-18" caption="Sử dụng `Pin` và `Box::pin` để làm cho `Vec` kiểm tra được kiểu" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Nếu chúng ta biên dịch và chạy mã này, cuối cùng chúng ta sẽ nhận được kết quả
mong muốn:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
received 'hi'
received 'more'
received 'from'
received 'messages'
received 'the'
received 'for'
received 'future'
received 'you'
```

Xong rồi!

Có một chút điều cần khám phá thêm ở đây. Một mặt, việc sử dụng `Pin<Box<T>>`
thêm một lượng nhỏ chi phí từ việc đưa các future này lên heap với `Box`—và
chúng ta chỉ làm điều đó để làm cho các kiểu phù hợp. Chúng ta thực sự không
_cần_ cấp phát trên heap, sau tất cả: các future này là cục bộ cho hàm cụ thể
này. Như đã lưu ý trước đó, `Pin` tự nó là một kiểu bao bọc, vì vậy chúng ta có
thể nhận được lợi ích của việc có một kiểu duy nhất trong `Vec`—lý do ban đầu
chúng ta chọn `Box`—mà không cần cấp phát heap. Chúng ta có thể sử dụng `Pin`
trực tiếp với mỗi future, sử dụng macro `std::pin::pin`.

Tuy nhiên, chúng ta vẫn phải rõ ràng về kiểu của tham chiếu được ghim; nếu
không, Rust vẫn không biết cách diễn giải chúng như các đối tượng trait động, đó
là những gì chúng ta cần chúng trở thành trong `Vec`. Do đó, chúng ta thêm `pin`
vào danh sách import của chúng ta từ `std::pin`. Sau đó, chúng ta có thể `pin!`
mỗi future khi chúng ta định nghĩa nó và định nghĩa `futures` là một `Vec` chứa
các tham chiếu có thể thay đổi được ghim đến kiểu future động, như trong Listing
17-19.

<Listing number="17-19" caption="Sử dụng `Pin` trực tiếp với macro `pin!` để tránh cấp phát heap không cần thiết" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:here}}
```

</Listing>

Chúng ta đã đi đến đây bằng cách bỏ qua thực tế rằng chúng ta có thể có các kiểu
`Output` khác nhau. Ví dụ, trong Listing 17-20, future ẩn danh cho `a` thực thi
`Future<Output = u32>`, future ẩn danh cho `b` thực thi `Future<Output = &str>`,
và future ẩn danh cho `c` thực thi `Future<Output = bool>`.

<Listing number="17-20" caption="Ba future với các kiểu riêng biệt" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:here}}
```

</Listing>

Chúng ta có thể sử dụng `trpl::join!` để đợi chúng, bởi vì nó cho phép chúng ta
truyền vào nhiều kiểu future và tạo ra một tuple của các kiểu đó. Chúng ta
_không thể_ sử dụng `trpl::join_all`, bởi vì nó yêu cầu tất cả các future được
truyền vào phải có cùng kiểu. Hãy nhớ rằng, lỗi đó là những gì khiến chúng ta
bắt đầu cuộc phiêu lưu này với `Pin`!

Đây là một sự đánh đổi cơ bản: chúng ta có thể xử lý một số lượng động của các
future với `join_all`, miễn là tất cả chúng đều có cùng kiểu, hoặc chúng ta có
thể xử lý một tập hợp cố định số lượng future với các hàm `join` hoặc macro
`join!`, ngay cả khi chúng có các kiểu khác nhau. Đây là cùng một kịch bản chúng
ta gặp phải khi làm việc với bất kỳ kiểu nào khác trong Rust. Future không có gì
đặc biệt, mặc dù chúng ta có một số cú pháp tốt để làm việc với chúng, và đó là
một điều tốt.

### Đua Các Future

Khi chúng ta "join" các future với họ hàm `join` và các macro, chúng ta yêu cầu
_tất cả_ chúng phải hoàn thành trước khi chúng ta tiếp tục. Đôi khi, tuy nhiên,
chúng ta chỉ cần _một số_ future từ một tập hợp hoàn thành trước khi chúng ta
tiếp tục—hơi giống với việc đua một future với một future khác.

Trong Listing 17-21, chúng ta một lần nữa sử dụng `trpl::race` để chạy hai
future, `slow` và `fast`, đua nhau.

<Listing number="17-21" caption="Sử dụng `race` để lấy kết quả của future nào hoàn thành trước" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:here}}
```

</Listing>

Mỗi future in một thông báo khi nó bắt đầu chạy, tạm dừng một khoảng thời gian
bằng cách gọi và đợi `sleep`, và sau đó in một thông báo khác khi nó hoàn thành.
Sau đó, chúng ta truyền cả `slow` và `fast` cho `trpl::race` và đợi một trong số
chúng hoàn thành. (Kết quả ở đây không quá đáng ngạc nhiên: `fast` thắng.) Không
giống như khi chúng ta sử dụng `race` trở lại trong ["Our First Async
Program"][async-program]<!--
ignore -->, chúng ta chỉ bỏ qua thể hiện `Either` mà nó trả về ở đây, bởi vì tất
cả các hành vi thú vị xảy ra trong phần thân của các async block.

Chú ý rằng nếu bạn đảo ngược thứ tự các đối số cho `race`, thứ tự của các thông
báo "started" sẽ thay đổi, mặc dù future `fast` luôn hoàn thành trước. Đó là vì
việc thực thi của hàm `race` cụ thể này là không công bằng. Nó luôn chạy các
future được truyền vào làm đối số theo thứ tự chúng được truyền vào. Các cách
thực thi khác _là_ công bằng và sẽ chọn ngẫu nhiên future nào được poll đầu
tiên. Bất kể việc thực thi race mà chúng ta đang sử dụng có công bằng hay không,
_một_ trong các future sẽ chạy đến điểm `await` đầu tiên trong phần thân của nó
trước khi một tác vụ khác có thể bắt đầu.

Nhớ lại từ [Our First Async Program][async-program]<!-- ignore --> rằng tại mỗi
điểm await, Rust cho phép runtime có cơ hội tạm dừng tác vụ và chuyển sang một
tác vụ khác nếu future đang được await chưa sẵn sàng. Điều ngược lại cũng đúng:
Rust _chỉ_ tạm dừng các async block và trả quyền kiểm soát lại cho runtime tại
một điểm await. Mọi thứ giữa các điểm await là đồng bộ.

Điều đó có nghĩa là nếu bạn thực hiện một loạt công việc trong một async block
mà không có điểm await, future đó sẽ chặn bất kỳ future nào khác thực hiện tiến
trình. Đôi khi bạn có thể nghe điều này được gọi là một future _đói_ các future
khác. Trong một số trường hợp, điều đó có thể không phải là vấn đề lớn. Tuy
nhiên, nếu bạn đang thực hiện một số loại thiết lập đắt tiền hoặc công việc chạy
dài, hoặc nếu bạn có một future sẽ tiếp tục thực hiện một số tác vụ cụ thể vô
thời hạn, bạn sẽ cần suy nghĩ về khi nào và ở đâu để bàn giao quyền kiểm soát
cho runtime.

Cũng vậy, nếu bạn có các hoạt động chặn chạy dài, async có thể là một công cụ
hữu ích để cung cấp các cách cho các phần khác nhau của chương trình liên quan
với nhau.

Nhưng _làm thế nào_ bạn sẽ bàn giao quyền kiểm soát cho runtime trong những
trường hợp đó?

<!-- Old headings. Do not remove or links may break. -->

<a id="yielding"></a>

### Nhường Quyền Kiểm Soát cho Runtime

Hãy mô phỏng một hoạt động chạy dài. Listing 17-22 giới thiệu một hàm `slow`.

<Listing number="17-22" caption="Sử dụng `thread::sleep` để mô phỏng các hoạt động chậm" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-22/src/main.rs:slow}}
```

</Listing>

<Listing number="17-22" caption="Sử dụng `thread::sleep` để mô phỏng các hoạt động chậm" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-14/src/main.rs:slow}}
```

</Listing>

Mã này sử dụng `std::thread::sleep` thay vì `trpl::sleep` để khi gọi `slow` sẽ
chặn thread hiện tại trong một số mili giây. Chúng ta có thể sử dụng `slow` để
đại diện cho các hoạt động trong thế giới thực vừa chạy lâu vừa chặn.

Trong Listing 17-15, chúng ta sử dụng `slow` để mô phỏng việc thực hiện loại
công việc gắn với CPU này trong một cặp future.

<Listing number="17-15" caption="Gọi hàm `slow` để mô phỏng các hoạt động chậm" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-15/src/main.rs:slow-futures}}
```

</Listing>

Để bắt đầu, mỗi future chỉ trả lại quyền kiểm soát cho runtime _sau khi_ thực
hiện một loạt các hoạt động chậm. Nếu bạn chạy mã này, bạn sẽ thấy kết quả này:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-23/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Mỗi future trả lại quyền kiểm soát cho runtime chỉ _sau khi_ thực hiện một loạt
các hoạt động chậm. Nếu bạn chạy mã này, bạn sẽ thấy kết quả này:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-15/
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'a' ran for 10ms
'a' ran for 20ms
'b' started.
'b' ran for 75ms
'b' ran for 10ms
'b' ran for 15ms
'b' ran for 350ms
'a' finished.
```

Giống như ví dụ Listing 17-5 khi chúng ta sử dụng `trpl::select` để đua các
future lấy hai URL, `select` vẫn kết thúc ngay khi `a` hoàn thành. Không có sự
xen kẽ giữa các lệnh gọi `slow` trong hai future. Future `a` thực hiện tất cả
công việc của nó cho đến khi lệnh gọi `trpl::sleep` được await, sau đó future
`b` thực hiện tất cả công việc của nó cho đến khi lệnh gọi `trpl::sleep` của
riêng nó được await, và cuối cùng future `a` hoàn thành. Để cho phép cả hai
future thực hiện tiến trình giữa các tác vụ chậm của chúng, chúng ta cần các
điểm await để chúng ta có thể bàn giao quyền kiểm soát cho runtime. Điều đó có
nghĩa là chúng ta cần thứ gì đó mà chúng ta có thể await!

Chúng ta đã có thể thấy loại bàn giao này xảy ra trong Listing 17-15: nếu chúng
ta loại bỏ `trpl::sleep` ở cuối future `a`, nó sẽ hoàn thành mà không có future
`b` chạy _chút nào_. Hãy thử sử dụng hàm `trpl::sleep` làm điểm khởi đầu để cho
phép các hoạt động luân phiên thực hiện tiến trình, như được hiển thị trong
Listing 17-16.

<Listing number="17-16" caption="Sử dụng `trpl::sleep` để cho phép các hoạt động luân phiên thực hiện tiến trình" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-16/src/main.rs:here}}
```

</Listing>

Chúng ta đã thêm các lệnh gọi `trpl::sleep` với các điểm await giữa mỗi lệnh gọi
đến `slow`. Bây giờ công việc của hai future được xen kẽ:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo run
copy just the output
-->

```text
'a' started.
'a' ran for 30ms
'b' started.
'b' ran for 75ms
'a' ran for 10ms
'b' ran for 10ms
'a' ran for 20ms
'b' ran for 15ms
'a' finished.
```

Future `a` vẫn chạy một chút trước khi bàn giao quyền kiểm soát cho `b`, bởi vì
nó gọi `slow` trước khi từng gọi `trpl::sleep`, nhưng sau đó các future hoán đổi
qua lại mỗi khi một trong số chúng đạt đến một điểm await. Trong trường hợp này,
chúng ta đã làm điều đó sau mỗi lệnh gọi đến `slow`, nhưng chúng ta có thể chia
nhỏ công việc theo bất kỳ cách nào hợp lý nhất đối với chúng ta.

Tuy nhiên, chúng ta không thực sự muốn _ngủ_ ở đây: chúng ta muốn thực hiện tiến
trình nhanh nhất có thể. Chúng ta chỉ cần trả lại quyền kiểm soát cho runtime.
Chúng ta có thể làm điều đó trực tiếp, sử dụng hàm `trpl::yield_now`. Trong
Listing 17-17, chúng ta thay thế tất cả những lệnh gọi `trpl::sleep` bằng
`trpl::yield_now`.

<Listing number="17-17" caption="Sử dụng `yield_now` để cho phép các hoạt động luân phiên thực hiện tiến trình" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-17/src/main.rs:yields}}
```

</Listing>

Mã này vừa rõ ràng hơn về ý định thực tế vừa có thể nhanh hơn đáng kể so với
việc sử dụng `sleep`, bởi vì các bộ đếm thời gian như bộ được sử dụng bởi
`sleep` thường có giới hạn về mức độ chi tiết mà chúng có thể có. Phiên bản
`sleep` mà chúng ta đang sử dụng, ví dụ, sẽ luôn ngủ ít nhất một mili giây, ngay
cả khi chúng ta truyền cho nó một `Duration` một nano giây. Một lần nữa, máy
tính hiện đại _nhanh_: chúng có thể làm rất nhiều trong một mili giây!

Điều này có nghĩa là async có thể hữu ích ngay cả cho các tác vụ gắn với tính
toán, tùy thuộc vào những gì chương trình của bạn đang làm, bởi vì nó cung cấp
một công cụ hữu ích để cấu trúc các mối quan hệ giữa các phần khác nhau của
chương trình (nhưng với chi phí của overhead từ state machine async). Đây là một
hình thức _đa nhiệm hợp tác_, trong đó mỗi future có quyền xác định khi nào nó
bàn giao quyền kiểm soát thông qua các điểm await. Mỗi future do đó cũng có
trách nhiệm tránh chặn quá lâu. Trong một số hệ điều hành nhúng dựa trên Rust,
đây là _loại duy nhất_ của đa nhiệm!

Trong mã thực tế, bạn thường sẽ không thay đổi các lệnh gọi hàm với các điểm
await trên mỗi dòng, tất nhiên. Mặc dù nhường quyền kiểm soát theo cách này là
tương đối không tốn kém, nhưng nó không miễn phí. Trong nhiều trường hợp, việc
cố gắng chia nhỏ một tác vụ gắn với tính toán có thể làm cho nó chậm hơn đáng
kể, vì vậy đôi khi tốt hơn cho hiệu suất _tổng thể_ là để một hoạt động chặn
ngắn. Luôn đo lường để xem thực sự các nút thắt cổ chai hiệu suất của mã của bạn
là gì. Động lực cơ bản là điều quan trọng cần ghi nhớ, tuy nhiên, nếu bạn _đang_
thấy nhiều công việc xảy ra tuần tự mà bạn mong đợi xảy ra đồng thời!

### Xây Dựng Các Sự Trừu Tượng Async Của Riêng Chúng Ta

Chúng ta cũng có thể kết hợp các future lại với nhau để tạo ra các mẫu mới. Ví
dụ, chúng ta có thể xây dựng một hàm `timeout` với các khối xây dựng async mà
chúng ta đã có. Khi chúng ta hoàn thành, kết quả sẽ là một khối xây dựng khác mà
chúng ta có thể sử dụng để tạo ra thêm các sự trừu tượng async.

Listing 17-18 hiển thị cách chúng ta mong đợi `timeout` này hoạt động với một
future chậm.

<Listing number="17-18" caption="Sử dụng `timeout` tưởng tượng của chúng ta để chạy một hoạt động chậm với giới hạn thời gian" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-18/src/main.rs:here}}
```

</Listing>

Hãy triển khai điều này! Để bắt đầu, hãy suy nghĩ về API cho `timeout`:

- Nó cần phải là một hàm async để chúng ta có thể await nó.
- Tham số đầu tiên của nó phải là một future để chạy. Chúng ta có thể làm cho nó
  generic để cho phép nó hoạt động với bất kỳ future nào.
- Tham số thứ hai của nó sẽ là thời gian tối đa để đợi. Nếu chúng ta sử dụng một
  `Duration`, điều đó sẽ làm cho nó dễ dàng để truyền cho `trpl::sleep`.
- Nó sẽ trả về một `Result`. Nếu future hoàn thành thành công, `Result` sẽ là
  `Ok` với giá trị được tạo ra bởi future. Nếu timeout hết hạn trước, `Result`
  sẽ là `Err` với duration mà timeout đã đợi.

Listing 17-19 hiển thị khai báo này.

<!-- This is not tested because it intentionally does not compile. -->

<Listing number="17-19" caption="Định nghĩa chữ ký của `timeout`" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-19/src/main.rs:declaration}}
```

</Listing>

Điều đó đáp ứng các mục tiêu của chúng ta cho các kiểu. Bây giờ hãy suy nghĩ về
_hành vi_ mà chúng ta cần: chúng ta muốn đua future được truyền vào với
duration. Chúng ta có thể sử dụng `trpl::sleep` để tạo một future bộ đếm thời
gian từ duration, và sử dụng `trpl::select` để chạy bộ đếm thời gian đó với
future mà người gọi truyền vào.

Trong Listing 17-20, chúng ta triển khai `timeout` bằng cách match trên kết quả
của việc await `trpl::select`.

<Listing number="17-20" caption="Định nghĩa `timeout` với `select` và `sleep`" file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-20/src/main.rs:implementation}}
```

</Listing>

Việc triển khai `trpl::select` không công bằng: nó luôn poll các đối số theo thứ
tự mà chúng được truyền vào (các triển khai `select` khác sẽ chọn ngẫu nhiên đối
số nào được poll đầu tiên). Do đó, chúng ta truyền `future_to_try` cho `select`
trước để nó có cơ hội hoàn thành ngay cả khi `max_time` là một duration rất
ngắn. Nếu `future_to_try` hoàn thành trước, `select` sẽ trả về `Left` với đầu ra
từ `future_to_try`. Nếu `timer` hoàn thành trước, `select` sẽ trả về `Right` với
đầu ra của timer là `()`.

Nếu `future_to_try` thành công và chúng ta nhận được `Left(output)`, chúng ta
trả về `Ok(output)`. Nếu bộ đếm thời gian ngủ hết hạn thay vào đó và chúng ta
nhận được `Right(())`, chúng ta bỏ qua `()` với `_` và trả về `Err(max_time)`
thay thế.

Với điều đó, chúng ta có một `timeout` hoạt động được xây dựng từ hai trợ giúp
async khác. Nếu chúng ta chạy mã của mình, nó sẽ in chế độ thất bại sau timeout:

```text
Failed after 2 seconds
```

Bởi vì future kết hợp với các future khác, bạn có thể xây dựng các công cụ thực
sự mạnh mẽ sử dụng các khối xây dựng async nhỏ hơn. Ví dụ, bạn có thể sử dụng
cùng cách tiếp cận này để kết hợp timeout với thử lại, và đến lượt sử dụng chúng
với các hoạt động như các cuộc gọi mạng (như những cuộc gọi trong Listing 17-5).

Trong thực tế, bạn thường sẽ làm việc trực tiếp với `async` và `await`, và thứ
hai là với các hàm như `select` và macro như `join!` để kiểm soát cách các
future bên ngoài được thực thi.

Bây giờ chúng ta đã thấy một số cách để làm việc với nhiều future cùng một lúc.
Tiếp theo, chúng ta sẽ xem cách làm việc với nhiều future trong một chuỗi theo
thời gian với _streams_.

[async-program]: ch17-01-futures-and-syntax.html#our-first-async-program
