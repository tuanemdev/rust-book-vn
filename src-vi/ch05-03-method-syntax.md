## Cú pháp Method

_Method_ tương tự như các hàm: chúng ta khai báo chúng với từ khóa `fn` và một
tên, chúng có thể có các tham số và giá trị trả về, và chúng chứa một số mã được
chạy khi method được gọi từ nơi khác. Không giống như các hàm, method được định
nghĩa trong ngữ cảnh của một struct (hoặc một enum hoặc một trait object, mà
chúng ta sẽ đề cập trong [Chương 6][enums]<!-- ignore --> và [Chương
18][trait-objects]<!-- ignore -->, tương ứng), và tham số đầu tiên của chúng
luôn là `self`, đại diện cho instance của struct mà method đang được gọi.

### Định nghĩa Method

Hãy thay đổi hàm `area` có một instance `Rectangle` làm tham số và thay vào đó,
tạo một method `area` được định nghĩa trên struct `Rectangle`, như thể hiện
trong Listing 5-13.

<Listing number="5-13" file-name="src/main.rs" caption="Định nghĩa một method `area` trên struct `Rectangle`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-13/src/main.rs}}
```

</Listing>

Để định nghĩa hàm trong ngữ cảnh của `Rectangle`, chúng ta bắt đầu một khối
`impl` (implementation) cho `Rectangle`. Mọi thứ trong khối `impl` này sẽ được
liên kết với kiểu `Rectangle`. Sau đó, chúng ta di chuyển hàm `area` vào trong
dấu ngoặc nhọn `impl` và thay đổi tham số đầu tiên (và trong trường hợp này, duy
nhất) thành `self` trong chữ ký và mọi nơi trong nội dung hàm. Trong `main`, nơi
chúng ta đã gọi hàm `area` và truyền `rect1` như một đối số, chúng ta có thể sử
dụng _cú pháp method_ để gọi method `area` trên instance `Rectangle` của chúng
ta. Cú pháp method được đặt sau một instance: chúng ta thêm một dấu chấm theo
sau bởi tên method, dấu ngoặc đơn, và bất kỳ đối số nào.

Trong chữ ký cho `area`, chúng ta sử dụng `&self` thay vì
`rectangle: &Rectangle`. `&self` thực tế là viết tắt của `self: &Self`. Trong
một khối `impl`, kiểu `Self` là bí danh cho kiểu mà khối `impl` đang áp dụng.
Method phải có một tham số tên là `self` kiểu `Self` làm tham số đầu tiên, vì
vậy Rust cho phép bạn viết tắt điều này với chỉ tên `self` ở vị trí tham số đầu
tiên. Lưu ý rằng chúng ta vẫn cần sử dụng `&` trước viết tắt `self` để chỉ ra
rằng method này mượn instance `Self`, giống như chúng ta đã làm trong
`rectangle: &Rectangle`. Method có thể lấy quyền sở hữu của `self`, mượn `self`
không thay đổi như chúng ta đã làm ở đây, hoặc mượn `self` có thể thay đổi,
giống như chúng có thể với bất kỳ tham số nào khác.

Chúng ta chọn `&self` ở đây vì cùng lý do chúng ta đã sử dụng `&Rectangle` trong
phiên bản hàm: chúng ta không muốn lấy quyền sở hữu, và chúng ta chỉ muốn đọc dữ
liệu trong struct, không viết vào nó. Nếu chúng ta muốn thay đổi instance mà
chúng ta đã gọi method như một phần của những gì method làm, chúng ta sẽ sử dụng
`&mut self` làm tham số đầu tiên. Có một method lấy quyền sở hữu của instance
bằng cách chỉ sử dụng `self` làm tham số đầu tiên là hiếm; kỹ thuật này thường
được sử dụng khi method chuyển đổi `self` thành một cái gì đó khác và bạn muốn
ngăn người gọi sử dụng instance ban đầu sau khi chuyển đổi.

Lý do chính để sử dụng method thay vì hàm, ngoài việc cung cấp cú pháp method và
không phải lặp lại kiểu của `self` trong mọi chữ ký method, là để tổ chức. Chúng
ta đã đặt tất cả những thứ có thể làm với một instance của một kiểu trong một
khối `impl` thay vì làm cho người dùng tương lai của mã chúng ta phải tìm kiếm
khả năng của `Rectangle` ở nhiều nơi khác nhau trong thư viện mà chúng ta cung
cấp.

Lưu ý rằng chúng ta có thể chọn đặt tên cho một method giống với tên của một
trong các trường của struct. Ví dụ, chúng ta có thể định nghĩa một method trên
`Rectangle` cũng được đặt tên là `width`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-06-method-field-interaction/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta đang chọn cách làm cho method `width` trả về `true` nếu giá trị
trong trường `width` của instance lớn hơn `0` và `false` nếu giá trị là `0`:
chúng ta có thể sử dụng một trường trong một method cùng tên cho bất kỳ mục đích
nào. Trong `main`, khi chúng ta theo sau `rect1.width` với dấu ngoặc đơn, Rust
biết chúng ta đang đề cập đến method `width`. Khi chúng ta không sử dụng dấu
ngoặc đơn, Rust biết chúng ta đang đề cập đến trường `width`.

Thường xuyên, nhưng không phải lúc nào cũng vậy, khi chúng ta đặt cho một method
cùng tên với một trường, chúng ta muốn nó chỉ trả về giá trị trong trường đó và
không làm gì khác. Các method như thế này được gọi là _getter_, và Rust không tự
động triển khai chúng cho các trường của struct như một số ngôn ngữ khác. Getter
rất hữu ích vì bạn có thể làm cho trường này riêng tư nhưng method là công khai,
và do đó cho phép truy cập chỉ đọc vào trường đó như một phần của API công khai
của kiểu. Chúng ta sẽ thảo luận về các khái niệm công khai và riêng tư là gì và
cách chỉ định một trường hoặc method là công khai hay riêng tư trong [Chương
7][public]<!-- ignore -->.

> ### Toán tử `->` Ở Đâu?
>
> Trong C và C++, hai toán tử khác nhau được sử dụng để gọi method: bạn sử dụng
> `.` nếu bạn đang gọi một method trực tiếp trên đối tượng và `->` nếu bạn đang
> gọi method trên một con trỏ đến đối tượng và cần giải tham chiếu con trỏ
> trước. Nói cách khác, nếu `object` là một con trỏ, `object->something()` tương
> tự như `(*object).something()`.
>
> Rust không có một toán tử tương đương với `->`; thay vào đó, Rust có một tính
> năng gọi là _tham chiếu và giải tham chiếu tự động_. Gọi method là một trong
> số ít nơi trong Rust có hành vi này.
>
> Đây là cách nó hoạt động: khi bạn gọi một method bằng `object.something()`,
> Rust tự động thêm `&`, `&mut`, hoặc `*` để `object` khớp với chữ ký của
> method. Nói cách khác, những điều sau đây là giống nhau:
>
> <!-- CAN'T EXTRACT SEE BUG https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust
> # #[derive(Debug,Copy,Clone)]
> # struct Point {
> #     x: f64,
> #     y: f64,
> # }
> #
> # impl Point {
> #    fn distance(&self, other: &Point) -> f64 {
> #        let x_squared = f64::powi(other.x - self.x, 2);
> #        let y_squared = f64::powi(other.y - self.y, 2);
> #
> #        f64::sqrt(x_squared + y_squared)
> #    }
> # }
> # let p1 = Point { x: 0.0, y: 0.0 };
> # let p2 = Point { x: 5.0, y: 6.5 };
> p1.distance(&p2);
> (&p1).distance(&p2);
> ```
>
> Cái đầu tiên trông gọn gàng hơn nhiều. Hành vi tham chiếu tự động này hoạt
> động bởi vì method có một người nhận rõ ràng—kiểu của `self`. Với người nhận
> và tên của một method, Rust có thể xác định rõ ràng liệu method đang đọc
> (`&self`), biến đổi (`&mut self`), hay tiêu thụ (`self`). Thực tế là Rust làm
> cho việc mượn ngầm định cho người nhận method là một phần lớn của việc làm cho
> quyền sở hữu trở nên tiện dụng trong thực tế.

