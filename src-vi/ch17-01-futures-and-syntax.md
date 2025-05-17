## Futures và Cú Pháp Async

Các yếu tố chính của lập trình bất đồng bộ trong Rust là _futures_ và các từ
khóa `async` và `await` của Rust.

Một _future_ là một giá trị có thể chưa sẵn sàng ngay bây giờ nhưng sẽ trở nên
sẵn sàng tại một thời điểm nào đó trong tương lai. (Khái niệm này xuất hiện
trong nhiều ngôn ngữ, đôi khi dưới các tên khác như _task_ hoặc _promise_.) Rust
cung cấp một đặc tính `Future` như một khối xây dựng để các thao tác bất đồng bộ
khác nhau có thể được triển khai với các cấu trúc dữ liệu khác nhau nhưng với
một giao diện chung. Trong Rust, futures là các kiểu triển khai đặc tính
`Future`. Mỗi future chứa thông tin riêng về tiến trình đã được thực hiện và
"sẵn sàng" có nghĩa là gì.

Bạn có thể áp dụng từ khóa `async` cho các khối và hàm để chỉ định rằng chúng có
thể bị gián đoạn và tiếp tục. Trong một khối async hoặc hàm async, bạn có thể sử
dụng từ khóa `await` để _đợi một future_ (nghĩa là, đợi nó trở nên sẵn sàng).
Bất kỳ điểm nào mà bạn đợi một future trong một khối async hoặc hàm là một vị
trí tiềm năng để khối async hoặc hàm đó tạm dừng và tiếp tục. Quá trình kiểm tra
với một future để xem giá trị của nó đã có sẵn chưa được gọi là _polling_.

Một số ngôn ngữ khác, như C# và JavaScript, cũng sử dụng các từ khóa `async` và
`await` cho lập trình bất đồng bộ. Nếu bạn quen thuộc với những ngôn ngữ đó, bạn
có thể nhận thấy một số khác biệt đáng kể trong cách Rust thực hiện mọi thứ, bao
gồm cách nó xử lý cú pháp. Điều đó có lý do chính đáng, như chúng ta sẽ thấy!

Khi viết Rust bất đồng bộ, chúng ta sử dụng các từ khóa `async` và `await` trong
hầu hết thời gian. Rust biên dịch chúng thành mã tương đương sử dụng đặc tính
`Future`, giống như nó biên dịch vòng lặp `for` thành mã tương đương sử dụng đặc
tính `Iterator`. Tuy nhiên, vì Rust cung cấp đặc tính `Future`, bạn cũng có thể
triển khai nó cho các kiểu dữ liệu của riêng bạn khi cần. Nhiều hàm mà chúng ta
sẽ thấy trong suốt chương này trả về các kiểu có triển khai `Future` riêng của
chúng. Chúng ta sẽ quay lại định nghĩa của đặc tính vào cuối chương và đi sâu
vào cách nó hoạt động, nhưng đây là đủ chi tiết để giúp chúng ta tiếp tục tiến
về phía trước.

Điều này có thể cảm thấy hơi trừu tượng, vì vậy hãy viết chương trình bất đồng
bộ đầu tiên của chúng ta: một trình thu thập web nhỏ. Chúng ta sẽ truyền vào hai
URL từ dòng lệnh, lấy cả hai cùng một lúc, và trả về kết quả của URL nào hoàn
thành trước. Ví dụ này sẽ có khá nhiều cú pháp mới, nhưng đừng lo lắng — chúng
ta sẽ giải thích mọi thứ bạn cần biết khi tiến hành.

## Chương Trình Async Đầu Tiên Của Chúng Ta

