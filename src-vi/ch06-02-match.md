<!-- Old heading. Do not remove or links may break. -->

<a id="the-match-control-flow-operator"></a>

## Cấu trúc Điều khiển Luồng `match`

Rust có một cấu trúc điều khiển luồng cực kỳ mạnh mẽ gọi là `match`, cho phép
bạn so sánh một giá trị với một loạt các mẫu và sau đó thực thi mã dựa trên mẫu
nào phù hợp. Các mẫu có thể được tạo thành từ các giá trị văn bản, tên biến, ký
tự đại diện, và nhiều thứ khác; [Chương 19][ch19-00-patterns]<!-- ignore --> bao
gồm tất cả các loại mẫu khác nhau và chức năng của chúng. Sức mạnh của `match`
đến từ tính biểu đạt của các mẫu và việc trình biên dịch xác nhận rằng tất cả
các trường hợp có thể xảy ra đều được xử lý.

Hãy nghĩ về biểu thức `match` giống như một máy phân loại tiền xu: các đồng xu
trượt xuống một đường ray với các lỗ có kích thước khác nhau dọc theo nó, và mỗi
đồng xu rơi qua lỗ đầu tiên mà nó gặp và vừa với nó. Tương tự như vậy, các giá
trị đi qua từng mẫu trong một `match`, và tại mẫu đầu tiên mà giá trị "vừa vặn,"
giá trị đó rơi vào khối mã liên quan để được sử dụng trong quá trình thực thi.

Nói về tiền xu, hãy sử dụng chúng làm ví dụ với `match`! Chúng ta có thể viết
một hàm nhận một đồng xu không xác định của Hoa Kỳ và, tương tự như máy đếm, xác
định loại đồng xu nào và trả về giá trị của nó bằng xu, như trong Listing 6-3.

<Listing number="6-3" caption="Một enum và một biểu thức `match` có các biến thể của enum làm mẫu của nó">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-03/src/main.rs:here}}
```

</Listing>

Hãy phân tích `match` trong hàm `value_in_cents`. Đầu tiên, chúng ta liệt kê từ
khóa `match` và theo sau là một biểu thức, trong trường hợp này là giá trị
`coin`. Điều này có vẻ rất giống với một biểu thức điều kiện được sử dụng với
`if`, nhưng có một sự khác biệt lớn: với `if`, điều kiện cần phải đánh giá thành
một giá trị Boolean, nhưng ở đây nó có thể là bất kỳ kiểu dữ liệu nào. Kiểu của
`coin` trong ví dụ này là enum `Coin` mà chúng ta đã định nghĩa ở dòng đầu tiên.

Tiếp theo là các nhánh của `match`. Một nhánh có hai phần: một mẫu và một đoạn
mã. Nhánh đầu tiên ở đây có mẫu là giá trị `Coin::Penny` và sau đó là toán tử
`=>` tách biệt mẫu và mã sẽ chạy. Mã trong trường hợp này chỉ đơn giản là giá
trị `1`. Mỗi nhánh được tách biệt với nhánh tiếp theo bằng một dấu phẩy.

Khi biểu thức `match` thực thi, nó so sánh giá trị kết quả với mẫu của mỗi
nhánh, theo thứ tự. Nếu một mẫu khớp với giá trị, đoạn mã liên kết với mẫu đó sẽ
được thực thi. Nếu mẫu đó không khớp với giá trị, việc thực thi tiếp tục đến
nhánh tiếp theo, tương tự như trong máy phân loại tiền xu. Chúng ta có thể có
nhiều nhánh tùy theo nhu cầu: trong Listing 6-3, `match` của chúng ta có bốn
nhánh.

Đoạn mã liên kết với mỗi nhánh là một biểu thức, và giá trị kết quả của biểu
thức trong nhánh khớp là giá trị được trả về cho toàn bộ biểu thức `match`.

Chúng ta thường không sử dụng dấu ngoặc nhọn nếu mã trong nhánh match ngắn gọn,
như trong Listing 6-3 nơi mỗi nhánh chỉ trả về một giá trị. Nếu bạn muốn chạy
nhiều dòng mã trong một nhánh match, bạn phải sử dụng dấu ngoặc nhọn, và dấu
phẩy sau nhánh sau đó là tùy chọn. Ví dụ, đoạn mã sau in ra "Lucky penny!" mỗi
khi phương thức được gọi với một `Coin::Penny`, nhưng vẫn trả về giá trị cuối
cùng của khối, `1`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-08-match-arm-multiple-lines/src/main.rs:here}}
```