### Method với Nhiều Tham số

Hãy thực hành sử dụng method bằng cách triển khai một method thứ hai trên struct
`Rectangle`. Lần này chúng ta muốn một instance của `Rectangle` lấy một instance
khác của `Rectangle` và trả về `true` nếu `Rectangle` thứ hai có thể nằm hoàn
toàn bên trong `self` (tức là `Rectangle` đầu tiên); nếu không, nó sẽ trả về
`false`. Nghĩa là, một khi chúng ta đã định nghĩa method `can_hold`, chúng ta
muốn có thể viết chương trình được hiển thị trong Listing 5-14.

<Listing number="5-14" file-name="src/main.rs" caption="Sử dụng method `can_hold` chưa được viết">

```rust,ignore
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-14/src/main.rs}}
```

</Listing>

Đầu ra dự kiến sẽ trông giống như sau bởi vì cả hai kích thước của `rect2` đều
nhỏ hơn kích thước của `rect1`, nhưng `rect3` rộng hơn `rect1`:

```text
Can rect1 hold rect2? true
Can rect1 hold rect3? false
```

Chúng ta biết mình muốn định nghĩa một method, vì vậy nó sẽ nằm trong khối
`impl Rectangle`. Tên method sẽ là `can_hold`, và nó sẽ lấy một bản mượn không
thay đổi của một `Rectangle` khác làm tham số. Chúng ta có thể biết kiểu của
tham số sẽ là gì bằng cách nhìn vào mã gọi method: `rect1.can_hold(&rect2)`
truyền vào `&rect2`, đó là một bản mượn không thay đổi cho `rect2`, một instance
của `Rectangle`. Điều này có ý nghĩa vì chúng ta chỉ cần đọc `rect2` (thay vì
viết, có nghĩa là chúng ta cần một bản mượn có thể thay đổi), và chúng ta muốn
`main` giữ quyền sở hữu của `rect2` để chúng ta có thể sử dụng nó lại sau khi
gọi method `can_hold`. Giá trị trả về của `can_hold` sẽ là một giá trị Boolean,
và việc triển khai sẽ kiểm tra xem chiều rộng và chiều cao của `self` có lớn hơn
chiều rộng và chiều cao của `Rectangle` khác hay không, tương ứng. Hãy thêm
method `can_hold` mới vào khối `impl` từ Listing 5-13, như được hiển thị trong
Listing 5-15.

