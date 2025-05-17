## Chương trình Ví dụ Sử dụng Structs

Để hiểu khi nào chúng ta nên sử dụng structs, hãy viết một chương trình tính
diện tích của hình chữ nhật. Chúng ta sẽ bắt đầu bằng việc sử dụng các biến đơn
lẻ, và sau đó cải tiến chương trình cho đến khi chúng ta sử dụng structs.

Hãy tạo một dự án binary mới với Cargo có tên _rectangles_ để tính diện tích
hình chữ nhật dựa trên chiều rộng và chiều cao được chỉ định bằng pixel. Listing
5-8 hiển thị một chương trình ngắn với một cách thực hiện chính xác điều đó
trong tệp _src/main.rs_ của dự án chúng ta.

<Listing number="5-8" file-name="src/main.rs" caption="Tính diện tích của hình chữ nhật được xác định bằng các biến chiều rộng và chiều cao riêng biệt">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:all}}
```

</Listing>

Bây giờ, chạy chương trình này bằng `cargo run`:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/output.txt}}
```

Đoạn mã này thành công trong việc tính diện tích của hình chữ nhật bằng cách gọi
hàm `area` với mỗi kích thước, nhưng chúng ta có thể làm nhiều hơn nữa để làm
cho mã này rõ ràng và dễ đọc hơn.

Vấn đề với đoạn mã này thể hiện rõ trong chữ ký của hàm `area`:

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-08/src/main.rs:here}}
```

Hàm `area` được cho là để tính diện tích của một hình chữ nhật, nhưng hàm mà
chúng ta đã viết có hai tham số, và không có chỗ nào trong chương trình của
chúng ta làm rõ rằng các tham số này có liên quan đến nhau. Sẽ dễ đọc và dễ quản
lý hơn nếu nhóm chiều rộng và chiều cao lại với nhau. Chúng ta đã thảo luận về
một cách chúng ta có thể làm điều đó trong phần ["Kiểu
Tuple"][the-tuple-type]<!-- ignore --> của Chương 3: bằng cách sử dụng các
tuple.

### Cải tiến với Tuples

Listing 5-9 hiển thị một phiên bản khác của chương trình chúng ta sử dụng
tuples.

<Listing number="5-9" file-name="src/main.rs" caption="Chỉ định chiều rộng và chiều cao của hình chữ nhật bằng một tuple">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-09/src/main.rs}}
```

</Listing>

Ở một khía cạnh, chương trình này tốt hơn. Tuples cho phép chúng ta thêm một
chút cấu trúc và giờ đây chúng ta chỉ truyền một đối số. Nhưng ở một khía cạnh
khác, phiên bản này ít rõ ràng hơn: tuples không đặt tên cho các phần tử của
chúng, vì vậy chúng ta phải lập chỉ mục vào các phần của tuple, làm cho phép
tính của chúng ta kém rõ ràng hơn.

Việc nhầm lẫn chiều rộng và chiều cao sẽ không ảnh hưởng đến phép tính diện
tích, nhưng nếu chúng ta muốn vẽ hình chữ nhật trên màn hình, điều đó sẽ quan
trọng! Chúng ta sẽ phải nhớ rằng `width` là chỉ mục tuple `0` và `height` là chỉ
mục tuple `1`. Điều này sẽ càng khó hơn cho người khác để hiểu và ghi nhớ nếu họ
sử dụng mã của chúng ta. Bởi vì chúng ta chưa truyền đạt ý nghĩa của dữ liệu
trong mã của mình, việc dễ gây ra lỗi hơn.

### Cải tiến với Structs: Thêm Ý nghĩa

Chúng ta sử dụng structs để thêm ý nghĩa bằng cách gắn nhãn cho dữ liệu. Chúng
ta có thể chuyển đổi tuple chúng ta đang sử dụng thành một struct với một tên
cho toàn bộ cũng như các tên cho các phần, như được hiển thị trong Listing 5-10.

<Listing number="5-10" file-name="src/main.rs" caption="Định nghĩa struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-10/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đã định nghĩa một struct và đặt tên nó là `Rectangle`. Bên trong
dấu ngoặc nhọn, chúng ta định nghĩa các trường là `width` và `height`, cả hai
đều có kiểu `u32`. Sau đó, trong `main`, chúng ta tạo một instance cụ thể của
`Rectangle` có chiều rộng `30` và chiều cao `50`.

Hàm `area` của chúng ta bây giờ được định nghĩa với một tham số, mà chúng ta đã
đặt tên là `rectangle`, có kiểu là một tham chiếu không thể thay đổi đến một
instance struct `Rectangle`. Như đã đề cập trong Chương 4, chúng ta muốn mượn
struct thay vì lấy quyền sở hữu của nó. Bằng cách này, `main` giữ quyền sở hữu
và có thể tiếp tục sử dụng `rect1`, đó là lý do tại sao chúng ta sử dụng `&`
trong chữ ký hàm và nơi chúng ta gọi hàm.

