## Xem xét kỹ hơn về các Trait cho Async

<!-- Old headings. Do not remove or links may break. -->

<a id="digging-into-the-traits-for-async"></a>

Trong suốt chương này, chúng ta đã sử dụng các trait `Future`, `Pin`, `Unpin`,
`Stream`, và `StreamExt` theo nhiều cách khác nhau. Tuy nhiên, cho đến nay,
chúng ta đã tránh đi quá sâu vào chi tiết về cách chúng hoạt động hoặc cách
chúng kết hợp với nhau, điều này phù hợp với công việc Rust hàng ngày của bạn
trong hầu hết thời gian. Tuy nhiên, đôi khi, bạn sẽ gặp phải những tình huống mà
bạn cần hiểu thêm một vài chi tiết này. Trong phần này, chúng ta sẽ đào sâu đủ
để giúp đỡ trong những kịch bản đó, vẫn để lại việc đào sâu _thực sự_ cho các
tài liệu khác.

<!-- Old headings. Do not remove or links may break. -->

<a id="future"></a>

### Trait `Future`

Hãy bắt đầu bằng cách xem xét kỹ hơn cách trait `Future` hoạt động. Đây là cách
Rust định nghĩa nó:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Định nghĩa trait đó bao gồm một loạt các kiểu mới và cũng có một số cú pháp mà
chúng ta chưa thấy trước đây, vì vậy hãy đi qua định nghĩa từng phần một.

Đầu tiên, kiểu kết hợp `Output` của `Future` nói lên những gì future giải quyết.
Điều này tương tự như kiểu kết hợp `Item` của trait `Iterator`. Thứ hai,
`Future` cũng có phương thức `poll`, phương thức này lấy một tham chiếu `Pin`
đặc biệt cho tham số `self` và một tham chiếu có thể thay đổi đến kiểu
`Context`, và trả về một `Poll<Self::Output>`. Chúng ta sẽ nói thêm về `Pin` và
`Context` trong một lát. Hiện tại, hãy tập trung vào những gì phương thức trả
về, kiểu `Poll`:

```rust
enum Poll<T> {
    Ready(T),
    Pending,
}
```

Kiểu `Poll` này tương tự như một `Option`. Nó có một biến thể có giá trị,
`Ready(T)`, và một biến thể không có, `Pending`. Tuy nhiên, `Poll` có ý nghĩa
khá khác với `Option`! Biến thể `Pending` chỉ ra rằng future vẫn còn việc phải
làm, vì vậy người gọi sẽ cần kiểm tra lại sau. Biến thể `Ready` chỉ ra rằng
future đã hoàn thành công việc của nó và giá trị `T` đã có sẵn.

> Lưu ý: Với hầu hết các future, người gọi không nên gọi `poll` lại sau khi
> future đã trả về `Ready`. Nhiều future sẽ panic nếu bị poll lại sau khi đã sẵn
> sàng. Các future an toàn để poll lại sẽ nêu rõ điều đó trong tài liệu của
> chúng. Điều này tương tự như cách `Iterator::next` hoạt động.

Khi bạn thấy mã sử dụng `await`, Rust biên dịch nó bên dưới thành mã gọi `poll`.
Nếu bạn nhìn lại Listing 17-4, nơi chúng ta in tiêu đề trang cho một URL duy
nhất sau khi nó được giải quyết, Rust biên dịch nó thành thứ gì đó kiểu như (mặc
dù không chính xác) như thế này:

```rust,ignore
match page_title(url).poll() {
    Ready(page_title) => match page_title {
        Some(title) => println!("The title for {url} was {title}"),
        None => println!("{url} had no title"),
    }
    Pending => {
        // But what goes here?
    }
}
```

Chúng ta nên làm gì khi future vẫn còn `Pending`? Chúng ta cần một cách nào đó
để thử lại, và lại, và lại, cho đến khi future cuối cùng sẵn sàng. Nói cách
khác, chúng ta cần một vòng lặp:

