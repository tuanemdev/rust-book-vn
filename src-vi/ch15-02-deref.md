## Xử lý Con trỏ Thông minh Như Tham chiếu Thông thường với `Deref`

<!-- Old link, do not remove -->

<a id="treating-smart-pointers-like-regular-references-with-the-deref-trait"></a>

Việc triển khai trait `Deref` cho phép bạn tùy chỉnh hành vi của _toán tử giải
tham chiếu_ `*` (không nên nhầm lẫn với toán tử nhân hoặc toán tử glob). Bằng
cách triển khai `Deref` theo cách mà một con trỏ thông minh có thể được xử lý
như một tham chiếu thông thường, bạn có thể viết mã hoạt động trên tham chiếu và
sử dụng mã đó với con trỏ thông minh.

Đầu tiên, hãy xem cách toán tử giải tham chiếu hoạt động với tham chiếu thông
thường. Sau đó, chúng ta sẽ thử định nghĩa một kiểu tùy chỉnh hoạt động giống
như `Box<T>`, và xem tại sao toán tử giải tham chiếu không hoạt động như một
tham chiếu trên kiểu mới định nghĩa của chúng ta. Chúng ta sẽ khám phá cách
triển khai trait `Deref` khiến con trỏ thông minh có thể hoạt động tương tự như
tham chiếu. Sau đó, chúng ta sẽ xem tính năng _chuyển đổi giải tham chiếu_
(deref coercion) của Rust và cách nó cho phép chúng ta làm việc với cả tham
chiếu hoặc con trỏ thông minh.

> Lưu ý: Có một sự khác biệt lớn giữa kiểu `MyBox<T>` mà chúng ta sắp xây dựng
> và `Box<T>` thực tế: phiên bản của chúng ta sẽ không lưu trữ dữ liệu trên
> heap. Chúng ta đang tập trung ví dụ này vào `Deref`, vì vậy nơi dữ liệu thực
> sự được lưu trữ ít quan trọng hơn hành vi giống con trỏ.

<!-- Old links, do not remove -->

<a id="following-the-pointer-to-the-value-with-the-dereference-operator"></a>
<a id="following-the-pointer-to-the-value"></a>

### Theo dõi Tham chiếu đến Giá trị

Một tham chiếu thông thường là một loại con trỏ, và một cách để nghĩ về con trỏ
là như một mũi tên đến một giá trị được lưu trữ ở đâu đó. Trong Listing 15-6,
chúng ta tạo một tham chiếu đến một giá trị `i32` và sau đó sử dụng toán tử giải
tham chiếu để theo dõi tham chiếu đến giá trị.

