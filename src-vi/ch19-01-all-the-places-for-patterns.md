## Tất cả những nơi có thể sử dụng Mẫu

Mẫu xuất hiện ở nhiều nơi trong Rust, và bạn đã sử dụng chúng rất nhiều mà không
nhận ra! Phần này sẽ thảo luận về tất cả những nơi mà mẫu có hiệu lực.

### Các nhánh của `match`

Như đã thảo luận trong Chương 6, chúng ta sử dụng mẫu trong các nhánh của biểu
thức `match`. Chính thức thì, biểu thức `match` được định nghĩa bằng từ khóa
`match`, một giá trị để so khớp, và một hoặc nhiều nhánh khớp bao gồm một mẫu và
một biểu thức để chạy nếu giá trị khớp với mẫu của nhánh đó, như thế này:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre><code>match <em>VALUE</em> {
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
    <em>PATTERN</em> => <em>EXPRESSION</em>,
}</code></pre>

Ví dụ, đây là biểu thức `match` từ Listing 6-5 khớp với một giá trị
`Option<i32>` trong biến `x`:

```rust,ignore
match x {
    None => None,
    Some(i) => Some(i + 1),
}
```

Các mẫu trong biểu thức `match` này là `None` và `Some(i)` ở bên trái của mỗi
mũi tên.

Một yêu cầu đối với biểu thức `match` là chúng cần phải _toàn diện_ theo nghĩa
là tất cả các khả năng cho giá trị trong biểu thức `match` phải được tính đến.
Một cách để đảm bảo bạn đã bao quát mọi khả năng là có một mẫu bao trùm tất cả
cho nhánh cuối cùng: ví dụ, một tên biến khớp với bất kỳ giá trị nào không bao
giờ thất bại và do đó bao gồm mọi trường hợp còn lại.

