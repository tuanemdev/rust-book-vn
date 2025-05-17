## Các Kiểu Dữ Liệu Generic

Chúng ta sử dụng generics để tạo các định nghĩa cho các thành phần như chữ ký
hàm hoặc structs, mà sau đó chúng ta có thể sử dụng với nhiều kiểu dữ liệu cụ
thể khác nhau. Hãy xem trước cách định nghĩa các hàm, structs, enums và phương
thức sử dụng generics. Sau đó chúng ta sẽ thảo luận về cách generics ảnh hưởng
đến hiệu năng mã.

### Trong Định Nghĩa Hàm

Khi định nghĩa một hàm sử dụng generics, chúng ta đặt generics vào chữ ký của
hàm nơi mà chúng ta thường xác định kiểu dữ liệu của các tham số và giá trị trả
về. Làm như vậy giúp mã của chúng ta linh hoạt hơn và cung cấp nhiều chức năng
hơn cho người gọi hàm của chúng ta đồng thời ngăn chặn trùng lặp mã.

Tiếp tục với hàm `largest` của chúng ta, Listing 10-4 hiển thị hai hàm mà cả hai
đều tìm giá trị lớn nhất trong một slice. Sau đó chúng ta sẽ kết hợp chúng thành
một hàm duy nhất sử dụng generics.

<Listing number="10-4" file-name="src/main.rs" caption="Hai hàm chỉ khác nhau ở tên và kiểu trong chữ ký của chúng">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-04/src/main.rs:here}}
```

</Listing>

Hàm `largest_i32` là hàm chúng ta đã trích xuất trong Listing 10-3 để tìm giá
trị `i32` lớn nhất trong một slice. Hàm `largest_char` tìm giá trị `char` lớn
nhất trong một slice. Phần thân của hai hàm có cùng mã nguồn, vì vậy hãy loại bỏ
sự trùng lặp bằng cách giới thiệu một tham số kiểu generic trong một hàm duy
nhất.

Để tham số hóa các kiểu trong một hàm đơn lẻ mới, chúng ta cần đặt tên cho tham
số kiểu, giống như chúng ta làm cho các tham số giá trị cho một hàm. Bạn có thể
sử dụng bất kỳ định danh nào làm tên tham số kiểu. Nhưng chúng ta sẽ sử dụng `T`
vì, theo quy ước, tên tham số kiểu trong Rust thường ngắn, thường chỉ một chữ
cái, và quy ước đặt tên kiểu của Rust là CamelCase. Viết tắt của _type_, `T` là
lựa chọn mặc định của hầu hết lập trình viên Rust.

Khi chúng ta sử dụng một tham số trong thân hàm, chúng ta phải khai báo tên tham
số trong chữ ký để trình biên dịch biết tên đó có ý nghĩa gì. Tương tự, khi
chúng ta sử dụng tên tham số kiểu trong chữ ký hàm, chúng ta phải khai báo tên
tham số kiểu trước khi sử dụng nó. Để định nghĩa hàm generic `largest`, chúng ta
đặt khai báo tên kiểu bên trong dấu ngoặc nhọn, `<>`, giữa tên của hàm và danh
sách tham số, như sau:

```rust,ignore
fn largest<T>(list: &[T]) -> &T {
```

Chúng ta đọc định nghĩa này là: hàm `largest` là generic trên một số kiểu `T`.
Hàm này có một tham số tên là `list`, là một slice của các giá trị của kiểu `T`.
Hàm `largest` sẽ trả về một tham chiếu đến một giá trị cùng kiểu `T`.

Listing 10-5 hiển thị định nghĩa hàm `largest` kết hợp sử dụng kiểu dữ liệu
generic trong chữ ký của nó. Listing này cũng cho thấy cách chúng ta có thể gọi
hàm với một slice của các giá trị `i32` hoặc giá trị `char`. Lưu ý rằng mã này
sẽ không biên dịch được.

<Listing number="10-5" file-name="src/main.rs" caption="Hàm `largest` sử dụng tham số kiểu generic; mã này chưa biên dịch được">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/src/main.rs}}
```

</Listing>

Nếu chúng ta biên dịch mã này ngay bây giờ, chúng ta sẽ nhận được lỗi này:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-05/output.txt}}
```

Văn bản trợ giúp đề cập đến `std::cmp::PartialOrd`, đó là một _trait_, và chúng
ta sẽ nói về traits trong phần tiếp theo. Hiện tại, hãy biết rằng lỗi này cho
biết rằng phần thân của `largest` sẽ không hoạt động cho tất cả các kiểu có thể
của `T`. Bởi vì chúng ta muốn so sánh các giá trị của kiểu `T` trong phần thân,
chúng ta chỉ có thể sử dụng các kiểu mà các giá trị của nó có thể được sắp xếp
theo thứ tự. Để cho phép so sánh, thư viện tiêu chuẩn có trait
`std::cmp::PartialOrd` mà bạn có thể thực hiện trên các kiểu (xem Phụ lục C để
biết thêm về trait này). Để sửa Listing 10-5, chúng ta có thể làm theo gợi ý của
văn bản trợ giúp và giới hạn các kiểu hợp lệ cho `T` chỉ với những kiểu thực
hiện `PartialOrd`. Ví dụ này sau đó sẽ biên dịch, vì thư viện tiêu chuẩn đã thực
hiện `PartialOrd` cho cả `i32` và `char`.

### Trong Định Nghĩa Struct

Chúng ta cũng có thể định nghĩa structs để sử dụng tham số kiểu generic trong
một hoặc nhiều trường bằng cách sử dụng cú pháp `<>`. Listing 10-6 định nghĩa
một struct `Point<T>` để chứa các giá trị tọa độ `x` và `y` của bất kỳ kiểu nào.

<Listing number="10-6" file-name="src/main.rs" caption="Một struct `Point<T>` chứa các giá trị `x` và `y` của kiểu `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-06/src/main.rs}}
```

</Listing>

Cú pháp để sử dụng generics trong các định nghĩa struct tương tự với cú pháp
được sử dụng trong các định nghĩa hàm. Đầu tiên chúng ta khai báo tên của tham
số kiểu bên trong dấu ngoặc nhọn ngay sau tên của struct. Sau đó chúng ta sử
dụng kiểu generic trong định nghĩa struct ở những vị trí mà chúng ta muốn chỉ
định kiểu dữ liệu cụ thể.

Lưu ý rằng vì chúng ta chỉ sử dụng một kiểu generic để định nghĩa `Point<T>`,
nên định nghĩa này nói rằng struct `Point<T>` là generic trên một số kiểu `T`,
và các trường `x` và `y` _đều_ có cùng kiểu đó, bất kể kiểu đó là gì. Nếu chúng
ta tạo một thể hiện của `Point<T>` có giá trị của các kiểu khác nhau, như trong
Listing 10-7, mã của chúng ta sẽ không biên dịch.

<Listing number="10-7" file-name="src/main.rs" caption="Các trường `x` và `y` phải có cùng kiểu vì cả hai đều có cùng kiểu dữ liệu generic `T`.">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/src/main.rs}}
```