### Các Mẫu Gắn với Giá trị

Một tính năng hữu ích khác của nhánh match là chúng có thể gắn với các phần của
giá trị khớp với mẫu. Đây là cách chúng ta có thể trích xuất các giá trị từ các
biến thể enum.

Ví dụ, hãy thay đổi một trong các biến thể enum của chúng ta để lưu trữ dữ liệu
bên trong nó. Từ năm 1999 đến 2008, Hoa Kỳ đã đúc đồng 25 xu với các thiết kế
khác nhau cho mỗi tiểu bang trong số 50 tiểu bang trên một mặt. Không có đồng xu
nào khác có thiết kế tiểu bang, vì vậy chỉ có đồng quarter có giá trị bổ sung
này. Chúng ta có thể thêm thông tin này vào `enum` bằng cách thay đổi biến thể
`Quarter` để bao gồm một giá trị `UsState` được lưu trữ bên trong nó, như chúng
ta đã làm trong Listing 6-4.

<Listing number="6-4" caption="Một enum `Coin` trong đó biến thể `Quarter` cũng giữ một giá trị `UsState`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-04/src/main.rs:here}}
```

</Listing>

Hãy tưởng tượng rằng một người bạn đang cố gắng sưu tập tất cả 50 đồng quarter
của các tiểu bang. Trong khi chúng ta phân loại tiền lẻ theo loại đồng xu, chúng
ta cũng sẽ nói to tên của tiểu bang liên quan đến mỗi đồng quarter để nếu đó là
một đồng mà bạn của chúng ta chưa có, họ có thể thêm nó vào bộ sưu tập của mình.

Trong biểu thức match cho đoạn mã này, chúng ta thêm một biến có tên `state` vào
mẫu khớp với các giá trị của biến thể `Coin::Quarter`. Khi một `Coin::Quarter`
khớp, biến `state` sẽ gắn với giá trị của tiểu bang của đồng quarter đó. Sau đó,
chúng ta có thể sử dụng `state` trong đoạn mã cho nhánh đó, như sau:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-09-variable-in-pattern/src/main.rs:here}}
```

Nếu chúng ta gọi `value_in_cents(Coin::Quarter(UsState::Alaska))`, `coin` sẽ là
`Coin::Quarter(UsState::Alaska)`. Khi chúng ta so sánh giá trị đó với từng nhánh
match, không có nhánh nào khớp cho đến khi chúng ta đến `Coin::Quarter(state)`.
Tại thời điểm đó, giá trị gắn với `state` sẽ là `UsState::Alaska`. Chúng ta có
thể sử dụng giá trị gắn đó trong biểu thức `println!`, từ đó lấy được giá trị
tiểu bang bên trong ra khỏi biến thể enum `Coin` cho `Quarter`.

### Khớp với `Option<T>`

Trong phần trước, chúng ta muốn lấy giá trị `T` bên trong ra khỏi trường hợp
`Some` khi sử dụng `Option<T>`; chúng ta cũng có thể xử lý `Option<T>` bằng cách
sử dụng `match`, như chúng ta đã làm với enum `Coin`! Thay vì so sánh các đồng
xu, chúng ta sẽ so sánh các biến thể của `Option<T>`, nhưng cách biểu thức
`match` hoạt động vẫn giữ nguyên.

Giả sử chúng ta muốn viết một hàm nhận một `Option<i32>` và, nếu có giá trị bên
trong, cộng thêm 1 vào giá trị đó. Nếu không có giá trị bên trong, hàm sẽ trả về
giá trị `None` và không cố gắng thực hiện bất kỳ thao tác nào.

Hàm này rất dễ viết, nhờ `match`, và sẽ trông giống như Listing 6-5.