<Listing number="5-15" file-name="src/main.rs" caption="Triển khai method `can_hold` trên `Rectangle` lấy một instance `Rectangle` khác làm tham số">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-15/src/main.rs:here}}
```

</Listing>

Khi chúng ta chạy mã này với hàm `main` trong Listing 5-14, chúng ta sẽ nhận
được đầu ra mong muốn. Method có thể lấy nhiều tham số mà chúng ta thêm vào chữ
ký sau tham số `self`, và những tham số đó hoạt động giống như tham số trong
hàm.

### Hàm Liên kết

Tất cả các hàm được định nghĩa trong một khối `impl` được gọi là _hàm liên kết_
bởi vì chúng được liên kết với kiểu được đặt tên sau `impl`. Chúng ta có thể
định nghĩa các hàm liên kết mà không có `self` là tham số đầu tiên của chúng (và
do đó không phải là method) bởi vì chúng không cần một instance của kiểu để làm
việc với. Chúng ta đã sử dụng một hàm như thế này: hàm `String::from` mà được
định nghĩa trên kiểu `String`.

Các hàm liên kết không phải là method thường được sử dụng cho các constructor sẽ
trả về một instance mới của struct. Những hàm này thường được gọi là `new`,
nhưng `new` không phải là một tên đặc biệt và không được tích hợp sẵn trong ngôn
ngữ. Ví dụ, chúng ta có thể chọn cung cấp một hàm liên kết có tên `square` sẽ có
một tham số kích thước và sử dụng nó làm cả chiều rộng và chiều cao, từ đó giúp
tạo một `Rectangle` vuông dễ dàng hơn thay vì phải chỉ định cùng một giá trị hai
lần:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-03-associated-functions/src/main.rs:here}}
```

Từ khóa `Self` trong kiểu trả về và trong nội dung của hàm là bí danh cho kiểu
xuất hiện sau từ khóa `impl`, trong trường hợp này là `Rectangle`.

Để gọi hàm liên kết này, chúng ta sử dụng cú pháp `::` với tên struct;
`let sq = Rectangle::square(3);` là một ví dụ. Hàm này được đặt tên trong không
gian tên bởi struct: cú pháp `::` được sử dụng cho cả hàm liên kết và không gian
tên được tạo bởi các module. Chúng ta sẽ thảo luận về các module trong [Chương
7][modules]<!-- ignore -->.

### Nhiều Khối `impl`

Mỗi struct được phép có nhiều khối `impl`. Ví dụ, Listing 5-15 tương đương với
mã được hiển thị trong Listing 5-16, trong đó mỗi method nằm trong khối `impl`
riêng của nó.

<Listing number="5-16" caption="Viết lại Listing 5-15 sử dụng nhiều khối `impl`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-16/src/main.rs:here}}
```

</Listing>

Không có lý do gì để tách các method này thành nhiều khối `impl` ở đây, nhưng
đây là cú pháp hợp lệ. Chúng ta sẽ thấy một trường hợp trong đó nhiều khối
`impl` là hữu ích trong Chương 10, nơi chúng ta thảo luận về các kiểu và trait
generic.

## Tóm tắt

Struct cho phép bạn tạo các kiểu tùy chỉnh có ý nghĩa cho miền của bạn. Bằng
cách sử dụng struct, bạn có thể giữ các phần dữ liệu liên quan với nhau và đặt
tên cho mỗi phần để làm mã của bạn rõ ràng. Trong các khối `impl`, bạn có thể
định nghĩa các hàm được liên kết với kiểu của bạn, và method là một loại hàm
liên kết cho phép bạn chỉ định hành vi mà các instance của struct của bạn có.

Nhưng struct không phải là cách duy nhất để bạn tạo kiểu tùy chỉnh: hãy chuyển
sang tính năng enum của Rust để thêm một công cụ khác vào hộp công cụ của bạn.

[enums]: ch06-00-enums.html
[trait-objects]: ch18-02-trait-objects.md
[public]:
  ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[modules]: ch07-02-defining-modules-to-control-scope-and-privacy.html
