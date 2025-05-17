## Sử dụng Đối tượng Trait để Trừu tượng hóa Hành vi Chung

<!-- Old headings. Do not remove or links may break. -->

<a id="using-trait-objects-that-allow-for-values-of-different-types"></a>

Trong Chương 8, chúng ta đã đề cập rằng một hạn chế của vector là chúng chỉ có
thể lưu trữ các phần tử của cùng một kiểu. Chúng ta đã tạo ra một giải pháp thay
thế trong Listing 8-9, trong đó chúng ta định nghĩa một enum `SpreadsheetCell`
có các biến thể để chứa số nguyên, số thực và văn bản. Điều này có nghĩa là
chúng ta có thể lưu trữ các kiểu dữ liệu khác nhau trong mỗi ô và vẫn có một
vector đại diện cho một hàng các ô. Đây là một giải pháp hoàn toàn phù hợp khi
các mục có thể thay thế của chúng ta là một tập hợp các kiểu cố định mà chúng ta
biết khi mã của chúng ta được biên dịch.

Tuy nhiên, đôi khi chúng ta muốn người dùng thư viện của mình có thể mở rộng tập
hợp các kiểu hợp lệ trong một tình huống cụ thể. Để minh họa cách chúng ta có
thể thực hiện điều này, chúng ta sẽ tạo một ví dụ về công cụ giao diện người
dùng đồ họa (GUI) lặp qua danh sách các mục, gọi phương thức `draw` trên mỗi mục
để vẽ nó lên màn hình—một kỹ thuật phổ biến cho các công cụ GUI. Chúng ta sẽ tạo
một crate thư viện có tên `gui` chứa cấu trúc của thư viện GUI. Crate này có thể
bao gồm một số kiểu cho mọi người sử dụng, chẳng hạn như `Button` hoặc
`TextField`. Ngoài ra, người dùng `gui` sẽ muốn tạo các kiểu riêng của họ có thể
được vẽ: ví dụ, một lập trình viên có thể thêm `Image` và một lập trình viên
khác có thể thêm `SelectBox`.

Tại thời điểm viết thư viện, chúng ta không thể biết và định nghĩa tất cả các
kiểu mà các lập trình viên khác có thể muốn tạo. Nhưng chúng ta biết rằng `gui`
cần theo dõi nhiều giá trị của các kiểu khác nhau và cần gọi phương thức `draw`
trên mỗi giá trị có kiểu khác nhau này. Nó không cần biết chính xác điều gì sẽ
xảy ra khi chúng ta gọi phương thức `draw`, chỉ cần biết rằng giá trị đó sẽ có
phương thức sẵn sàng để gọi.

Để thực hiện điều này trong ngôn ngữ có tính kế thừa, chúng ta có thể định nghĩa
một lớp có tên `Component` có phương thức tên là `draw`. Các lớp khác, chẳng hạn
như `Button`, `Image` và `SelectBox`, sẽ kế thừa từ `Component` và do đó kế thừa
phương thức `draw`. Mỗi lớp có thể ghi đè phương thức `draw` để định nghĩa hành
vi tùy chỉnh của nó, nhưng framework có thể xử lý tất cả các kiểu như thể chúng
là các thể hiện của `Component` và gọi `draw` trên chúng. Nhưng vì Rust không có
tính kế thừa, chúng ta cần một cách khác để cấu trúc thư viện `gui` để cho phép
người dùng tạo các kiểu mới tương thích với thư viện.

### Định nghĩa một Trait cho Hành vi Chung