<Listing number="6-5" caption="Một hàm sử dụng biểu thức `match` trên một `Option<i32>`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:here}}
```

</Listing>

Hãy xem xét lần thực thi đầu tiên của `plus_one` chi tiết hơn. Khi chúng ta gọi
`plus_one(five)`, biến `x` trong thân của `plus_one` sẽ có giá trị `Some(5)`.
Sau đó, chúng ta so sánh giá trị đó với từng nhánh match:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Giá trị `Some(5)` không khớp với mẫu `None`, vì vậy chúng ta tiếp tục đến nhánh
tiếp theo:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:second_arm}}
```

`Some(5)` có khớp với `Some(i)` không? Có! Chúng ta có cùng một biến thể. `i`
gắn với giá trị được chứa trong `Some`, vì vậy `i` nhận giá trị `5`. Sau đó, mã
trong nhánh match được thực thi, vì vậy chúng ta cộng 1 vào giá trị của `i` và
tạo ra một giá trị `Some` mới với tổng `6` bên trong.

Bây giờ hãy xem xét lần gọi thứ hai của `plus_one` trong Listing 6-5, trong đó
`x` là `None`. Chúng ta đi vào `match` và so sánh với nhánh đầu tiên:

```rust,ignore
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-05/src/main.rs:first_arm}}
```

Nó khớp! Không có giá trị để cộng thêm, vì vậy chương trình dừng lại và trả về
giá trị `None` ở phía bên phải của `=>`. Bởi vì nhánh đầu tiên đã khớp, không có
nhánh nào khác được so sánh.

Kết hợp `match` và enum rất hữu ích trong nhiều tình huống. Bạn sẽ thấy mẫu này
rất nhiều trong mã Rust: `match` đối với một enum, gắn một biến với dữ liệu bên
trong, và sau đó thực thi mã dựa trên nó. Ban đầu có thể hơi khó hiểu, nhưng một
khi bạn đã quen với nó, bạn sẽ ước rằng mọi ngôn ngữ đều có nó. Nó luôn là một
tính năng yêu thích của người dùng.

### Các Match Đều Phải Đầy đủ

Có một khía cạnh khác của `match` mà chúng ta cần thảo luận: các mẫu của nhánh
phải bao gồm tất cả các khả năng. Hãy xem xét phiên bản này của hàm `plus_one`
của chúng ta, có một lỗi và sẽ không biên dịch được:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/src/main.rs:here}}
```

Chúng ta không xử lý trường hợp `None`, vì vậy mã này sẽ gây ra lỗi. May mắn
thay, đây là một lỗi mà Rust biết cách phát hiện. Nếu chúng ta cố gắng biên dịch
mã này, chúng ta sẽ nhận được lỗi này:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-10-non-exhaustive-match/output.txt}}
```

Rust biết rằng chúng ta không bao gồm mọi trường hợp có thể xảy ra, và thậm chí
còn biết mẫu nào chúng ta đã quên! Các match trong Rust là _đầy đủ_: chúng ta
phải liệt kê tất cả các khả năng để mã hợp lệ. Đặc biệt là trong trường hợp
`Option<T>`, khi Rust ngăn chúng ta quên xử lý rõ ràng trường hợp `None`, nó bảo
vệ chúng ta khỏi việc giả định rằng chúng ta có một giá trị khi chúng ta có thể
có null, từ đó làm cho lỗi hàng tỷ đô la đã thảo luận trước đó trở nên không
thể.

### Mẫu Bắt tất cả và Placeholder `_`

