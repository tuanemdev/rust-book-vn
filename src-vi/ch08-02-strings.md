## Lưu Trữ Văn Bản Mã Hóa UTF-8 với Chuỗi

Chúng ta đã nói về chuỗi trong Chương 4, nhưng bây giờ chúng ta sẽ xem xét chúng
chi tiết hơn. Những người mới học Rust thường bị mắc kẹt với chuỗi vì sự kết hợp
của ba lý do: khuynh hướng của Rust trong việc tiết lộ các lỗi có thể xảy ra,
chuỗi là một cấu trúc dữ liệu phức tạp hơn nhiều so với những gì nhiều lập trình
viên đánh giá, và UTF-8. Những yếu tố này kết hợp theo cách dường như khó khăn
khi bạn đến từ các ngôn ngữ lập trình khác.

Chúng ta thảo luận về chuỗi trong bối cảnh của các bộ sưu tập vì chuỗi được
triển khai như một bộ sưu tập các byte, cộng thêm một số phương thức để cung cấp
chức năng hữu ích khi những byte này được diễn giải dưới dạng văn bản. Trong
phần này, chúng ta sẽ nói về các hoạt động trên `String` mà mọi loại bộ sưu tập
đều có, chẳng hạn như tạo, cập nhật và đọc. Chúng ta cũng sẽ thảo luận về những
cách mà `String` khác với các bộ sưu tập khác, cụ thể là cách mà việc lập chỉ
mục vào một `String` bị phức tạp hóa bởi sự khác biệt giữa cách con người và máy
tính diễn giải dữ liệu `String`.

### Chuỗi Là Gì?

Đầu tiên, chúng ta sẽ định nghĩa những gì chúng ta hiểu bằng thuật ngữ _chuỗi_.
Rust chỉ có một kiểu chuỗi trong ngôn ngữ cốt lõi, đó là lát cắt chuỗi `str`
thường được thấy dưới dạng mượn `&str`. Trong Chương 4, chúng ta đã nói về _lát
cắt chuỗi_, là tham chiếu đến một số dữ liệu chuỗi mã hóa UTF-8 được lưu trữ ở
nơi khác. Các chuỗi nghĩa đen, chẳng hạn, được lưu trữ trong tệp nhị phân của
chương trình và do đó là lát cắt chuỗi.

Kiểu `String`, được cung cấp bởi thư viện chuẩn của Rust thay vì được mã hóa
trong ngôn ngữ cốt lõi, là một kiểu chuỗi có thể phát triển, có thể thay đổi,
được sở hữu, mã hóa UTF-8. Khi người dùng Rust nhắc đến "chuỗi" trong Rust, họ
có thể nhắc đến kiểu `String` hoặc kiểu lát cắt chuỗi `&str`, không chỉ một
trong những kiểu đó. Mặc dù phần này chủ yếu nói về `String`, cả hai kiểu đều
được sử dụng nhiều trong thư viện chuẩn của Rust, và cả `String` và lát cắt
chuỗi đều được mã hóa UTF-8.

### Tạo Một Chuỗi Mới

Nhiều hoạt động tương tự có sẵn với `Vec<T>` cũng có sẵn với `String` vì
`String` thực sự được triển khai như một bao bọc xung quanh một vector byte với
một số đảm bảo, hạn chế và khả năng bổ sung. Một ví dụ về một hàm hoạt động
giống nhau với `Vec<T>` và `String` là hàm `new` để tạo một thể hiện, như hiển
thị trong Listing 8-11.

<Listing number="8-11" caption="Tạo một `String` mới, rỗng">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-11/src/main.rs:here}}
```

</Listing>

Dòng này tạo ra một chuỗi mới, rỗng có tên là `s`, mà chúng ta sau đó có thể tải
dữ liệu vào. Thường thì chúng ta sẽ có một số dữ liệu ban đầu mà chúng ta muốn
bắt đầu chuỗi với. Để làm điều đó, chúng ta sử dụng phương thức `to_string`, có
sẵn trên bất kỳ kiểu nào triển khai trait `Display`, như các chuỗi nghĩa đen có.
Listing 8-12 hiển thị hai ví dụ.

<Listing number="8-12" caption="Sử dụng phương thức `to_string` để tạo một `String` từ một chuỗi nghĩa đen">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-12/src/main.rs:here}}
```

</Listing>

Mã này tạo ra một chuỗi chứa `initial contents`.