</Listing>

Trong ví dụ này, khi chúng ta gán giá trị số nguyên `5` cho `x`, chúng ta cho
trình biên dịch biết rằng kiểu generic `T` sẽ là một số nguyên cho thể hiện này
của `Point<T>`. Sau đó khi chúng ta chỉ định `4.0` cho `y`, mà chúng ta đã định
nghĩa là có cùng kiểu với `x`, chúng ta sẽ nhận được lỗi không khớp kiểu như
sau:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-07/output.txt}}
```

Để định nghĩa một struct `Point` trong đó `x` và `y` đều là generics nhưng có
thể có các kiểu khác nhau, chúng ta có thể sử dụng nhiều tham số kiểu generic.
Ví dụ, trong Listing 10-8, chúng ta thay đổi định nghĩa của `Point` để generic
trên các kiểu `T` và `U` trong đó `x` có kiểu `T` và `y` có kiểu `U`.

<Listing number="10-8" file-name="src/main.rs" caption="Một `Point<T, U>` generic trên hai kiểu để `x` và `y` có thể là các giá trị của các kiểu khác nhau">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-08/src/main.rs}}
```

</Listing>

Bây giờ tất cả các thể hiện của `Point` được hiển thị đều được cho phép! Bạn có
thể sử dụng nhiều tham số kiểu generic trong một định nghĩa tùy thích, nhưng
việc sử dụng quá nhiều sẽ làm cho mã của bạn khó đọc. Nếu bạn thấy mình cần
nhiều kiểu generic trong mã của mình, điều đó có thể chỉ ra rằng mã của bạn cần
được cấu trúc lại thành các phần nhỏ hơn.

### Trong Định Nghĩa Enum

Như chúng ta đã làm với structs, chúng ta có thể định nghĩa enums để giữ các
kiểu dữ liệu generic trong các biến thể của chúng. Hãy xem lại enum `Option<T>`
mà thư viện tiêu chuẩn cung cấp, mà chúng ta đã sử dụng trong Chương 6:

```rust
enum Option<T> {
    Some(T),
    None,
}
```