<Listing number="15-6" file-name="src/main.rs" caption="Sử dụng toán tử giải tham chiếu để theo dõi một tham chiếu đến một giá trị `i32`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-06/src/main.rs}}
```

</Listing>

Biến `x` chứa một giá trị `i32` là `5`. Chúng ta đặt `y` bằng một tham chiếu đến
`x`. Chúng ta có thể khẳng định rằng `x` bằng `5`. Tuy nhiên, nếu chúng ta muốn
khẳng định về giá trị trong `y`, chúng ta phải sử dụng `*y` để theo dõi tham
chiếu đến giá trị mà nó đang trỏ tới (do đó _giải tham chiếu_) để trình biên
dịch có thể so sánh giá trị thực. Một khi chúng ta giải tham chiếu `y`, chúng ta
có quyền truy cập vào giá trị số nguyên mà `y` đang trỏ tới mà chúng ta có thể
so sánh với `5`.

Nếu chúng ta cố gắng viết `assert_eq!(5, y);` thay vào đó, chúng ta sẽ nhận được
lỗi biên dịch này:

```console
{{#include ../listings/ch15-smart-pointers/output-only-01-comparing-to-reference/output.txt}}
```

So sánh một số và một tham chiếu đến một số không được phép vì chúng là các kiểu
khác nhau. Chúng ta phải sử dụng toán tử giải tham chiếu để theo dõi tham chiếu
đến giá trị mà nó đang trỏ tới.

### Sử dụng `Box<T>` Như một Tham chiếu

Chúng ta có thể viết lại mã trong Listing 15-6 để sử dụng `Box<T>` thay vì một
tham chiếu; toán tử giải tham chiếu được sử dụng trên `Box<T>` trong Listing
15-7 hoạt động theo cùng một cách như toán tử giải tham chiếu được sử dụng trên
tham chiếu trong Listing 15-6:

<Listing number="15-7" file-name="src/main.rs" caption="Sử dụng toán tử giải tham chiếu trên `Box<i32>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-07/src/main.rs}}
```

</Listing>

Sự khác biệt chính giữa Listing 15-7 và Listing 15-6 là ở đây chúng ta đặt `y`
là một thực thể của một box trỏ đến một giá trị sao chép của `x` thay vì một
tham chiếu trỏ đến giá trị của `x`. Trong khẳng định cuối cùng, chúng ta có thể
sử dụng toán tử giải tham chiếu để theo dõi con trỏ của box theo cách tương tự
như khi `y` là một tham chiếu. Tiếp theo, chúng ta sẽ khám phá điều gì đặc biệt
về `Box<T>` cho phép chúng ta sử dụng toán tử giải tham chiếu bằng cách định
nghĩa kiểu của riêng chúng ta.

### Định nghĩa Con trỏ Thông minh Riêng của Chúng ta

Hãy xây dựng một con trỏ thông minh tương tự như kiểu `Box<T>` được cung cấp bởi
thư viện chuẩn để trải nghiệm cách con trỏ thông minh hoạt động khác với tham
chiếu theo mặc định. Sau đó, chúng ta sẽ xem cách thêm khả năng sử dụng toán tử
giải tham chiếu.

Kiểu `Box<T>` cuối cùng được định nghĩa như một tuple struct với một phần tử, vì
vậy Listing 15-8 định nghĩa một kiểu `MyBox<T>` theo cách tương tự. Chúng ta
cũng sẽ định nghĩa một hàm `new` để phù hợp với hàm `new` được định nghĩa trên
`Box<T>`.

<Listing number="15-8" file-name="src/main.rs" caption="Định nghĩa một kiểu `MyBox<T>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-08/src/main.rs:here}}
```

</Listing>

Chúng ta định nghĩa một struct có tên `MyBox` và khai báo một tham số generic
`T` vì chúng ta muốn kiểu của chúng ta giữ các giá trị của bất kỳ kiểu nào. Kiểu
`MyBox` là một tuple struct với một phần tử kiểu `T`. Hàm `MyBox::new` nhận một
tham số kiểu `T` và trả về một thực thể `MyBox` chứa giá trị được truyền vào.

Hãy thử thêm hàm `main` trong Listing 15-7 vào Listing 15-8 và thay đổi nó để sử
dụng kiểu `MyBox<T>` mà chúng ta đã định nghĩa thay vì `Box<T>`. Mã trong
Listing 15-9 sẽ không biên dịch vì Rust không biết cách giải tham chiếu `MyBox`.

<Listing number="15-9" file-name="src/main.rs" caption="Cố gắng sử dụng `MyBox<T>` theo cách tương tự như chúng ta đã sử dụng tham chiếu và `Box<T>`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-09/src/main.rs:here}}
```

</Listing>

Đây là lỗi biên dịch kết quả:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-09/output.txt}}
```

Kiểu `MyBox<T>` của chúng ta không thể được giải tham chiếu vì chúng ta chưa
triển khai khả năng đó trên kiểu của chúng ta. Để kích hoạt giải tham chiếu với
toán tử `*`, chúng ta triển khai trait `Deref`.

<!-- Old link, do not remove -->

<a id="treating-a-type-like-a-reference-by-implementing-the-deref-trait"></a>

### Triển khai Trait `Deref`

Như đã thảo luận trong ["Triển khai một Trait trên một
Kiểu"][impl-trait]<!-- ignore --> trong Chương 10, để triển khai một trait,
chúng ta cần cung cấp các triển khai cho các phương thức yêu cầu của trait.
Trait `Deref`, được cung cấp bởi thư viện chuẩn, yêu cầu chúng ta triển khai một
phương thức có tên `deref` mượn `self` và trả về một tham chiếu đến dữ liệu bên
trong. Listing 15-10 chứa một triển khai của `Deref` để thêm vào định nghĩa của
`MyBox<T>`.

<Listing number="15-10" file-name="src/main.rs" caption="Triển khai `Deref` trên `MyBox<T>`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-10/src/main.rs:here}}
```

</Listing>

Cú pháp `type Target = T;` định nghĩa một kiểu liên kết cho trait `Deref` để sử
dụng. Kiểu liên kết là một cách khác để khai báo một tham số generic, nhưng bạn
không cần phải lo lắng về chúng bây giờ; chúng ta sẽ đề cập đến chúng chi tiết
hơn trong Chương 20.