Chúng ta cũng có thể sử dụng hàm `String::from` để tạo một `String` từ một chuỗi
nghĩa đen. Mã trong Listing 8-13 tương đương với mã trong Listing 8-12 sử dụng
`to_string`.

<Listing number="8-13" caption="Sử dụng hàm `String::from` để tạo một `String` từ một chuỗi nghĩa đen">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-13/src/main.rs:here}}
```

</Listing>

Vì chuỗi được sử dụng cho rất nhiều thứ, chúng ta có thể sử dụng nhiều API chung
khác nhau cho chuỗi, cung cấp cho chúng ta rất nhiều tùy chọn. Một số trong số
chúng có vẻ thừa thãi, nhưng tất cả đều có vị trí riêng của mình! Trong trường
hợp này, `String::from` và `to_string` làm cùng một việc, vì vậy việc bạn chọn
cái nào là vấn đề của phong cách và khả năng đọc.

Hãy nhớ rằng chuỗi được mã hóa UTF-8, vì vậy chúng ta có thể bao gồm bất kỳ dữ
liệu nào được mã hóa đúng cách trong chúng, như hiển thị trong Listing 8-14.

<Listing number="8-14" caption="Lưu trữ lời chào trong các ngôn ngữ khác nhau trong chuỗi">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:here}}
```

</Listing>

Tất cả những điều này đều là các giá trị `String` hợp lệ.

### Cập Nhật một Chuỗi

Một `String` có thể tăng kích thước và nội dung của nó có thể thay đổi, giống
như nội dung của một `Vec<T>`, nếu bạn đẩy thêm dữ liệu vào nó. Ngoài ra, bạn có
thể thuận tiện sử dụng toán tử `+` hoặc macro `format!` để nối các giá trị
`String`.

#### Thêm vào Chuỗi với `push_str` và `push`

Chúng ta có thể làm cho một `String` phát triển bằng cách sử dụng phương thức
`push_str` để thêm một lát cắt chuỗi, như hiển thị trong Listing 8-15.

<Listing number="8-15" caption="Thêm một lát cắt chuỗi vào một `String` bằng phương thức `push_str`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-15/src/main.rs:here}}
```

</Listing>

Sau hai dòng này, `s` sẽ chứa `foobar`. Phương thức `push_str` lấy một lát cắt
chuỗi vì chúng ta không nhất thiết muốn lấy quyền sở hữu của tham số. Ví dụ,
trong mã trong Listing 8-16, chúng ta muốn có thể sử dụng `s2` sau khi thêm nội
dung của nó vào `s1`.

<Listing number="8-16" caption="Sử dụng một lát cắt chuỗi sau khi thêm nội dung của nó vào một `String`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-16/src/main.rs:here}}
```

</Listing>

Nếu phương thức `push_str` lấy quyền sở hữu của `s2`, chúng ta sẽ không thể in
giá trị của nó trên dòng cuối cùng. Tuy nhiên, mã này hoạt động như chúng ta
mong đợi!

Phương thức `push` lấy một ký tự duy nhất làm tham số và thêm nó vào `String`.
Listing 8-17 thêm chữ cái _l_ vào một `String` bằng cách sử dụng phương thức
`push`.

<Listing number="8-17" caption="Thêm một ký tự vào giá trị `String` bằng cách sử dụng `push`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-17/src/main.rs:here}}
```

</Listing>

Kết quả là, `s` sẽ chứa `lol`.

#### Nối với Toán Tử `+` hoặc Macro `format!`

Thường thì bạn sẽ muốn kết hợp hai chuỗi hiện có. Một cách để làm điều đó là sử
dụng toán tử `+`, như hiển thị trong Listing 8-18.

<Listing number="8-18" caption="Sử dụng toán tử `+` để kết hợp hai giá trị `String` thành một giá trị `String` mới">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-18/src/main.rs:here}}
```

</Listing>

Chuỗi `s3` sẽ chứa `Hello, world!`. Lý do `s1` không còn hợp lệ sau khi thêm, và
lý do chúng ta sử dụng một tham chiếu đến `s2`, có liên quan đến chữ ký của
phương thức được gọi khi chúng ta sử dụng toán tử `+`. Toán tử `+` sử dụng
phương thức `add`, có chữ ký trông giống như thế này:

```rust,ignore
fn add(self, s: &str) -> String {
```

Trong thư viện chuẩn, bạn sẽ thấy `add` được định nghĩa bằng cách sử dụng
generics và liên kết kiểu. Ở đây, chúng ta đã thay thế bằng các kiểu cụ thể,
điều này xảy ra khi chúng ta gọi phương thức này với các giá trị `String`. Chúng
ta sẽ thảo luận về generics trong Chương 10. Chữ ký này cung cấp cho chúng ta
những manh mối chúng ta cần để hiểu các phần phức tạp của toán tử `+`.

