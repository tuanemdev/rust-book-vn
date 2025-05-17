## Sử dụng `Box<T>` để Trỏ đến Dữ liệu trên Heap

Con trỏ thông minh đơn giản nhất là một box, có kiểu được viết là `Box<T>`.
_Box_ cho phép bạn lưu trữ dữ liệu trên heap thay vì stack. Những gì còn lại
trên stack là con trỏ đến dữ liệu heap. Xem lại Chương 4 để ôn lại sự khác biệt
giữa stack và heap.

Box không tạo ra chi phí hiệu suất, ngoại trừ việc lưu trữ dữ liệu của chúng
trên heap thay vì trên stack. Nhưng chúng cũng không có nhiều khả năng bổ sung.
Bạn sẽ sử dụng chúng thường xuyên nhất trong những tình huống sau:

- Khi bạn có một kiểu mà kích thước không thể biết được tại thời điểm biên dịch
  và bạn muốn sử dụng một giá trị của kiểu đó trong một ngữ cảnh yêu cầu kích
  thước chính xác
- Khi bạn có một lượng lớn dữ liệu và bạn muốn chuyển quyền sở hữu nhưng đảm bảo
  dữ liệu sẽ không bị sao chép khi làm như vậy
- Khi bạn muốn sở hữu một giá trị và bạn chỉ quan tâm rằng nó là một kiểu triển
  khai một trait cụ thể thay vì thuộc về một kiểu cụ thể

