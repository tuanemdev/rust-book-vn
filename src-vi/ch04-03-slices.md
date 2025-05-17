## Kiểu Slice

_Slices_ cho phép bạn tham chiếu đến một chuỗi liên tiếp các phần tử trong một
[bộ sưu tập](ch08-00-common-collections.md)<!-- ignore -->. Slice là một loại
tham chiếu, nên nó không có quyền sở hữu.

Đây là một bài toán nhỏ: viết một hàm nhận vào một chuỗi các từ được phân tách
bởi khoảng trắng và trả về từ đầu tiên mà nó tìm thấy trong chuỗi đó. Nếu hàm
không tìm thấy khoảng trắng trong chuỗi, toàn bộ chuỗi phải là một từ, do đó
toàn bộ chuỗi sẽ được trả về.

> Lưu ý: Để giới thiệu về string slice, chúng ta giả định chỉ xử lý ASCII trong
> phần này; một thảo luận chi tiết hơn về xử lý UTF-8 nằm trong phần ["Lưu trữ
> văn bản được mã hóa UTF-8 với Strings"][strings]<!-- ignore --> ở Chương 8.

Hãy cùng xem xét cách viết chữ ký của hàm này mà không sử dụng slices, để hiểu
vấn đề mà slices sẽ giải quyết:

```rust,ignore
fn first_word(s: &String) -> ?
```

Hàm `first_word` có một tham số kiểu `&String`. Chúng ta không cần quyền sở hữu,
nên điều này là hợp lý. (Theo cách viết thông thường của Rust, hàm không lấy
quyền sở hữu của các đối số của chúng trừ khi cần thiết, và lý do sẽ trở nên rõ
ràng hơn khi chúng ta tiếp tục.) Nhưng chúng ta nên trả về gì? Chúng ta không
thực sự có cách để nói về _một phần_ của một chuỗi. Tuy nhiên, chúng ta có thể
trả về chỉ số của vị trí kết thúc từ, được chỉ định bằng một khoảng trắng. Hãy
thử điều đó, như trong Listing 4-7.

<Listing number="4-7" file-name="src/main.rs" caption="Hàm `first_word` trả về một giá trị chỉ số byte vào tham số `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:here}}
```

</Listing>

Vì chúng ta cần đi qua từng phần tử của `String` và kiểm tra xem một giá trị có
phải là khoảng trắng không, chúng ta sẽ chuyển đổi `String` thành một mảng byte
bằng phương thức `as_bytes`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:as_bytes}}
```

Tiếp theo, chúng ta tạo một iterator qua mảng byte bằng phương thức `iter`:

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:iter}}
```

Chúng ta sẽ thảo luận về iterators chi tiết hơn trong [Chương
13][ch13]<!-- ignore -->. Hiện tại, hãy biết rằng `iter` là một phương thức trả
về mỗi phần tử trong một bộ sưu tập và `enumerate` bao bọc kết quả của `iter` và
trả về mỗi phần tử dưới dạng một tuple. Phần tử đầu tiên của tuple trả về từ
`enumerate` là chỉ số, và phần tử thứ hai là tham chiếu đến phần tử. Điều này
thuận tiện hơn một chút so với việc tự tính toán chỉ số.

Vì phương thức `enumerate` trả về một tuple, chúng ta có thể sử dụng các mẫu để
phân rã tuple đó. Chúng ta sẽ thảo luận về các mẫu chi tiết hơn trong [Chương
6][ch6]<!-- ignore -->. Trong vòng lặp `for`, chúng ta chỉ định một mẫu có `i`
cho chỉ số trong tuple và `&item` cho byte đơn lẻ trong tuple. Bởi vì chúng ta
có được tham chiếu đến phần tử từ `.iter().enumerate()`, chúng ta sử dụng `&`
trong mẫu.

Bên trong vòng lặp `for`, chúng ta tìm kiếm byte đại diện cho khoảng trắng bằng
cách sử dụng cú pháp byte literal. Nếu chúng ta tìm thấy một khoảng trắng, chúng
ta trả về vị trí. Nếu không, chúng ta trả về độ dài của chuỗi bằng cách sử dụng
`s.len()`.

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-07/src/main.rs:inside_for}}
```

