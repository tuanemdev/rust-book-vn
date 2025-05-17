## Traits: Định Nghĩa Hành Vi Được Chia Sẻ

Một _trait_ định nghĩa chức năng mà một kiểu cụ thể có và có thể chia sẻ với các
kiểu khác. Chúng ta có thể sử dụng traits để định nghĩa hành vi được chia sẻ
theo cách trừu tượng. Chúng ta có thể sử dụng _trait bounds_ để chỉ định rằng
một kiểu generic có thể là bất kỳ kiểu nào có hành vi nhất định.

> Lưu ý: Traits tương tự với một tính năng thường được gọi là _interfaces_ trong
> các ngôn ngữ khác, mặc dù có một số khác biệt.

### Định Nghĩa một Trait

Hành vi của một kiểu bao gồm các phương thức mà chúng ta có thể gọi trên kiểu
đó. Các kiểu khác nhau chia sẻ cùng một hành vi nếu chúng ta có thể gọi các
phương thức giống nhau trên tất cả các kiểu đó. Định nghĩa trait là một cách để
nhóm các chữ ký phương thức lại với nhau để định nghĩa một tập hợp các hành vi
cần thiết để hoàn thành một mục đích nào đó.

Ví dụ, giả sử chúng ta có nhiều struct chứa các loại và số lượng văn bản khác
nhau: một struct `NewsArticle` chứa một câu chuyện tin tức được lưu trữ ở một vị
trí cụ thể và một `SocialPost` có thể có, nhiều nhất là 280 ký tự cùng với
metadata cho biết liệu đó là một bài viết mới, một bài đăng lại, hoặc một trả
lời cho bài viết khác.

Chúng ta muốn tạo một thư viện tổng hợp phương tiện có tên là `aggregator` có
thể hiển thị tóm tắt dữ liệu có thể được lưu trữ trong một thể hiện
`NewsArticle` hoặc `SocialPost`. Để làm điều này, chúng ta cần một bản tóm tắt
từ mỗi loại, và chúng ta sẽ yêu cầu bản tóm tắt đó bằng cách gọi phương thức
`summarize` trên một thể hiện. Listing 10-12 hiển thị định nghĩa của một trait
`Summary` công khai thể hiện hành vi này.

<Listing number="10-12" file-name="src/lib.rs" caption="Một trait `Summary` bao gồm hành vi được cung cấp bởi phương thức `summarize`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-12/src/lib.rs}}
```

</Listing>

Ở đây, chúng ta khai báo một trait bằng cách sử dụng từ khóa `trait` và sau đó
là tên của trait, là `Summary` trong trường hợp này. Chúng ta cũng khai báo
trait là `pub` để các crate phụ thuộc vào crate này cũng có thể sử dụng trait
này, như chúng ta sẽ thấy trong một vài ví dụ. Bên trong dấu ngoặc nhọn, chúng
ta khai báo các chữ ký phương thức mô tả các hành vi của các kiểu thực hiện
trait này, trong trường hợp này là `fn summarize(&self) -> String`.

Sau chữ ký phương thức, thay vì cung cấp một triển khai trong dấu ngoặc nhọn,
chúng ta sử dụng dấu chấm phẩy. Mỗi kiểu triển khai trait này phải cung cấp hành
vi tùy chỉnh riêng của nó cho phần thân của phương thức. Trình biên dịch sẽ áp
dụng rằng bất kỳ kiểu nào có trait `Summary` sẽ phải có phương thức `summarize`
được định nghĩa với chữ ký này một cách chính xác.

Một trait có thể có nhiều phương thức trong thân: các chữ ký phương thức được
liệt kê mỗi phương thức một dòng, và mỗi dòng kết thúc bằng dấu chấm phẩy.

### Triển Khai một Trait trên một Kiểu

Bây giờ chúng ta đã định nghĩa các chữ ký mong muốn của các phương thức của
trait `Summary`, chúng ta có thể triển khai nó trên các kiểu trong trình tổng
hợp phương tiện của chúng ta. Listing 10-13 hiển thị một triển khai của trait
`Summary` trên struct `NewsArticle` sử dụng tiêu đề, tác giả và vị trí để tạo
giá trị trả về của `summarize`. Đối với struct `SocialPost`, chúng ta định nghĩa
`summarize` là tên người dùng theo sau là toàn bộ văn bản của bài viết, giả định
rằng nội dung bài viết đã giới hạn ở 280 ký tự.

<Listing number="10-13" file-name="src/lib.rs" caption="Triển khai trait `Summary` trên các kiểu `NewsArticle` và `SocialPost`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-13/src/lib.rs:here}}
```

