## Hàm và Closure Nâng Cao

Phần này khám phá một số tính năng nâng cao liên quan đến hàm và closure, bao
gồm con trỏ hàm và trả về closure.

### Con Trỏ Hàm

Chúng ta đã nói về cách truyền closure cho các hàm; bạn cũng có thể truyền các
hàm thông thường cho hàm! Kỹ thuật này hữu ích khi bạn muốn truyền một hàm bạn
đã định nghĩa thay vì định nghĩa một closure mới. Hàm được ép kiểu thành kiểu
`fn` (với _f_ viết thường), không nên nhầm lẫn với trait closure `Fn`. Kiểu `fn`
được gọi là _con trỏ hàm_. Truyền hàm với con trỏ hàm sẽ cho phép bạn sử dụng
hàm làm đối số cho các hàm khác.

Cú pháp để chỉ định rằng một tham số là con trỏ hàm tương tự như của closure,
như được hiển thị trong Listing 20-28, nơi chúng ta đã định nghĩa một hàm
`add_one` để cộng 1 vào tham số của nó. Hàm `do_twice` nhận hai tham số: một con
trỏ hàm tới bất kỳ hàm nào nhận một tham số `i32` và trả về một `i32`, và một
giá trị `i32`. Hàm `do_twice` gọi hàm `f` hai lần, truyền cho nó giá trị `arg`,
sau đó cộng hai kết quả gọi hàm lại với nhau. Hàm `main` gọi `do_twice` với các
đối số `add_one` và `5`.

<Listing number="20-28" file-name="src/main.rs" caption="Sử dụng kiểu `fn` để chấp nhận con trỏ hàm làm đối số">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-28/src/main.rs}}
```

</Listing>

Mã này in ra `The answer is: 12`. Chúng ta chỉ định rằng tham số `f` trong
`do_twice` là một `fn` nhận một tham số kiểu `i32` và trả về một `i32`. Sau đó,
chúng ta có thể gọi `f` trong thân hàm của `do_twice`. Trong `main`, chúng ta có
thể truyền tên hàm `add_one` làm đối số đầu tiên cho `do_twice`.

Không giống như closure, `fn` là một kiểu chứ không phải một trait, vì vậy chúng
ta chỉ định `fn` làm kiểu tham số trực tiếp thay vì khai báo một tham số kiểu
generic với một trong các trait `Fn` làm ràng buộc trait.

Con trỏ hàm triển khai cả ba trait closure (`Fn`, `FnMut`, và `FnOnce`), có
nghĩa là bạn luôn có thể truyền con trỏ hàm làm đối số cho một hàm mà mong đợi
một closure. Tốt nhất là viết các hàm sử dụng một kiểu generic và một trong các
trait closure để hàm của bạn có thể chấp nhận cả hàm hoặc closure.

Dù vậy, một ví dụ về trường hợp bạn muốn chỉ chấp nhận `fn` và không chấp nhận
closure là khi giao tiếp với mã bên ngoài không có closure: Các hàm C có thể
chấp nhận hàm làm đối số, nhưng C không có closure.

Để xem ví dụ về nơi bạn có thể sử dụng một closure được định nghĩa inline hoặc
một hàm có tên, hãy xem xét việc sử dụng phương thức `map` được cung cấp bởi
trait `Iterator` trong thư viện chuẩn. Để sử dụng phương thức `map` để biến một
vector số thành một vector chuỗi, chúng ta có thể sử dụng một closure, như trong
Listing 20-29.

<Listing number="20-29" caption="Sử dụng closure với phương thức `map` để chuyển đổi số thành chuỗi">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-29/src/main.rs:here}}
```

</Listing>

Hoặc chúng ta có thể đặt tên một hàm làm đối số cho `map` thay vì closure.
Listing 20-30 cho thấy điều này sẽ trông như thế nào.