Bây giờ chúng ta đã có cách để tìm ra chỉ số của vị trí kết thúc từ đầu tiên
trong chuỗi, nhưng có một vấn đề. Chúng ta đang trả về một `usize` riêng lẻ,
nhưng đó chỉ là một số có ý nghĩa trong ngữ cảnh của `&String`. Nói cách khác,
vì nó là một giá trị tách biệt khỏi `String`, không có gì đảm bảo rằng nó sẽ vẫn
hợp lệ trong tương lai. Xem xét chương trình trong Listing 4-8 sử dụng hàm
`first_word` từ Listing 4-7.

<Listing number="4-8" file-name="src/main.rs" caption="Lưu trữ kết quả từ việc gọi hàm `first_word` và sau đó thay đổi nội dung của `String`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-08/src/main.rs:here}}
```

</Listing>

Chương trình này biên dịch mà không có lỗi nào và cũng sẽ như vậy nếu chúng ta
sử dụng `word` sau khi gọi `s.clear()`. Bởi vì `word` không liên kết với trạng
thái của `s` chút nào, `word` vẫn chứa giá trị `5`. Chúng ta có thể sử dụng giá
trị `5` đó với biến `s` để cố gắng trích xuất từ đầu tiên, nhưng điều này sẽ là
một lỗi vì nội dung của `s` đã thay đổi kể từ khi chúng ta lưu `5` vào `word`.

Việc phải lo lắng về chỉ số trong `word` không đồng bộ với dữ liệu trong `s` là
tẻ nhạt và dễ gây lỗi! Việc quản lý các chỉ số này thậm chí còn mong manh hơn
nếu chúng ta viết một hàm `second_word`. Chữ ký của nó sẽ phải trông như thế
này:

```rust,ignore
fn second_word(s: &String) -> (usize, usize) {
```

Bây giờ chúng ta đang theo dõi một chỉ số _bắt đầu_ _và_ một chỉ số _kết thúc_,
và chúng ta có nhiều giá trị hơn được tính toán từ dữ liệu trong một trạng thái
cụ thể nhưng không gắn liền với trạng thái đó chút nào. Chúng ta có ba biến
không liên quan đến nhau nổi xung quanh cần phải được giữ đồng bộ.

May mắn thay, Rust có một giải pháp cho vấn đề này: string slices.

### String Slices

Một _string slice_ là một tham chiếu đến một chuỗi liên tiếp các phần tử của một
`String`, và nó trông như thế này:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-17-slice/src/main.rs:here}}
```

Thay vì một tham chiếu đến toàn bộ `String`, `hello` là một tham chiếu đến một
phần của `String`, được chỉ định trong phần `[0..5]`. Chúng ta tạo slices bằng
cách sử dụng một phạm vi trong ngoặc vuông bằng cách chỉ định
`[starting_index..ending_index]`, trong đó _`starting_index`_ là vị trí đầu tiên
trong slice và _`ending_index`_ là vị trí sau vị trí cuối cùng trong slice. Về
mặt nội bộ, cấu trúc dữ liệu slice lưu trữ vị trí bắt đầu và độ dài của slice,
mà tương ứng với _`ending_index`_ trừ đi _`starting_index`_. Vì vậy, trong
trường hợp `let world = &s[6..11];`, `world` sẽ là một slice chứa một con trỏ
đến byte tại chỉ số 6 của `s` với giá trị độ dài là `5`.

Hình 4-7 minh họa điều này trong một sơ đồ.

<img alt="Ba bảng: một bảng đại diện cho dữ liệu stack của s, nó trỏ đến
byte tại chỉ số 0 trong một bảng dữ liệu chuỗi &quot;hello world&quot; trên
heap. Bảng thứ ba đại diện cho dữ liệu stack của slice world, nó
có giá trị độ dài là 5 và trỏ đến byte thứ 6 của bảng dữ liệu heap."
src="img/trpl04-07.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 4-7: String slice tham chiếu đến một phần của một
`String`</span>

Với cú pháp phạm vi `..` của Rust, nếu bạn muốn bắt đầu từ chỉ số 0, bạn có thể
bỏ giá trị trước hai dấu chấm. Nói cách khác, hai cách viết sau là tương đương:

```rust
let s = String::from("hello");

let slice = &s[0..2];
let slice = &s[..2];
```

Tương tự, nếu slice của bạn bao gồm byte cuối cùng của `String`, bạn có thể bỏ
số cuối. Điều đó có nghĩa là các cách viết sau là tương đương:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[3..len];
let slice = &s[3..];
```

Bạn cũng có thể bỏ cả hai giá trị để lấy một slice của toàn bộ chuỗi. Vì vậy,
hai cách viết sau là tương đương:

```rust
let s = String::from("hello");

let len = s.len();

let slice = &s[0..len];
let slice = &s[..];
```

> Lưu ý: Các chỉ số phạm vi của string slice phải nằm tại các ranh giới ký tự
> UTF-8 hợp lệ. Nếu bạn cố gắng tạo một string slice ở giữa một ký tự đa byte,
> chương trình của bạn sẽ kết thúc với một lỗi.

Với tất cả thông tin này, hãy viết lại `first_word` để trả về một slice. Kiểu
dùng để biểu thị "string slice" được viết là `&str`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-18-first-word-slice/src/main.rs:here}}
```

</Listing>

Chúng ta lấy chỉ số cho vị trí kết thúc của từ theo cách tương tự như trong
Listing 4-7, bằng cách tìm kiếm lần xuất hiện đầu tiên của một khoảng trắng. Khi
chúng ta tìm thấy một khoảng trắng, chúng ta trả về một string slice sử dụng
phần đầu của chuỗi và chỉ số của khoảng trắng làm chỉ số bắt đầu và kết thúc.

Bây giờ khi chúng ta gọi `first_word`, chúng ta nhận được một giá trị duy nhất
gắn liền với dữ liệu cơ bản. Giá trị bao gồm một tham chiếu đến điểm bắt đầu của
slice và số phần tử trong slice.

Việc trả về một slice cũng sẽ hoạt động cho một hàm `second_word`:

```rust,ignore
fn second_word(s: &String) -> &str {
```

Bây giờ chúng ta có một API đơn giản hơn nhiều mà khó bị sai sót hơn bởi vì
trình biên dịch sẽ đảm bảo các tham chiếu vào `String` vẫn hợp lệ. Hãy nhớ lại
lỗi trong chương trình ở Listing 4-8, khi chúng ta nhận được chỉ số đến vị trí
kết thúc từ đầu tiên nhưng sau đó xóa chuỗi khiến cho chỉ số của chúng ta không
còn hợp lệ? Đoạn mã đó về mặt logic là không đúng nhưng không hiển thị lỗi ngay
lập tức. Các vấn đề sẽ xuất hiện sau nếu chúng ta tiếp tục cố gắng sử dụng chỉ
số từ đầu tiên với một chuỗi đã bị làm rỗng. Slices làm cho lỗi này là không thể
và cho chúng ta biết rằng chúng ta có vấn đề với mã của mình sớm hơn nhiều. Sử
dụng phiên bản slice của `first_word` sẽ báo lỗi khi biên dịch:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/src/main.rs:here}}
```

</Listing>

Đây là lỗi biên dịch:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-19-slice-error/output.txt}}
```