</Listing>

Triển khai một trait trên một kiểu tương tự như triển khai các phương thức thông
thường. Sự khác biệt là sau `impl`, chúng ta đặt tên trait mà chúng ta muốn
triển khai, sau đó sử dụng từ khóa `for`, và sau đó chỉ định tên của kiểu mà
chúng ta muốn triển khai trait cho. Trong khối `impl`, chúng ta đặt các chữ ký
phương thức mà định nghĩa trait đã định nghĩa. Thay vì thêm dấu chấm phẩy sau
mỗi chữ ký, chúng ta sử dụng dấu ngoặc nhọn và điền vào phần thân phương thức
với hành vi cụ thể mà chúng ta muốn các phương thức của trait có đối với kiểu cụ
thể.

Bây giờ thư viện đã triển khai trait `Summary` trên `NewsArticle` và
`SocialPost`, người dùng của crate có thể gọi các phương thức trait trên các thể
hiện của `NewsArticle` và `SocialPost` giống như cách chúng ta gọi các phương
thức thông thường. Sự khác biệt duy nhất là người dùng phải đưa trait vào phạm
vi cũng như các kiểu. Đây là một ví dụ về cách một binary crate có thể sử dụng
thư viện `aggregator` của chúng ta:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-01-calling-trait-method/src/main.rs}}
```

Mã này in ra
`1 new post: horse_ebooks: of course, as you probably already know, people`.

Các crate khác phụ thuộc vào crate `aggregator` cũng có thể đưa trait `Summary`
vào phạm vi để triển khai `Summary` trên các kiểu riêng của họ. Một hạn chế cần
lưu ý là chúng ta chỉ có thể triển khai một trait trên một kiểu nếu hoặc trait
hoặc kiểu, hoặc cả hai, là local đối với crate của chúng ta. Ví dụ, chúng ta có
thể triển khai các trait của thư viện tiêu chuẩn như `Display` trên một kiểu tùy
chỉnh như `SocialPost` như một phần của chức năng crate `aggregator` của chúng
ta vì kiểu `SocialPost` là local đối với crate `aggregator` của chúng ta. Chúng
ta cũng có thể triển khai `Summary` trên `Vec<T>` trong crate `aggregator` của
chúng ta vì trait `Summary` là local đối với crate `aggregator` của chúng ta.

Nhưng chúng ta không thể triển khai các trait bên ngoài trên các kiểu bên ngoài.
Ví dụ, chúng ta không thể triển khai trait `Display` trên `Vec<T>` trong crate
`aggregator` của chúng ta vì `Display` và `Vec<T>` đều được định nghĩa trong thư
viện tiêu chuẩn và không local đối với crate `aggregator` của chúng ta. Hạn chế
này là một phần của thuộc tính được gọi là _coherence_, và cụ thể hơn là _quy
tắc mồ côi_, được đặt tên như vậy bởi vì kiểu cha không có mặt. Quy tắc này đảm
bảo rằng mã của người khác không thể phá vỡ mã của bạn và ngược lại. Nếu không
có quy tắc này, hai crate có thể triển khai cùng một trait cho cùng một kiểu, và
Rust sẽ không biết triển khai nào để sử dụng.

### Các Triển Khai Mặc Định

Đôi khi việc có hành vi mặc định cho một số hoặc tất cả các phương thức trong
một trait thay vì yêu cầu triển khai cho tất cả các phương thức trên mọi kiểu là
hữu ích. Sau đó, khi chúng ta triển khai trait trên một kiểu cụ thể, chúng ta có
thể giữ nguyên hoặc ghi đè hành vi mặc định của từng phương thức.

Trong Listing 10-14, chúng ta chỉ định một chuỗi mặc định cho phương thức
`summarize` của trait `Summary` thay vì chỉ định nghĩa chữ ký phương thức, như
chúng ta đã làm trong Listing 10-12.

<Listing number="10-14" file-name="src/lib.rs" caption="Định nghĩa một trait `Summary` với một triển khai mặc định của phương thức `summarize`">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-14/src/lib.rs:here}}
```

</Listing>

Để sử dụng một triển khai mặc định để tóm tắt các thể hiện của `NewsArticle`,
chúng ta chỉ định một khối `impl` trống với `impl Summary for NewsArticle {}`.