Để triển khai hành vi mà chúng ta muốn `gui` có, chúng ta sẽ định nghĩa một
trait có tên `Draw` có một phương thức tên `draw`. Sau đó, chúng ta có thể định
nghĩa một vector nhận một đối tượng trait. Một _đối tượng trait_ trỏ đến cả một
thể hiện của một kiểu triển khai trait được chỉ định của chúng ta và một bảng
được sử dụng để tra cứu các phương thức trait trên kiểu đó tại thời điểm chạy.
Chúng ta tạo một đối tượng trait bằng cách chỉ định một loại con trỏ, chẳng hạn
như tham chiếu `&` hoặc con trỏ thông minh `Box<T>`, sau đó là từ khóa `dyn`, và
sau đó chỉ định trait liên quan. (Chúng ta sẽ nói về lý do đối tượng trait phải
sử dụng con trỏ trong ["Các kiểu Kích thước Động và Trait
`Sized`"][dynamically-sized]<!-- ignore --> trong Chương 20.) Chúng ta có thể sử
dụng đối tượng trait thay thế cho kiểu generic hoặc kiểu cụ thể. Bất cứ khi nào
chúng ta sử dụng đối tượng trait, hệ thống kiểu của Rust sẽ đảm bảo tại thời
điểm biên dịch rằng bất kỳ giá trị nào được sử dụng trong ngữ cảnh đó sẽ triển
khai trait của đối tượng trait. Do đó, chúng ta không cần phải biết tất cả các
kiểu có thể có tại thời điểm biên dịch.

Chúng ta đã đề cập rằng, trong Rust, chúng ta tránh gọi các struct và enum là
"đối tượng" để phân biệt chúng với đối tượng của các ngôn ngữ khác. Trong một
struct hoặc enum, dữ liệu trong các trường struct và hành vi trong các khối
`impl` được tách biệt, trong khi ở các ngôn ngữ khác, dữ liệu và hành vi kết hợp
thành một khái niệm thường được gọi là đối tượng. Đối tượng trait khác với đối
tượng trong các ngôn ngữ khác ở chỗ chúng ta không thể thêm dữ liệu vào đối
tượng trait. Đối tượng trait không hữu ích một cách tổng quát như đối tượng
trong các ngôn ngữ khác: mục đích cụ thể của chúng là cho phép trừu tượng hóa
qua hành vi chung.

Listing 18-3 cho thấy cách định nghĩa một trait có tên `Draw` với một phương
thức có tên `draw`.

<Listing number="18-3" file-name="src/lib.rs" caption="Định nghĩa của trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-03/src/lib.rs}}
```

</Listing>

Cú pháp này có thể quen thuộc từ các cuộc thảo luận của chúng ta về cách định
nghĩa trait trong Chương 10. Tiếp theo là một cú pháp mới: Listing 18-4 định
nghĩa một struct có tên `Screen` chứa một vector có tên `components`. Vector này
có kiểu `Box<dyn Draw>`, là một đối tượng trait; nó là một đại diện cho bất kỳ
kiểu nào bên trong một `Box` triển khai trait `Draw`.

<Listing number="18-4" file-name="src/lib.rs" caption="Định nghĩa của struct `Screen` với trường `components` chứa một vector các đối tượng trait triển khai trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-04/src/lib.rs:here}}
```

</Listing>

Trên struct `Screen`, chúng ta sẽ định nghĩa một phương thức có tên `run` sẽ gọi
phương thức `draw` trên mỗi `components` của nó, như được hiển thị trong Listing
18-5.

<Listing number="18-5" file-name="src/lib.rs" caption="Phương thức `run` trên `Screen` gọi phương thức `draw` trên mỗi thành phần">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-05/src/lib.rs:here}}
```

</Listing>

Điều này hoạt động khác với việc định nghĩa một struct sử dụng tham số kiểu
generic với ràng buộc trait. Một tham số kiểu generic chỉ có thể được thay thế
bằng một kiểu cụ thể tại một thời điểm, trong khi đối tượng trait cho phép nhiều
kiểu cụ thể điền vào đối tượng trait tại thời điểm chạy. Ví dụ, chúng ta có thể
đã định nghĩa struct `Screen` sử dụng kiểu generic và ràng buộc trait, như trong
Listing 18-6.

<Listing number="18-6" file-name="src/lib.rs" caption="Một triển khai thay thế của struct `Screen` và phương thức `run` của nó sử dụng generic và ràng buộc trait">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-06/src/lib.rs:here}}
```

</Listing>

Điều này giới hạn chúng ta vào một thể hiện `Screen` có danh sách các thành phần
tất cả đều có kiểu `Button` hoặc tất cả đều có kiểu `TextField`. Nếu bạn chỉ
muốn có các bộ sưu tập đồng nhất, sử dụng generic và ràng buộc trait là tốt hơn
vì các định nghĩa sẽ được đơn hình hóa tại thời điểm biên dịch để sử dụng các
kiểu cụ thể.