Hãy nhớ lại từ quy tắc mượn rằng nếu chúng ta có một tham chiếu bất biến đến một
thứ gì đó, chúng ta không thể đồng thời lấy một tham chiếu có thể thay đổi. Bởi
vì `clear` cần cắt ngắn `String`, nó cần lấy một tham chiếu có thể thay đổi.
`println!` sau lời gọi đến `clear` sử dụng tham chiếu trong `word`, do đó tham
chiếu bất biến phải vẫn còn hoạt động tại thời điểm đó. Rust không cho phép tham
chiếu có thể thay đổi trong `clear` và tham chiếu bất biến trong `word` tồn tại
cùng một lúc, và việc biên dịch thất bại. Rust không chỉ làm cho API của chúng
ta dễ sử dụng hơn, mà còn loại bỏ toàn bộ một lớp lỗi tại thời điểm biên dịch!

<!-- Old heading. Do not remove or links may break. -->

<a id="string-literals-are-slices"></a>

#### String Literals dưới dạng Slices

Nhớ lại rằng chúng ta đã nói về việc string literals được lưu trữ bên trong mã
nhị phân. Bây giờ khi chúng ta đã biết về slices, chúng ta có thể hiểu đúng về
string literals:

```rust
let s = "Hello, world!";
```

Kiểu của `s` ở đây là `&str`: đó là một slice trỏ đến một điểm cụ thể của mã nhị
phân. Đây cũng là lý do tại sao string literals là bất biến; `&str` là một tham
chiếu bất biến.