Mẫu đặc biệt `_` sẽ khớp với bất cứ thứ gì, nhưng nó không bao giờ gắn kết với
một biến, vì vậy nó thường được sử dụng trong nhánh match cuối cùng. Mẫu `_` có
thể hữu ích khi bạn muốn bỏ qua bất kỳ giá trị nào không được chỉ định, ví dụ.
Chúng ta sẽ đề cập đến mẫu `_` chi tiết hơn trong ["Bỏ qua các giá trị trong
Mẫu"][ignoring-values-in-a-pattern]<!-- ignore --> sau trong chương này.

### Câu lệnh `let`

Trước chương này, chúng ta chỉ thảo luận rõ ràng về việc sử dụng mẫu với `match`
và `if let`, nhưng trên thực tế, chúng ta đã sử dụng mẫu ở những nơi khác nữa,
bao gồm cả câu lệnh `let`. Ví dụ, hãy xem xét việc gán biến đơn giản này với
`let`:

```rust
let x = 5;
```

Mỗi lần bạn sử dụng câu lệnh `let` như thế này, bạn đã sử dụng mẫu, mặc dù bạn
có thể không nhận ra điều đó! Chính thức hơn, câu lệnh `let` trông như thế này:

<!--
  Manually formatted rather than using Markdown intentionally: Markdown does not
  support italicizing code in the body of a block like this!
-->

<pre>
<code>let <em>PATTERN</em> = <em>EXPRESSION</em>;</code>
</pre>

Trong các câu lệnh như `let x = 5;` với tên biến trong vị trí PATTERN, tên biến
chỉ là một dạng đặc biệt đơn giản của mẫu. Rust so sánh biểu thức với mẫu và gán
bất kỳ tên nào nó tìm thấy. Vì vậy, trong ví dụ `let x = 5;`, `x` là một mẫu có
nghĩa là "gán cái gì khớp ở đây vào biến `x`." Bởi vì tên `x` là toàn bộ mẫu,
mẫu này về cơ bản có nghĩa là "gán mọi thứ vào biến `x`, bất kể giá trị là gì."

Để thấy rõ hơn khía cạnh khớp mẫu của `let`, hãy xem xét Listing 19-1, sử dụng
một mẫu với `let` để phân rã một tuple.

<Listing number="19-1" caption="Sử dụng mẫu để phân rã một tuple và tạo ba biến cùng một lúc">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta khớp một tuple với một mẫu. Rust so sánh giá trị `(1, 2, 3)` với
mẫu `(x, y, z)` và thấy rằng giá trị khớp với mẫu, ở chỗ số lượng phần tử là
giống nhau trong cả hai, nên Rust gắn `1` với `x`, `2` với `y`, và `3` với `z`.
Bạn có thể xem mẫu tuple này như là lồng ba mẫu biến cá nhân bên trong nó.

Nếu số lượng phần tử trong mẫu không khớp với số lượng phần tử trong tuple, kiểu
tổng thể sẽ không khớp và chúng ta sẽ gặp lỗi trình biên dịch. Ví dụ, Listing
19-2 cho thấy một nỗ lực phân rã một tuple với ba phần tử thành hai biến, điều
này sẽ không hoạt động.

<Listing number="19-2" caption="Xây dựng sai một mẫu có các biến không khớp với số lượng phần tử trong tuple">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Cố gắng biên dịch mã này dẫn đến lỗi kiểu này:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-02/output.txt}}
```

Để sửa lỗi, chúng ta có thể bỏ qua một hoặc nhiều giá trị trong tuple bằng cách
sử dụng `_` hoặc `..`, như bạn sẽ thấy trong phần ["Bỏ qua các giá trị trong
Mẫu"][ignoring-values-in-a-pattern]<!-- ignore -->. Nếu vấn đề là chúng ta có
quá nhiều biến trong mẫu, giải pháp là làm cho các kiểu khớp bằng cách loại bỏ
các biến để số lượng biến bằng với số lượng phần tử trong tuple.

### Biểu thức điều kiện `if let`

Trong Chương 6, chúng ta đã thảo luận về cách sử dụng biểu thức `if let` chủ yếu
như một cách ngắn hơn để viết tương đương với một `match` chỉ khớp một trường
hợp. Tùy chọn, `if let` có thể có một `else` tương ứng chứa mã để chạy nếu mẫu
trong `if let` không khớp.

Listing 19-1 cho thấy rằng cũng có thể kết hợp `if let`, `else if`, và
`else if let`. Làm như vậy giúp chúng ta linh hoạt hơn so với biểu thức `match`
mà chúng ta chỉ có thể biểu thị một giá trị để so sánh với các mẫu. Ngoài ra,
Rust không yêu cầu các điều kiện trong một chuỗi `if let`, `else if`,
`else if let` phải liên quan đến nhau.

Mã trong Listing 19-1 xác định màu nền dựa trên một loạt các kiểm tra cho một số
điều kiện. Đối với ví dụ này, chúng tôi đã tạo các biến với giá trị cứng mà một
chương trình thực sự có thể nhận từ người dùng đầu vào.

<Listing number="19-1" file-name="src/main.rs" caption="Kết hợp `if let`, `else if`, `else if let`, và `else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-01/src/main.rs}}
```

</Listing>

Nếu người dùng chỉ định một màu yêu thích, màu đó được sử dụng làm nền. Nếu
không có màu yêu thích nào được chỉ định và hôm nay là thứ Ba, màu nền là xanh
lá cây. Nếu không, nếu người dùng chỉ định tuổi của họ dưới dạng chuỗi và chúng
ta có thể phân tích nó thành một số thành công, màu sắc sẽ là tím hoặc cam tùy
thuộc vào giá trị của số đó. Nếu không có điều kiện nào áp dụng, màu nền là xanh
dương.

Cấu trúc điều kiện này cho phép chúng ta hỗ trợ các yêu cầu phức tạp. Với các
giá trị cứng mà chúng ta có ở đây, ví dụ này sẽ in ra
`Using purple as the background color`.

Bạn có thể thấy rằng `if let` cũng có thể giới thiệu các biến mới che khuất các
biến hiện có theo cách tương tự như các nhánh `match`: dòng
`if let Ok(age) = age` giới thiệu một biến `age` mới chứa giá trị bên trong biến
thể `Ok`, che khuất biến `age` hiện có. Điều này có nghĩa là chúng ta cần đặt
điều kiện `if age > 30` trong khối đó: chúng ta không thể kết hợp hai điều kiện
này thành `if let Ok(age) = age && age > 30`. Biến `age` mới mà chúng ta muốn so
sánh với 30 không hợp lệ cho đến khi phạm vi mới bắt đầu với dấu ngoặc nhọn.

Nhược điểm của việc sử dụng biểu thức `if let` là trình biên dịch không kiểm tra
tính toàn diện, trong khi với biểu thức `match` thì có. Nếu chúng ta bỏ qua khối
`else` cuối cùng và do đó bỏ lỡ việc xử lý một số trường hợp, trình biên dịch sẽ
không cảnh báo chúng ta về lỗi logic có thể xảy ra.

### Vòng lặp điều kiện `while let`

Tương tự như cấu trúc `if let`, vòng lặp điều kiện `while let` cho phép vòng lặp
`while` chạy miễn là một mẫu tiếp tục khớp. Trong Listing 19-3 chúng ta thấy một
vòng lặp `while let` đợi các tin nhắn được gửi giữa các luồng, nhưng trong
trường hợp này kiểm tra một `Result` thay vì một `Option`.

<Listing number="19-3" caption="Sử dụng vòng lặp `while let` để in giá trị miễn là `rx.recv()` trả về `Ok`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-02/src/main.rs:here}}
```