Mặt khác, với phương pháp sử dụng đối tượng trait, một thể hiện `Screen` có thể
chứa một `Vec<T>` bao gồm cả `Box<Button>` và `Box<TextField>`. Hãy xem cách
điều này hoạt động, và sau đó chúng ta sẽ nói về các hệ quả hiệu suất thời gian
chạy.

### Triển khai Trait

Bây giờ chúng ta sẽ thêm một số kiểu triển khai trait `Draw`. Chúng ta sẽ cung
cấp kiểu `Button`. Một lần nữa, việc thực sự triển khai một thư viện GUI nằm
ngoài phạm vi của cuốn sách này, vì vậy phương thức `draw` sẽ không có bất kỳ
triển khai hữu ích nào trong phần thân của nó. Để hình dung triển khai có thể
trông như thế nào, một struct `Button` có thể có các trường cho `width`,
`height` và `label`, như được hiển thị trong Listing 18-7.

<Listing number="18-7" file-name="src/lib.rs" caption="Struct `Button` triển khai trait `Draw`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-07/src/lib.rs:here}}
```

</Listing>

Các trường `width`, `height` và `label` trên `Button` sẽ khác với các trường
trên các thành phần khác; ví dụ, một kiểu `TextField` có thể có những trường
giống nhau cộng với trường `placeholder`. Mỗi kiểu mà chúng ta muốn vẽ trên màn
hình sẽ triển khai trait `Draw` nhưng sẽ sử dụng mã khác nhau trong phương thức
`draw` để định nghĩa cách vẽ kiểu cụ thể đó, như `Button` đã làm ở đây (không có
mã GUI thực tế, như đã đề cập). Kiểu `Button`, ví dụ, có thể có một khối `impl`
bổ sung chứa các phương thức liên quan đến những gì xảy ra khi người dùng nhấp
vào nút. Những loại phương thức này sẽ không áp dụng cho các kiểu như
`TextField`.

Nếu ai đó sử dụng thư viện của chúng ta quyết định triển khai một struct
`SelectBox` có các trường `width`, `height` và `options`, họ cũng sẽ triển khai
trait `Draw` trên kiểu `SelectBox`, như được hiển thị trong Listing 18-8.

<Listing number="18-8" file-name="src/main.rs" caption="Một crate khác sử dụng `gui` và triển khai trait `Draw` trên struct `SelectBox`">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-08/src/main.rs:here}}
```

</Listing>

Người dùng thư viện của chúng ta bây giờ có thể viết hàm `main` của họ để tạo
một thể hiện `Screen`. Với thể hiện `Screen`, họ có thể thêm một `SelectBox` và
một `Button` bằng cách đặt mỗi cái vào một `Box<T>` để trở thành một đối tượng
trait. Sau đó, họ có thể gọi phương thức `run` trên thể hiện `Screen`, sẽ gọi
`draw` trên mỗi thành phần. Listing 18-9 cho thấy triển khai này.

<Listing number="18-9" file-name="src/main.rs" caption="Sử dụng đối tượng trait để lưu trữ các giá trị của các kiểu khác nhau triển khai cùng một trait">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-09/src/main.rs:here}}
```

</Listing>

Khi chúng ta viết thư viện, chúng ta không biết rằng ai đó có thể thêm kiểu
`SelectBox`, nhưng triển khai `Screen` của chúng ta có thể hoạt động với kiểu
mới và vẽ nó vì `SelectBox` triển khai trait `Draw`, có nghĩa là nó triển khai
phương thức `draw`.

Khái niệm này—chỉ quan tâm đến các thông điệp mà một giá trị phản hồi thay vì
kiểu cụ thể của giá trị—tương tự như khái niệm _duck typing_ trong các ngôn ngữ
kiểu động: nếu nó đi như một con vịt và kêu quạc quạc như một con vịt, thì nó
phải là một con vịt! Trong triển khai của `run` trên `Screen` trong Listing
18-5, `run` không cần biết kiểu cụ thể của mỗi thành phần là gì. Nó không kiểm
tra liệu một thành phần có phải là một thể hiện của `Button` hay `SelectBox`, nó
chỉ gọi phương thức `draw` trên thành phần. Bằng cách chỉ định `Box<dyn Draw>`
là kiểu của các giá trị trong vector `components`, chúng ta đã định nghĩa
`Screen` cần các giá trị mà chúng ta có thể gọi phương thức `draw` trên đó.

Lợi thế của việc sử dụng đối tượng trait và hệ thống kiểu của Rust để viết mã
tương tự như mã sử dụng duck typing là chúng ta không bao giờ phải kiểm tra liệu
một giá trị có triển khai một phương thức cụ thể nào đó tại thời điểm chạy hoặc
lo lắng về việc gặp lỗi nếu một giá trị không triển khai một phương thức nhưng
chúng ta vẫn gọi nó. Rust sẽ không biên dịch mã của chúng ta nếu các giá trị
không triển khai các trait mà đối tượng trait cần.

Ví dụ, Listing 18-10 cho thấy điều gì xảy ra nếu chúng ta cố gắng tạo một
`Screen` với một `String` làm thành phần.

<Listing number="18-10" file-name="src/main.rs" caption="Cố gắng sử dụng một kiểu không triển khai trait của đối tượng trait">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-10/src/main.rs}}
```