Đầu tiên, `s2` có một `&`, có nghĩa là chúng ta đang thêm một _tham chiếu_ của
chuỗi thứ hai vào chuỗi đầu tiên. Điều này là do tham số `s` trong hàm `add`:
chúng ta chỉ có thể thêm một `&str` vào một `String`; chúng ta không thể thêm
hai giá trị `String` với nhau. Nhưng khoan—kiểu của `&s2` là `&String`, không
phải `&str`, như đã được chỉ định trong tham số thứ hai cho `add`. Vậy tại sao
Listing 8-18 biên dịch?

Lý do chúng ta có thể sử dụng `&s2` trong lệnh gọi đến `add` là vì trình biên
dịch có thể _ép buộc_ đối số `&String` thành `&str`. Khi chúng ta gọi phương
thức `add`, Rust sử dụng _ép buộc deref_, nơi đây biến `&s2` thành `&s2[..]`.
Chúng ta sẽ thảo luận về ép buộc deref chi tiết hơn trong Chương 15. Bởi vì
`add` không lấy quyền sở hữu của tham số `s`, `s2` vẫn sẽ là một `String` hợp lệ
sau hoạt động này.

Thứ hai, chúng ta có thể thấy trong chữ ký rằng `add` lấy quyền sở hữu của
`self` vì `self` _không_ có `&`. Điều này có nghĩa `s1` trong Listing 8-18 sẽ bị
di chuyển vào lệnh gọi `add` và sẽ không còn hợp lệ sau đó. Vì vậy, mặc dù
`let s3 = s1 + &s2;` trông như thể nó sẽ sao chép cả hai chuỗi và tạo một chuỗi
mới, câu lệnh này thực sự lấy quyền sở hữu của `s1`, thêm một bản sao nội dung
của `s2`, và sau đó trả lại quyền sở hữu của kết quả. Nói cách khác, nó trông
như thể nó đang tạo ra rất nhiều bản sao, nhưng không phải vậy; việc triển khai
hiệu quả hơn là sao chép.

Nếu chúng ta cần nối nhiều chuỗi, hành vi của toán tử `+` trở nên khó xử lý:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-01-concat-multiple-strings/src/main.rs:here}}
```

Lúc này, `s` sẽ là `tic-tac-toe`. Với tất cả các ký tự `+` và `"`, khó có thể
thấy điều gì đang xảy ra. Để kết hợp các chuỗi theo cách phức tạp hơn, chúng ta
có thể sử dụng macro `format!`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-02-format/src/main.rs:here}}
```

Mã này cũng đặt `s` thành `tic-tac-toe`. Macro `format!` hoạt động giống như
`println!`, nhưng thay vì in đầu ra ra màn hình, nó trả về một `String` với nội
dung. Phiên bản mã sử dụng `format!` dễ đọc hơn nhiều, và mã được tạo ra bởi
macro `format!` sử dụng tham chiếu nên lời gọi này không lấy quyền sở hữu của
bất kỳ tham số nào của nó.

### Lập Chỉ Mục vào Chuỗi

Trong nhiều ngôn ngữ lập trình khác, việc truy cập các ký tự riêng lẻ trong một
chuỗi bằng cách tham chiếu đến chúng bằng chỉ mục là một hoạt động hợp lệ và phổ
biến. Tuy nhiên, nếu bạn cố gắng truy cập các phần của một `String` sử dụng cú
pháp lập chỉ mục trong Rust, bạn sẽ gặp lỗi. Xem xét mã không hợp lệ trong
Listing 8-19.

<Listing number="8-19" caption="Cố gắng sử dụng cú pháp lập chỉ mục với một String">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-19/src/main.rs:here}}
```

</Listing>

Mã này sẽ dẫn đến lỗi sau:

```console
{{#include ../listings/ch08-common-collections/listing-08-19/output.txt}}
```

Lỗi và ghi chú kể câu chuyện: chuỗi Rust không hỗ trợ lập chỉ mục. Nhưng tại sao
không? Để trả lời câu hỏi đó, chúng ta cần thảo luận về cách Rust lưu trữ chuỗi
trong bộ nhớ.

#### Biểu Diễn Nội Bộ

