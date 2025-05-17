## Kiểu Dữ Liệu Nâng Cao

Hệ thống kiểu dữ liệu của Rust có một số tính năng mà chúng ta đã đề cập nhưng
chưa thảo luận chi tiết. Chúng ta sẽ bắt đầu bằng việc thảo luận về newtype nói
chung khi chúng ta xem xét tại sao newtype là kiểu dữ liệu hữu ích. Sau đó,
chúng ta sẽ chuyển sang bí danh kiểu (type aliases), một tính năng tương tự như
newtype nhưng có ngữ nghĩa hơi khác một chút. Chúng ta cũng sẽ thảo luận về kiểu
`!` và các kiểu có kích thước động.

### Sử Dụng Mẫu Newtype cho An Toàn Kiểu và Trừu Tượng Hóa

Phần này giả định bạn đã đọc phần trước ["Sử Dụng Mẫu Newtype để Triển Khai
Trait Bên Ngoài trên Kiểu Bên
Ngoài."][using-the-newtype-pattern]<!-- ignore -->. Mẫu newtype cũng hữu ích cho
các nhiệm vụ ngoài những gì chúng ta đã thảo luận cho đến nay, bao gồm việc đảm
bảo tĩnh rằng các giá trị không bao giờ bị nhầm lẫn và chỉ ra đơn vị của một giá
trị. Bạn đã thấy một ví dụ về việc sử dụng newtype để chỉ ra đơn vị trong
Listing 20-16: nhớ lại rằng các struct `Millimeters` và `Meters` bọc các giá trị
`u32` trong một newtype. Nếu chúng ta viết một hàm với tham số có kiểu
`Millimeters`, chúng ta sẽ không thể biên dịch một chương trình mà vô tình cố
gắng gọi hàm đó với một giá trị có kiểu `Meters` hoặc một `u32` đơn giản.

Chúng ta cũng có thể sử dụng mẫu newtype để trừu tượng hóa một số chi tiết triển
khai của một kiểu: kiểu mới có thể hiển thị một API công khai khác với API của
kiểu bên trong riêng tư.

Newtype cũng có thể ẩn triển khai nội bộ. Ví dụ, chúng ta có thể cung cấp một
kiểu `People` để bọc một `HashMap<i32, String>` lưu trữ ID của một người kết hợp
với tên của họ. Mã sử dụng `People` sẽ chỉ tương tác với API công khai mà chúng
ta cung cấp, chẳng hạn như một phương thức để thêm một chuỗi tên vào bộ sưu tập
`People`; mã đó sẽ không cần biết rằng chúng ta gán một ID `i32` cho tên một
cách nội bộ. Mẫu newtype là một cách nhẹ nhàng để đạt được sự đóng gói để ẩn chi
tiết triển khai, điều mà chúng ta đã thảo luận trong ["Đóng Gói ẩn chi tiết
triển khai"][encapsulation-that-hides-implementation-details]<!-- ignore -->
trong Chương 18.

### Tạo Đồng Nghĩa Kiểu với Bí Danh Kiểu

Rust cung cấp khả năng khai báo một _bí danh kiểu_ để đặt tên khác cho một kiểu
hiện có. Đối với điều này, chúng ta sử dụng từ khóa `type`. Ví dụ, chúng ta có
thể tạo bí danh `Kilometers` cho `i32` như sau:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:here}}
```

Bây giờ, bí danh `Kilometers` là một _từ đồng nghĩa_ cho `i32`; không giống như
các kiểu `Millimeters` và `Meters` mà chúng ta đã tạo trong Listing 20-16,
`Kilometers` không phải là một kiểu mới, riêng biệt. Các giá trị có kiểu
`Kilometers` sẽ được xử lý giống như các giá trị kiểu `i32`:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-04-kilometers-alias/src/main.rs:there}}
```

Bởi vì `Kilometers` và `i32` là cùng một kiểu, chúng ta có thể cộng các giá trị
của cả hai kiểu và chúng ta có thể truyền các giá trị `Kilometers` cho các hàm
nhận tham số `i32`. Tuy nhiên, khi sử dụng phương pháp này, chúng ta không nhận
được lợi ích kiểm tra kiểu mà chúng ta nhận được từ mẫu newtype đã thảo luận
trước đó. Nói cách khác, nếu chúng ta nhầm lẫn giữa các giá trị `Kilometers` và
`i32` ở đâu đó, trình biên dịch sẽ không đưa ra lỗi.