Chúng ta điền vào thân của phương thức `deref` với `&self.0` để `deref` trả về
một tham chiếu đến giá trị mà chúng ta muốn truy cập với toán tử `*`; nhớ lại từ
["Sử dụng Tuple Structs Không Có Trường Có Tên để Tạo Các Kiểu Khác
nhau"][tuple-structs]<!-- ignore --> trong Chương 5 rằng `.0` truy cập giá trị
đầu tiên trong một tuple struct. Hàm `main` trong Listing 15-9 gọi `*` trên giá
trị `MyBox<T>` bây giờ biên dịch được, và các khẳng định đều thành công!

Không có trait `Deref`, trình biên dịch chỉ có thể giải tham chiếu tham chiếu
`&`. Phương thức `deref` cung cấp cho trình biên dịch khả năng lấy một giá trị
của bất kỳ kiểu nào triển khai `Deref` và gọi phương thức `deref` để lấy tham
chiếu `&` mà nó biết cách giải tham chiếu.

Khi chúng ta nhập `*y` trong Listing 15-9, đằng sau hậu trường Rust thực sự chạy
mã này:

```rust,ignore
*(y.deref())
```

Rust thay thế toán tử `*` bằng một lệnh gọi đến phương thức `deref` và sau đó là
một giải tham chiếu thông thường để chúng ta không phải suy nghĩ về việc liệu
chúng ta có cần gọi phương thức `deref` hay không. Tính năng Rust này cho phép
chúng ta viết mã hoạt động giống hệt nhau dù chúng ta có một tham chiếu thông
thường hay một kiểu triển khai `Deref`.

Lý do phương thức `deref` trả về một tham chiếu đến một giá trị, và giải tham
chiếu thông thường bên ngoài dấu ngoặc đơn trong `*(y.deref())` vẫn cần thiết,
liên quan đến hệ thống sở hữu. Nếu phương thức `deref` trả về giá trị trực tiếp
thay vì một tham chiếu đến giá trị, giá trị sẽ bị di chuyển ra khỏi `self`.
Chúng ta không muốn lấy quyền sở hữu của giá trị bên trong `MyBox<T>` trong
trường hợp này hoặc trong hầu hết các trường hợp khi chúng ta sử dụng toán tử
giải tham chiếu.

Lưu ý rằng toán tử `*` được thay thế bằng một lệnh gọi đến phương thức `deref`
và sau đó là một lệnh gọi đến toán tử `*` chỉ một lần, mỗi lần chúng ta sử dụng
`*` trong mã của chúng ta. Vì sự thay thế của toán tử `*` không đệ quy vô hạn,
chúng ta kết thúc với dữ liệu kiểu `i32`, phù hợp với `5` trong `assert_eq!`
trong Listing 15-9.

### Chuyển đổi Giải tham chiếu Ngầm với Hàm và Phương thức

_Chuyển đổi giải tham chiếu_ (Deref coercion) chuyển đổi một tham chiếu đến một
kiểu triển khai trait `Deref` thành một tham chiếu đến một kiểu khác. Ví dụ,
chuyển đổi giải tham chiếu có thể chuyển đổi `&String` thành `&str` vì `String`
triển khai trait `Deref` sao cho nó trả về `&str`. Chuyển đổi giải tham chiếu là
một tiện ích Rust thực hiện trên các đối số cho hàm và phương thức, và chỉ hoạt
động trên các kiểu triển khai trait `Deref`. Nó xảy ra tự động khi chúng ta
truyền một tham chiếu đến giá trị của một kiểu cụ thể làm đối số cho một hàm
hoặc phương thức mà không khớp với kiểu tham số trong định nghĩa hàm hoặc phương
thức. Một chuỗi các lệnh gọi đến phương thức `deref` chuyển đổi kiểu chúng ta đã
cung cấp thành kiểu mà tham số cần.

Chuyển đổi giải tham chiếu được thêm vào Rust để các lập trình viên viết lệnh
gọi hàm và phương thức không cần thêm quá nhiều tham chiếu và giải tham chiếu rõ
ràng với `&` và `*`. Tính năng chuyển đổi giải tham chiếu cũng cho phép chúng ta
viết nhiều mã hơn có thể hoạt động cho cả tham chiếu hoặc con trỏ thông minh.

Để thấy chuyển đổi giải tham chiếu trong hành động, hãy sử dụng kiểu `MyBox<T>`
mà chúng ta đã định nghĩa trong Listing 15-8 cũng như triển khai của `Deref` mà
chúng ta đã thêm vào trong Listing 15-10. Listing 15-11 hiển thị định nghĩa của
một hàm có tham số là một lát chuỗi.

<Listing number="15-11" file-name="src/main.rs" caption="Một hàm `hello` có tham số `name` kiểu `&str`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-11/src/main.rs:here}}
```

</Listing>

Chúng ta có thể gọi hàm `hello` với một lát chuỗi làm đối số, chẳng hạn như
`hello("Rust");`. Chuyển đổi giải tham chiếu làm cho việc gọi `hello` với một
tham chiếu đến một giá trị kiểu `MyBox<String>` trở nên khả thi, như trong
Listing 15-12.

<Listing number="15-12" file-name="src/main.rs" caption="Gọi `hello` với một tham chiếu đến một giá trị `MyBox<String>`, hoạt động vì chuyển đổi giải tham chiếu">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-12/src/main.rs:here}}
```

</Listing>

Ở đây chúng ta đang gọi hàm `hello` với đối số `&m`, là một tham chiếu đến một
giá trị `MyBox<String>`. Vì chúng ta đã triển khai trait `Deref` trên `MyBox<T>`
trong Listing 15-10, Rust có thể chuyển đổi `&MyBox<String>` thành `&String`
bằng cách gọi `deref`. Thư viện chuẩn cung cấp một triển khai của `Deref` trên
`String` trả về một lát chuỗi, và điều này có trong tài liệu API cho `Deref`.
Rust gọi `deref` một lần nữa để chuyển đổi `&String` thành `&str`, khớp với định
nghĩa của hàm `hello`.

Nếu Rust không triển khai chuyển đổi giải tham chiếu, chúng ta sẽ phải viết mã
trong Listing 15-13 thay vì mã trong Listing 15-12 để gọi `hello` với một giá
trị kiểu `&MyBox<String>`.

<Listing number="15-13" file-name="src/main.rs" caption="Mã chúng ta sẽ phải viết nếu Rust không có chuyển đổi giải tham chiếu">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-13/src/main.rs:here}}
```

</Listing>

`(*m)` giải tham chiếu `MyBox<String>` thành một `String`. Sau đó, `&` và `[..]`
lấy một lát chuỗi của `String` bằng với toàn bộ chuỗi để khớp với chữ ký của
`hello`. Mã này không có chuyển đổi giải tham chiếu khó đọc, viết và hiểu hơn
với tất cả các ký hiệu liên quan. Chuyển đổi giải tham chiếu cho phép Rust xử lý
các chuyển đổi này cho chúng ta tự động.

Khi trait `Deref` được định nghĩa cho các kiểu liên quan, Rust sẽ phân tích các
kiểu và sử dụng `Deref::deref` nhiều lần nếu cần để lấy một tham chiếu khớp với
kiểu của tham số. Số lần cần thiết để chèn `Deref::deref` được giải quyết tại
thời điểm biên dịch, vì vậy không có hình phạt thời gian chạy khi tận dụng lợi
thế của chuyển đổi giải tham chiếu!

### Cách Chuyển đổi Giải tham chiếu Tương tác với Tính Thay đổi

Tương tự như cách bạn sử dụng trait `Deref` để ghi đè lên toán tử `*` trên tham
chiếu không thay đổi, bạn có thể sử dụng trait `DerefMut` để ghi đè lên toán tử
`*` trên tham chiếu có thể thay đổi.

Rust thực hiện chuyển đổi giải tham chiếu khi nó tìm thấy các kiểu và triển khai
trait trong ba trường hợp:

1. Từ `&T` đến `&U` khi `T: Deref<Target=U>`
2. Từ `&mut T` đến `&mut U` khi `T: DerefMut<Target=U>`
3. Từ `&mut T` đến `&U` khi `T: Deref<Target=U>`

Hai trường hợp đầu tiên giống nhau ngoại trừ trường hợp thứ hai triển khai tính
thay đổi. Trường hợp đầu tiên nêu rằng nếu bạn có một `&T`, và `T` triển khai
`Deref` đến một kiểu `U` nào đó, bạn có thể nhận được một `&U` một cách trong
suốt. Trường hợp thứ hai nêu rằng sự chuyển đổi giải tham chiếu tương tự cũng
xảy ra cho tham chiếu có thể thay đổi.

Trường hợp thứ ba phức tạp hơn: Rust cũng sẽ chuyển đổi một tham chiếu có thể
thay đổi thành một tham chiếu không thay đổi. Nhưng điều ngược lại _không_ khả
thi: tham chiếu không thay đổi sẽ không bao giờ chuyển đổi thành tham chiếu có
thể thay đổi. Vì quy tắc mượn, nếu bạn có một tham chiếu có thể thay đổi, tham
chiếu có thể thay đổi đó phải là tham chiếu duy nhất đến dữ liệu đó (nếu không,
chương trình sẽ không biên dịch). Chuyển đổi một tham chiếu có thể thay đổi
thành một tham chiếu không thay đổi sẽ không bao giờ phá vỡ quy tắc mượn. Chuyển
đổi một tham chiếu không thay đổi thành một tham chiếu có thể thay đổi sẽ yêu
cầu tham chiếu không thay đổi ban đầu là tham chiếu không thay đổi duy nhất đến
dữ liệu đó, nhưng quy tắc mượn không đảm bảo điều đó. Do đó, Rust không thể đưa
ra giả định rằng việc chuyển đổi một tham chiếu không thay đổi thành một tham
chiếu có thể thay đổi là khả thi.

[impl-trait]: ch10-02-traits.html#implementing-a-trait-on-a-type
[tuple-structs]:
  ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