Một `String` là một bao bọc trên một `Vec<u8>`. Hãy xem một số ví dụ chuỗi được
mã hóa UTF-8 đúng của chúng ta từ Listing 8-14. Đầu tiên là ví dụ này:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:spanish}}
```

Trong trường hợp này, `len` sẽ là `4`, có nghĩa là vector lưu trữ chuỗi `"Hola"`
dài 4 byte. Mỗi chữ cái này chiếm một byte khi được mã hóa trong UTF-8. Tuy
nhiên, dòng sau đây có thể làm bạn ngạc nhiên (lưu ý rằng chuỗi này bắt đầu bằng
chữ cái Cyrillic viết hoa _Ze_, không phải số 3):

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-14/src/main.rs:russian}}
```

Nếu bạn được hỏi chuỗi dài bao nhiêu, bạn có thể nói 12. Thực tế, câu trả lời
của Rust là 24: đó là số byte cần thiết để mã hóa "Здравствуйте" trong UTF-8, vì
mỗi giá trị vô hướng Unicode trong chuỗi đó chiếm 2 byte lưu trữ. Do đó, một chỉ
mục vào byte của chuỗi sẽ không phải lúc nào cũng tương quan với một giá trị vô
hướng Unicode hợp lệ. Để minh họa, hãy xem xét mã Rust không hợp lệ này:

```rust,ignore,does_not_compile
let hello = "Здравствуйте";
let answer = &hello[0];
```

Bạn đã biết rằng `answer` sẽ không phải là `З`, chữ cái đầu tiên. Khi được mã
hóa trong UTF-8, byte đầu tiên của `З` là `208` và byte thứ hai là `151`, vì vậy
nó sẽ có vẻ như `answer` thực sự nên là `208`, nhưng `208` không phải là một ký
tự hợp lệ tự nó. Trả về `208` có thể không phải là điều mà người dùng muốn nếu
họ hỏi về chữ cái đầu tiên của chuỗi này; tuy nhiên, đó là dữ liệu duy nhất mà
Rust có tại chỉ mục byte 0. Người dùng thường không muốn giá trị byte được trả
về, ngay cả khi chuỗi chỉ chứa các chữ cái Latin: nếu `&"hi"[0]` là mã hợp lệ
trả về giá trị byte, nó sẽ trả về `104`, không phải `h`.

Câu trả lời, vì vậy, là để tránh trả về một giá trị không mong đợi và gây ra lỗi
có thể không được phát hiện ngay lập tức, Rust không biên dịch mã này và ngăn
chặn hiểu lầm sớm trong quá trình phát triển.

#### Byte, Giá Trị Vô Hướng và Cụm Tự Hình! Ôi Trời!

Một điểm khác về UTF-8 là thực sự có ba cách liên quan để nhìn chuỗi từ góc độ
của Rust: dưới dạng byte, giá trị vô hướng, và cụm tự hình (điều gần nhất với
những gì chúng ta gọi là _chữ cái_).

Nếu chúng ta nhìn vào từ tiếng Hindi "नमस्ते" được viết bằng chữ viết
Devanagari, nó được lưu trữ dưới dạng một vector các giá trị `u8` trông như thế
này:

```text
[224, 164, 168, 224, 164, 174, 224, 164, 184, 224, 165, 141, 224, 164, 164,
224, 165, 135]
```

Đó là 18 byte và đây là cách máy tính cuối cùng lưu trữ dữ liệu này. Nếu chúng
ta nhìn vào chúng như các giá trị vô hướng Unicode, đó là kiểu `char` của Rust,
những byte đó trông như thế này:

```text
['न', 'म', 'स', '्', 'त', 'े']
```

Có sáu giá trị `char` ở đây, nhưng giá trị thứ tư và thứ sáu không phải là chữ
cái: đó là các dấu phụ không có ý nghĩa khi đứng một mình. Cuối cùng, nếu chúng
ta nhìn vào chúng như cụm tự hình, chúng ta sẽ có được những gì mà một người sẽ
gọi là bốn chữ cái tạo nên từ tiếng Hindi:

```text
["न", "म", "स्", "ते"]
```

Rust cung cấp các cách khác nhau để diễn giải dữ liệu chuỗi thô mà máy tính lưu
trữ để mỗi chương trình có thể chọn cách diễn giải mà nó cần, bất kể ngôn ngữ
con người nào mà dữ liệu đó thuộc về.

Một lý do cuối cùng mà Rust không cho phép chúng ta lập chỉ mục vào một `String`
để lấy một ký tự là vì các hoạt động lập chỉ mục được mong đợi luôn mất thời
gian không đổi (O(1)). Nhưng không thể đảm bảo hiệu suất đó với một `String`, vì
Rust sẽ phải đi qua nội dung từ đầu đến chỉ mục để xác định có bao nhiêu ký tự
hợp lệ.