Hàm `area` truy cập vào các trường `width` và `height` của instance `Rectangle`
(lưu ý rằng việc truy cập các trường của một instance struct đã được mượn không
di chuyển các giá trị trường, đó là lý do tại sao bạn thường thấy việc mượn các
struct). Chữ ký hàm cho `area` bây giờ nói chính xác những gì chúng ta muốn:
tính diện tích của `Rectangle`, sử dụng các trường `width` và `height`. Điều này
truyền đạt rằng chiều rộng và chiều cao có liên quan đến nhau, và nó đưa ra các
tên mô tả cho các giá trị thay vì sử dụng các giá trị chỉ mục tuple `0` và `1`.
Đây là một thắng lợi cho sự rõ ràng.

### Thêm Chức năng Hữu ích với Derived Traits

Sẽ rất hữu ích nếu có thể in ra một instance của `Rectangle` trong khi chúng ta
đang gỡ lỗi chương trình và xem các giá trị cho tất cả các trường của nó.
Listing 5-11 thử sử dụng [macro `println!`][println]<!-- ignore --> như chúng ta
đã sử dụng trong các chương trước. Tuy nhiên, điều này sẽ không hoạt động.

<Listing number="5-11" file-name="src/main.rs" caption="Thử in ra một instance `Rectangle`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/src/main.rs}}
```

</Listing>

Khi chúng ta biên dịch đoạn mã này, chúng ta nhận được một lỗi với thông báo cốt
lõi này:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:3}}
```

Macro `println!` có thể thực hiện nhiều loại định dạng, và theo mặc định, dấu
ngoặc nhọn báo cho `println!` sử dụng định dạng được gọi là `Display`: đầu ra
dành cho người dùng cuối trực tiếp. Các kiểu nguyên thủy mà chúng ta đã thấy cho
đến nay triển khai `Display` theo mặc định vì chỉ có một cách bạn muốn hiển thị
`1` hoặc bất kỳ kiểu nguyên thủy nào khác cho người dùng. Nhưng với các struct,
cách `println!` nên định dạng đầu ra ít rõ ràng hơn vì có nhiều khả năng hiển
thị hơn: Bạn có muốn dấu phẩy hay không? Bạn có muốn in dấu ngoặc nhọn? Tất cả
các trường có nên được hiển thị không? Do tính mơ hồ này, Rust không cố đoán
những gì chúng ta muốn, và các struct không có sẵn thực thi của `Display` để sử
dụng với `println!` và placeholder `{}`.

Nếu chúng ta tiếp tục đọc các lỗi, chúng ta sẽ tìm thấy ghi chú hữu ích này:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-11/output.txt:9:10}}
```

Hãy thử nó! Lệnh gọi macro `println!` bây giờ sẽ trông như
`println!("rect1 is {rect1:?}");`. Đặt đặc tả `:?` bên trong dấu ngoặc nhọn báo
cho `println!` rằng chúng ta muốn sử dụng một định dạng đầu ra được gọi là
`Debug`. Trait `Debug` cho phép chúng ta in struct theo cách hữu ích cho các nhà
phát triển để chúng ta có thể thấy giá trị của nó trong khi chúng ta đang gỡ lỗi
mã của mình.

Biên dịch mã với thay đổi này. Chết tiệt! Chúng ta vẫn nhận được một lỗi:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:3}}
```

Nhưng một lần nữa, trình biên dịch cung cấp cho chúng ta một ghi chú hữu ích:

```text
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-01-debug/output.txt:9:10}}
```

Rust _có_ bao gồm chức năng để in ra thông tin gỡ lỗi, nhưng chúng ta phải chọn
tham gia một cách rõ ràng để làm cho chức năng đó có sẵn cho struct của chúng
ta. Để làm điều đó, chúng ta thêm thuộc tính ngoài `#[derive(Debug)]` ngay trước
định nghĩa struct, như được hiển thị trong Listing 5-12.

<Listing number="5-12" file-name="src/main.rs" caption="Thêm thuộc tính để triển khai trait `Debug` và in instance `Rectangle` bằng định dạng gỡ lỗi">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/src/main.rs}}
```

</Listing>

Bây giờ khi chúng ta chạy chương trình, chúng ta sẽ không gặp bất kỳ lỗi nào, và
chúng ta sẽ thấy đầu ra sau:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/listing-05-12/output.txt}}
```