```rust,ignore
let mut page_title_fut = page_title(url);
loop {
    match page_title_fut.poll() {
        Ready(value) => match page_title {
            Some(title) => println!("The title for {url} was {title}"),
            None => println!("{url} had no title"),
        }
        Pending => {
            // continue
        }
    }
}
```

Tuy nhiên, nếu Rust biên dịch nó thành chính xác mã đó, thì mỗi `await` sẽ
chặn—chính xác là điều ngược lại với những gì chúng ta đang hướng đến! Thay vào
đó, Rust đảm bảo rằng vòng lặp có thể chuyển quyền kiểm soát cho thứ gì đó có
thể tạm dừng công việc trên future này để làm việc trên các future khác và sau
đó kiểm tra lại future này sau. Như chúng ta đã thấy, thứ đó là một runtime
async, và công việc lập lịch và phối hợp này là một trong những nhiệm vụ chính
của nó.

Trước đó trong chương này, chúng ta đã mô tả việc chờ đợi `rx.recv`. Lệnh gọi
`recv` trả về một future, và việc await future sẽ poll nó. Chúng ta đã lưu ý
rằng runtime sẽ tạm dừng future cho đến khi nó sẵn sàng với `Some(message)` hoặc
`None` khi kênh đóng. Với sự hiểu biết sâu sắc hơn về trait `Future`, và cụ thể
là `Future::poll`, chúng ta có thể thấy cách nó hoạt động. Runtime biết future
chưa sẵn sàng khi nó trả về `Poll::Pending`. Ngược lại, runtime biết future _đã_
sẵn sàng và tiến nó khi `poll` trả về `Poll::Ready(Some(message))` hoặc
`Poll::Ready(None)`.

Chi tiết chính xác về cách runtime thực hiện điều đó nằm ngoài phạm vi của cuốn
sách này, nhưng chìa khóa là thấy được cơ chế cơ bản của futures: một runtime
_polls_ mỗi future mà nó chịu trách nhiệm, đưa future trở lại trạng thái ngủ khi
nó chưa sẵn sàng.

<!-- Old headings. Do not remove or links may break. -->

<a id="pinning-and-the-pin-and-unpin-traits"></a>

### Các Trait `Pin` và `Unpin`

Khi chúng ta giới thiệu ý tưởng về pinning trong Listing 17-16, chúng ta đã gặp
phải một thông báo lỗi rất khó hiểu. Đây là phần liên quan của nó một lần nữa:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-16
cargo build
copy *only* the final `error` block from the errors
-->

```text
error[E0277]: `{async block@src/main.rs:10:23: 10:33}` cannot be unpinned
  --> src/main.rs:48:33
   |
48 |         trpl::join_all(futures).await;
   |                                 ^^^^^ the trait `Unpin` is not implemented for `{async block@src/main.rs:10:23: 10:33}`
   |
   = note: consider using the `pin!` macro
           consider using `Box::pin` if you need to access the pinned value outside of the current scope
   = note: required for `Box<{async block@src/main.rs:10:23: 10:33}>` to implement `Future`
note: required by a bound in `futures_util::future::join_all::JoinAll`
  --> file:///home/.cargo/registry/src/index.crates.io-1949cf8c6b5b557f/futures-util-0.3.30/src/future/join_all.rs:29:8
   |
27 | pub struct JoinAll<F>
   |            ------- required by a bound in this struct
28 | where
29 |     F: Future,
   |        ^^^^^^ required by this bound in `JoinAll`
```

Thông báo lỗi này cho chúng ta biết không chỉ rằng chúng ta cần ghim các giá trị
mà còn cả lý do tại sao pinning là bắt buộc. Hàm `trpl::join_all` trả về một
struct có tên `JoinAll`. Struct đó là generic trên một kiểu `F`, bị ràng buộc để
thực thi trait `Future`. Await trực tiếp một future với `await` ghim future một
cách ngầm định. Đó là lý do tại sao chúng ta không cần sử dụng `pin!` ở mọi nơi
chúng ta muốn await futures.