### Cắt Chuỗi

Lập chỉ mục vào một chuỗi thường là một ý tưởng tồi vì không rõ kiểu trả về của
hoạt động lập chỉ mục chuỗi nên là gì: một giá trị byte, một ký tự, một cụm tự
hình, hay một lát cắt chuỗi. Do đó, nếu bạn thực sự cần sử dụng chỉ mục để tạo
lát cắt chuỗi, Rust yêu cầu bạn cần cụ thể hơn.

Thay vì lập chỉ mục bằng `[]` với một số duy nhất, bạn có thể sử dụng `[]` với
một phạm vi để tạo một lát cắt chuỗi chứa các byte cụ thể:

```rust
let hello = "Здравствуйте";

let s = &hello[0..4];
```

Ở đây, `s` sẽ là một `&str` chứa bốn byte đầu tiên của chuỗi. Trước đó, chúng ta
đã đề cập rằng mỗi ký tự này chiếm hai byte, điều đó có nghĩa `s` sẽ là `Зд`.

Nếu chúng ta cố gắng cắt chỉ một phần byte của một ký tự với một cái gì đó như
`&hello[0..1]`, Rust sẽ hoảng loạn tại thời điểm chạy theo cách tương tự như khi
một chỉ mục không hợp lệ được truy cập trong một vector:

```console
{{#include ../listings/ch08-common-collections/output-only-01-not-char-boundary/output.txt}}
```

Bạn nên thận trọng khi tạo lát cắt chuỗi với phạm vi, vì làm như vậy có thể làm
sập chương trình của bạn.

### Phương Thức để Lặp Qua Chuỗi

Cách tốt nhất để hoạt động trên các phần của chuỗi là rõ ràng về việc bạn muốn
ký tự hay byte. Đối với các giá trị vô hướng Unicode riêng lẻ, hãy sử dụng
phương thức `chars`. Gọi `chars` trên "Зд" tách ra và trả về hai giá trị kiểu
`char`, và bạn có thể lặp qua kết quả để truy cập từng phần tử:

```rust
for c in "Зд".chars() {
    println!("{c}");
}
```

Mã này sẽ in ra như sau:

```text
З
д
```

Ngoài ra, phương thức `bytes` trả về từng byte thô, điều này có thể phù hợp cho
miền của bạn:

```rust
for b in "Зд".bytes() {
    println!("{b}");
}
```

Mã này sẽ in ra bốn byte tạo nên chuỗi này:

```text
208
151
208
180
```

Nhưng hãy nhớ rằng các giá trị vô hướng Unicode hợp lệ có thể bao gồm nhiều hơn
một byte.

Lấy cụm tự hình từ chuỗi, như với chữ viết Devanagari, là phức tạp, vì vậy chức
năng này không được cung cấp bởi thư viện chuẩn. Các crate có sẵn trên
[crates.io](https://crates.io/)<!-- ignore --> nếu đây là chức năng bạn cần.

### Chuỗi Không Đơn Giản

Tóm lại, chuỗi là phức tạp. Các ngôn ngữ lập trình khác nhau đưa ra những lựa
chọn khác nhau về cách trình bày sự phức tạp này cho lập trình viên. Rust đã
chọn làm cho việc xử lý dữ liệu `String` đúng cách trở thành hành vi mặc định
cho tất cả các chương trình Rust, điều này có nghĩa là lập trình viên phải suy
nghĩ nhiều hơn về việc xử lý dữ liệu UTF-8 ngay từ đầu. Sự đánh đổi này tiết lộ
nhiều sự phức tạp của chuỗi hơn là rõ ràng trong các ngôn ngữ lập trình khác,
nhưng nó ngăn bạn phải xử lý lỗi liên quan đến các ký tự không phải ASCII sau
này trong vòng đời phát triển của bạn.

Tin tốt là thư viện chuẩn cung cấp rất nhiều chức năng được xây dựng dựa trên
các kiểu `String` và `&str` để giúp xử lý các tình huống phức tạp này một cách
chính xác. Hãy chắc chắn kiểm tra tài liệu để biết các phương thức hữu ích như
`contains` để tìm kiếm trong một chuỗi và `replace` để thay thế các phần của một
chuỗi bằng một chuỗi khác.

Hãy chuyển sang một thứ ít phức tạp hơn một chút: bảng băm!