Mặc dù chúng ta không còn định nghĩa phương thức `summarize` trên `NewsArticle`
trực tiếp, chúng ta đã cung cấp một triển khai mặc định và chỉ định rằng
`NewsArticle` triển khai trait `Summary`. Kết quả là, chúng ta vẫn có thể gọi
phương thức `summarize` trên một thể hiện của `NewsArticle`, như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-02-calling-default-impl/src/main.rs:here}}
```

Mã này in ra `New article available! (Read more...)`.

Việc tạo một triển khai mặc định không yêu cầu chúng ta thay đổi bất cứ điều gì
về việc triển khai `Summary` trên `SocialPost` trong Listing 10-13. Lý do là cú
pháp để ghi đè một triển khai mặc định giống với cú pháp để triển khai một
phương thức trait không có triển khai mặc định.

Các triển khai mặc định có thể gọi các phương thức khác trong cùng trait, ngay
cả khi những phương thức khác đó không có triển khai mặc định. Bằng cách này,
một trait có thể cung cấp nhiều chức năng hữu ích và chỉ yêu cầu những người
triển khai chỉ định một phần nhỏ của nó. Ví dụ, chúng ta có thể định nghĩa trait
`Summary` có một phương thức `summarize_author` mà triển khai của nó là bắt
buộc, và sau đó định nghĩa một phương thức `summarize` có triển khai mặc định
gọi phương thức `summarize_author`:

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:here}}
```

Để sử dụng phiên bản này của `Summary`, chúng ta chỉ cần định nghĩa
`summarize_author` khi triển khai trait trên một kiểu:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/lib.rs:impl}}
```

Sau khi chúng ta định nghĩa `summarize_author`, chúng ta có thể gọi `summarize`
trên các thể hiện của struct `SocialPost`, và triển khai mặc định của
`summarize` sẽ gọi định nghĩa của `summarize_author` mà chúng ta đã cung cấp.
Bởi vì chúng ta đã triển khai `summarize_author`, trait `Summary` đã cung cấp
cho chúng ta hành vi của phương thức `summarize` mà không yêu cầu chúng ta viết
thêm bất kỳ mã nào. Đây là cách nó hoạt động:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-03-default-impl-calls-other-methods/src/main.rs:here}}
```

Mã này in ra `1 new post: (Read more from @horse_ebooks...)`.

Lưu ý rằng không thể gọi triển khai mặc định từ một triển khai ghi đè của cùng
một phương thức.

### Traits như Tham Số

Bây giờ bạn đã biết cách định nghĩa và triển khai traits, chúng ta có thể khám
phá cách sử dụng traits để định nghĩa các hàm chấp nhận nhiều kiểu khác nhau.
Chúng ta sẽ sử dụng trait `Summary` mà chúng ta đã triển khai trên các kiểu
`NewsArticle` và `SocialPost` trong Listing 10-13 để định nghĩa một hàm `notify`
gọi phương thức `summarize` trên tham số `item` của nó, là một kiểu nào đó triển
khai trait `Summary`. Để làm điều này, chúng ta sử dụng cú pháp `impl Trait`,
như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-04-traits-as-parameters/src/lib.rs:here}}
```

Thay vì một kiểu cụ thể cho tham số `item`, chúng ta chỉ định từ khóa `impl` và
tên trait. Tham số này chấp nhận bất kỳ kiểu nào triển khai trait được chỉ định.
Trong thân của `notify`, chúng ta có thể gọi bất kỳ phương thức nào trên `item`
mà đến từ trait `Summary`, chẳng hạn như `summarize`. Chúng ta có thể gọi
`notify` và truyền vào bất kỳ thể hiện nào của `NewsArticle` hoặc `SocialPost`.
Mã gọi hàm với bất kỳ kiểu nào khác, chẳng hạn như `String` hoặc `i32`, sẽ không
biên dịch vì các kiểu đó không triển khai `Summary`.

<!-- Old headings. Do not remove or links may break. -->

<a id="fixing-the-largest-function-with-trait-bounds"></a>

#### Cú Pháp Trait Bound

Cú pháp `impl Trait` hoạt động cho các trường hợp đơn giản nhưng thực sự là cú
pháp đường tắt cho một hình thức dài hơn được gọi là _trait bound_; nó trông như
thế này:

```rust,ignore
pub fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

Hình thức dài hơn này tương đương với ví dụ trong phần trước nhưng dài dòng hơn.
Chúng ta đặt các trait bound với khai báo của tham số kiểu generic sau dấu hai
chấm và bên trong dấu ngoặc nhọn.