Trường hợp sử dụng chính cho bí danh kiểu là giảm sự lặp lại. Ví dụ, chúng ta có
thể có một kiểu dài như thế này:

```rust,ignore
Box<dyn Fn() + Send + 'static>
```

Viết kiểu dài này trong chữ ký hàm và chú thích kiểu trên khắp mã có thể nhàm
chán và dễ gây lỗi. Hãy tưởng tượng có một dự án đầy mã như trong Listing 20-25.

<Listing number="20-25" caption="Sử dụng một kiểu dài ở nhiều nơi">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-25/src/main.rs:here}}
```

</Listing>

Một bí danh kiểu làm cho mã này dễ quản lý hơn bằng cách giảm sự lặp lại. Trong
Listing 20-26, chúng ta đã giới thiệu một bí danh có tên `Thunk` cho kiểu dài
dòng và có thể thay thế tất cả các sử dụng của kiểu đó bằng bí danh ngắn gọn hơn
`Thunk`.

<Listing number="20-26" caption="Giới thiệu bí danh kiểu `Thunk` để giảm sự lặp lại">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-26/src/main.rs:here}}
```

</Listing>

Mã này dễ đọc và viết hơn nhiều! Việc chọn một tên có ý nghĩa cho bí danh kiểu
có thể giúp truyền đạt ý định của bạn (_thunk_ là một từ chỉ mã được đánh giá
vào thời điểm sau, vì vậy đó là một tên thích hợp cho một closure được lưu trữ).

Bí danh kiểu cũng thường được sử dụng với kiểu `Result<T, E>` để giảm sự lặp
lại. Xem xét module `std::io` trong thư viện chuẩn. Các hoạt động I/O thường trả
về một `Result<T, E>` để xử lý các tình huống khi các hoạt động không thành
công. Thư viện này có một struct `std::io::Error` đại diện cho tất cả các lỗi
I/O có thể xảy ra. Nhiều hàm trong `std::io` sẽ trả về `Result<T, E>` trong đó
`E` là `std::io::Error`, chẳng hạn như các hàm trong trait `Write`:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-05-write-trait/src/lib.rs}}
```

`Result<..., Error>` được lặp lại rất nhiều. Do đó, `std::io` có khai báo bí
danh kiểu này:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:here}}
```

Vì khai báo này nằm trong module `std::io`, chúng ta có thể sử dụng bí danh đầy
đủ `std::io::Result<T>`; đó là một `Result<T, E>` với `E` được điền là
`std::io::Error`. Chữ ký hàm của trait `Write` cuối cùng trông như thế này:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-06-result-alias/src/lib.rs:there}}
```

Bí danh kiểu giúp ích theo hai cách: nó làm cho mã dễ viết hơn _và_ cung cấp cho
chúng ta một giao diện nhất quán trên toàn bộ `std::io`. Bởi vì nó là một bí
danh, nó chỉ là một `Result<T, E>` khác, có nghĩa là chúng ta có thể sử dụng bất
kỳ phương thức nào hoạt động với `Result<T, E>` với nó, cũng như cú pháp đặc
biệt như toán tử `?`.

### Kiểu Never Không Bao Giờ Trả Về

Rust có một kiểu đặc biệt có tên là `!` được biết đến trong thuật ngữ lý thuyết
kiểu là _kiểu rỗng_ vì nó không có giá trị. Chúng ta thích gọi nó là _kiểu
never_ vì nó đứng ở vị trí của kiểu trả về khi một hàm không bao giờ trả về. Đây
là một ví dụ:

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-07-never-type/src/lib.rs:here}}
```

Mã này được đọc là "hàm `bar` không bao giờ trả về." Các hàm không bao giờ trả
về được gọi là _hàm phân kỳ_. Chúng ta không thể tạo giá trị của kiểu `!` nên
`bar` không thể trả về.

Nhưng ích lợi gì của một kiểu mà bạn không bao giờ có thể tạo giá trị cho nó?
Nhớ lại mã từ Listing 2-5, một phần của trò chơi đoán số; chúng ta đã tái hiện
một phần của nó ở đây trong Listing 20-27.

<Listing number="20-27" caption="Một `match` với một nhánh kết thúc bằng `continue`">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:ch19}}
```

</Listing>

Tại thời điểm đó, chúng ta đã bỏ qua một số chi tiết trong mã này. Trong ["Toán
Tử Điều Khiển Luồng `match`"][the-match-control-flow-operator]<!-- ignore -->
trong Chương 6, chúng ta đã thảo luận rằng các nhánh của `match` phải trả về
cùng một kiểu. Vì vậy, ví dụ, mã sau đây không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-08-match-arms-different-types/src/main.rs:here}}
```