Tuy nhiên, chúng ta không await trực tiếp một future ở đây. Thay vào đó, chúng
ta xây dựng một future mới, `JoinAll`, bằng cách truyền một bộ sưu tập các
future vào hàm `join_all`. Chữ ký cho `join_all` yêu cầu rằng các kiểu của các
mục trong bộ sưu tập đều thực thi trait `Future`, và `Box<T>` thực thi `Future`
chỉ khi `T` mà nó bọc là một future thực thi trait `Unpin`.

Đó là rất nhiều để hấp thụ! Để thực sự hiểu nó, hãy đi sâu hơn một chút vào cách
trait `Future` thực sự hoạt động, đặc biệt là xung quanh _pinning_.

Nhìn lại định nghĩa của trait `Future`:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

pub trait Future {
    type Output;

    // Required method
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}
```

Tham số `cx` và kiểu `Context` của nó là chìa khóa để một runtime thực sự biết
khi nào kiểm tra bất kỳ future nào mà vẫn lười biếng. Một lần nữa, chi tiết về
cách điều đó hoạt động nằm ngoài phạm vi của chương này, và bạn thường chỉ cần
suy nghĩ về điều này khi viết một cách triển khai `Future` tùy chỉnh. Chúng ta
sẽ tập trung thay vào đó vào kiểu cho `self`, vì đây là lần đầu tiên chúng ta
thấy một phương thức trong đó `self` có một chú thích kiểu. Một chú thích kiểu
cho `self` hoạt động như chú thích kiểu cho các tham số hàm khác, nhưng với hai
khác biệt chính:

- Nó cho Rust biết `self` phải là kiểu gì để phương thức được gọi.

- Nó không thể là bất kỳ kiểu nào. Nó bị giới hạn trong kiểu mà phương thức được
  triển khai, một tham chiếu hoặc con trỏ thông minh đến kiểu đó, hoặc một `Pin`
  bao bọc một tham chiếu đến kiểu đó.

Chúng ta sẽ thấy thêm về cú pháp này trong [Chương 18][ch-18]<!-- ignore -->.
Hiện tại, điều đủ để biết là nếu chúng ta muốn poll một future để kiểm tra xem
nó là `Pending` hay `Ready(Output)`, chúng ta cần một tham chiếu có thể thay đổi
được bọc trong `Pin` đến kiểu.

`Pin` là một wrapper cho các kiểu giống con trỏ như `&`, `&mut`, `Box`, và `Rc`.
(Về mặt kỹ thuật, `Pin` hoạt động với các kiểu triển khai các trait `Deref` hoặc
`DerefMut`, nhưng điều này hiệu quả tương đương với việc chỉ làm việc với các
con trỏ.) `Pin` không phải là một con trỏ và không có bất kỳ hành vi nào của
riêng nó như `Rc` và `Arc` với việc đếm tham chiếu; nó thuần túy là một công cụ
mà trình biên dịch có thể sử dụng để áp đặt các ràng buộc về việc sử dụng con
trỏ.

Nhớ lại rằng `await` được triển khai dưới dạng các lệnh gọi đến `poll` bắt đầu
giải thích thông báo lỗi chúng ta đã thấy trước đó, nhưng điều đó là về `Unpin`,
không phải `Pin`. Vậy chính xác thì `Pin` liên quan đến `Unpin` như thế nào, và
tại sao `Future` cần `self` là một kiểu `Pin` để gọi `poll`?

Hãy nhớ từ trước đó trong chương này, một chuỗi các điểm await trong một future
được biên dịch thành một máy trạng thái, và trình biên dịch đảm bảo rằng máy
trạng thái đó tuân theo tất cả các quy tắc thông thường của Rust xung quanh sự
an toàn, bao gồm cả việc mượn và sở hữu. Để làm cho điều đó hoạt động, Rust xem
xét dữ liệu nào là cần thiết giữa một điểm await và điểm await tiếp theo hoặc
cuối của async block. Nó sau đó tạo ra một biến thể tương ứng trong máy trạng
thái được biên dịch. Mỗi biến thể nhận quyền truy cập mà nó cần đến dữ liệu sẽ
được sử dụng trong phần đó của mã nguồn, dù là bằng cách lấy quyền sở hữu dữ
liệu đó hoặc bằng cách nhận một tham chiếu có thể thay đổi hoặc không thể thay
đổi đến nó.

Cho đến nay, vẫn tốt: nếu chúng ta làm bất cứ điều gì sai về quyền sở hữu hoặc
tham chiếu trong một async block nhất định, trình kiểm tra mượn sẽ cho chúng ta
biết. Khi chúng ta muốn di chuyển future tương ứng với block đó—như đưa nó vào
một `Vec` để truyền vào `join_all`—mọi thứ trở nên khó khăn hơn.

Khi chúng ta di chuyển một future—dù là bằng cách đẩy nó vào một cấu trúc dữ
liệu để sử dụng như một iterator với `join_all` hoặc bằng cách trả về nó từ một
hàm—điều đó thực sự có nghĩa là di chuyển máy trạng thái Rust tạo ra cho chúng
ta. Và không giống như hầu hết các kiểu khác trong Rust, các future mà Rust tạo
ra cho các async block có thể kết thúc với các tham chiếu đến chính nó trong các
trường của bất kỳ biến thể nhất định nào, như được minh họa đơn giản trong Hình
17-4.

<figure>

<img alt="A single-column, three-row table representing a future, fut1, which has data values 0 and 1 in the first two rows and an arrow pointing from the third row back to the second row, representing an internal reference within the future." src="img/trpl17-04.svg" class="center" />

<figcaption>Hình 17-4: Một kiểu dữ liệu tự tham chiếu.</figcaption>

</figure>

Tuy nhiên, theo mặc định, bất kỳ đối tượng nào có tham chiếu đến chính nó đều
không an toàn để di chuyển, bởi vì các tham chiếu luôn trỏ đến địa chỉ bộ nhớ
thực tế của bất cứ thứ gì chúng tham chiếu đến (xem Hình 17-5). Nếu bạn di
chuyển chính cấu trúc dữ liệu, những tham chiếu nội bộ đó sẽ bị để lại trỏ đến
vị trí cũ. Tuy nhiên, vị trí bộ nhớ đó giờ đây không hợp lệ. Một mặt, giá trị
của nó sẽ không được cập nhật khi bạn thay đổi cấu trúc dữ liệu. Mặt khác—điều
quan trọng hơn—là máy tính hiện có thể tự do sử dụng lại bộ nhớ đó cho các mục
đích khác! Bạn có thể kết thúc bằng việc đọc dữ liệu hoàn toàn không liên quan
sau này.

<figure>

<img alt="Two tables, depicting two futures, fut1 and fut2, each of which has one column and three rows, representing the result of having moved a future out of fut1 into fut2. The first, fut1, is grayed out, with a question mark in each index, representing unknown memory. The second, fut2, has 0 and 1 in the first and second rows and an arrow pointing from its third row back to the second row of fut1, representing a pointer that is referencing the old location in memory of the future before it was moved." src="img/trpl17-05.svg" class="center" />

<figcaption>Hình 17-5: Kết quả không an toàn của việc di chuyển một kiểu dữ liệu tự tham chiếu</figcaption>

</figure>

Về mặt lý thuyết, trình biên dịch Rust có thể cố gắng cập nhật mọi tham chiếu
đến một đối tượng bất cứ khi nào nó bị di chuyển, nhưng điều đó có thể thêm rất
nhiều chi phí hiệu suất, đặc biệt nếu toàn bộ mạng lưới các tham chiếu cần cập
nhật. Nếu thay vào đó chúng ta có thể đảm bảo rằng cấu trúc dữ liệu đang được đề
cập _không di chuyển trong bộ nhớ_, chúng ta sẽ không phải cập nhật bất kỳ tham
chiếu nào. Đây chính xác là những gì trình kiểm tra mượn của Rust yêu cầu: trong
mã an toàn, nó ngăn bạn di chuyển bất kỳ mục nào có tham chiếu đang hoạt động
đến nó.

`Pin` xây dựng trên điều đó để cung cấp cho chúng ta chính xác sự đảm bảo mà
chúng ta cần. Khi chúng ta _ghim_ một giá trị bằng cách bọc một con trỏ đến giá
trị đó trong `Pin`, nó không còn có thể di chuyển. Do đó, nếu bạn có
`Pin<Box<SomeType>>`, bạn thực sự ghim giá trị `SomeType`, _không_ phải con trỏ
`Box`. Hình 17-6 minh họa quá trình này.

<figure>

<img alt="Three boxes laid out side by side. The first is labeled "Pin", the
second "b1", and the third "pinned". Within "pinned" is a table labeled "fut",
with a single column; it represents a future with cells for each part of the
data structure. Its first cell has the value "0", its second cell has an arrow
coming out of it and pointing to the fourth and final cell, which has the value
"1" in it, and the third cell has dashed lines and an ellipsis to indicate there
may be other parts to the data structure. All together, the "fut" table
represents a future which is self-referential. An arrow leaves the box labeled
"Pin", goes through the box labeled "b1" and has terminates inside the "pinned"
box at the "fut" table." src="img/trpl17-06.svg" class="center" />

<figcaption>Hình 17-6: Ghim một `Box` trỏ đến một kiểu future tự tham chiếu.</figcaption>

</figure>

Trên thực tế, con trỏ `Box` vẫn có thể di chuyển tự do. Hãy nhớ: chúng ta quan
tâm đến việc đảm bảo rằng dữ liệu cuối cùng được tham chiếu ở yên vị trí của nó.
Nếu một con trỏ di chuyển xung quanh, _nhưng dữ liệu mà nó trỏ đến ở cùng một vị
trí_, như trong Hình 17-7, không có vấn đề tiềm ẩn nào. Như một bài tập độc lập,
hãy xem tài liệu cho các kiểu cũng như module `std::pin` và cố gắng tìm ra cách
bạn sẽ làm điều này với một `Pin` bao bọc một `Box`.) Điều quan trọng là kiểu tự
tham chiếu không thể di chuyển, bởi vì nó vẫn được ghim.

<figure>

<img alt="Four boxes laid out in three rough columns, identical to the previous
diagram with a change to the second column. Now there are two boxes in the
second column, labeled "b1" and "b2", "b1" is grayed out, and the arrow from
"Pin" goes through "b2" instead of "b1", indicating that the pointer has moved
from "b1" to "b2", but the data in "pinned" has not moved."
src="img/trpl17-07.svg" class="center" />

<figcaption>Hình 17-7: Di chuyển một `Box` trỏ đến một kiểu future tự tham chiếu.</figcaption>

</figure>

Tuy nhiên, hầu hết các kiểu đều hoàn toàn an toàn để di chuyển, ngay cả khi
chúng tình cờ đứng sau một wrapper `Pin`. Chúng ta chỉ cần nghĩ về việc ghim khi
các mục có tham chiếu nội bộ. Các giá trị nguyên thủy như số và Boolean là an
toàn bởi vì chúng rõ ràng không có bất kỳ tham chiếu nội bộ nào. Hầu hết các
kiểu mà bạn thường làm việc trong Rust cũng vậy. Bạn có thể di chuyển một `Vec`,
ví dụ, mà không cần lo lắng. Với những gì chúng ta đã thấy cho đến nay, nếu bạn
có một `Pin<Vec<String>>`, bạn sẽ phải làm mọi thứ thông qua các API an toàn
nhưng hạn chế được cung cấp bởi `Pin`, mặc dù một `Vec<String>` luôn an toàn để
di chuyển nếu không có tham chiếu nào khác đến nó. Chúng ta cần một cách để nói
với trình biên dịch rằng việc di chuyển các mục xung quanh trong những trường
hợp như thế này là tốt—và đó là nơi `Unpin` đi vào cuộc chơi.

`Unpin` là một trait đánh dấu, tương tự như các trait `Send` và `Sync` mà chúng
ta đã thấy trong Chương 16, và do đó không có chức năng của riêng nó. Các trait
đánh dấu tồn tại chỉ để nói với trình biên dịch rằng việc sử dụng kiểu thực thi
một trait nhất định trong một ngữ cảnh cụ thể là an toàn. `Unpin` thông báo cho
trình biên dịch rằng một kiểu nhất định _không_ cần duy trì bất kỳ đảm bảo nào
về việc liệu giá trị đang được đề cập có thể được di chuyển an toàn.

<!--
  The inline `<code>` in the next block is to allow the inline `<em>` inside it,
  matching what NoStarch does style-wise, and emphasizing within the text here
  that it is something distinct from a normal type.
-->

Giống như với `Send` và `Sync`, trình biên dịch triển khai `Unpin` tự động cho
tất cả các kiểu mà nó có thể chứng minh là an toàn. Một trường hợp đặc biệt, một
lần nữa tương tự như `Send` và `Sync`, là nơi `Unpin` _không_ được triển khai
cho một kiểu. Ký hiệu cho điều này là <code>impl !Unpin for
<em>SomeType</em></code>, trong đó <code><em>SomeType</em></code> là tên của một
kiểu _thực sự_ cần duy trì những đảm bảo đó để an toàn bất cứ khi nào một con
trỏ đến kiểu đó được sử dụng trong một `Pin`.

Nói cách khác, có hai điều cần ghi nhớ về mối quan hệ giữa `Pin` và `Unpin`. Đầu
tiên, `Unpin` là trường hợp "bình thường", và `!Unpin` là trường hợp đặc biệt.
Thứ hai, liệu một kiểu triển khai `Unpin` hay `!Unpin` _chỉ_ quan trọng khi bạn
đang sử dụng một con trỏ được ghim cho kiểu đó như <code>Pin<&mut
<em>SomeType</em>></code>.

Để làm cho điều đó cụ thể, hãy nghĩ về một `String`: nó có một độ dài và các ký
tự Unicode tạo nên nó. Chúng ta có thể bọc một `String` trong `Pin`, như trong
Hình 17-8. Tuy nhiên, `String` tự động triển khai `Unpin`, cũng như hầu hết các
kiểu khác trong Rust.

<figure>

<img alt="Concurrent work flow" src="img/trpl17-08.svg" class="center" />

<figcaption>Hình 17-8: Ghim một `String`; đường đứt nét chỉ ra rằng `String` triển khai trait `Unpin`, và do đó không được ghim.</figcaption>

</figure>

Do đó, chúng ta có thể làm những điều mà sẽ bất hợp pháp nếu `String` triển khai
`!Unpin` thay thế, chẳng hạn như thay thế một chuỗi bằng một chuỗi khác tại
chính xác cùng một vị trí trong bộ nhớ như trong Hình 17-9. Điều này không vi
phạm hợp đồng `Pin`, bởi vì `String` không có tham chiếu nội bộ làm cho nó không
an toàn để di chuyển! Đó chính xác là lý do tại sao nó triển khai `Unpin` chứ
không phải `!Unpin`.

<figure>

<img alt="Concurrent work flow" src="img/trpl17-09.svg" class="center" />

<figcaption>Hình 17-9: Thay thế `String` bằng một `String` hoàn toàn khác trong bộ nhớ.</figcaption>

</figure>

Bây giờ chúng ta biết đủ để hiểu các lỗi được báo cáo cho lệnh gọi `join_all` từ
Listing 17-17. Ban đầu, chúng ta đã cố gắng di chuyển các future được tạo ra bởi
các async block vào một `Vec<Box<dyn Future<Output = ()>>>`, nhưng như chúng ta
đã thấy, những future đó có thể có tham chiếu nội bộ, vì vậy chúng không triển
khai `Unpin`. Chúng cần được ghim, và sau đó chúng ta có thể truyền kiểu `Pin`
vào `Vec`, tự tin rằng dữ liệu cơ bản trong các future sẽ _không_ bị di chuyển.

`Pin` và `Unpin` chủ yếu quan trọng cho việc xây dựng các thư viện cấp thấp hơn,
hoặc khi bạn đang xây dựng chính một runtime, hơn là cho mã Rust hàng ngày. Tuy
nhiên, khi bạn thấy các trait này trong thông báo lỗi, bây giờ bạn sẽ có một
hiểu biết tốt hơn về cách sửa mã của mình!

> Lưu ý: Sự kết hợp này của `Pin` và `Unpin` làm cho nó có thể triển khai an
> toàn toàn bộ một lớp các kiểu phức tạp trong Rust mà nếu không sẽ chứng minh
> đầy thách thức bởi vì chúng tự tham chiếu. Các kiểu yêu cầu `Pin` xuất hiện
> phổ biến nhất trong async Rust ngày nay, nhưng thỉnh thoảng, bạn có thể thấy
> chúng trong các ngữ cảnh khác.
>
> Các chi tiết cụ thể về cách `Pin` và `Unpin` hoạt động, và các quy tắc mà
> chúng được yêu cầu để tuân thủ, được đề cập rộng rãi trong tài liệu API cho
> `std::pin`, vì vậy nếu bạn quan tâm đến việc tìm hiểu thêm, đó là một nơi
> tuyệt vời để bắt đầu.
>
> Nếu bạn muốn hiểu cách mọi thứ hoạt động bên dưới nắp capo một cách chi tiết
> hơn, hãy xem Chương [2][under-the-hood] và [4][pinning] của [_Asynchronous
> Programming in Rust_][async-book].

### Trait `Stream`

Bây giờ bạn đã có một hiểu biết sâu sắc hơn về các trait `Future`, `Pin`, và
`Unpin`, chúng ta có thể chuyển sự chú ý của mình đến trait `Stream`. Như bạn đã
học được trước đó trong chương, streams tương tự như các iterator bất đồng bộ.
Không giống như `Iterator` và `Future`, tuy nhiên, `Stream` không có định nghĩa
trong thư viện chuẩn vào thời điểm viết bài này, nhưng _có_ một định nghĩa rất
phổ biến từ crate `futures` được sử dụng trong toàn bộ hệ sinh thái.

Hãy xem lại các định nghĩa của các trait `Iterator` và `Future` trước khi xem
xét cách một trait `Stream` có thể hợp nhất chúng lại với nhau. Từ `Iterator`,
chúng ta có ý tưởng về một chuỗi: phương thức `next` của nó cung cấp một
`Option<Self::Item>`. Từ `Future`, chúng ta có ý tưởng về sự sẵn sàng theo thời
gian: phương thức `poll` của nó cung cấp một `Poll<Self::Output>`. Để đại diện
cho một chuỗi các mục trở nên sẵn sàng theo thời gian, chúng ta định nghĩa một
trait `Stream` kết hợp các tính năng đó:

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Stream {
    type Item;

    fn poll_next(
        self: Pin<&mut Self>,
        cx: &mut Context<'_>
    ) -> Poll<Option<Self::Item>>;
}
```

