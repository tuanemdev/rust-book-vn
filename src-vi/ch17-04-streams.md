## Streams: Future theo Trình tự

<!-- Old headings. Do not remove or links may break. -->

<a id="streams"></a>

Cho đến nay trong chương này, chúng ta chủ yếu đã làm việc với các future riêng
lẻ. Một ngoại lệ lớn là kênh async mà chúng ta đã sử dụng. Hãy nhớ lại cách
chúng ta đã sử dụng bộ thu cho kênh async của chúng ta trước đó trong chương này
ở phần ["Message Passing"][17-02-messages]<!-- ignore -->. Phương thức `recv`
async tạo ra một chuỗi các mục theo thời gian. Đây là một ví dụ của một mẫu tổng
quát hơn được gọi là _stream_.

Chúng ta đã thấy một chuỗi các mục trong Chương 13, khi chúng ta xem xét trait
`Iterator` trong phần [The Iterator Trait and the `next`
Method][iterator-trait]<!-- ignore
-->, nhưng có hai sự khác biệt giữa iterators và bộ thu kênh async. Sự khác biệt
đầu tiên là thời gian: iterators là đồng bộ, trong khi bộ thu kênh là bất đồng bộ.
Thứ hai là API. Khi làm việc trực tiếp với `Iterator`, chúng ta gọi phương thức `next`
đồng bộ của nó. Với stream `trpl::Receiver` cụ thể, chúng ta đã gọi một phương thức
`recv` bất đồng bộ thay thế. Ngoài ra, các API này cảm thấy rất giống nhau, và sự
tương đồng đó không phải là sự trùng hợp. Một stream giống như một hình thức lặp
bất đồng bộ. Trong khi `trpl::Receiver` cụ thể đợi để nhận tin nhắn, thì API stream
đa năng chung rộng hơn nhiều: nó cung cấp mục tiếp theo theo cách `Iterator` làm,
nhưng bất đồng bộ.

Sự tương đồng giữa iterators và streams trong Rust có nghĩa là chúng ta thực sự
có thể tạo một stream từ bất kỳ iterator nào. Giống như với một iterator, chúng
ta có thể làm việc với một stream bằng cách gọi phương thức `next` của nó và sau
đó đợi kết quả, như trong Listing 17-30.

<Listing number="17-30" caption="Tạo một stream từ iterator và in các giá trị của nó" file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-21/src/main.rs:stream}}
```

</Listing>

Chúng ta bắt đầu với một mảng các số, mà chúng ta chuyển đổi thành một iterator
và sau đó gọi `map` trên nó để nhân đôi tất cả các giá trị. Sau đó, chúng ta
chuyển đổi iterator thành stream bằng cách sử dụng hàm `trpl::stream_from_iter`.
Tiếp theo, chúng ta lặp qua các mục trong stream khi chúng xuất hiện với vòng
lặp `while let`.

Thật không may, khi chúng ta thử chạy mã, nó không biên dịch được, mà thay vào
đó báo cáo rằng không có phương thức `next` nào khả dụng:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-21
cargo build
copy only the error output
-->

```console
error[E0599]: no method named `next` found for struct `Iter` in the current scope
  --> src/main.rs:10:40
   |