Tuyệt! Đây không phải là đầu ra đẹp nhất, nhưng nó hiển thị các giá trị của tất
cả các trường cho instance này, điều này chắc chắn sẽ giúp ích trong quá trình
gỡ lỗi. Khi chúng ta có các struct lớn hơn, sẽ hữu ích khi có đầu ra dễ đọc hơn
một chút; trong những trường hợp đó, chúng ta có thể sử dụng `{:#?}` thay vì
`{:?}` trong chuỗi `println!`. Trong ví dụ này, sử dụng kiểu `{:#?}` sẽ xuất ra
như sau:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/output-only-02-pretty-debug/output.txt}}
```

Một cách khác để in ra một giá trị sử dụng định dạng `Debug` là sử dụng [macro
`dbg!`][dbg]<!-- ignore -->, lấy quyền sở hữu của một biểu thức (trái ngược với
`println!`, lấy một tham chiếu), in ra tệp và số dòng nơi lệnh gọi macro `dbg!`
đó xuất hiện trong mã của bạn cùng với giá trị kết quả của biểu thức đó, và trả
lại quyền sở hữu của giá trị.

> Lưu ý: Gọi macro `dbg!` in ra luồng bảng điều khiển lỗi tiêu chuẩn (`stderr`),
> trái ngược với `println!`, in ra luồng bảng điều khiển đầu ra tiêu chuẩn
> (`stdout`). Chúng ta sẽ nói thêm về `stderr` và `stdout` trong phần ["Viết
> Thông báo Lỗi cho Standard Error Thay vì Standard Output" trong Chương
> 12][err]<!-- ignore -->.

Đây là một ví dụ trong đó chúng ta quan tâm đến giá trị được gán cho trường
`width`, cũng như giá trị của toàn bộ struct trong `rect1`:

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/src/main.rs}}
```

Chúng ta có thể đặt `dbg!` xung quanh biểu thức `30 * scale` và, bởi vì `dbg!`
trả lại quyền sở hữu của giá trị biểu thức, trường `width` sẽ nhận được cùng một
giá trị như thể chúng ta không có lệnh gọi `dbg!` ở đó. Chúng ta không muốn
`dbg!` lấy quyền sở hữu của `rect1`, vì vậy chúng ta sử dụng một tham chiếu đến
`rect1` trong lệnh gọi tiếp theo. Đây là đầu ra của ví dụ này:

```console
{{#include ../listings/ch05-using-structs-to-structure-related-data/no-listing-05-dbg-macro/output.txt}}
```

Chúng ta có thể thấy phần đầu tiên của đầu ra đến từ dòng 10 của _src/main.rs_
nơi chúng ta đang gỡ lỗi biểu thức `30 * scale`, và giá trị kết quả của nó là
`60` (định dạng `Debug` được thực hiện cho các số nguyên là chỉ in giá trị của
chúng). Lệnh gọi `dbg!` trên dòng 14 của _src/main.rs_ xuất ra giá trị của
`&rect1`, đó là struct `Rectangle`. Đầu ra này sử dụng định dạng `Debug` đẹp của
kiểu `Rectangle`. Macro `dbg!` có thể thực sự hữu ích khi bạn đang cố gắng tìm
hiểu xem mã của bạn đang làm gì!

Ngoài trait `Debug`, Rust đã cung cấp một số trait cho chúng ta sử dụng với
thuộc tính `derive` có thể thêm hành vi hữu ích cho các kiểu tùy chỉnh của chúng
ta. Những trait đó và hành vi của chúng được liệt kê trong [Phụ lục
C][app-c]<!-- ignore -->. Chúng ta sẽ đề cập đến cách triển khai các trait này
với hành vi tùy chỉnh cũng như cách tạo trait của riêng mình trong Chương 10.
Ngoài ra còn có nhiều thuộc tính khác ngoài `derive`; để biết thêm thông tin,
xem [phần "Thuộc tính" của Tham chiếu Rust][attributes].

Hàm `area` của chúng ta rất cụ thể: nó chỉ tính diện tích của hình chữ nhật. Sẽ
hữu ích hơn nếu gắn hành vi này chặt chẽ hơn với struct `Rectangle` của chúng ta
vì nó sẽ không hoạt động với bất kỳ kiểu nào khác. Hãy xem xét làm thế nào chúng
ta có thể tiếp tục cải tiến mã này bằng cách chuyển hàm `area` thành _phương
thức_ `area` được định nghĩa trên kiểu `Rectangle` của chúng ta.

[the-tuple-type]: ch03-02-data-types.html#the-tuple-type
[app-c]: appendix-03-derivable-traits.md
[println]: ../std/macro.println.html
[dbg]: ../std/macro.dbg.html
[err]: ch12-06-writing-to-stderr-instead-of-stdout.html
[attributes]: ../reference/attributes.html