Để giữ trọng tâm của chương này vào việc học async thay vì tung hứng các phần
của hệ sinh thái, chúng ta đã tạo ra crate `trpl` (`trpl` là viết tắt của "The
Rust Programming Language"). Nó tái xuất tất cả các kiểu, đặc tính và hàm mà bạn
sẽ cần, chủ yếu từ các crate [`futures`][futures-crate]<!-- ignore --> và
[`tokio`][tokio]<!-- ignore -->. Crate `futures` là nơi chính thức để thử nghiệm
mã async của Rust, và đó thực sự là nơi đặc tính `Future` được thiết kế ban đầu.
Tokio là runtime async được sử dụng rộng rãi nhất trong Rust hiện nay, đặc biệt
là cho các ứng dụng web. Có những runtime tuyệt vời khác, và chúng có thể phù
hợp hơn cho mục đích của bạn. Chúng ta sử dụng crate `tokio` bên dưới cho `trpl`
vì nó đã được kiểm tra kỹ lưỡng và được sử dụng rộng rãi.

Trong một số trường hợp, `trpl` cũng đổi tên hoặc bọc các API gốc để giúp bạn
tập trung vào các chi tiết liên quan đến chương này. Nếu bạn muốn hiểu những gì
crate làm, chúng tôi khuyến khích bạn kiểm tra [mã nguồn của
nó][crate-source]<!-- ignore -->. Bạn sẽ có thể thấy mỗi tái xuất đến từ crate
nào, và chúng tôi đã để lại các bình luận mở rộng giải thích những gì crate làm.

Tạo một dự án nhị phân mới có tên `hello-async` và thêm crate `trpl` làm phụ
thuộc:

```console
$ cargo new hello-async
$ cd hello-async
$ cargo add trpl
```

Bây giờ chúng ta có thể sử dụng các phần khác nhau được cung cấp bởi `trpl` để
viết chương trình async đầu tiên của chúng ta. Chúng ta sẽ xây dựng một công cụ
dòng lệnh nhỏ lấy hai trang web, kéo phần tử `<title>` từ mỗi trang, và in ra
tiêu đề của trang nào hoàn thành toàn bộ quá trình đó trước.

### Định Nghĩa Hàm page_title

Hãy bắt đầu bằng cách viết một hàm nhận một URL trang làm tham số, gửi yêu cầu
đến nó, và trả về văn bản của phần tử tiêu đề (xem Listing 17-1).

<Listing number="17-1" file-name="src/main.rs" caption="Định nghĩa một hàm async để lấy phần tử tiêu đề từ một trang HTML">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-01/src/main.rs:all}}
```

</Listing>

Đầu tiên, chúng ta định nghĩa một hàm có tên `page_title` và đánh dấu nó bằng từ
khóa `async`. Sau đó, chúng ta sử dụng hàm `trpl::get` để lấy bất kỳ URL nào
được truyền vào và thêm từ khóa `await` để đợi phản hồi. Để lấy văn bản của phản
hồi, chúng ta gọi phương thức `text` của nó, và một lần nữa đợi nó với từ khóa
`await`. Cả hai bước này đều bất đồng bộ. Đối với hàm `get`, chúng ta phải đợi
máy chủ gửi lại phần đầu tiên của phản hồi, bao gồm các tiêu đề HTTP, cookie,
v.v., và có thể được gửi riêng biệt với phần thân phản hồi. Đặc biệt nếu phần
thân rất lớn, có thể mất một thời gian để tất cả đến. Bởi vì chúng ta phải đợi
_toàn bộ_ phản hồi đến, phương thức `text` cũng là bất đồng bộ.

Chúng ta phải rõ ràng đợi cả hai future này, bởi vì futures trong Rust là _lười
biếng_: chúng không làm gì cho đến khi bạn yêu cầu chúng làm việc với từ khóa
`await`. (Thực tế, Rust sẽ hiển thị cảnh báo trình biên dịch nếu bạn không sử
dụng một future.) Điều này có thể nhắc bạn về cuộc thảo luận về iterators trong
Chương 13 ở phần [Xử Lý Một Loạt Các Mục Với
Iterators][iterators-lazy]<!-- ignore -->. Các iterator không làm gì trừ khi bạn
gọi phương thức `next` của chúng — dù trực tiếp hay bằng cách sử dụng vòng lặp
`for` hoặc các phương thức như `map` sử dụng `next` bên dưới. Tương tự, futures
không làm gì trừ khi bạn rõ ràng yêu cầu chúng làm. Tính lười biếng này cho phép
Rust tránh chạy mã async cho đến khi nó thực sự cần thiết.

> Lưu ý: Điều này khác với hành vi chúng ta đã thấy trong chương trước khi sử
> dụng `thread::spawn` trong [Tạo Một Thread Mới với
> spawn][thread-spawn]<!--ignore-->, nơi closure mà chúng ta chuyển cho một
> thread khác bắt đầu chạy ngay lập tức. Nó cũng khác với cách nhiều ngôn ngữ
> khác tiếp cận async. Nhưng nó quan trọng để Rust có thể cung cấp các đảm bảo
> hiệu suất của nó, giống như với iterators.

Khi chúng ta có `response_text`, chúng ta có thể phân tích nó thành một instance
của kiểu `Html` bằng cách sử dụng `Html::parse`. Thay vì một chuỗi thô, bây giờ
chúng ta có một kiểu dữ liệu có thể sử dụng để làm việc với HTML như một cấu
trúc dữ liệu phong phú hơn. Cụ thể, chúng ta có thể sử dụng phương thức
`select_first` để tìm instance đầu tiên của một bộ chọn CSS. Bằng cách truyền
chuỗi `"title"`, chúng ta sẽ nhận được phần tử `<title>` đầu tiên trong tài
liệu, nếu có. Bởi vì có thể không có phần tử nào phù hợp, `select_first` trả về
một `Option<ElementRef>`. Cuối cùng, chúng ta sử dụng phương thức `Option::map`,
cho phép chúng ta làm việc với mục trong `Option` nếu nó hiện diện, và không làm
gì nếu không. (Chúng ta cũng có thể sử dụng biểu thức `match` ở đây, nhưng `map`
thường được sử dụng hơn.) Trong phần thân của hàm mà chúng ta cung cấp cho
`map`, chúng ta gọi `inner_html` trên `title` để lấy nội dung của nó, đó là một
`String`. Khi tất cả đã nói và làm, chúng ta có một `Option<String>`.

Lưu ý rằng từ khóa `await` của Rust đi _sau_ biểu thức bạn đang đợi, không phải
trước nó. Nghĩa là, nó là một từ khóa _hậu tố_. Điều này có thể khác với những
gì bạn quen thuộc nếu bạn đã sử dụng `async` trong các ngôn ngữ khác, nhưng
trong Rust nó làm cho chuỗi các phương thức dễ dàng làm việc hơn nhiều. Kết quả
là, chúng ta có thể thay đổi phần thân của `page_title` để nối các lệnh gọi hàm
`trpl::get` và `text` với nhau với `await` ở giữa, như được hiển thị trong
Listing 17-2.

<Listing number="17-2" file-name="src/main.rs" caption="Nối chuỗi với từ khóa `await`">

```rust
{{#rustdoc_include ../listings/ch17-async-await/listing-17-02/src/main.rs:chaining}}
```

</Listing>

Với điều đó, chúng ta đã viết thành công hàm async đầu tiên của mình! Trước khi
chúng ta thêm một số mã trong `main` để gọi nó, hãy nói thêm một chút về những
gì chúng ta đã viết và ý nghĩa của nó.

Khi Rust thấy một khối được đánh dấu bằng từ khóa `async`, nó biên dịch nó thành
một kiểu dữ liệu duy nhất, ẩn danh triển khai đặc tính `Future`. Khi Rust thấy
một hàm được đánh dấu bằng `async`, nó biên dịch nó thành một hàm không bất đồng
bộ có phần thân là một khối async. Kiểu trả về của một hàm async là kiểu của
kiểu dữ liệu ẩn danh mà trình biên dịch tạo ra cho khối async đó.

Do đó, viết `async fn` tương đương với viết một hàm trả về một _future_ của kiểu
trả về. Đối với trình biên dịch, một định nghĩa hàm như `async fn page_title`
trong Listing 17-1 tương đương với một hàm không bất đồng bộ được định nghĩa như
sau:

```rust
# extern crate trpl; // required for mdbook test
use std::future::Future;
use trpl::Html;

fn page_title(url: &str) -> impl Future<Output = Option<String>> {
    async move {
        let text = trpl::get(url).await.text().await;
        Html::parse(&text)
            .select_first("title")
            .map(|title| title.inner_html())
    }
}
```

Hãy đi qua từng phần của phiên bản được chuyển đổi:

- Nó sử dụng cú pháp `impl Trait` mà chúng ta đã thảo luận trở lại Chương 10
  trong phần ["Đặc Tính như Tham Số"][impl-trait]<!-- ignore -->.
- Đặc tính được trả về là một `Future` với một kiểu liên kết là `Output`. Lưu ý
  rằng kiểu `Output` là `Option<String>`, giống với kiểu trả về ban đầu từ phiên
  bản `async fn` của `page_title`.
- Tất cả mã được gọi trong phần thân của hàm ban đầu được bọc trong một khối
  `async move`. Hãy nhớ rằng các khối là biểu thức. Toàn bộ khối này là biểu
  thức được trả về từ hàm.
- Khối async này tạo ra một giá trị với kiểu `Option<String>`, như vừa mô tả.
  Giá trị đó khớp với kiểu `Output` trong kiểu trả về. Điều này giống như các
  khối khác bạn đã thấy.
- Phần thân hàm mới là một khối `async move` vì cách nó sử dụng tham số `url`.
  (Chúng ta sẽ nói nhiều hơn về `async` so với `async move` sau trong chương.)

Bây giờ chúng ta có thể gọi `page_title` trong `main`.

## Xác Định Tiêu Đề Một Trang Duy Nhất

Để bắt đầu, chúng ta sẽ chỉ lấy tiêu đề cho một trang duy nhất. Trong Listing
17-3, chúng ta theo cùng một mẫu mà chúng ta đã sử dụng trong Chương 12 để lấy
đối số dòng lệnh trong phần [Chấp Nhận Đối Số Dòng
Lệnh][cli-args]<!-- ignore -->. Sau đó, chúng ta truyền URL đầu tiên cho
`page_title` và đợi kết quả. Bởi vì giá trị được tạo ra bởi future là một
`Option<String>`, chúng ta sử dụng biểu thức `match` để in các thông báo khác
nhau để tính đến việc liệu trang có `<title>` hay không.

<Listing number="17-3" file-name="src/main.rs" caption="Gọi hàm `page_title` từ `main` với một đối số do người dùng cung cấp">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch17-async-await/listing-17-03/src/main.rs:main}}
```

</Listing>

Thật không may, mã này không biên dịch. Nơi duy nhất chúng ta có thể sử dụng từ
khóa `await` là trong các hàm hoặc khối async, và Rust sẽ không cho phép chúng
ta đánh dấu hàm đặc biệt `main` là `async`.

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-03
cargo build
copy just the compiler error
-->

```text
error[E0752]: `main` function is not allowed to be `async`
 --> src/main.rs:6:1
  |
6 | async fn main() {
  | ^^^^^^^^^^^^^^^ `main` function is not allowed to be `async`
```

Lý do `main` không thể được đánh dấu `async` là vì mã async cần một _runtime_:
một crate Rust quản lý các chi tiết của việc thực thi mã bất đồng bộ. Hàm `main`
của một chương trình có thể _khởi tạo_ một runtime, nhưng nó không phải là một
runtime _bản thân nó_. (Chúng ta sẽ thấy thêm về lý do tại sao trong một chút.)
Mỗi chương trình Rust thực thi mã async có ít nhất một nơi mà nó thiết lập
runtime và thực thi các futures.

Hầu hết các ngôn ngữ hỗ trợ async đều gói gọn một runtime, nhưng Rust thì không.
Thay vào đó, có nhiều runtime async khác nhau có sẵn, mỗi cái thực hiện những
đánh đổi khác nhau phù hợp với trường hợp sử dụng mà nó nhắm tới. Ví dụ, một máy
chủ web có thông lượng cao với nhiều lõi CPU và một lượng lớn RAM có nhu cầu rất
khác so với một vi điều khiển với một lõi duy nhất, một lượng nhỏ RAM, và không
có khả năng cấp phát heap. Các crate cung cấp các runtime đó cũng thường cung
cấp các phiên bản async của các chức năng phổ biến như I/O tệp hoặc mạng.

Ở đây, và trong suốt phần còn lại của chương này, chúng ta sẽ sử dụng hàm `run`
từ crate `trpl`, lấy một future làm đối số và chạy nó đến khi hoàn thành. Đằng
sau hậu trường, gọi `run` thiết lập một runtime được sử dụng để chạy future được
truyền vào. Một khi future hoàn thành, `run` trả về bất cứ giá trị nào mà future
đã tạo ra.

Chúng ta có thể truyền future được trả về bởi `page_title` trực tiếp cho `run`,
và một khi nó hoàn thành, chúng ta có thể khớp trên `Option<String>` kết quả,
như chúng ta đã cố gắng làm trong Listing 17-3. Tuy nhiên, đối với hầu hết các
ví dụ trong chương (và hầu hết mã async trong thế giới thực), chúng ta sẽ làm
nhiều hơn là chỉ một lệnh gọi hàm async, vì vậy thay vào đó chúng ta sẽ truyền
một khối `async` và rõ ràng đợi kết quả của lệnh gọi `page_title`, như trong
Listing 17-4.

<Listing number="17-4" caption="Đợi một khối async với `trpl::run`" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook test does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-04/src/main.rs:run}}
```

</Listing>

Khi chúng ta chạy mã này, chúng ta nhận được hành vi mà chúng ta mong đợi ban
đầu:

<!-- manual-regeneration
cd listings/ch17-async-await/listing-17-04
cargo build # skip all the build noise
cargo run https://www.rust-lang.org
# copy the output here
-->

```console
$ cargo run -- https://www.rust-lang.org
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.05s
     Running `target/debug/async_await 'https://www.rust-lang.org'`