Trait `Stream` định nghĩa một kiểu kết hợp được gọi là `Item` cho kiểu của các
mục được tạo ra bởi stream. Điều này tương tự như `Iterator`, nơi có thể có từ
không đến nhiều mục, và không giống như `Future`, nơi luôn có một `Output` duy
nhất, ngay cả khi đó là kiểu đơn vị `()`.

`Stream` cũng định nghĩa một phương thức để lấy các mục đó. Chúng ta gọi nó là
`poll_next`, để làm rõ rằng nó poll theo cùng cách `Future::poll` làm và tạo ra
một chuỗi các mục theo cùng cách `Iterator::next` làm. Kiểu trả về của nó kết
hợp `Poll` với `Option`. Kiểu bên ngoài là `Poll`, bởi vì nó phải được kiểm tra
sự sẵn sàng, giống như một future. Kiểu bên trong là `Option`, bởi vì nó cần
phải báo hiệu liệu có còn tin nhắn nữa hay không, giống như một iterator làm.

Một thứ gì đó rất giống với định nghĩa này sẽ có khả năng trở thành một phần của
thư viện chuẩn của Rust. Trong khi đó, nó là một phần của bộ công cụ của hầu hết
các runtime, vì vậy bạn có thể dựa vào nó, và mọi thứ chúng ta đề cập tiếp theo
nhìn chung sẽ áp dụng!