Sử dụng enum, chúng ta cũng có thể thực hiện các hành động đặc biệt cho một số
giá trị cụ thể, nhưng đối với tất cả các giá trị khác, thực hiện một hành động
mặc định. Hãy tưởng tượng chúng ta đang triển khai một trò chơi, trong đó, nếu
bạn tung được số 3 trên xúc xắc, người chơi của bạn không di chuyển, mà thay vào
đó nhận được một chiếc mũ đẹp mới. Nếu bạn tung được số 7, người chơi của bạn
mất một chiếc mũ đẹp. Đối với tất cả các giá trị khác, người chơi của bạn di
chuyển số ô đó trên bàn chơi. Dưới đây là một `match` triển khai logic đó, với
kết quả của việc tung xúc xắc được cố định thay vì một giá trị ngẫu nhiên, và
tất cả logic khác được biểu diễn bằng các hàm không có nội dung vì việc triển
khai chúng nằm ngoài phạm vi của ví dụ này:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-15-binding-catchall/src/main.rs:here}}
```

Đối với hai nhánh đầu tiên, các mẫu là các giá trị văn bản `3` và `7`. Đối với
nhánh cuối cùng bao gồm mọi giá trị có thể khác, mẫu là biến mà chúng ta đã chọn
đặt tên là `other`. Mã chạy cho nhánh `other` sử dụng biến bằng cách truyền nó
vào hàm `move_player`.

Mã này biên dịch được, mặc dù chúng ta chưa liệt kê tất cả các giá trị có thể có
của một `u8`, bởi vì mẫu cuối cùng sẽ khớp với tất cả các giá trị không được
liệt kê cụ thể. Mẫu bắt tất cả này đáp ứng yêu cầu rằng `match` phải đầy đủ. Lưu
ý rằng chúng ta phải đặt nhánh bắt tất cả cuối cùng vì các mẫu được đánh giá
theo thứ tự. Nếu chúng ta đặt nhánh bắt tất cả sớm hơn, các nhánh khác sẽ không
bao giờ chạy, vì vậy Rust sẽ cảnh báo chúng ta nếu chúng ta thêm nhánh sau một
nhánh bắt tất cả!

Rust cũng có một mẫu mà chúng ta có thể sử dụng khi muốn bắt tất cả nhưng không
muốn _sử dụng_ giá trị trong mẫu bắt tất cả: `_` là một mẫu đặc biệt khớp với
bất kỳ giá trị nào và không gắn với giá trị đó. Điều này nói với Rust rằng chúng
ta sẽ không sử dụng giá trị, vì vậy Rust sẽ không cảnh báo chúng ta về một biến
không được sử dụng.

Hãy thay đổi luật của trò chơi: bây giờ, nếu bạn tung bất kỳ số nào khác ngoài 3
hoặc 7, bạn phải tung lại. Chúng ta không còn cần sử dụng giá trị bắt tất cả
nữa, vì vậy chúng ta có thể thay đổi mã của mình để sử dụng `_` thay vì biến có
tên `other`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-16-underscore-catchall/src/main.rs:here}}
```

Ví dụ này cũng đáp ứng yêu cầu đầy đủ vì chúng ta đang rõ ràng bỏ qua tất cả các
giá trị khác trong nhánh cuối cùng; chúng ta không quên bất cứ điều gì.

Cuối cùng, chúng ta sẽ thay đổi luật của trò chơi một lần nữa để không có gì
khác xảy ra trong lượt của bạn nếu bạn tung bất kỳ số nào khác ngoài 3 hoặc 7.
Chúng ta có thể biểu đạt điều đó bằng cách sử dụng giá trị đơn vị (kiểu tuple
rỗng mà chúng ta đã đề cập trong phần ["Kiểu Tuple"][tuples]<!-- ignore -->) làm
mã đi kèm với nhánh `_`:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-17-underscore-unit/src/main.rs:here}}
```

Ở đây, chúng ta đang nói với Rust một cách rõ ràng rằng chúng ta sẽ không sử
dụng bất kỳ giá trị nào khác không khớp với mẫu trong một nhánh trước đó, và
chúng ta không muốn chạy bất kỳ mã nào trong trường hợp này.

Còn nhiều điều về mẫu và khớp mà chúng ta sẽ đề cập trong [Chương
19][ch19-00-patterns]<!-- ignore -->. Hiện tại, chúng ta sẽ tiếp tục với cú pháp
`if let`, có thể hữu ích trong các tình huống mà biểu thức `match` hơi dài dòng.

[tuples]: ch03-02-data-types.html#the-tuple-type
[ch19-00-patterns]: ch19-00-patterns.html