Cú pháp `impl Trait` thuận tiện và tạo ra mã ngắn gọn hơn trong các trường hợp
đơn giản, trong khi cú pháp trait bound đầy đủ hơn có thể biểu đạt phức tạp hơn
trong các trường hợp khác. Ví dụ, chúng ta có thể có hai tham số triển khai
`Summary`. Thực hiện điều này với cú pháp `impl Trait` trông như thế này:

```rust,ignore
pub fn notify(item1: &impl Summary, item2: &impl Summary) {
```

Sử dụng `impl Trait` là phù hợp nếu chúng ta muốn hàm này cho phép `item1` và
`item2` có các kiểu khác nhau (miễn là cả hai kiểu đều triển khai `Summary`).
Nếu chúng ta muốn buộc cả hai tham số phải có cùng kiểu, tuy nhiên, chúng ta
phải sử dụng một trait bound, như thế này:

```rust,ignore
pub fn notify<T: Summary>(item1: &T, item2: &T) {
```

Kiểu generic `T` được chỉ định làm kiểu của các tham số `item1` và `item2` ràng
buộc hàm sao cho kiểu cụ thể của giá trị được truyền dưới dạng đối số cho
`item1` và `item2` phải giống nhau.

#### Chỉ Định Nhiều Trait Bounds với Cú Pháp `+`

Chúng ta cũng có thể chỉ định nhiều hơn một trait bound. Giả sử chúng ta muốn
`notify` sử dụng định dạng hiển thị cũng như `summarize` trên `item`: chúng ta
chỉ định trong định nghĩa `notify` rằng `item` phải triển khai cả `Display` và
`Summary`. Chúng ta có thể làm như vậy bằng cách sử dụng cú pháp `+`:

```rust,ignore
pub fn notify(item: &(impl Summary + Display)) {
```

Cú pháp `+` cũng hợp lệ với trait bounds trên các kiểu generic:

```rust,ignore
pub fn notify<T: Summary + Display>(item: &T) {
```

Với hai trait bounds đã được chỉ định, thân của `notify` có thể gọi `summarize`
và sử dụng `{}` để định dạng `item`.

#### Trait Bounds Rõ Ràng Hơn với Mệnh Đề `where`

Việc sử dụng quá nhiều trait bounds có nhược điểm. Mỗi generic có trait bound
riêng của nó, vì vậy các hàm với nhiều tham số kiểu generic có thể chứa rất
nhiều thông tin trait bound giữa tên của hàm và danh sách tham số của nó, làm
cho chữ ký hàm khó đọc. Vì lý do này, Rust có cú pháp thay thế để chỉ định trait
bounds bên trong một mệnh đề `where` sau chữ ký hàm. Vì vậy, thay vì viết như
thế này:

```rust,ignore
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
```

chúng ta có thể sử dụng một mệnh đề `where`, như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-07-where-clause/src/lib.rs:here}}
```

Chữ ký của hàm này ít rối hơn: tên hàm, danh sách tham số, và kiểu trả về ở gần
nhau, tương tự như một hàm không có nhiều trait bounds.

### Trả Về các Kiểu Triển Khai Traits

Chúng ta cũng có thể sử dụng cú pháp `impl Trait` ở vị trí trả về để trả về một
giá trị của một số kiểu triển khai một trait, như được hiển thị ở đây:

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-05-returning-impl-trait/src/lib.rs:here}}
```

Bằng cách sử dụng `impl Summary` cho kiểu trả về, chúng ta chỉ định rằng hàm
`returns_summarizable` trả về một số kiểu triển khai trait `Summary` mà không
cần đặt tên kiểu cụ thể. Trong trường hợp này, `returns_summarizable` trả về một
`SocialPost`, nhưng mã gọi hàm này không cần biết điều đó.

Khả năng chỉ định một kiểu trả về chỉ bằng trait mà nó triển khai đặc biệt hữu
ích trong ngữ cảnh của closures và iterators, mà chúng ta sẽ đề cập trong
Chương 13. Closures và iterators tạo ra các kiểu mà chỉ trình biên dịch biết
hoặc các kiểu rất dài để chỉ định. Cú pháp `impl Trait` cho phép bạn ngắn gọn
chỉ định rằng một hàm trả về một số kiểu triển khai trait `Iterator` mà không
cần viết ra một kiểu rất dài.