Trong ví dụ chúng ta đã thấy trong phần về streaming, tuy nhiên, chúng ta đã
không sử dụng `poll_next` _hoặc_ `Stream`, mà thay vào đó đã sử dụng `next` và
`StreamExt`. Chúng ta _có thể_ làm việc trực tiếp với API `poll_next` bằng cách
tự viết các máy trạng thái `Stream` của chúng ta, tất nhiên, giống như chúng ta
_có thể_ làm việc với futures trực tiếp thông qua phương thức `poll` của chúng.
Tuy nhiên, sử dụng `await` tốt hơn nhiều, và trait `StreamExt` cung cấp phương
thức `next` để chúng ta có thể làm điều đó:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-stream-ext/src/lib.rs:here}}
```

<!--
TODO: update this if/when tokio/etc. update their MSRV and switch to using async functions
in traits, since the lack thereof is the reason they do not yet have this.
-->

> Lưu ý: Định nghĩa thực tế mà chúng ta đã sử dụng trước đó trong chương trông
> hơi khác với điều này, bởi vì nó hỗ trợ các phiên bản của Rust chưa hỗ trợ
> việc sử dụng các hàm async trong traits. Kết quả là, nó trông như thế này:
>
> ```rust,ignore
> fn next(&mut self) -> Next<'_, Self> where Self: Unpin;
> ```
>
> Kiểu `Next` đó là một `struct` triển khai `Future` và cho phép chúng ta đặt
> tên cho lifetime của tham chiếu đến `self` với `Next<'_, Self>`, để `await` có
> thể làm việc với phương thức này.