Kiểu của `guess` trong mã này sẽ phải là một số nguyên _và_ một chuỗi, và Rust
yêu cầu `guess` chỉ có một kiểu. Vậy `continue` trả về gì? Làm thế nào mà chúng
ta được phép trả về một `u32` từ một nhánh và có một nhánh khác kết thúc bằng
`continue` trong Listing 20-27?

Như bạn có thể đã đoán, `continue` có một giá trị `!`. Nghĩa là, khi Rust tính
toán kiểu của `guess`, nó xem xét cả hai nhánh match, nhánh trước với giá trị
`u32` và nhánh sau với giá trị `!`. Bởi vì `!` không bao giờ có thể có một giá
trị, Rust quyết định rằng kiểu của `guess` là `u32`.

Cách chính thức để mô tả hành vi này là các biểu thức của kiểu `!` có thể được
ép buộc vào bất kỳ kiểu nào khác. Chúng ta được phép kết thúc nhánh `match` này
với `continue` bởi vì `continue` không trả về giá trị; thay vào đó, nó chuyển
điều khiển trở lại đầu vòng lặp, vì vậy trong trường hợp `Err`, chúng ta không
bao giờ gán giá trị cho `guess`.

Kiểu never cũng hữu ích với macro `panic!`. Nhớ lại hàm `unwrap` mà chúng ta gọi
trên các giá trị `Option<T>` để tạo ra một giá trị hoặc gây panic với định nghĩa
này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-09-unwrap-definition/src/lib.rs:here}}
```

Trong mã này, điều tương tự xảy ra như trong `match` ở Listing 20-27: Rust thấy
rằng `val` có kiểu `T` và `panic!` có kiểu `!`, vì vậy kết quả của toàn bộ biểu
thức `match` là `T`. Mã này hoạt động bởi vì `panic!` không tạo ra một giá trị;
nó kết thúc chương trình. Trong trường hợp `None`, chúng ta sẽ không trả về giá
trị từ `unwrap`, vì vậy mã này hợp lệ.

Một biểu thức cuối cùng có kiểu `!` là vòng lặp `loop`:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-10-loop-returns-never/src/main.rs:here}}
```

Ở đây, vòng lặp không bao giờ kết thúc, vì vậy `!` là giá trị của biểu thức. Tuy
nhiên, điều này sẽ không đúng nếu chúng ta đưa vào một `break`, bởi vì vòng lặp
sẽ kết thúc khi nó đến `break`.

### Kiểu Có Kích Thước Động và Trait `Sized`

Rust cần biết một số chi tiết về các kiểu của nó, chẳng hạn như cần bao nhiêu
không gian để cấp phát cho một giá trị của một kiểu cụ thể. Điều này để lại một
góc của hệ thống kiểu hơi khó hiểu lúc đầu: khái niệm về _kiểu có kích thước
động_. Đôi khi được gọi là _DST_ hoặc _kiểu không có kích thước_, những kiểu này
cho phép chúng ta viết mã sử dụng các giá trị mà chúng ta chỉ có thể biết kích
thước vào thời điểm chạy.