10 |         while let Some(value) = stream.next().await {
   |                                        ^^^^
   |
   = note: the full type name has been written to 'file:///projects/async-await/target/debug/deps/async_await-575db3dd3197d257.long-type-14490787947592691573.txt'
   = note: consider using `--verbose` to print the full type name to the console
   = help: items from traits can only be used if the trait is in scope
help: the following traits which provide `next` are implemented but not in scope; perhaps you want to import one of them
   |
1  + use crate::trpl::StreamExt;
   |
1  + use futures_util::stream::stream::StreamExt;
   |
1  + use std::iter::Iterator;
   |
1  + use std::str::pattern::Searcher;
   |
help: there is a method `try_next` with a similar name
   |
10 |         while let Some(value) = stream.try_next().await {
   |                                        ~~~~~~~~
```

Như kết quả này giải thích, lý do cho lỗi biên dịch là chúng ta cần trait đúng
trong phạm vi để có thể sử dụng phương thức `next`. Dựa trên cuộc thảo luận của
chúng ta cho đến nay, bạn có thể mong đợi một cách hợp lý rằng trait đó là
`Stream`, nhưng nó thực sự là `StreamExt`. Viết tắt của _extension_, `Ext` là
một mẫu phổ biến trong cộng đồng Rust để mở rộng một trait với một trait khác.

Chúng ta sẽ giải thích các trait `Stream` và `StreamExt` chi tiết hơn một chút
vào cuối chương, nhưng hiện tại tất cả những gì bạn cần biết là trait `Stream`
định nghĩa một giao diện cấp thấp mà hiệu quả là kết hợp các trait `Iterator` và
`Future`. `StreamExt` cung cấp một tập hợp API cấp cao hơn trên `Stream`, bao
gồm phương thức `next` cũng như các phương thức tiện ích khác tương tự như những
phương thức được cung cấp bởi trait `Iterator`. `Stream` và `StreamExt` chưa
phải là một phần của thư viện chuẩn của Rust, nhưng hầu hết các crate trong hệ
sinh thái sử dụng cùng định nghĩa.

Cách sửa lỗi biên dịch là thêm câu lệnh `use` cho `trpl::StreamExt`, như trong
Listing 17-31.

<Listing number="17-31" caption="Sử dụng thành công một iterator làm cơ sở cho một stream" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-38 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Với tất cả các phần đó được kết hợp lại, mã này hoạt động theo cách chúng ta
muốn! Hơn nữa, bây giờ chúng ta có `StreamExt` trong phạm vi, chúng ta có thể sử
dụng tất cả các phương thức tiện ích của nó, giống như với các iterator. Ví dụ,
trong Listing 17-32, chúng ta sử dụng phương thức `filter` để lọc ra tất cả trừ
bội số của ba và năm.

<Listing number="17-32" caption="Lọc một stream với phương thức `StreamExt::filter`" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-37 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Tất nhiên, điều này không quá thú vị, vì chúng ta có thể làm tương tự với các
iterator thông thường và không cần async gì cả. Hãy xem những gì chúng ta có thể
làm mà _là_ độc đáo cho streams.

### Kết hợp các Streams

Nhiều khái niệm tự nhiên được biểu diễn dưới dạng streams: các mục trở nên có
sẵn trong một hàng đợi, các đoạn dữ liệu được kéo dần từ hệ thống tập tin khi
tập dữ liệu đầy đủ quá lớn đối với bộ nhớ của máy tính, hoặc dữ liệu đến qua
mạng theo thời gian. Vì streams là futures, chúng ta có thể sử dụng chúng với
bất kỳ loại future nào khác và kết hợp chúng theo những cách thú vị. Ví dụ,
chúng ta có thể gộp các sự kiện để tránh kích hoạt quá nhiều cuộc gọi mạng, đặt
thời gian chờ trên chuỗi các hoạt động chạy dài, hoặc hạn chế các sự kiện giao
diện người dùng để tránh làm công việc không cần thiết.

Hãy bắt đầu bằng việc xây dựng một stream nhỏ gồm các tin nhắn như một thay thế
cho stream dữ liệu mà chúng ta có thể thấy từ WebSocket hoặc giao thức giao tiếp
thời gian thực khác, như được hiển thị trong Listing 17-33.

<Listing number="17-33" caption="Sử dụng bộ thu `rx` như một `ReceiverStream`" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-33 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Đầu tiên, chúng ta tạo một hàm gọi là `get_messages` mà trả về
`impl Stream<Item = String>`. Đối với cách thực hiện của nó, chúng ta tạo một
kênh async, lặp qua 10 chữ cái đầu tiên của bảng chữ cái tiếng Anh, và gửi chúng
qua kênh.

Chúng ta cũng sử dụng một kiểu mới: `ReceiverStream`, nó chuyển đổi bộ thu `rx`
từ `trpl::channel` thành một `Stream` với một phương thức `next`. Trở lại trong
`main`, chúng ta sử dụng một vòng lặp `while let` để in tất cả các tin nhắn từ
stream.

Khi chúng ta chạy mã này, chúng ta nhận được chính xác kết quả mà chúng ta mong
đợi:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Message: 'a'
Message: 'b'
Message: 'c'
Message: 'd'
Message: 'e'
Message: 'f'
Message: 'g'
Message: 'h'
Message: 'i'
Message: 'j'
```

Một lần nữa, chúng ta có thể làm điều này với API `Receiver` thông thường hoặc
thậm chí API `Iterator` thông thường, mặc dù vậy, nên hãy thêm một tính năng yêu
cầu streams: thêm một timeout áp dụng cho mọi mục trong stream, và một độ trễ
trên các mục mà chúng ta phát ra, như trong Listing 17-34.

<Listing number="17-34" caption="Sử dụng phương thức `StreamExt::timeout` để đặt giới hạn thời gian cho các mục trong một stream" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-34 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Chúng ta bắt đầu bằng việc thêm một timeout vào stream với phương thức
`timeout`, nó đến từ trait `StreamExt`. Sau đó, chúng ta cập nhật thân của vòng
lặp `while let`, bởi vì stream bây giờ trả về một `Result`. Biến thể `Ok` chỉ ra
rằng một tin nhắn đã đến kịp thời; biến thể `Err` chỉ ra rằng timeout đã hết hạn
trước khi bất kỳ tin nhắn nào đến. Chúng ta `match` trên kết quả đó và hoặc in
tin nhắn khi chúng ta nhận được nó thành công hoặc in một thông báo về timeout.
Cuối cùng, chú ý rằng chúng ta ghim các tin nhắn sau khi áp dụng timeout cho
chúng, bởi vì trợ giúp timeout tạo ra một stream cần được ghim để được poll.

Tuy nhiên, bởi vì không có độ trễ giữa các tin nhắn, timeout này không thay đổi
hành vi của chương trình. Hãy thêm một độ trễ thay đổi cho các tin nhắn mà chúng
ta gửi, như trong Listing 17-35.

<Listing number="17-35" caption="Gửi tin nhắn qua `tx` với một độ trễ async mà không làm cho `get_messages` trở thành một hàm async" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-35 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Trong `get_messages`, chúng ta sử dụng phương thức iterator `enumerate` với mảng
`messages` để chúng ta có thể lấy chỉ số của mỗi mục mà chúng ta đang gửi cùng
với chính mục đó. Sau đó, chúng ta áp dụng một độ trễ 100 mili giây cho các mục
có chỉ số chẵn và một độ trễ 300 mili giây cho các mục có chỉ số lẻ để mô phỏng
các độ trễ khác nhau mà chúng ta có thể thấy từ một stream các tin nhắn trong
thế giới thực. Bởi vì timeout của chúng ta là 200 mili giây, điều này sẽ ảnh
hưởng đến một nửa các tin nhắn.

Để ngủ giữa các tin nhắn trong hàm `get_messages` mà không chặn, chúng ta cần sử
dụng async. Tuy nhiên, chúng ta không thể biến `get_messages` thành một hàm
async, bởi vì sau đó chúng ta sẽ trả về một
`Future<Output = Stream<Item = String>>` thay vì một `Stream<Item = String>>`.
Người gọi sẽ phải await chính `get_messages` để có quyền truy cập vào stream.
Nhưng hãy nhớ: mọi thứ trong một future nhất định xảy ra tuyến tính; tính đồng
thời xảy ra _giữa_ các future. Awaiting `get_messages` sẽ yêu cầu nó gửi tất cả
các tin nhắn, bao gồm cả độ trễ sleep giữa mỗi tin nhắn, trước khi trả về stream
bộ thu. Kết quả là, timeout sẽ vô dụng. Sẽ không có độ trễ trong chính stream;
tất cả chúng sẽ xảy ra trước khi stream thậm chí có sẵn.

Thay vào đó, chúng ta để `get_messages` như một hàm thông thường trả về một
stream, và chúng ta spawn một task để xử lý các lệnh gọi `sleep` async.

> Lưu ý: Gọi `spawn_task` theo cách này hoạt động bởi vì chúng ta đã thiết lập
> runtime của mình; nếu không, nó sẽ gây ra hoảng loạn. Các cách thực hiện khác
> lựa chọn các sự đánh đổi khác nhau: chúng có thể spawn một runtime mới và
> tránh hoảng loạn nhưng cuối cùng có một chút chi phí bổ sung, hoặc chúng có
> thể đơn giản không cung cấp một cách độc lập để spawn các task mà không cần
> tham chiếu đến runtime. Hãy đảm bảo rằng bạn biết sự đánh đổi mà runtime của
> bạn đã chọn và viết mã của bạn tương ứng!

Bây giờ mã của chúng ta có một kết quả thú vị hơn nhiều. Giữa mỗi cặp tin nhắn
khác, một lỗi `Problem: Elapsed(())`.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Message: 'a'
Problem: Elapsed(())
Message: 'b'
Message: 'c'
Problem: Elapsed(())
Message: 'd'
Message: 'e'
Problem: Elapsed(())
Message: 'f'
Message: 'g'
Problem: Elapsed(())
Message: 'h'
Message: 'i'
Problem: Elapsed(())
Message: 'j'
```

Timeout không ngăn chặn các tin nhắn đến cuối cùng. Chúng ta vẫn nhận được tất
cả các tin nhắn ban đầu, bởi vì kênh của chúng ta là _unbounded_: nó có thể chứa
nhiều tin nhắn như chúng ta có thể vừa trong bộ nhớ. Nếu tin nhắn không đến
trước khi timeout, bộ xử lý stream của chúng ta sẽ tính đến điều đó, nhưng khi
nó poll stream một lần nữa, tin nhắn bây giờ có thể đã đến.

Bạn có thể nhận được hành vi khác nhau nếu cần bằng cách sử dụng các loại kênh
khác hoặc các loại stream khác nói chung. Hãy xem một trong những điều đó trong
thực tế bằng cách kết hợp một stream các khoảng thời gian với stream tin nhắn
này.

### Hợp nhất các Streams

Trước tiên, hãy tạo một stream khác, nó sẽ phát ra một mục mỗi mili giây nếu
chúng ta để nó chạy trực tiếp. Để đơn giản, chúng ta có thể sử dụng hàm `sleep`
để gửi một tin nhắn trên một độ trễ và kết hợp nó với cùng cách tiếp cận mà
chúng ta đã sử dụng trong `get_messages` của việc tạo một stream từ một kênh. Sự
khác biệt là lần này, chúng ta sẽ gửi lại số lượng khoảng thời gian đã trôi qua,
vì vậy kiểu trả về sẽ là `impl Stream<Item = u32>`, và chúng ta có thể gọi hàm
`get_intervals` (xem Listing 17-36).

<Listing number="17-36" caption="Tạo một stream với một bộ đếm sẽ được phát ra mỗi mili giây" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-36 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Chúng ta bắt đầu bằng việc định nghĩa một `count` trong task. (Chúng ta cũng có
thể định nghĩa nó bên ngoài task, nhưng rõ ràng hơn là giới hạn phạm vi của bất
kỳ biến nhất định nào.) Sau đó, chúng ta tạo một vòng lặp vô hạn. Mỗi lần lặp
của vòng lặp ngủ bất đồng bộ trong một mili giây, tăng số đếm, và sau đó gửi nó
qua kênh. Bởi vì tất cả điều này được bọc trong task được tạo bởi `spawn_task`,
tất cả nó—bao gồm cả vòng lặp vô hạn—sẽ được dọn dẹp cùng với runtime.

Loại vòng lặp vô hạn này, nó chỉ kết thúc khi toàn bộ runtime bị tháo dỡ, khá
phổ biến trong Rust async: nhiều chương trình cần tiếp tục chạy vô thời hạn. Với
async, điều này không chặn bất cứ thứ gì khác, miễn là có ít nhất một điểm await
trong mỗi lần lặp qua vòng lặp.

Bây giờ, trở lại async block của hàm main của chúng ta, chúng ta có thể cố gắng
hợp nhất các stream `messages` và `intervals`, như trong Listing 17-37.

<Listing number="17-37" caption="Cố gắng hợp nhất các stream `messages` và `intervals`" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-32 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Chúng ta bắt đầu bằng cách gọi `get_intervals`. Sau đó, chúng ta hợp nhất các
stream `messages` và `intervals` với phương thức `merge`, nó kết hợp nhiều
stream thành một stream mà tạo ra các mục từ bất kỳ stream nguồn nào ngay khi
các mục có sẵn, mà không áp đặt bất kỳ thứ tự cụ thể nào. Cuối cùng, chúng ta
lặp qua stream kết hợp đó thay vì qua `messages`.

Tại thời điểm này, không `messages` cũng không `intervals` cần được ghim hoặc có
thể thay đổi, bởi vì cả hai sẽ được kết hợp vào stream `merged` duy nhất. Tuy
nhiên, lệnh gọi `merge` này không biên dịch! (Lệnh gọi `next` trong vòng lặp
`while let` cũng không, nhưng chúng ta sẽ quay lại điều đó.) Đây là bởi vì hai
stream có các kiểu khác nhau. Stream `messages` có kiểu
`Timeout<impl Stream<Item = String>>`, trong đó `Timeout` là kiểu thực thi
`Stream` cho một lệnh gọi `timeout`. Stream `intervals` có kiểu
`impl Stream<Item = u32>`. Để hợp nhất hai stream này, chúng ta cần chuyển đổi
một trong chúng để khớp với stream kia. Chúng ta sẽ cải tạo stream intervals,
bởi vì messages đã ở định dạng cơ bản mà chúng ta muốn và phải xử lý các lỗi
timeout (xem Listing 17-38).

<!-- We cannot directly test this one, because it never stops. -->

<Listing number="17-38" caption="Căn chỉnh kiểu của stream `intervals` với kiểu của stream `messages`" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-38 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Đầu tiên, chúng ta có thể sử dụng phương thức trợ giúp `map` để chuyển đổi
`intervals` thành một chuỗi. Thứ hai, chúng ta cần phải khớp với `Timeout` từ
`messages`. Bởi vì chúng ta không thực sự _muốn_ một timeout cho `intervals`,
tuy nhiên, chúng ta có thể chỉ tạo một timeout dài hơn các khoảng thời gian khác
mà chúng ta đang sử dụng. Ở đây, chúng ta tạo một timeout 10 giây với
`Duration::from_secs(10)`. Cuối cùng, chúng ta cần làm cho `stream` có thể thay
đổi, để các lệnh gọi `next` của vòng lặp `while let` có thể lặp qua stream, và
ghim nó để an toàn khi làm như vậy. Điều đó đưa chúng ta _gần_ đến nơi chúng ta
cần phải đến. Mọi thứ kiểm tra kiểu. Tuy nhiên, nếu bạn chạy điều này, sẽ có hai
vấn đề. Đầu tiên, nó sẽ không bao giờ dừng lại! Bạn sẽ cần dừng nó với
<span class="keystroke">ctrl-c</span>. Thứ hai, các tin nhắn từ bảng chữ cái
tiếng Anh sẽ bị chôn vùi giữa tất cả các tin nhắn bộ đếm khoảng thời gian:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the tasks running differently rather than
changes in the compiler -->

```text
--snip--
Interval: 38
Interval: 39
Interval: 40
Message: 'a'
Interval: 41
Interval: 42
Interval: 43
--snip--
```

Listing 17-39 hiển thị một cách để giải quyết hai vấn đề cuối cùng này.

<Listing number="17-39" caption="Sử dụng `throttle` và `take` để quản lý các stream đã hợp nhất" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-39 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Đầu tiên, chúng ta sử dụng phương thức `throttle` trên stream `intervals` để nó
không áp đảo stream `messages`. _Throttling_ là một cách để giới hạn tốc độ mà
một hàm sẽ được gọi—hoặc, trong trường hợp này, bao lâu một lần stream sẽ được
poll. Mỗi 100 mili giây sẽ đủ, bởi vì đó là khoảng thời gian tin nhắn của chúng
ta đến.

Để giới hạn số lượng mục mà chúng ta sẽ chấp nhận từ một stream, chúng ta áp
dụng phương thức `take` cho stream `merged`, bởi vì chúng ta muốn giới hạn đầu
ra cuối cùng, không chỉ một stream này hoặc stream kia.

Bây giờ khi chúng ta chạy chương trình, nó dừng lại sau khi kéo 20 mục từ
stream, và các khoảng thời gian không áp đảo các tin nhắn. Chúng ta cũng không
nhận được `Interval: 100` hoặc `Interval: 200` hoặc vân vân, mà thay vào đó nhận
được `Interval: 1`, `Interval: 2`, và v.v.—mặc dù chúng ta có một stream nguồn
có _thể_ tạo ra một sự kiện mỗi mili giây. Đó là bởi vì lệnh gọi `throttle` tạo
ra một stream mới bao bọc stream ban đầu để stream ban đầu chỉ được poll ở tốc
độ throttle, không phải tốc độ "tự nhiên" của nó. Chúng ta không có một đống tin
nhắn khoảng thời gian không xử lý mà chúng ta đang chọn bỏ qua. Thay vào đó,
chúng ta không bao giờ tạo ra những tin nhắn khoảng thời gian đó ngay từ đầu!
Đây là tính "lười biếng" vốn có của các future của Rust làm việc một lần nữa,
cho phép chúng ta chọn các đặc điểm hiệu suất của mình.

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Interval: 1
Message: 'a'
Interval: 2
Interval: 3
Problem: Elapsed(())
Interval: 4
Message: 'b'
Interval: 5
Message: 'c'
Interval: 6
Interval: 7
Problem: Elapsed(())
Interval: 8
Message: 'd'
Interval: 9
Message: 'e'
Interval: 10
Interval: 11
Problem: Elapsed(())
Interval: 12
```

Có một điều cuối cùng chúng ta cần xử lý: lỗi! Với cả hai stream dựa trên kênh
này, các lệnh gọi `send` có thể thất bại khi phía bên kia của kênh đóng—và đó
chỉ là một vấn đề về cách runtime thực thi các future tạo nên stream. Cho đến
bây giờ, chúng ta đã bỏ qua khả năng này bằng cách gọi `unwrap`, nhưng trong một
ứng dụng hoạt động tốt, chúng ta nên xử lý lỗi một cách rõ ràng, tối thiểu là
bằng cách kết thúc vòng lặp để chúng ta không cố gắng gửi thêm tin nhắn. Listing
17-40 hiển thị một chiến lược lỗi đơn giản: in vấn đề và sau đó `break` khỏi các
vòng lặp.

<Listing number="17-40" caption="Xử lý lỗi và tắt các vòng lặp">

```rust,ignore
// Note: Listing 17-40 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Như thường lệ, cách đúng để xử lý lỗi gửi tin nhắn sẽ thay đổi; chỉ cần đảm bảo
bạn có một chiến lược.

Bây giờ sau khi chúng ta đã thấy rất nhiều async trong thực tế, hãy lui lại một
bước và đào sâu vào một vài chi tiết về cách `Future`, `Stream`, và các trait
quan trọng khác mà Rust sử dụng để làm cho async hoạt động.

[17-02-messages]: ch17-02-concurrency-with-async.html#message-passing
[iterator-trait]: ch13-02-iterators.html#the-iterator-trait-and-the-next-method
