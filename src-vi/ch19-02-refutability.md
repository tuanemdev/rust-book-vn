## Tính bác bỏ: Liệu một Mẫu Có Thể Không Khớp

Các mẫu có hai dạng: bác bỏ được (refutable) và không bác bỏ được (irrefutable).
Các mẫu sẽ khớp với bất kỳ giá trị có thể nào được truyền vào được gọi là _không
bác bỏ được_. Một ví dụ là `x` trong câu lệnh `let x = 5;` bởi vì `x` khớp với
bất cứ thứ gì và do đó không thể thất bại khi khớp. Các mẫu có thể không khớp
với một số giá trị có thể có được gọi là _bác bỏ được_. Một ví dụ là `Some(x)`
trong biểu thức `if let Some(x) = a_value` bởi vì nếu giá trị trong biến
`a_value` là `None` thay vì `Some`, thì mẫu `Some(x)` sẽ không khớp.

Tham số hàm, câu lệnh `let`, và vòng lặp `for` chỉ có thể chấp nhận các mẫu
không bác bỏ được bởi vì chương trình không thể làm bất cứ điều gì có ý nghĩa
khi giá trị không khớp. Các biểu thức `if let` và `while let` và câu lệnh
`let...else` chấp nhận cả mẫu bác bỏ được và không bác bỏ được, nhưng trình biên
dịch cảnh báo đối với các mẫu không bác bỏ được bởi vì, theo định nghĩa, chúng
được dùng để xử lý khả năng thất bại: chức năng của một điều kiện nằm ở khả năng
thực hiện khác nhau tùy thuộc vào thành công hay thất bại.

Nhìn chung, bạn không nên phải lo lắng về sự khác biệt giữa các mẫu bác bỏ được
và không bác bỏ được; tuy nhiên, bạn cần phải làm quen với khái niệm tính bác bỏ
để có thể phản ứng khi bạn thấy nó trong thông báo lỗi. Trong những trường hợp
đó, bạn sẽ cần thay đổi hoặc mẫu hoặc cấu trúc mà bạn đang sử dụng với mẫu, tùy
thuộc vào hành vi dự định của mã.

Hãy xem một ví dụ về điều gì xảy ra khi chúng ta cố gắng sử dụng một mẫu bác bỏ
được trong khi Rust yêu cầu một mẫu không bác bỏ được và ngược lại. Listing 19-8
cho thấy một câu lệnh `let`, nhưng đối với mẫu, chúng ta đã chỉ định `Some(x)`,
một mẫu bác bỏ được. Như bạn có thể mong đợi, mã này sẽ không biên dịch.

<Listing number="19-8" caption="Cố gắng sử dụng một mẫu bác bỏ được với `let`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-08/src/main.rs:here}}
```

</Listing>

Nếu `some_option_value` là một giá trị `None`, nó sẽ không khớp với mẫu
`Some(x)`, có nghĩa là mẫu là bác bỏ được. Tuy nhiên, câu lệnh `let` chỉ có thể
chấp nhận một mẫu không bác bỏ được vì không có điều gì hợp lệ mà mã có thể làm
với một giá trị `None`. Tại thời điểm biên dịch, Rust sẽ phàn nàn rằng chúng ta
đã cố gắng sử dụng một mẫu bác bỏ được ở nơi yêu cầu một mẫu không bác bỏ được:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-08/output.txt}}
```

Bởi vì chúng ta không bao quát (và không thể bao quát!) mọi giá trị hợp lệ với
mẫu `Some(x)`, Rust đúng đắn tạo ra một lỗi biên dịch.

Nếu chúng ta có một mẫu bác bỏ được ở nơi cần một mẫu không bác bỏ được, chúng
ta có thể sửa nó bằng cách thay đổi mã sử dụng mẫu: thay vì sử dụng `let`, chúng
ta có thể sử dụng `let...else`. Sau đó, nếu mẫu không khớp, mã trong ngoặc nhọn sẽ
xử lý giá trị. Listing 19-9 cho thấy cách sửa mã trong Listing 19-8.

<Listing number="19-9" caption="Sử dụng `let...else` và một khối với các mẫu bác bỏ được thay vì `let`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-09/src/main.rs:here}}
```

</Listing>

Chúng ta đã cho mã một lối thoát! Mã này bây giờ hoàn toàn hợp lệ, mặc dù điều
này có nghĩa là chúng ta không thể sử dụng mẫu không bác bỏ được mà không nhận
được cảnh báo. Nếu chúng ta cung cấp cho `let...else` một mẫu sẽ luôn khớp,
chẳng hạn như `x`, như được hiển thị trong Listing 19-10, trình biên dịch sẽ đưa
ra cảnh báo.

<Listing number="19-10" caption="Cố gắng sử dụng một mẫu không bác bỏ được với `let...else`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-10/src/main.rs:here}}
```

</Listing>

Rust phàn nàn rằng việc sử dụng `let...else` với một mẫu không bác bỏ được là
không hợp lý:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-10/output.txt}}
```

Vì lý do này, các nhánh của match phải sử dụng các mẫu bác bỏ được, ngoại trừ
nhánh cuối cùng, nên khớp với bất kỳ giá trị còn lại nào bằng một mẫu không bác
bỏ được. Rust cho phép chúng ta sử dụng một mẫu không bác bỏ được trong một
`match` với chỉ một nhánh, nhưng cú pháp này không đặc biệt hữu ích và có thể
được thay thế bằng một câu lệnh `let` đơn giản hơn.

Bây giờ bạn đã biết nơi sử dụng các mẫu và sự khác biệt giữa các mẫu bác bỏ được
và không bác bỏ được, hãy cùng xem tất cả các cú pháp mà chúng ta có thể sử dụng
để tạo mẫu.