</Listing>

Chúng ta sẽ nhận được lỗi này vì `String` không triển khai trait `Draw`:

```console
{{#include ../listings/ch18-oop/listing-18-10/output.txt}}
```

Lỗi này cho chúng ta biết rằng hoặc là chúng ta đang truyền một thứ gì đó cho
`Screen` mà chúng ta không có ý định truyền và do đó nên truyền một kiểu khác,
hoặc chúng ta nên triển khai `Draw` trên `String` để `Screen` có thể gọi `draw`
trên nó.

### Đối tượng Trait Thực hiện Điều phối Động

Nhớ lại trong ["Hiệu suất của Mã Sử dụng
Generic"][performance-of-code-using-generics]<!-- ignore --> trong Chương 10,
chúng ta đã thảo luận về quá trình đơn hình hóa được thực hiện trên generic bởi
trình biên dịch: trình biên dịch tạo ra các triển khai không generic của các hàm
và phương thức cho mỗi kiểu cụ thể mà chúng ta sử dụng thay thế cho tham số kiểu
generic. Mã kết quả từ quá trình đơn hình hóa đang thực hiện _điều phối tĩnh_,
đó là khi trình biên dịch biết phương thức nào bạn đang gọi tại thời điểm biên
dịch. Điều này trái ngược với _điều phối động_, là khi trình biên dịch không thể
biết tại thời điểm biên dịch phương thức nào bạn đang gọi. Trong các trường hợp
điều phối động, trình biên dịch tạo ra mã mà tại thời điểm chạy sẽ biết phương
thức nào để gọi.

Khi chúng ta sử dụng đối tượng trait, Rust phải sử dụng điều phối động. Trình
biên dịch không biết tất cả các kiểu có thể được sử dụng với mã đang sử dụng đối
tượng trait, vì vậy nó không biết phương thức nào được triển khai trên kiểu nào
để gọi. Thay vào đó, tại thời điểm chạy, Rust sử dụng các con trỏ bên trong đối
tượng trait để biết phương thức nào cần gọi. Việc tra cứu này phát sinh chi phí
thời gian chạy không xảy ra với điều phối tĩnh. Điều phối động cũng ngăn trình
biên dịch chọn để nội tuyến mã của một phương thức, từ đó ngăn chặn một số tối
ưu hóa, và Rust có một số quy tắc về nơi bạn có thể và không thể sử dụng điều
phối động, được gọi là _khả năng tương thích dyn_. Những quy tắc đó nằm ngoài
phạm vi của cuộc thảo luận này, nhưng bạn có thể đọc thêm về chúng [trong tài
liệu tham khảo][dyn-compatibility]<!-- ignore -->. Tuy nhiên, chúng ta đã có
được sự linh hoạt bổ sung trong mã mà chúng ta đã viết trong Listing 18-5 và có
thể hỗ trợ trong Listing 18-9, vì vậy đó là một sự đánh đổi cần cân nhắc.

[performance-of-code-using-generics]:
  ch10-01-syntax.html#performance-of-code-using-generics
[dynamically-sized]:
  ch20-03-advanced-types.html#dynamically-sized-types-and-the-sized-trait
[dyn-compatibility]:
  https://doc.rust-lang.org/reference/items/traits.html#dyn-compatibility