#### String Slices dưới dạng Tham số

Khi biết rằng bạn có thể lấy slices của literals và giá trị `String` dẫn chúng
ta đến một cải tiến nữa cho `first_word`, và đó là chữ ký của nó:

```rust,ignore
fn first_word(s: &String) -> &str {
```

Một Rustacean có kinh nghiệm hơn sẽ viết chữ ký như trong Listing 4-9 thay vì
như trên vì nó cho phép chúng ta sử dụng cùng một hàm cho cả giá trị `&String`
và giá trị `&str`.

<Listing number="4-9" caption="Cải thiện hàm `first_word` bằng cách sử dụng string slice cho kiểu của tham số `s`">

```rust,ignore
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:here}}
```

</Listing>

Nếu chúng ta có một string slice, chúng ta có thể truyền nó trực tiếp. Nếu chúng
ta có một `String`, chúng ta có thể truyền một slice của `String` hoặc một tham
chiếu đến `String`. Tính linh hoạt này tận dụng _deref coercions_, một tính năng
mà chúng ta sẽ đề cập trong phần ["Implicit Deref Coercions with Functions and
Methods"][deref-coercions]<!--ignore--> của Chương 15.

Việc định nghĩa một hàm nhận một string slice thay vì một tham chiếu đến một
`String` làm cho API của chúng ta trở nên tổng quát và hữu ích hơn mà không mất
bất kỳ chức năng nào:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-09/src/main.rs:usage}}
```

</Listing>

### Các Slices Khác

String slices, như bạn có thể tưởng tượng, là đặc biệt dành cho chuỗi. Nhưng có
một kiểu slice chung hơn. Xét mảng này:

```rust
let a = [1, 2, 3, 4, 5];
```

Cũng như chúng ta có thể muốn tham chiếu đến một phần của một chuỗi, chúng ta có
thể muốn tham chiếu đến một phần của một mảng. Chúng ta sẽ thực hiện như sau:

```rust
let a = [1, 2, 3, 4, 5];

let slice = &a[1..3];

assert_eq!(slice, &[2, 3]);
```

Slice này có kiểu `&[i32]`. Nó hoạt động giống như string slices, bằng cách lưu
trữ một tham chiếu đến phần tử đầu tiên và một độ dài. Bạn sẽ sử dụng kiểu slice
này cho tất cả các loại bộ sưu tập khác. Chúng ta sẽ thảo luận về các bộ sưu tập
này chi tiết khi chúng ta nói về vectors trong Chương 8.

## Tóm tắt

Các khái niệm về quyền sở hữu, mượn và slices đảm bảo an toàn bộ nhớ trong các
chương trình Rust tại thời điểm biên dịch. Ngôn ngữ Rust cho bạn quyền kiểm soát
đối với việc sử dụng bộ nhớ của bạn theo cách giống như các ngôn ngữ lập trình
hệ thống khác, nhưng việc có chủ sở hữu của dữ liệu tự động dọn dẹp dữ liệu đó
khi chủ sở hữu ra khỏi phạm vi có nghĩa là bạn không cần phải viết và gỡ lỗi mã
bổ sung để có được sự kiểm soát này.

Quyền sở hữu ảnh hưởng đến cách thức hoạt động của nhiều phần khác của Rust, vì
vậy chúng ta sẽ nói về các khái niệm này thêm trong suốt phần còn lại của cuốn
sách. Hãy chuyển sang Chương 5 và xem xét việc nhóm các phần dữ liệu lại với
nhau trong một `struct`.

[ch13]: ch13-02-iterators.html
[ch6]: ch06-02-match.html#patterns-that-bind-to-values
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[deref-coercions]:
  ch15-02-deref.html#implicit-deref-coercions-with-functions-and-methods