Tuy nhiên, bạn chỉ có thể sử dụng `impl Trait` nếu bạn đang trả về một kiểu duy
nhất. Ví dụ, mã này trả về một `NewsArticle` hoặc một `SocialPost` với kiểu trả
về được chỉ định là `impl Summary` sẽ không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-06-impl-trait-returns-one-type/src/lib.rs:here}}
```

Việc trả về một `NewsArticle` hoặc một `SocialPost` không được phép do các hạn
chế xung quanh cách cú pháp `impl Trait` được triển khai trong trình biên dịch.
Chúng ta sẽ đề cập đến cách viết một hàm với hành vi này trong phần ["Sử dụng
Trait Objects Cho Phép các Giá Trị của Các Kiểu Khác
Nhau"][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore
--> của Chương 18.

### Sử Dụng Trait Bounds để Triển Khai Các Phương Thức Có Điều Kiện

Bằng cách sử dụng một trait bound với một khối `impl` sử dụng các tham số kiểu
generic, chúng ta có thể triển khai các phương thức có điều kiện cho các kiểu
triển khai các traits được chỉ định. Ví dụ, kiểu `Pair<T>` trong Listing 10-15
luôn triển khai hàm `new` để trả về một thể hiện mới của `Pair<T>` (nhớ lại từ
phần ["Định Nghĩa Phương Thức"][methods]<!-- ignore --> của Chương 5 rằng `Self`
là một bí danh kiểu cho kiểu của khối `impl`, trong trường hợp này là
`Pair<T>`). Nhưng trong khối `impl` tiếp theo, `Pair<T>` chỉ triển khai phương
thức `cmp_display` nếu kiểu bên trong `T` của nó triển khai trait `PartialOrd`
cho phép so sánh _và_ trait `Display` cho phép in.

<Listing number="10-15" file-name="src/lib.rs" caption="Triển khai có điều kiện các phương thức trên một kiểu generic tùy thuộc vào trait bounds">

```rust,noplayground
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-15/src/lib.rs}}
```

</Listing>

Chúng ta cũng có thể triển khai có điều kiện một trait cho bất kỳ kiểu nào triển
khai một trait khác. Việc triển khai một trait trên bất kỳ kiểu nào thỏa mãn các
trait bounds được gọi là _blanket implementations_ và được sử dụng rộng rãi
trong thư viện tiêu chuẩn Rust. Ví dụ, thư viện tiêu chuẩn triển khai trait
`ToString` trên bất kỳ kiểu nào triển khai trait `Display`. Khối `impl` trong
thư viện tiêu chuẩn trông tương tự như mã này:

```rust,ignore
impl<T: Display> ToString for T {
    // --snip--
}
```

Bởi vì thư viện tiêu chuẩn có blanket implementation này, chúng ta có thể gọi
phương thức `to_string` được định nghĩa bởi trait `ToString` trên bất kỳ kiểu
nào triển khai trait `Display`. Ví dụ, chúng ta có thể chuyển các số nguyên
thành các giá trị `String` tương ứng của chúng như thế này vì số nguyên triển
khai `Display`:

```rust
let s = 3.to_string();
```

Các blanket implementation xuất hiện trong tài liệu của trait trong phần
"Implementors".

Traits và trait bounds cho phép chúng ta viết mã sử dụng các tham số kiểu
generic để giảm sự trùng lặp nhưng cũng chỉ định cho trình biên dịch rằng chúng
ta muốn kiểu generic có hành vi cụ thể. Trình biên dịch sau đó có thể sử dụng
thông tin trait bound để kiểm tra rằng tất cả các kiểu cụ thể được sử dụng với
mã của chúng ta cung cấp hành vi chính xác. Trong các ngôn ngữ kiểu động, chúng
ta sẽ gặp lỗi tại thời điểm chạy nếu chúng ta gọi một phương thức trên một kiểu
không định nghĩa phương thức đó. Nhưng Rust chuyển các lỗi này sang thời điểm
biên dịch để chúng ta buộc phải sửa các vấn đề trước khi mã của chúng ta thậm
chí có thể chạy. Ngoài ra, chúng ta không phải viết mã kiểm tra hành vi tại thời
điểm chạy vì chúng ta đã kiểm tra tại thời điểm biên dịch. Làm như vậy cải thiện
hiệu suất mà không phải từ bỏ tính linh hoạt của generics.

[using-trait-objects-that-allow-for-values-of-different-types]:
  ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[methods]: ch05-03-method-syntax.html#defining-methods
