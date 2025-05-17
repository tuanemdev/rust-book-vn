## Điều khiển Luồng Ngắn Gọn với `if let` và `let...else`

Cú pháp `if let` cho phép bạn kết hợp `if` và `let` thành một cách ít dài dòng
hơn để xử lý các giá trị khớp với một mẫu trong khi bỏ qua phần còn lại. Hãy xem
xét chương trình trong Listing 6-6 khớp với một giá trị `Option<u8>` trong biến
`config_max` nhưng chỉ muốn thực thi mã nếu giá trị là biến thể `Some`.

<Listing number="6-6" caption="Một `match` chỉ quan tâm đến việc thực thi mã khi giá trị là `Some`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-06/src/main.rs:here}}
```

</Listing>

Nếu giá trị là `Some`, chúng ta in ra giá trị trong biến thể `Some` bằng cách
gắn giá trị với biến `max` trong mẫu. Chúng ta không muốn làm gì với giá trị
`None`. Để thỏa mãn biểu thức `match`, chúng ta phải thêm `_ => ()` sau khi xử
lý chỉ một biến thể, đó là mã mẫu khó chịu cần thêm.

Thay vào đó, chúng ta có thể viết điều này ngắn gọn hơn bằng cách sử dụng
`if let`. Đoạn mã sau hoạt động giống như `match` trong Listing 6-6:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-12-if-let/src/main.rs:here}}
```

Cú pháp `if let` lấy một mẫu và một biểu thức được phân tách bằng dấu bằng. Nó
hoạt động giống cách như `match`, trong đó biểu thức được đưa vào `match` và mẫu
là nhánh đầu tiên của nó. Trong trường hợp này, mẫu là `Some(max)`, và `max` gắn
với giá trị bên trong `Some`. Sau đó, chúng ta có thể sử dụng `max` trong phần
thân của khối `if let` tương tự như cách chúng ta sử dụng `max` trong nhánh
`match` tương ứng. Mã trong khối `if let` chỉ chạy nếu giá trị khớp với mẫu.

Sử dụng `if let` có nghĩa là ít phải gõ hơn, ít thụt đầu dòng, và ít mã mẫu. Tuy
nhiên, bạn mất đi kiểm tra tính đầy đủ mà `match` bắt buộc để đảm bảo bạn không
quên xử lý bất kỳ trường hợp nào. Việc lựa chọn giữa `match` và `if let` phụ
thuộc vào những gì bạn đang làm trong tình huống cụ thể và liệu việc đạt được sự
ngắn gọn có phải là một sự đánh đổi phù hợp cho việc mất đi kiểm tra tính đầy đủ
hay không.

Nói cách khác, bạn có thể coi `if let` như là cú pháp đường tắt cho `match` mà
chạy mã khi giá trị khớp với một mẫu và sau đó bỏ qua tất cả các giá trị khác.

Chúng ta có thể bao gồm một `else` với `if let`. Khối mã đi kèm với `else` giống
như khối mã sẽ đi kèm với trường hợp `_` trong biểu thức `match` tương đương với
`if let` và `else`. Nhớ lại định nghĩa enum `Coin` trong Listing 6-4, trong đó
biến thể `Quarter` cũng chứa một giá trị `UsState`. Nếu chúng ta muốn đếm tất cả
các đồng xu không phải quarter mà chúng ta thấy đồng thời thông báo về tiểu bang
của các quarter, chúng ta có thể làm điều đó với biểu thức `match`, như thế này:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-13-count-and-announce-match/src/main.rs:here}}
```

Hoặc chúng ta có thể sử dụng biểu thức `if let` và `else`, như thế này:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-14-count-and-announce-if-let-else/src/main.rs:here}}
```

## Duy trì trên "Đường Hạnh phúc" với `let...else`