Hãy đi sâu vào chi tiết của một kiểu có kích thước động có tên là `str`, mà
chúng ta đã sử dụng trong suốt cuốn sách. Đúng vậy, không phải `&str`, mà là
`str` tự nó, là một DST. Chúng ta không thể biết chuỗi dài bao nhiêu cho đến
thời điểm chạy, có nghĩa là chúng ta không thể tạo một biến kiểu `str`, cũng
không thể nhận một đối số kiểu `str`. Xem xét mã sau, mã này không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-11-cant-create-str/src/main.rs:here}}
```

Rust cần biết cần cấp phát bao nhiêu bộ nhớ cho bất kỳ giá trị nào của một kiểu
cụ thể, và tất cả các giá trị của một kiểu phải sử dụng cùng một lượng bộ nhớ.
Nếu Rust cho phép chúng ta viết mã này, hai giá trị `str` này sẽ cần chiếm cùng
một lượng không gian. Nhưng chúng có độ dài khác nhau: `s1` cần 12 byte lưu trữ
và `s2` cần 15. Đó là lý do tại sao không thể tạo một biến chứa một kiểu có kích
thước động.

Vậy chúng ta làm gì? Trong trường hợp này, bạn đã biết câu trả lời: chúng ta làm
cho kiểu của `s1` và `s2` là `&str` thay vì `str`. Nhớ lại từ ["Slice
Chuỗi"][string-slices]<!-- ignore --> trong Chương 4 rằng cấu trúc dữ liệu slice
chỉ lưu trữ vị trí bắt đầu và độ dài của slice. Vì vậy, mặc dù một `&T` là một
giá trị đơn lẻ lưu trữ địa chỉ bộ nhớ của nơi `T` nằm, một `&str` là _hai_ giá
trị: địa chỉ của `str` và độ dài của nó. Do đó, chúng ta có thể biết kích thước
của một giá trị `&str` tại thời điểm biên dịch: nó gấp đôi độ dài của một
`usize`. Nghĩa là, chúng ta luôn biết kích thước của một `&str`, dù chuỗi mà nó
tham chiếu đến dài bao nhiêu. Nói chung, đây là cách mà các kiểu có kích thước
động được sử dụng trong Rust: chúng có một bit siêu dữ liệu bổ sung lưu trữ kích
thước của thông tin động. Quy tắc vàng của các kiểu có kích thước động là chúng
ta phải luôn đặt các giá trị của kiểu có kích thước động sau một con trỏ nào đó.

Chúng ta có thể kết hợp `str` với tất cả các loại con trỏ: ví dụ, `Box<str>`
hoặc `Rc<str>`. Thực tế, bạn đã thấy điều này trước đây nhưng với một kiểu có
kích thước động khác: trait. Mỗi trait là một kiểu có kích thước động mà chúng
ta có thể tham chiếu bằng cách sử dụng tên của trait. Trong ["Sử Dụng Đối Tượng
Trait Cho Phép Cho Giá Trị Của Các Kiểu Khác
Nhau"][using-trait-objects-that-allow-for-values-of-different-types]<!-- ignore -->
trong Chương 18, chúng ta đã đề cập rằng để sử dụng trait làm đối tượng trait,
chúng ta phải đặt chúng sau một con trỏ, chẳng hạn như `&dyn Trait` hoặc
`Box<dyn Trait>` (`Rc<dyn Trait>` cũng sẽ hoạt động).

Để làm việc với DST, Rust cung cấp trait `Sized` để xác định liệu kích thước của
một kiểu có được biết tại thời điểm biên dịch hay không. Trait này được tự động
triển khai cho mọi thứ có kích thước được biết tại thời điểm biên dịch. Ngoài
ra, Rust ngầm thêm một ràng buộc về `Sized` vào mọi hàm generic. Nghĩa là, một
định nghĩa hàm generic như thế này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-12-generic-fn-definition/src/lib.rs}}
```

thực tế được xử lý như thể chúng ta đã viết điều này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-13-generic-implicit-sized-bound/src/lib.rs}}
```

Theo mặc định, các hàm generic sẽ chỉ hoạt động trên các kiểu có kích thước đã
biết tại thời điểm biên dịch. Tuy nhiên, bạn có thể sử dụng cú pháp đặc biệt sau
đây để nới lỏng hạn chế này:

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-14-generic-maybe-sized/src/lib.rs}}
```

Một ràng buộc trait trên `?Sized` có nghĩa là "`T` có thể hoặc có thể không là
`Sized`" và ký hiệu này ghi đè mặc định rằng các kiểu generic phải có kích thước
đã biết tại thời điểm biên dịch. Cú pháp `?Trait` với ý nghĩa này chỉ có sẵn cho
`Sized`, không cho bất kỳ trait nào khác.

Cũng lưu ý rằng chúng ta đã chuyển kiểu của tham số `t` từ `T` sang `&T`. Bởi vì
kiểu có thể không phải là `Sized`, chúng ta cần sử dụng nó sau một loại con trỏ
nào đó. Trong trường hợp này, chúng ta đã chọn một tham chiếu.

Tiếp theo, chúng ta sẽ nói về các hàm và closure!

[encapsulation-that-hides-implementation-details]:
  ch18-01-what-is-oo.html#encapsulation-that-hides-implementation-details
[string-slices]: ch04-03-slices.html#string-slices
[the-match-control-flow-operator]:
  ch06-02-match.html#the-match-control-flow-operator
[using-trait-objects-that-allow-for-values-of-different-types]:
  ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[using-the-newtype-pattern]:
  ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