The title for https://www.rust-lang.org was
            Rust Programming Language
```

Phù—cuối cùng chúng ta đã có một số mã async hoạt động! Nhưng trước khi chúng ta
thêm mã để đua hai trang web với nhau, hãy nhanh chóng trở lại cách futures hoạt
động.

Mỗi _điểm đợi_—nghĩa là, mỗi nơi mà mã sử dụng từ khóa `await`—đại diện cho một
nơi mà quyền kiểm soát được trao lại cho runtime. Để làm cho điều đó hoạt động,
Rust cần phải theo dõi trạng thái liên quan đến khối async để runtime có thể
khởi động một số công việc khác và sau đó quay lại khi nó sẵn sàng để cố gắng
tiếp tục cái đầu tiên. Đây là một máy trạng thái vô hình, như thể bạn đã viết
một enum như thế này để lưu trạng thái hiện tại tại mỗi điểm đợi:

```rust
{{#rustdoc_include ../listings/ch17-async-await/no-listing-state-machine/src/lib.rs:enum}}
```

Tuy nhiên, viết mã để chuyển đổi giữa mỗi trạng thái bằng tay sẽ là tẻ nhạt và
dễ bị lỗi, đặc biệt là khi bạn cần thêm nhiều chức năng và nhiều trạng thái vào
mã sau này. May mắn thay, trình biên dịch Rust tạo và quản lý các cấu trúc dữ
liệu máy trạng thái cho mã async tự động. Các quy tắc bình thường về việc mượn
và sở hữu xung quanh các cấu trúc dữ liệu vẫn áp dụng, và may mắn thay, trình
biên dịch cũng xử lý việc kiểm tra những điều đó cho chúng ta và cung cấp các
thông báo lỗi hữu ích. Chúng ta sẽ làm việc thông qua một vài điều đó sau trong
chương.

Cuối cùng, một cái gì đó phải thực thi máy trạng thái này, và thứ đó là một
runtime. (Đây là lý do tại sao bạn có thể gặp phải các tham chiếu đến
_executors_ khi tìm hiểu về runtimes: một executor là phần của runtime chịu
trách nhiệm thực thi mã async.)

Bây giờ bạn có thể thấy tại sao trình biên dịch đã ngăn chúng ta làm cho `main`
bản thân là một hàm async trở lại trong Listing 17-3. Nếu `main` là một hàm
async, một cái gì đó khác sẽ cần quản lý máy trạng thái cho bất kỳ future nào
`main` trả về, nhưng `main` là điểm bắt đầu của chương trình! Thay vào đó, chúng
ta đã gọi hàm `trpl::run` trong `main` để thiết lập một runtime và chạy future
được trả về bởi khối `async` cho đến khi nó hoàn thành.

> Lưu ý: Một số runtime cung cấp các macro để bạn _có thể_ viết một hàm `main`
> async. Những macro đó viết lại `async fn main() { ... }` để là một `fn main`
> bình thường, làm cùng một việc mà chúng ta đã làm bằng tay trong Listing 17-4:
> gọi một hàm chạy một future đến khi hoàn thành giống như `trpl::run` làm.

Bây giờ hãy kết hợp các phần này lại với nhau và xem cách chúng ta có thể viết
mã đồng thời.

### Đua Hai URL Của Chúng Ta Với Nhau

Trong Listing 17-5, chúng ta gọi `page_title` với hai URL khác nhau được truyền
vào từ dòng lệnh và đua chúng.

<Listing number="17-5" caption="" file-name="src/main.rs">

<!-- should_panic,noplayground because mdbook does not pass args -->

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch17-async-await/listing-17-05/src/main.rs:all}}
```

</Listing>

Chúng ta bắt đầu bằng cách gọi `page_title` cho mỗi URL do người dùng cung cấp.
Chúng ta lưu các future kết quả là `title_fut_1` và `title_fut_2`. Hãy nhớ rằng,
chúng không làm gì cả, bởi vì futures là lười biếng và chúng ta chưa đợi chúng.
Sau đó, chúng ta truyền các future cho `trpl::race`, trả về một giá trị để chỉ
ra futures nào trong số các futures được truyền cho nó hoàn thành đầu tiên.

> Lưu ý: Bên dưới, `race` được xây dựng trên một hàm tổng quát hơn, `select`, mà
> bạn sẽ gặp nhiều hơn trong mã Rust thực tế. Một hàm `select` có thể làm nhiều
> thứ mà hàm `trpl::race` không thể, nhưng nó cũng có một số phức tạp bổ sung mà
> chúng ta có thể bỏ qua bây giờ.

Một trong hai future có thể "thắng" một cách hợp pháp, vì vậy sẽ không hợp lý
khi trả về một `Result`. Thay vào đó, `race` trả về một kiểu mà chúng ta chưa
thấy trước đây, `trpl::Either`. Kiểu `Either` hơi giống với một `Result` ở chỗ
nó có hai trường hợp. Không giống như `Result`, tuy nhiên, không có khái niệm
thành công hay thất bại được nướng vào `Either`. Thay vào đó, nó sử dụng `Left`
và `Right` để chỉ "một hoặc cái kia":

```rust
enum Either<A, B> {
    Left(A),
    Right(B),
}
```

Hàm `race` trả về `Left` với đầu ra từ đối số future đầu tiên nếu nó kết thúc
trước, hoặc `Right` với đầu ra của đối số future thứ hai nếu đó là cái kết thúc
trước. Điều này phù hợp với thứ tự các đối số xuất hiện khi gọi hàm: đối số đầu
tiên nằm ở bên trái của đối số thứ hai.

Chúng ta cũng cập nhật `page_title` để trả về cùng URL đã truyền vào. Bằng cách
đó, nếu trang trả về đầu tiên không có `<title>` mà chúng ta có thể giải quyết,
chúng ta vẫn có thể in một thông báo có ý nghĩa. Với thông tin đó có sẵn, chúng
ta kết thúc bằng cách cập nhật đầu ra `println!` của chúng ta để chỉ ra cả URL
nào kết thúc trước và `<title>` là gì, nếu có, cho trang web tại URL đó.

Bây giờ bạn đã xây dựng một trình thu thập web nhỏ hoạt động! Chọn một cặp URL
và chạy công cụ dòng lệnh. Bạn có thể khám phá ra rằng một số trang web nhất
quán nhanh hơn các trang khác, trong khi trong các trường hợp khác, trang nhanh
hơn thay đổi từ lần chạy này sang lần chạy khác. Quan trọng hơn, bạn đã học được
những điều cơ bản về làm việc với futures, vì vậy bây giờ chúng ta có thể đi sâu
hơn vào những gì chúng ta có thể làm với async.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
[iterators-lazy]: ch13-02-iterators.html
[thread-spawn]: ch16-01-threads.html#creating-a-new-thread-with-spawn
[cli-args]: ch12-01-accepting-command-line-arguments.html

<!-- TODO: map source link version to version of Rust? -->

[crate-source]: https://github.com/rust-lang/book/tree/main/packages/trpl
[futures-crate]: https://crates.io/crates/futures
[tokio]: https://tokio.rs