Mẫu phổ biến là thực hiện một số tính toán khi một giá trị hiện diện và trả về
một giá trị mặc định nếu không. Tiếp tục với ví dụ của chúng ta về tiền xu với
một giá trị `UsState`, nếu chúng ta muốn nói điều gì đó hài hước tùy thuộc vào
tuổi của tiểu bang trên đồng quarter, chúng ta có thể giới thiệu một phương thức
trên `UsState` để kiểm tra tuổi của một tiểu bang, như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:state}}
```

Sau đó, chúng ta có thể sử dụng `if let` để khớp với loại xu, giới thiệu một
biến `state` trong phần thân của điều kiện, như trong Listing 6-7.

<Listing number="6-7" caption="Kiểm tra xem một tiểu bang có tồn tại vào năm 1900 hay không bằng cách sử dụng điều kiện lồng nhau bên trong một `if let`.">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-07/src/main.rs:describe}}
```

</Listing>

Điều đó hoàn thành công việc, nhưng nó đã đẩy công việc vào trong phần thân của
câu lệnh `if let`, và nếu công việc cần làm phức tạp hơn, có thể khó theo dõi
chính xác cách các nhánh cấp cao liên quan đến nhau. Chúng ta cũng có thể tận
dụng thực tế là các biểu thức tạo ra một giá trị hoặc để tạo ra `state` từ
`if let` hoặc để trả về sớm, như trong Listing 6-8. (Bạn cũng có thể làm tương
tự với `match`.)

<Listing number="6-8" caption="Sử dụng `if let` để tạo ra một giá trị hoặc trả về sớm.">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-08/src/main.rs:describe}}
```

</Listing>

Điều này cũng hơi khó theo dõi theo cách riêng của nó! Một nhánh của `if let`
tạo ra một giá trị, và nhánh kia trả về hoàn toàn từ hàm.

Để làm cho mẫu phổ biến này dễ diễn đạt hơn, Rust có `let...else`. Cú pháp
`let...else` lấy một mẫu ở phía bên trái và một biểu thức ở bên phải, rất giống
với `if let`, nhưng nó không có nhánh `if`, chỉ có nhánh `else`. Nếu mẫu khớp,
nó sẽ gắn giá trị từ mẫu trong phạm vi bên ngoài. Nếu mẫu _không_ khớp, chương
trình sẽ chuyển vào nhánh `else`, phải trả về từ hàm.

Trong Listing 6-9, bạn có thể thấy Listing 6-8 trông như thế nào khi sử dụng
`let...else` thay cho `if let`.

<Listing number="6-9" caption="Sử dụng `let...else` để làm rõ luồng qua hàm.">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-09/src/main.rs:describe}}
```

</Listing>

Lưu ý rằng nó vẫn "trên đường hạnh phúc" trong phần thân chính của hàm theo cách
này, không có luồng điều khiển khác biệt đáng kể cho hai nhánh như `if let` đã
làm.

Nếu bạn có tình huống mà chương trình của bạn có logic quá dài dòng để diễn đạt
bằng `match`, hãy nhớ rằng `if let` và `let...else` cũng có trong bộ công cụ
Rust của bạn.

## Tóm tắt

Bây giờ chúng ta đã đề cập đến cách sử dụng enum để tạo các kiểu tùy chỉnh có
thể là một trong một tập hợp các giá trị được liệt kê. Chúng ta đã thấy cách
kiểu `Option<T>` của thư viện tiêu chuẩn giúp bạn sử dụng hệ thống kiểu để ngăn
chặn lỗi. Khi các giá trị enum có dữ liệu bên trong, bạn có thể sử dụng `match`
hoặc `if let` để trích xuất và sử dụng các giá trị đó, tùy thuộc vào số lượng
trường hợp bạn cần xử lý.

Các chương trình Rust của bạn giờ đây có thể diễn đạt các khái niệm trong miền
của bạn bằng cách sử dụng struct và enum. Việc tạo các kiểu tùy chỉnh để sử dụng
trong API của bạn đảm bảo tính an toàn kiểu: trình biên dịch sẽ đảm bảo rằng các
hàm của bạn chỉ nhận các giá trị thuộc kiểu mà mỗi hàm mong đợi.

Để cung cấp một API được tổ chức tốt cho người dùng của bạn, dễ dàng sử dụng và
chỉ hiển thị chính xác những gì người dùng của bạn sẽ cần, bây giờ hãy chuyển
sang các mô-đun của Rust.