Trait `StreamExt` cũng là nơi ở của tất cả các phương thức thú vị có sẵn để sử
dụng với streams. `StreamExt` được triển khai tự động cho mọi kiểu triển khai
`Stream`, nhưng các trait này được định nghĩa riêng biệt để cho phép cộng đồng
lặp lại trên các API tiện lợi mà không ảnh hưởng đến trait nền tảng.

Trong phiên bản của `StreamExt` được sử dụng trong crate `trpl`, trait không chỉ
định nghĩa phương thức `next` mà còn cung cấp một triển khai mặc định của `next`
xử lý đúng các chi tiết của việc gọi `Stream::poll_next`. Điều này có nghĩa là
ngay cả khi bạn cần viết kiểu dữ liệu streaming của riêng mình, bạn _chỉ_ phải
triển khai `Stream`, và sau đó bất kỳ ai sử dụng kiểu dữ liệu của bạn có thể sử
dụng `StreamExt` và các phương thức của nó với nó một cách tự động.

Đó là tất cả những gì chúng ta sắp đề cập cho các chi tiết cấp thấp hơn về các
trait này. Để kết thúc, hãy xem xét cách các futures (bao gồm streams), tasks,
và threads đều phù hợp với nhau!

[ch-18]: ch18-00-oop.html
[async-book]: https://rust-lang.github.io/async-book/
[under-the-hood]:
  https://rust-lang.github.io/async-book/02_execution/01_chapter.html
[pinning]: https://rust-lang.github.io/async-book/04_pinning/01_chapter.html
[first-async]: ch17-01-futures-and-syntax.html#our-first-async-program
[any-number-futures]:
  ch17-03-more-futures.html#working-with-any-number-of-futures