</Listing>

Ví dụ này in ra `1`, `2`, và sau đó `3`. Phương thức `recv` lấy tin nhắn đầu
tiên từ phía nhận của kênh và trả về `Ok(value)`. Khi chúng ta lần đầu tiên thấy
`recv` trở lại trong Chương 16, chúng ta đã unwrap lỗi trực tiếp, hoặc tương tác
với nó như một iterator sử dụng vòng lặp `for`. Như Listing 19-2 cho thấy, tuy
nhiên, chúng ta cũng có thể sử dụng `while let`, vì phương thức `recv` trả về
`Ok` mỗi lần một tin nhắn đến, miễn là người gửi tồn tại, và sau đó tạo ra một
`Err` khi phía người gửi ngắt kết nối.

### Vòng lặp `for`

Trong vòng lặp `for`, giá trị theo sau từ khóa `for` là một mẫu. Ví dụ, trong
`for x in y`, `x` là mẫu. Listing 19-4 minh họa cách sử dụng mẫu trong vòng lặp
`for` để phân rã, hoặc chia nhỏ, một tuple như một phần của vòng lặp `for`.

<Listing number="19-4" caption="Sử dụng mẫu trong vòng lặp `for` để phân rã một tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-03/src/main.rs:here}}
```

</Listing>

Mã trong Listing 19-4 sẽ in ra như sau:

```console
0: a
1: b
2: c
```

Chúng ta điều chỉnh một iterator bằng phương thức `enumerate` để nó tạo ra một
giá trị và chỉ số cho giá trị đó, đặt vào một tuple. Giá trị đầu tiên được tạo
ra là tuple `(0, 'a')`. Khi giá trị này được khớp với mẫu `(index, value)`,
`index` sẽ là `0` và `value` sẽ là `'a'`, in ra dòng đầu tiên của đầu ra.

### Tham số hàm

Tham số hàm cũng có thể là mẫu. Mã trong Listing 19-5, khai báo một hàm tên
`foo` nhận một tham số tên `x` kiểu `i32`, giờ đây có lẽ đã trông quen thuộc.

<Listing number="19-5" caption="Một chữ ký hàm sử dụng mẫu trong các tham số">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-06/src/main.rs:here}}
```

</Listing>

Phần `x` là một mẫu! Giống như với `let`, chúng ta có thể khớp một tuple trong
đối số của một hàm với mẫu. Listing 19-6 chia các giá trị trong một tuple khi
chúng ta truyền nó vào một hàm.

<Listing number="19-6" file-name="src/main.rs" caption="Một hàm với các tham số phân rã một tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-07/src/main.rs}}
```

</Listing>

Mã này in ra `Current location: (3, 5)`. Các giá trị `&(3, 5)` khớp với mẫu
`&(x, y)`, vì vậy `x` là giá trị `3` và `y` là giá trị `5`.

Chúng ta cũng có thể sử dụng mẫu trong danh sách tham số closure giống như trong
danh sách tham số hàm vì closure tương tự như hàm, như đã thảo luận trong
Chương 13.

Ở thời điểm này, bạn đã thấy một số cách sử dụng mẫu, nhưng mẫu không hoạt động
giống nhau ở mọi nơi chúng ta có thể sử dụng chúng. Ở một số nơi, các mẫu phải
không thể bác bỏ; trong các trường hợp khác, chúng có thể bị bác bỏ. Chúng ta sẽ
thảo luận về hai khái niệm này tiếp theo.

[ignoring-values-in-a-pattern]:
  ch19-03-pattern-syntax.html#ignoring-values-in-a-pattern