Chúng ta sẽ minh họa tình huống đầu tiên trong
["Cho phép Kiểu Đệ quy với Boxes"](#enabling-recursive-types-with-boxes)<!-- ignore -->.
Trong trường hợp thứ hai, việc chuyển quyền sở hữu của một lượng lớn dữ liệu có
thể mất nhiều thời gian vì dữ liệu được sao chép trên stack. Để cải thiện hiệu
suất trong tình huống này, chúng ta có thể lưu trữ lượng lớn dữ liệu trên heap
trong một box. Sau đó, chỉ có một lượng nhỏ dữ liệu con trỏ được sao chép trên
stack, trong khi dữ liệu mà nó tham chiếu vẫn ở một vị trí trên heap. Trường hợp
thứ ba được gọi là _đối tượng trait_, và ["Sử dụng Đối tượng Trait Cho phép Các
Giá trị Có Kiểu Khác nhau,"][trait-objects]<!-- ignore --> trong Chương 18 được
dành cho chủ đề đó. Vì vậy, những gì bạn học ở đây sẽ áp dụng lại trong phần đó!

### Sử dụng `Box<T>` để Lưu trữ Dữ liệu trên Heap

Trước khi chúng ta thảo luận về trường hợp sử dụng lưu trữ heap cho `Box<T>`,
chúng ta sẽ đề cập đến cú pháp và cách tương tác với các giá trị được lưu trữ
trong một `Box<T>`.

Listing 15-1 cho thấy cách sử dụng box để lưu trữ một giá trị `i32` trên heap.

<Listing number="15-1" file-name="src/main.rs" caption="Lưu trữ một giá trị `i32` trên heap sử dụng box">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-01/src/main.rs}}
```

</Listing>

Chúng ta định nghĩa biến `b` có giá trị là một `Box` trỏ đến giá trị `5`, được
cấp phát trên heap. Chương trình này sẽ in ra `b = 5`; trong trường hợp này,
chúng ta có thể truy cập dữ liệu trong box tương tự như cách chúng ta sẽ làm nếu
dữ liệu này nằm trên stack. Giống như bất kỳ giá trị sở hữu nào, khi một box ra
khỏi phạm vi, như `b` ở cuối `main`, nó sẽ bị giải phóng. Việc giải phóng diễn
ra cả cho box (được lưu trữ trên stack) và dữ liệu mà nó trỏ đến (được lưu trữ
trên heap).

Đặt một giá trị đơn lẻ trên heap không quá hữu ích, vì vậy bạn sẽ không thường
xuyên sử dụng box theo cách này. Có các giá trị như một `i32` đơn lẻ trên stack,
nơi chúng được lưu trữ theo mặc định, thích hợp hơn trong đa số trường hợp. Hãy
xem xét một trường hợp trong đó box cho phép chúng ta định nghĩa các kiểu mà
chúng ta không được phép định nghĩa nếu không có box.

### Cho phép Kiểu Đệ quy với Boxes

Một giá trị của _kiểu đệ quy_ có thể có một giá trị khác của cùng kiểu như một
phần của chính nó. Kiểu đệ quy gây ra vấn đề vì Rust cần biết tại thời điểm biên
dịch kiểu chiếm bao nhiêu không gian. Tuy nhiên, việc lồng các giá trị của kiểu
đệ quy về mặt lý thuyết có thể tiếp tục vô hạn, vì vậy Rust không thể biết giá
trị cần bao nhiêu không gian. Vì box có kích thước đã biết, chúng ta có thể cho
phép kiểu đệ quy bằng cách chèn một box vào định nghĩa kiểu đệ quy.

Ví dụ về một kiểu đệ quy, hãy khám phá _cons list_. Đây là một kiểu dữ liệu
thường thấy trong các ngôn ngữ lập trình hàm. Kiểu cons list mà chúng ta sẽ định
nghĩa khá đơn giản ngoại trừ phần đệ quy; do đó, các khái niệm trong ví dụ chúng
ta sẽ làm việc sẽ hữu ích bất cứ khi nào bạn gặp phải các tình huống phức tạp
hơn liên quan đến kiểu đệ quy.

#### Thêm Thông tin về Cons List

Một _cons list_ là một cấu trúc dữ liệu xuất phát từ ngôn ngữ lập trình Lisp và
các phương ngữ của nó, được tạo thành từ các cặp lồng nhau, và là phiên bản Lisp
của danh sách liên kết. Tên của nó xuất phát từ hàm `cons` (viết tắt của
_construct function_) trong Lisp xây dựng một cặp mới từ hai đối số của nó. Bằng
cách gọi `cons` trên một cặp bao gồm một giá trị và một cặp khác, chúng ta có
thể xây dựng các cons list được tạo thành từ các cặp đệ quy.

Ví dụ, đây là biểu diễn mã giả của một cons list chứa danh sách `1, 2, 3` với
mỗi cặp trong ngoặc đơn:

```text
(1, (2, (3, Nil)))
```

Mỗi phần tử trong một cons list chứa hai thành phần: giá trị của phần tử hiện
tại và phần tử tiếp theo. Phần tử cuối cùng trong danh sách chỉ chứa một giá trị
gọi là `Nil` mà không có phần tử tiếp theo. Một cons list được tạo ra bằng cách
gọi đệ quy hàm `cons`. Tên quy ước để biểu thị trường hợp cơ sở của đệ quy là
`Nil`. Lưu ý rằng điều này không giống với khái niệm "null" hoặc "nil" được thảo
luận trong Chương 6, là một giá trị không hợp lệ hoặc vắng mặt.

Cons list không phải là cấu trúc dữ liệu thường được sử dụng trong Rust. Hầu hết
thời gian khi bạn có một danh sách các mục trong Rust, `Vec<T>` là một lựa chọn
tốt hơn để sử dụng. Các kiểu dữ liệu đệ quy phức tạp khác _thực sự_ hữu ích
trong nhiều tình huống khác nhau, nhưng bằng cách bắt đầu với cons list trong
chương này, chúng ta có thể khám phá cách box cho phép chúng ta định nghĩa một
kiểu dữ liệu đệ quy mà không bị phân tâm quá nhiều.

Listing 15-2 chứa định nghĩa enum cho một cons list. Lưu ý rằng mã này sẽ chưa
biên dịch được vì kiểu `List` không có kích thước đã biết, điều này chúng ta sẽ
minh họa.

<Listing number="15-2" file-name="src/main.rs" caption="Nỗ lực đầu tiên để định nghĩa một enum đại diện cho cấu trúc dữ liệu cons list của các giá trị `i32`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-02/src/main.rs:here}}
```

</Listing>

> Lưu ý: Chúng ta đang triển khai một cons list chỉ chứa các giá trị `i32` cho
> mục đích của ví dụ này. Chúng ta có thể đã triển khai nó bằng cách sử dụng
> generics, như chúng ta đã thảo luận trong Chương 10, để định nghĩa một kiểu
> cons list có thể lưu trữ các giá trị của bất kỳ kiểu nào.

Sử dụng kiểu `List` để lưu trữ danh sách `1, 2, 3` sẽ trông giống như mã trong
Listing 15-3.