Định nghĩa này bây giờ nên có ý nghĩa hơn với bạn. Như bạn có thể thấy, enum
`Option<T>` là generic trên kiểu `T` và có hai biến thể: `Some`, mà giữ một giá
trị của kiểu `T`, và một biến thể `None` không giữ bất kỳ giá trị nào. Bằng cách
sử dụng enum `Option<T>`, chúng ta có thể biểu đạt khái niệm trừu tượng của một
giá trị tùy chọn, và vì `Option<T>` là generic, chúng ta có thể sử dụng sự trừu
tượng này bất kể kiểu của giá trị tùy chọn là gì.

Enums cũng có thể sử dụng nhiều kiểu generic. Định nghĩa của enum `Result` mà
chúng ta đã sử dụng trong Chương 9 là một ví dụ:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

Enum `Result` là generic trên hai kiểu, `T` và `E`, và có hai biến thể: `Ok`,
giữ một giá trị của kiểu `T`, và `Err`, giữ một giá trị của kiểu `E`. Định nghĩa
này giúp thuận tiện để sử dụng enum `Result` ở bất kỳ nơi nào chúng ta có một
hoạt động có thể thành công (trả về một giá trị của một số kiểu `T`) hoặc thất
bại (trả về một lỗi của một số kiểu `E`). Trên thực tế, đây là những gì chúng ta
đã sử dụng để mở một tệp trong Listing 9-3, trong đó `T` được điền với kiểu
`std::fs::File` khi tệp được mở thành công và `E` được điền với kiểu
`std::io::Error` khi có vấn đề khi mở tệp.

Khi bạn nhận ra các tình huống trong mã của mình với nhiều định nghĩa struct
hoặc enum khác nhau chỉ ở kiểu của các giá trị mà chúng chứa, bạn có thể tránh
trùng lặp bằng cách sử dụng các kiểu generic thay thế.

### Trong Định Nghĩa Phương Thức

Chúng ta có thể triển khai các phương thức trên structs và enums (như chúng ta
đã làm trong Chương 5) và sử dụng các kiểu generic trong các định nghĩa của
chúng. Listing 10-9 hiển thị struct `Point<T>` mà chúng ta đã định nghĩa trong
Listing 10-6 với một phương thức có tên `x` được triển khai trên nó.

<Listing number="10-9" file-name="src/main.rs" caption="Triển khai một phương thức có tên `x` trên struct `Point<T>` sẽ trả về một tham chiếu đến trường `x` của kiểu `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-09/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đã định nghĩa một phương thức có tên `x` trên `Point<T>` trả về
một tham chiếu đến dữ liệu trong trường `x`.

Lưu ý rằng chúng ta phải khai báo `T` ngay sau `impl` để chúng ta có thể sử dụng
`T` để chỉ định rằng chúng ta đang triển khai các phương thức trên kiểu
`Point<T>`. Bằng cách khai báo `T` như là một kiểu generic sau `impl`, Rust có
thể xác định rằng kiểu trong dấu ngoặc nhọn trong `Point` là một kiểu generic
chứ không phải là một kiểu cụ thể. Chúng ta có thể đã chọn một tên khác cho tham
số generic này so với tham số generic đã khai báo trong định nghĩa struct, nhưng
việc sử dụng cùng một tên là quy ước. Nếu bạn viết một phương thức trong một
`impl` khai báo một kiểu generic, phương thức đó sẽ được định nghĩa trên bất kỳ
thể hiện nào của kiểu, bất kể kiểu cụ thể nào cuối cùng thay thế cho kiểu
generic.

Chúng ta cũng có thể chỉ định các ràng buộc trên các kiểu generic khi định nghĩa
các phương thức trên kiểu. Chúng ta có thể, ví dụ, triển khai các phương thức
chỉ trên các thể hiện `Point<f32>` thay vì trên các thể hiện `Point<T>` với bất
kỳ kiểu generic nào. Trong Listing 10-10, chúng ta sử dụng kiểu cụ thể `f32`,
nghĩa là chúng ta không khai báo bất kỳ kiểu nào sau `impl`.