<Listing number="20-30" caption="Sử dụng hàm `String::to_string` với phương thức `map` để chuyển đổi số thành chuỗi">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-30/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta phải sử dụng cú pháp đầy đủ mà chúng ta đã nói trong ["Trait
Nâng Cao"][advanced-traits]<!-- ignore --> bởi vì có nhiều hàm có sẵn có tên là
`to_string`.

Ở đây, chúng ta đang sử dụng hàm `to_string` được định nghĩa trong trait
`ToString`, mà thư viện chuẩn đã triển khai cho bất kỳ kiểu nào triển khai
`Display`.

Nhớ lại từ ["Giá trị Enum"][enum-values]<!-- ignore --> trong Chương 6 rằng tên
của mỗi biến thể enum mà chúng ta định nghĩa cũng trở thành một hàm khởi tạo.
Chúng ta có thể sử dụng các hàm khởi tạo này làm con trỏ hàm triển khai các
trait closure, điều này có nghĩa là chúng ta có thể chỉ định các hàm khởi tạo
làm đối số cho các phương thức nhận closure, như được thấy trong Listing 20-31.

<Listing number="20-31" caption="Sử dụng bộ khởi tạo enum với phương thức `map` để tạo một thể hiện `Status` từ các số">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-31/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta tạo các thể hiện `Status::Value` bằng cách sử dụng mỗi giá trị
`u32` trong phạm vi mà `map` được gọi bằng cách sử dụng hàm khởi tạo của
`Status::Value`. Một số người thích phong cách này và một số người thích sử dụng
closure. Chúng được biên dịch thành cùng một mã, vì vậy hãy sử dụng phong cách
nào rõ ràng hơn với bạn.

### Trả Về Closure

Closure được biểu diễn bởi các trait, có nghĩa là bạn không thể trả về closure
trực tiếp. Trong hầu hết các trường hợp mà bạn có thể muốn trả về một trait, bạn
có thể thay vào đó sử dụng kiểu cụ thể triển khai trait đó làm giá trị trả về
của hàm. Tuy nhiên, bạn thường không thể làm điều đó với closure vì chúng không
có kiểu cụ thể có thể trả về; bạn không được phép sử dụng con trỏ hàm `fn` làm
kiểu trả về nếu closure bắt bất kỳ giá trị nào từ phạm vi của nó, chẳng hạn.

Thay vào đó, bạn thường sẽ sử dụng cú pháp `impl Trait` mà chúng ta đã học trong
Chương 10. Bạn có thể trả về bất kỳ kiểu hàm nào, sử dụng `Fn`, `FnOnce` và
`FnMut`. Ví dụ, mã trong Listing 20-32 sẽ biên dịch tốt.

<Listing number="20-32" caption="Trả về closure từ một hàm bằng cách sử dụng cú pháp `impl Trait`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-32/src/lib.rs}}
```

</Listing>

Tuy nhiên, như chúng ta đã lưu ý trong ["Suy Luận Kiểu và Chú Thích
Closure"][closure-types]<!-- ignore --> trong Chương 13, mỗi closure cũng là
kiểu riêng biệt của nó. Nếu bạn cần làm việc với nhiều hàm có cùng chữ ký nhưng
các triển khai khác nhau, bạn sẽ cần sử dụng đối tượng trait cho chúng. Hãy xem
xét điều gì xảy ra nếu bạn viết mã như trong Listing 20-33.

<Listing file-name="src/main.rs" number="20-33" caption="Tạo một `Vec<T>` của các closure được định nghĩa bởi các hàm trả về các kiểu `impl Fn`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-33/src/main.rs}}
```

</Listing>

Ở đây chúng ta có hai hàm, `returns_closure` và `returns_initialized_closure`,
cả hai đều trả về `impl Fn(i32) -> i32`. Lưu ý rằng các closure mà chúng trả về
là khác nhau, mặc dù chúng triển khai cùng một kiểu. Nếu chúng ta cố gắng biên
dịch điều này, Rust cho chúng ta biết rằng nó sẽ không hoạt động:

```text
{{#include ../listings/ch20-advanced-features/listing-20-33/output.txt}}
```

Thông báo lỗi cho chúng ta biết rằng bất cứ khi nào chúng ta trả về một
`impl Trait`, Rust tạo ra một _kiểu mờ_ duy nhất, một kiểu mà chúng ta không thể
nhìn vào chi tiết của những gì Rust xây dựng cho chúng ta, cũng như không thể
đoán kiểu mà Rust sẽ tạo ra để tự viết. Vì vậy, mặc dù cả hai hàm này đều trả về
closure triển khai cùng một trait, `Fn(i32) -> i32`, các kiểu mờ mà Rust tạo ra
cho mỗi hàm là khác biệt. (Điều này tương tự như cách Rust tạo ra các kiểu cụ
thể khác nhau cho các khối async riêng biệt ngay cả khi chúng có cùng kiểu đầu
ra, như chúng ta đã thấy trong ["Làm việc với Bất kỳ Số Lượng
Futures"][any-number-of-futures]<!-- ignore --> trong Chương 17.) Chúng ta đã
thấy một giải pháp cho vấn đề này vài lần: chúng ta có thể sử dụng một đối tượng
trait, như trong Listing 20-34.

<Listing number="20-34" caption="Tạo một `Vec<T>` của các closure được định nghĩa bởi các hàm trả về `Box<dyn Fn>` để chúng có cùng kiểu">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-34/src/main.rs:here}}
```

</Listing>

Mã này sẽ biên dịch tốt. Để biết thêm về đối tượng trait, hãy tham khảo phần
["Sử Dụng Đối Tượng Trait Cho Phép Cho Giá Trị Của Các Kiểu Khác
Nhau"][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore -->
trong Chương 18.

Tiếp theo, hãy xem xét các macro!

[advanced-traits]: ch20-02-advanced-traits.html#advanced-traits
[enum-values]: ch06-01-defining-an-enum.html#enum-values
[closure-types]: ch13-01-closures.html#closure-type-inference-and-annotation
[any-number-of-futures]: ch17-03-more-futures.html
[using-trait-objects-to-abstract-over-shared-behavior]:
  ch18-02-trait-objects.html#using-trait-objects-to-abstract-over-shared-behavior