<Listing number="15-3" file-name="src/main.rs" caption="Sử dụng enum `List` để lưu trữ danh sách `1, 2, 3`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-03/src/main.rs:here}}
```

</Listing>

Giá trị `Cons` đầu tiên chứa `1` và một giá trị `List` khác. Giá trị `List` này
là một giá trị `Cons` khác chứa `2` và một giá trị `List` khác. Giá trị `List`
này là một giá trị `Cons` nữa chứa `3` và một giá trị `List`, cuối cùng là
`Nil`, biến thể không đệ quy biểu thị kết thúc của danh sách.

Nếu chúng ta cố gắng biên dịch mã trong Listing 15-3, chúng ta sẽ gặp lỗi như
trong Listing 15-4.

<Listing number="15-4" file-name="output.txt" caption="Lỗi chúng ta gặp phải khi cố gắng định nghĩa một enum đệ quy">

```console
{{#include ../listings/ch15-smart-pointers/listing-15-03/output.txt}}
```

</Listing>

Lỗi cho thấy kiểu này "có kích thước vô hạn." Lý do là chúng ta đã định nghĩa
`List` với một biến thể đệ quy: nó trực tiếp chứa một giá trị khác của chính nó.
Kết quả là, Rust không thể tìm ra cần bao nhiêu không gian để lưu trữ một giá
trị `List`. Hãy phân tích tại sao chúng ta gặp lỗi này. Đầu tiên, chúng ta sẽ
xem cách Rust quyết định cần bao nhiêu không gian để lưu trữ một giá trị của một
kiểu không đệ quy.

#### Tính toán Kích thước của một Kiểu Không Đệ quy

Nhớ lại enum `Message` mà chúng ta đã định nghĩa trong Listing 6-2 khi chúng ta
thảo luận về định nghĩa enum trong Chương 6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

Để xác định bao nhiêu không gian để cấp phát cho một giá trị `Message`, Rust đi
qua từng biến thể để xem biến thể nào cần nhiều không gian nhất. Rust thấy rằng
`Message::Quit` không cần bất kỳ không gian nào, `Message::Move` cần đủ không
gian để lưu trữ hai giá trị `i32`, và vân vân. Vì chỉ một biến thể sẽ được sử
dụng, không gian nhiều nhất mà một giá trị `Message` sẽ cần là không gian cần
thiết để lưu trữ biến thể lớn nhất trong số các biến thể của nó.

Đối chiếu với điều xảy ra khi Rust cố gắng xác định bao nhiêu không gian mà một
kiểu đệ quy như enum `List` trong Listing 15-2 cần. Trình biên dịch bắt đầu bằng
cách nhìn vào biến thể `Cons`, biến thể này chứa một giá trị của kiểu `i32` và
một giá trị của kiểu `List`. Do đó, `Cons` cần một lượng không gian bằng kích
thước của `i32` cộng với kích thước của `List`. Để tìm ra bộ nhớ mà kiểu `List`
cần, trình biên dịch xem xét các biến thể, bắt đầu với biến thể `Cons`. Biến thể
`Cons` chứa một giá trị kiểu `i32` và một giá trị kiểu `List`, và quá trình này
tiếp tục vô hạn, như được minh họa trong Hình 15-1.

<img alt="Một Cons list vô hạn: một hình chữ nhật có nhãn 'Cons' được chia thành hai hình chữ nhật nhỏ hơn. Hình chữ nhật nhỏ đầu tiên chứa nhãn 'i32', và hình chữ nhật nhỏ thứ hai chứa nhãn 'Cons' và một phiên bản nhỏ hơn của hình chữ nhật 'Cons' bên ngoài. Các hình chữ nhật 'Cons' tiếp tục chứa các phiên bản ngày càng nhỏ hơn của chính chúng cho đến khi hình chữ nhật nhỏ nhất có kích thước thoải mái chứa một ký hiệu vô hạn, cho biết rằng sự lặp lại này kéo dài mãi mãi" src="img/trpl15-01.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 15-1: Một `List` vô hạn bao gồm các biến thể `Cons`
vô hạn</span>

#### Sử dụng `Box<T>` để Có được Kiểu Đệ quy với Kích thước Đã biết

Vì Rust không thể tìm ra bao nhiêu không gian để cấp phát cho các kiểu được định
nghĩa đệ quy, trình biên dịch đưa ra một lỗi với gợi ý hữu ích này:

<!-- manual-regeneration
after doing automatic regeneration, look at listings/ch15-smart-pointers/listing-15-03/output.txt and copy the relevant line
-->

```text
help: insert some indirection (e.g., a `Box`, `Rc`, or `&`) to break the cycle
  |
2 |     Cons(i32, Box<List>),
  |               ++++    +
```

Trong gợi ý này, _sự gián tiếp_ có nghĩa là thay vì lưu trữ một giá trị trực
tiếp, chúng ta nên thay đổi cấu trúc dữ liệu để lưu trữ giá trị gián tiếp bằng
cách lưu trữ một con trỏ đến giá trị thay vì lưu trữ giá trị trực tiếp.

Vì `Box<T>` là một con trỏ, Rust luôn biết bao nhiêu không gian mà một `Box<T>`
cần: kích thước của một con trỏ không thay đổi dựa trên lượng dữ liệu mà nó trỏ
đến. Điều này có nghĩa là chúng ta có thể đặt một `Box<T>` bên trong biến thể
`Cons` thay vì một giá trị `List` trực tiếp. `Box<T>` sẽ trỏ đến giá trị `List`
tiếp theo sẽ nằm trên heap thay vì bên trong biến thể `Cons`. Về mặt khái niệm,
chúng ta vẫn có một danh sách, được tạo ra với các danh sách chứa các danh sách
khác, nhưng việc triển khai này bây giờ giống như đặt các mục cạnh nhau thay vì
bên trong nhau.

Chúng ta có thể thay đổi định nghĩa của enum `List` trong Listing 15-2 và cách
sử dụng `List` trong Listing 15-3 thành mã trong Listing 15-5, mã này sẽ biên
dịch được.

<Listing number="15-5" file-name="src/main.rs" caption="Định nghĩa của `List` sử dụng `Box<T>` để có kích thước đã biết">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-05/src/main.rs}}
```

</Listing>

Biến thể `Cons` cần kích thước của một `i32` cộng với không gian để lưu trữ dữ
liệu con trỏ của box. Biến thể `Nil` không lưu trữ giá trị nào, nên nó cần ít
không gian trên stack hơn biến thể `Cons`. Bây giờ chúng ta biết rằng bất kỳ giá
trị `List` nào sẽ chiếm kích thước của một `i32` cộng với kích thước của dữ liệu
con trỏ của box. Bằng cách sử dụng box, chúng ta đã phá vỡ chuỗi đệ quy vô hạn,
vì vậy trình biên dịch có thể tìm ra kích thước cần thiết để lưu trữ một giá trị
`List`. Hình 15-2 cho thấy biến thể `Cons` trông như thế nào bây giờ.

<img alt="Một hình chữ nhật có nhãn 'Cons' được chia thành hai hình chữ nhật nhỏ hơn. Hình chữ nhật nhỏ đầu tiên chứa nhãn 'i32', và hình chữ nhật nhỏ thứ hai chứa nhãn 'Box' với một hình chữ nhật bên trong chứa nhãn 'usize', biểu thị kích thước hữu hạn của con trỏ của box" src="img/trpl15-02.svg" class="center" />

<span class="caption">Hình 15-2: Một `List` không có kích thước vô hạn vì `Cons`
chứa một `Box`</span>

Box chỉ cung cấp sự gián tiếp và cấp phát heap; chúng không có bất kỳ khả năng
đặc biệt nào khác, như những gì chúng ta sẽ thấy với các kiểu con trỏ thông minh
khác. Chúng cũng không có chi phí hiệu suất mà các khả năng đặc biệt này gây ra,
vì vậy chúng có thể hữu ích trong các trường hợp như cons list, nơi sự gián tiếp
là tính năng duy nhất chúng ta cần. Chúng ta sẽ xem xét thêm các trường hợp sử
dụng cho box trong Chương 18.

Kiểu `Box<T>` là một con trỏ thông minh vì nó triển khai trait `Deref`, cho phép
các giá trị `Box<T>` được xử lý như tham chiếu. Khi một giá trị `Box<T>` ra khỏi
phạm vi, dữ liệu heap mà box đang trỏ đến cũng được dọn dẹp nhờ vào việc triển
khai trait `Drop`. Hai trait này sẽ càng quan trọng hơn đối với chức năng được
cung cấp bởi các kiểu con trỏ thông minh khác mà chúng ta sẽ thảo luận trong
phần còn lại của chương này. Hãy khám phá hai trait này chi tiết hơn.

[trait-objects]:
  ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