<Listing number="10-10" file-name="src/main.rs" caption="Một khối `impl` chỉ áp dụng cho một struct với một kiểu cụ thể cho tham số kiểu generic `T`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-10/src/main.rs:here}}
```

</Listing>

Mã này có nghĩa là kiểu `Point<f32>` sẽ có phương thức `distance_from_origin`;
các thể hiện khác của `Point<T>` mà `T` không phải là kiểu `f32` sẽ không có
phương thức này được định nghĩa. Phương thức đo lường khoảng cách từ điểm của
chúng ta đến điểm có tọa độ (0.0, 0.0) và sử dụng các phép toán toán học là chỉ
có sẵn cho các kiểu số thực.

Các tham số kiểu generic trong định nghĩa struct không phải lúc nào cũng giống
với các tham số bạn sử dụng trong chữ ký phương thức của cùng một struct đó.
Listing 10-11 sử dụng các kiểu generic `X1` và `Y1` cho struct `Point` và `X2`
`Y2` cho chữ ký phương thức `mixup` để làm cho ví dụ rõ ràng hơn. Phương thức
tạo một thể hiện `Point` mới với giá trị `x` từ `Point` `self` (kiểu `X1`) và
giá trị `y` từ `Point` được truyền vào (kiểu `Y2`).

<Listing number="10-11" file-name="src/main.rs" caption="Một phương thức sử dụng các kiểu generic khác với định nghĩa struct của nó">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-11/src/main.rs}}
```

</Listing>

Trong `main`, chúng ta đã định nghĩa một `Point` có một `i32` cho `x` (với giá
trị `5`) và một `f64` cho `y` (với giá trị `10.4`). Biến `p2` là một struct
`Point` có một string slice cho `x` (với giá trị `"Hello"`) và một `char` cho
`y` (với giá trị `c`). Gọi `mixup` trên `p1` với đối số `p2` cho chúng ta `p3`,
sẽ có một `i32` cho `x` vì `x` đến từ `p1`. Biến `p3` sẽ có một `char` cho `y`
vì `y` đến từ `p2`. Lời gọi macro `println!` sẽ in ra `p3.x = 5, p3.y = c`.

Mục đích của ví dụ này là để chứng minh một tình huống trong đó một số tham số
generic được khai báo với `impl` và một số được khai báo với định nghĩa phương
thức. Ở đây, các tham số generic `X1` và `Y1` được khai báo sau `impl` vì chúng
đi với định nghĩa struct. Các tham số generic `X2` và `Y2` được khai báo sau
`fn mixup` vì chúng chỉ liên quan đến phương thức.

### Hiệu Năng của Mã Sử Dụng Generics

Bạn có thể tự hỏi liệu có chi phí thời gian chạy khi sử dụng các tham số kiểu
generic không. Tin tốt là việc sử dụng các kiểu generic sẽ không làm cho chương
trình của bạn chạy chậm hơn so với sử dụng các kiểu cụ thể.

Rust thực hiện điều này bằng cách thực hiện monomorphization của mã sử dụng
generics tại thời điểm biên dịch. _Monomorphization_ là quá trình chuyển đổi mã
generic thành mã cụ thể bằng cách điền các kiểu cụ thể được sử dụng khi biên
dịch. Trong quá trình này, trình biên dịch làm ngược lại các bước chúng ta đã sử
dụng để tạo hàm generic trong Listing 10-5: trình biên dịch xem xét tất cả các
nơi mã generic được gọi và tạo mã cho các kiểu cụ thể mà mã generic được gọi
với.

Hãy xem cách hoạt động của nó bằng cách sử dụng enum generic `Option<T>` của thư
viện tiêu chuẩn:

```rust
let integer = Some(5);
let float = Some(5.0);
```

Khi Rust biên dịch mã này, nó thực hiện monomorphization. Trong quá trình đó,
trình biên dịch đọc các giá trị đã được sử dụng trong các thể hiện `Option<T>`
và xác định hai loại `Option<T>`: một là `i32` và loại kia là `f64`. Như vậy, nó
mở rộng định nghĩa generic của `Option<T>` thành hai định nghĩa chuyên biệt cho
`i32` và `f64`, từ đó thay thế định nghĩa generic bằng các định nghĩa cụ thể.

Phiên bản monomorphized của mã trông tương tự như sau (trình biên dịch sử dụng
các tên khác với những gì chúng ta đang sử dụng ở đây để minh họa):

<Listing file-name="src/main.rs">

```rust
enum Option_i32 {
    Some(i32),
    None,
}

enum Option_f64 {
    Some(f64),
    None,
}

fn main() {
    let integer = Option_i32::Some(5);
    let float = Option_f64::Some(5.0);
}
```

</Listing>

`Option<T>` generic được thay thế bằng các định nghĩa cụ thể được tạo bởi trình
biên dịch. Bởi vì Rust biên dịch mã generic thành mã chỉ định kiểu trong mỗi thể
hiện, chúng ta không phải trả chi phí thời gian chạy cho việc sử dụng generics.
Khi mã chạy, nó hoạt động giống như nếu chúng ta đã sao chép từng định nghĩa
bằng tay. Quá trình monomorphization làm cho generics của Rust cực kỳ hiệu quả
trong thời gian chạy.
