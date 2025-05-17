## Cú Pháp Mẫu

Trong phần này, chúng ta tập hợp tất cả các syntax hợp lệ trong các mẫu và thảo
luận về lý do và thời điểm bạn có thể muốn sử dụng mỗi syntax.

### Khớp với Giá Trị Cụ Thể

Như bạn đã thấy trong Chương 6, bạn có thể khớp các mẫu trực tiếp với các giá
trị cụ thể. Đoạn mã sau đây đưa ra một số ví dụ:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-01-literals/src/main.rs:here}}
```

Đoạn mã này in ra `one` vì giá trị trong `x` là 1. Cú pháp này hữu ích khi bạn
muốn mã của mình thực hiện một hành động nếu nó nhận được một giá trị cụ thể.

### Khớp với Biến Có Tên

Các biến có tên là các mẫu không thể bác bỏ (irrefutable) khớp với bất kỳ giá
trị nào, và chúng ta đã sử dụng chúng nhiều lần trong cuốn sách này. Tuy nhiên,
có một phức tạp khi bạn sử dụng các biến có tên trong expression `match`,
`if let` hoặc `while let`. Bởi vì mỗi loại expression này bắt đầu một phạm vi
mới, các biến được khai báo như một phần của mẫu bên trong expression sẽ che
khuất (shadow) các biến có cùng tên bên ngoài, như trường hợp với tất cả các
biến. Trong Listing 19-11, chúng ta khai báo một biến có tên `x` với giá trị
`Some(5)` và một biến `y` với giá trị `10`. Sau đó, chúng ta tạo một expression
`match` dựa trên giá trị `x`. Hãy nhìn vào các mẫu trong các arm của match và
`println!` ở cuối, và thử tìm hiểu xem đoạn mã sẽ in ra gì trước khi chạy mã này
hoặc đọc tiếp.

<Listing number="19-11" file-name="src/main.rs" caption="Một expression `match` với một arm giới thiệu một biến mới che khuất một biến `y` hiện có">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-11/src/main.rs:here}}
```

</Listing>

Hãy cùng xem xét những gì xảy ra khi biểu thức `match` chạy. Mẫu trong arm match
đầu tiên không khớp với giá trị đã định nghĩa của `x`, vì vậy mã tiếp tục.

Mẫu trong arm match thứ hai giới thiệu một biến mới có tên `y` sẽ khớp với bất
kỳ giá trị nào bên trong một giá trị `Some`. Vì chúng ta đang ở trong một phạm
vi mới bên trong biểu thức `match`, đây là một biến `y` mới, không phải biến `y`
mà chúng ta đã khai báo ở đầu với giá trị `10`. Ràng buộc `y` mới này sẽ khớp
với bất kỳ giá trị nào bên trong một `Some`, đó chính là những gì chúng ta có
trong `x`. Do đó, `y` mới này ràng buộc với giá trị bên trong của `Some` trong
`x`. Giá trị đó là `5`, vì vậy biểu thức cho arm đó thực thi và in ra
`Matched, y = 5`.

Nếu `x` đã là giá trị `None` thay vì `Some(5)`, thì các mẫu trong hai arm đầu
tiên sẽ không khớp, vì vậy giá trị sẽ khớp với dấu gạch dưới. Chúng ta không
giới thiệu biến `x` trong mẫu của arm dấu gạch dưới, vì vậy `x` trong biểu thức
vẫn là `x` bên ngoài chưa bị che khuất. Trong trường hợp giả định này, phép
`match` sẽ in ra `Default case, x = None`.

Khi biểu thức `match` kết thúc, phạm vi của nó cũng kết thúc, và phạm vi của `y`
bên trong cũng vậy. `println!` cuối cùng tạo ra
`at the end: x = Some(5), y = 10`.

Để tạo một biểu thức `match` so sánh giá trị của `x` và `y` bên ngoài, thay vì
giới thiệu một biến mới che khuất biến `y` hiện có, chúng ta cần sử dụng một
điều kiện bảo vệ match (match guard) thay thế. Chúng ta sẽ nói về match guard
sau trong phần
["Điều Kiện Bổ Sung với Match Guards"](#Điều-kiện-bổ-sung-với-match-guards)<!-- ignore -->.

### Nhiều Mẫu

Bạn có thể khớp nhiều mẫu bằng cách sử dụng cú pháp `|`, đó là toán tử _hoặc_
trong mẫu. Ví dụ, trong đoạn mã sau, chúng ta so khớp giá trị của `x` với các
arm match, arm đầu tiên có một tùy chọn _hoặc_, có nghĩa là nếu giá trị của `x`
khớp với bất kỳ giá trị nào trong arm đó, thì mã của arm đó sẽ chạy:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-02-multiple-patterns/src/main.rs:here}}
```

Đoạn mã này in ra `one or two`.

### Khớp với Phạm Vi Giá Trị bằng `..=`

Cú pháp `..=` cho phép chúng ta khớp với một phạm vi giá trị bao gồm
(inclusive). Trong đoạn mã sau, khi một mẫu khớp với bất kỳ giá trị nào trong
phạm vi đã cho, arm đó sẽ thực thi:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-03-ranges/src/main.rs:here}}
```

Nếu `x` là `1`, `2`, `3`, `4`, hoặc `5`, arm đầu tiên sẽ khớp. Cú pháp này thuận
tiện hơn cho nhiều giá trị khớp so với việc sử dụng toán tử `|` để biểu thị cùng
một ý tưởng; nếu chúng ta sử dụng `|`, chúng ta sẽ phải chỉ định
`1 | 2 | 3 | 4 | 5`. Việc chỉ định một phạm vi ngắn gọn hơn nhiều, đặc biệt là
nếu chúng ta muốn khớp, chẳng hạn, bất kỳ số nào từ 1 đến 1.000!

Trình biên dịch kiểm tra xem phạm vi có trống không tại thời điểm biên dịch, và
vì các loại duy nhất mà Rust có thể xác định xem một phạm vi có trống hay không
là các giá trị `char` và số, nên các phạm vi chỉ được phép với các giá trị số
hoặc `char`.

Đây là một ví dụ sử dụng phạm vi của các giá trị `char`:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-04-ranges-of-char/src/main.rs:here}}
```

Rust có thể biết rằng `'c'` nằm trong phạm vi của mẫu đầu tiên và in ra
`early ASCII letter`.

### Phá Vỡ để Tách Các Giá Trị

Chúng ta cũng có thể sử dụng các mẫu để phá vỡ các structs, enums và tuples để
sử dụng các phần khác nhau của các giá trị này. Hãy cùng xem xét từng giá trị.

#### Phá Vỡ Structs

Listing 19-12 cho thấy một struct `Point` với hai trường, `x` và `y`, mà chúng
ta có thể tách ra bằng cách sử dụng một mẫu với một câu lệnh `let`.

<Listing number="19-12" file-name="src/main.rs" caption="Tách các trường của một struct thành các biến riêng biệt">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-12/src/main.rs}}
```

</Listing>

Đoạn mã này tạo ra các biến `a` và `b` khớp với các giá trị của các trường `x`
và `y` của struct `p`. Ví dụ này cho thấy rằng tên của các biến trong mẫu không
cần phải khớp với tên của các trường của struct. Tuy nhiên, thông thường người
ta đặt tên biến trùng với tên trường để dễ nhớ biến nào đến từ trường nào. Vì
cách sử dụng phổ biến này, và vì việc viết `let Point { x: x, y: y } = p;` chứa
nhiều sự lặp lại, Rust có một cách viết ngắn gọn cho các mẫu khớp với các trường
struct: bạn chỉ cần liệt kê tên của trường struct, và các biến được tạo từ mẫu
sẽ có cùng tên. Listing 19-13 hoạt động giống như đoạn mã trong Listing 19-12,
nhưng các biến được tạo trong mẫu `let` là `x` và `y` thay vì `a` và `b`.

<Listing number="19-13" file-name="src/main.rs" caption="Phá vỡ các trường struct bằng cách sử dụng cách viết ngắn gọn của trường struct">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-13/src/main.rs}}
```

</Listing>

Đoạn mã này tạo ra các biến `x` và `y` khớp với các trường `x` và `y` của biến
`p`. Kết quả là các biến `x` và `y` chứa các giá trị từ struct `p`.

Chúng ta cũng có thể phá vỡ với các giá trị cụ thể như một phần của mẫu struct
thay vì tạo các biến cho tất cả các trường. Làm như vậy cho phép chúng ta kiểm
tra một số trường cho các giá trị cụ thể trong khi tạo các biến để phá vỡ các
trường khác.

Trong Listing 19-14, chúng ta có một biểu thức `match` phân tách các giá trị
`Point` thành ba trường hợp: các điểm nằm trực tiếp trên trục `x` (điều này đúng
khi `y = 0`), trên trục `y` (`x = 0`), hoặc không nằm trên trục nào.

<Listing number="19-14" file-name="src/main.rs" caption="Phá vỡ và khớp các giá trị cụ thể trong một mẫu">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-14/src/main.rs:here}}
```

</Listing>

Arm đầu tiên sẽ khớp với bất kỳ điểm nào nằm trên trục `x` bằng cách chỉ định
rằng trường `y` khớp nếu giá trị của nó khớp với giá trị cụ thể `0`. Mẫu vẫn tạo
ra một biến `x` mà chúng ta có thể sử dụng trong mã cho arm này.

Tương tự, arm thứ hai khớp với bất kỳ điểm nào trên trục `y` bằng cách chỉ định
rằng trường `x` khớp nếu giá trị của nó là `0` và tạo ra một biến `y` cho giá
trị của trường `y`. Arm thứ ba không chỉ định bất kỳ giá trị cụ thể nào, vì vậy
nó khớp với bất kỳ `Point` nào khác và tạo ra các biến cho cả trường `x` và `y`.

Trong ví dụ này, giá trị `p` khớp với arm thứ hai bởi vì `x` chứa một `0`, vì
vậy đoạn mã này sẽ in ra `On the y axis at 7`.

Hãy nhớ rằng biểu thức `match` dừng kiểm tra các arm ngay khi nó tìm thấy mẫu
khớp đầu tiên, vì vậy mặc dù `Point { x: 0, y: 0 }` nằm trên cả trục `x` và trục
`y`, đoạn mã này sẽ chỉ in ra `On the x axis at 0`.

#### Phá Vỡ Enums

Chúng ta đã phá vỡ enums trong cuốn sách này (ví dụ, Listing 6-5), nhưng chúng
ta chưa thảo luận rõ ràng rằng mẫu để phá vỡ một enum tương ứng với cách dữ liệu
được lưu trữ trong enum được định nghĩa. Ví dụ, trong Listing 19-15 chúng ta sử
dụng enum `Message` từ Listing 6-2 và viết một `match` với các mẫu sẽ phá vỡ
từng giá trị bên trong.

<Listing number="19-15" file-name="src/main.rs" caption="Phá vỡ các biến thể enum chứa các loại giá trị khác nhau">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-15/src/main.rs}}
```

</Listing>

Đoạn mã này sẽ in ra `Change color to red 0, green 160, and blue 255`. Hãy thử
thay đổi giá trị của `msg` để xem mã từ các arm khác chạy.

Đối với các biến thể enum không có dữ liệu, như `Message::Quit`, chúng ta không
thể phá vỡ giá trị thêm nữa. Chúng ta chỉ có thể khớp với giá trị cụ thể
`Message::Quit`, và không có biến nào trong mẫu đó.

Đối với các biến thể enum giống struct, như `Message::Move`, chúng ta có thể sử
dụng một mẫu tương tự như mẫu chúng ta chỉ định để khớp với các struct. Sau tên
biến thể, chúng ta đặt dấu ngoặc nhọn và sau đó liệt kê các trường với các biến
để chúng ta tách các phần ra để sử dụng trong mã cho arm này. Ở đây chúng ta sử
dụng dạng ngắn gọn như chúng ta đã làm trong Listing 19-13.

Đối với các biến thể enum giống tuple, như `Message::Write` chứa một tuple với
một phần tử và `Message::ChangeColor` chứa một tuple với ba phần tử, mẫu tương
tự như mẫu chúng ta chỉ định để khớp với các tuple. Số lượng biến trong mẫu phải
khớp với số lượng phần tử trong biến thể mà chúng ta đang khớp.

#### Phá Vỡ Các Structs và Enums Lồng Nhau

Cho đến nay, các ví dụ của chúng ta đều khớp với các struct hoặc enum một cấp độ
sâu, nhưng việc khớp cũng có thể hoạt động trên các mục lồng nhau! Ví dụ, chúng
ta có thể tái cấu trúc mã trong Listing 19-15 để hỗ trợ các màu RGB và HSV trong
thông điệp `ChangeColor`, như được hiển thị trong Listing 19-16.

<Listing number="19-16" caption="Khớp với các enum lồng nhau">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-16/src/main.rs}}
```

</Listing>

Mẫu của arm đầu tiên trong biểu thức `match` khớp với một biến thể enum
`Message::ChangeColor` chứa một biến thể `Color::Rgb`; sau đó mẫu ràng buộc với
ba giá trị `i32` bên trong. Mẫu của arm thứ hai cũng khớp với một biến thể enum
`Message::ChangeColor`, nhưng enum bên trong khớp với `Color::Hsv` thay thế.
Chúng ta có thể chỉ định các điều kiện phức tạp này trong một biểu thức `match`,
ngay cả khi hai enum có liên quan.

#### Phá Vỡ Structs và Tuples

Chúng ta có thể kết hợp, khớp, và lồng các mẫu phá vỡ theo những cách phức tạp
hơn. Ví dụ sau đây cho thấy một phép phá vỡ phức tạp trong đó chúng ta lồng các
struct và tuple bên trong một tuple và phá vỡ tất cả các giá trị nguyên thủy:

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/no-listing-05-destructuring-structs-and-tuples/src/main.rs:here}}
```

Đoạn mã này cho phép chúng ta phá vỡ các loại phức tạp thành các thành phần của
chúng để chúng ta có thể sử dụng riêng biệt các giá trị mà chúng ta quan tâm.

Phá vỡ với các mẫu là một cách thuận tiện để sử dụng các phần của giá trị, chẳng
hạn như giá trị từ mỗi trường trong một struct, tách biệt với nhau.

### Bỏ Qua Giá Trị trong Mẫu

Đôi khi việc bỏ qua các giá trị trong một mẫu rất hữu ích, chẳng hạn như trong
arm cuối cùng của `match`, để có một catchall không thực sự làm gì nhưng tính
đến tất cả các giá trị còn lại có thể có. Có một vài cách để bỏ qua toàn bộ giá
trị hoặc các phần của giá trị trong một mẫu: sử dụng mẫu `_` (mà bạn đã thấy),
sử dụng mẫu `_` trong một mẫu khác, sử dụng một tên bắt đầu bằng dấu gạch dưới,
hoặc sử dụng `..` để bỏ qua các phần còn lại của một giá trị. Hãy khám phá cách
và lý do sử dụng từng mẫu này.

<!-- Old link, do not remove -->

<a id="ignoring-an-entire-value-with-_"></a>

#### Toàn Bộ Giá Trị với `_`

Chúng ta đã sử dụng dấu gạch dưới làm mẫu đại diện (wildcard) sẽ khớp với bất kỳ
giá trị nào nhưng không ràng buộc với giá trị. Điều này đặc biệt hữu ích làm arm
cuối cùng trong biểu thức `match`, nhưng chúng ta cũng có thể sử dụng nó trong
bất kỳ mẫu nào, bao gồm cả tham số hàm, như được hiển thị trong Listing 19-17.

<Listing number="19-17" file-name="src/main.rs" caption="Sử dụng `_` trong chữ ký hàm">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-17/src/main.rs}}
```

</Listing>

Đoạn mã này sẽ hoàn toàn bỏ qua giá trị `3` được truyền làm đối số đầu tiên và
sẽ in ra `This code only uses the y parameter: 4`.

Trong hầu hết các trường hợp khi bạn không còn cần một tham số hàm cụ thể nào
đó, bạn sẽ thay đổi chữ ký sao cho nó không bao gồm tham số không sử dụng. Việc
bỏ qua một tham số hàm có thể đặc biệt hữu ích trong trường hợp, ví dụ, bạn đang
triển khai một trait khi bạn cần một chữ ký loại nhất định nhưng thân hàm trong
triển khai của bạn không cần một trong các tham số. Khi đó bạn tránh được cảnh
báo của trình biên dịch về các tham số hàm không sử dụng, như bạn sẽ gặp nếu bạn
sử dụng một tên thay thế.

<a id="ignoring-parts-of-a-value-with-a-nested-_"></a>

#### Các Phần của Giá Trị với `_` Lồng Nhau

Chúng ta cũng có thể sử dụng `_` bên trong một mẫu khác để bỏ qua chỉ một phần
của một giá trị, ví dụ, khi chúng ta muốn kiểm tra chỉ một phần của một giá trị
nhưng không sử dụng các phần khác trong mã tương ứng mà chúng ta muốn chạy.
Listing 19-18 hiển thị mã chịu trách nhiệm quản lý giá trị của một cài đặt. Yêu
cầu kinh doanh là người dùng không được phép ghi đè một tùy chỉnh hiện có của
cài đặt, nhưng có thể bỏ cài đặt và cung cấp giá trị cho nó nếu hiện tại nó chưa
được đặt.

<Listing number="19-18" caption="Sử dụng dấu gạch dưới trong các mẫu khớp với các biến thể `Some` khi chúng ta không cần sử dụng giá trị bên trong `Some`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-18/src/main.rs:here}}
```

</Listing>

Đoạn mã này sẽ in ra `Can't overwrite an existing customized value` và sau đó là
`setting is Some(5)`. Trong arm match đầu tiên, chúng ta không cần phải khớp
hoặc sử dụng các giá trị bên trong cả hai biến thể `Some`, nhưng chúng ta cần
kiểm tra trường hợp khi `setting_value` và `new_setting_value` là biến thể
`Some`. Trong trường hợp đó, chúng ta in ra lý do không thay đổi
`setting_value`, và nó không bị thay đổi.

Trong tất cả các trường hợp khác (nếu `setting_value` hoặc `new_setting_value`
là `None`) được biểu thị bởi mẫu `_` trong arm thứ hai, chúng ta muốn cho phép
`new_setting_value` trở thành `setting_value`.

Chúng ta cũng có thể sử dụng dấu gạch dưới ở nhiều vị trí trong một mẫu để bỏ
qua các giá trị cụ thể. Listing 19-19 cho thấy một ví dụ về việc bỏ qua giá trị
thứ hai và thứ tư trong một tuple gồm năm mục.

<Listing number="19-19" caption="Bỏ qua nhiều phần của một tuple">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-19/src/main.rs:here}}
```

</Listing>

Đoạn mã này sẽ in ra `Some numbers: 2, 8, 32`, và các giá trị `4` và `16` sẽ bị
bỏ qua.

<!-- Old link, do not remove -->

<a id="ignoring-an-unused-variable-by-starting-its-name-with-_"></a>

#### Một Biến Không Sử Dụng bằng Cách Bắt Đầu Tên Của Nó với `_`

Nếu bạn tạo một biến nhưng không sử dụng nó ở bất kỳ đâu, Rust thường sẽ đưa ra
cảnh báo vì một biến không sử dụng có thể là một lỗi. Tuy nhiên, đôi khi việc
tạo một biến mà bạn sẽ không sử dụng ngay là hữu ích, chẳng hạn như khi bạn đang
tạo nguyên mẫu hoặc chỉ mới bắt đầu một dự án. Trong tình huống này, bạn có thể
bảo Rust không cảnh báo bạn về biến không sử dụng bằng cách bắt đầu tên của biến
bằng dấu gạch dưới. Trong Listing 19-20, chúng ta tạo hai biến không sử dụng,
nhưng khi chúng ta biên dịch mã này, chúng ta chỉ nhận được cảnh báo về một
trong số chúng.

<Listing number="19-20" file-name="src/main.rs" caption="Bắt đầu tên biến bằng dấu gạch dưới để tránh nhận cảnh báo về biến không sử dụng">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-20/src/main.rs}}
```

</Listing>

Ở đây, chúng ta nhận được cảnh báo về việc không sử dụng biến `y`, nhưng chúng
ta không nhận được cảnh báo về việc không sử dụng `_x`.

Lưu ý rằng có sự khác biệt tinh tế giữa việc chỉ sử dụng `_` và sử dụng một tên
bắt đầu bằng dấu gạch dưới. Cú pháp `_x` vẫn ràng buộc giá trị với biến, trong
khi `_` hoàn toàn không ràng buộc. Để hiển thị một trường hợp mà sự khác biệt
này quan trọng, Listing 19-21 sẽ cung cấp cho chúng ta một lỗi.

<Listing number="19-21" caption="Một biến không sử dụng bắt đầu bằng dấu gạch dưới vẫn ràng buộc giá trị, có thể lấy quyền sở hữu của giá trị">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-21/src/main.rs:here}}
```

</Listing>

Chúng ta sẽ nhận được lỗi vì giá trị `s` vẫn sẽ được chuyển vào `_s`, điều này
ngăn chúng ta sử dụng lại `s`. Tuy nhiên, việc sử dụng dấu gạch dưới một mình
không bao giờ ràng buộc với giá trị. Listing 19-22 sẽ biên dịch mà không có bất
kỳ lỗi nào vì `s` không bị chuyển vào `_`.

<Listing number="19-22" caption="Sử dụng dấu gạch dưới không ràng buộc giá trị">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-22/src/main.rs:here}}
```

</Listing>

Đoạn mã này hoạt động tốt vì chúng ta không bao giờ ràng buộc `s` với bất cứ thứ
gì; nó không bị chuyển đi.

<a id="ignoring-remaining-parts-of-a-value-with-"></a>

#### Các Phần Còn Lại của Giá Trị với `..`

Với các giá trị có nhiều phần, chúng ta có thể sử dụng cú pháp `..` để sử dụng
các phần cụ thể và bỏ qua phần còn lại, tránh cần liệt kê dấu gạch dưới cho mỗi
giá trị bị bỏ qua. Mẫu `..` bỏ qua bất kỳ phần nào của giá trị mà chúng ta chưa
khớp rõ ràng trong phần còn lại của mẫu. Trong Listing 19-23, chúng ta có một
struct `Point` lưu trữ một tọa độ trong không gian ba chiều. Trong biểu thức
`match`, chúng ta chỉ muốn hoạt động trên tọa độ `x` và bỏ qua các giá trị trong
các trường `y` và `z`.

<Listing number="19-23" caption="Bỏ qua tất cả các trường của `Point` trừ `x` bằng cách sử dụng `..`">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-23/src/main.rs:here}}
```

</Listing>

Chúng ta liệt kê giá trị `x` và sau đó chỉ bao gồm mẫu `..`. Điều này nhanh hơn
việc phải liệt kê `y: _` và `z: _`, đặc biệt là khi chúng ta làm việc với các
struct có nhiều trường trong tình huống chỉ một hoặc hai trường là liên quan.

Cú pháp `..` sẽ mở rộng tới nhiều giá trị khi cần thiết. Listing 19-24 cho thấy
cách sử dụng `..` với một tuple.

<Listing number="19-24" file-name="src/main.rs" caption="Chỉ khớp với giá trị đầu tiên và cuối cùng trong một tuple và bỏ qua tất cả các giá trị khác">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-24/src/main.rs}}
```

</Listing>

Trong đoạn mã này, giá trị đầu tiên và cuối cùng được khớp với `first` và
`last`. `..` sẽ khớp và bỏ qua mọi thứ ở giữa.

Tuy nhiên, việc sử dụng `..` phải rõ ràng không mơ hồ. Nếu không rõ ràng giá trị
nào dùng để khớp và giá trị nào nên bị bỏ qua, Rust sẽ đưa ra lỗi cho chúng ta.
Listing 19-25 hiển thị một ví dụ về việc sử dụng `..` một cách mơ hồ, vì vậy nó
sẽ không biên dịch.

<Listing number="19-25" file-name="src/main.rs" caption="Một nỗ lực sử dụng `..` theo cách mơ hồ">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-25/src/main.rs}}
```

</Listing>

Khi chúng ta biên dịch ví dụ này, chúng ta nhận được lỗi này:

```console
{{#include ../listings/ch19-patterns-and-matching/listing-19-25/output.txt}}
```

Không thể nào để Rust xác định được có bao nhiêu giá trị trong tuple cần bỏ qua
trước khi khớp một giá trị với `second` và sau đó còn bao nhiêu giá trị nữa cần
bỏ qua sau đó. Đoạn mã này có thể có nghĩa là chúng ta muốn bỏ qua `2`, ràng
buộc `second` với `4`, rồi bỏ qua `8`, `16` và `32`; hoặc rằng chúng ta muốn bỏ
qua `2` và `4`, ràng buộc `second` với `8`, rồi bỏ qua `16` và `32`; và vân vân.
Tên biến `second` không có ý nghĩa đặc biệt gì đối với Rust, vì vậy chúng ta
nhận được lỗi biên dịch vì việc sử dụng `..` ở hai nơi như thế này là mơ hồ.

### Điều Kiện Bổ Sung với Match Guards

_Match guard_ là một điều kiện `if` bổ sung, được chỉ định sau mẫu trong một arm
match, cũng phải khớp để arm đó được chọn. Match guard hữu ích để biểu thị các ý
tưởng phức tạp hơn những gì chỉ riêng một mẫu cho phép. Tuy nhiên, lưu ý rằng
chúng chỉ có sẵn trong biểu thức `match`, không có trong biểu thức `if let` hoặc
`while let`.

Điều kiện có thể sử dụng các biến được tạo trong mẫu. Listing 19-26 hiển thị một
`match` trong đó arm đầu tiên có mẫu `Some(x)` và cũng có một match guard là
`if x % 2 == 0` (sẽ là `true` nếu số là chẵn).

<Listing number="19-26" caption="Thêm một match guard vào một mẫu">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-26/src/main.rs:here}}
```

</Listing>

Ví dụ này sẽ in ra `The number 4 is even`. Khi `num` được so sánh với mẫu trong
arm đầu tiên, nó khớp vì `Some(4)` khớp với `Some(x)`. Sau đó match guard kiểm
tra xem phần dư của việc chia `x` cho 2 có bằng 0 không, và vì vậy, arm đầu tiên
được chọn.

Nếu `num` đã là `Some(5)` thay vào đó, match guard trong arm đầu tiên sẽ là
`false` vì phần dư của 5 chia cho 2 là 1, không bằng 0. Rust sau đó sẽ đi đến
arm thứ hai, match do arm thứ hai không có match guard và do đó khớp với bất kỳ
biến thể `Some` nào.

Không có cách nào để biểu thị điều kiện `if x % 2 == 0` trong một mẫu, vì vậy
match guard cho chúng ta khả năng biểu thị logic này. Nhược điểm của tính biểu
đạt bổ sung này là trình biên dịch không cố gắng kiểm tra tính đầy đủ khi các
biểu thức match guard có liên quan.

Trong Listing 19-11, chúng ta đã đề cập rằng chúng ta có thể sử dụng match guard
để giải quyết vấn đề che khuất (shadowing) mẫu. Hãy nhớ rằng chúng ta đã tạo một
biến mới bên trong mẫu trong biểu thức `match` thay vì sử dụng biến bên ngoài
`match`. Biến mới đó có nghĩa là chúng ta không thể kiểm tra giá trị của biến
bên ngoài. Listing 19-27 cho thấy cách chúng ta có thể sử dụng match guard để
sửa lỗi này.

<Listing number="19-27" file-name="src/main.rs" caption="Sử dụng match guard để kiểm tra sự bằng nhau với một biến bên ngoài">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-27/src/main.rs}}
```

</Listing>

Đoạn mã này sẽ in ra `Default case, x = Some(5)`. Mẫu trong arm match thứ hai
không giới thiệu một biến mới `y` sẽ che khuất biến `y` bên ngoài, có nghĩa là
chúng ta có thể sử dụng `y` bên ngoài trong match guard. Thay vì chỉ định mẫu là
`Some(y)`, sẽ che khuất `y` bên ngoài, chúng ta chỉ định `Some(n)`. Điều này tạo
ra một biến mới `n` không che khuất bất cứ thứ gì vì không có biến `n` nào bên
ngoài `match`.

Match guard `if n == y` không phải là một mẫu và do đó không giới thiệu các biến
mới. `y` này _là_ `y` bên ngoài chứ không phải một `y` mới che khuất nó, và
chúng ta có thể tìm kiếm một giá trị có cùng giá trị với `y` bên ngoài bằng cách
so sánh `n` với `y`.

Bạn cũng có thể sử dụng toán tử _hoặc_ `|` trong một match guard để chỉ định
nhiều mẫu; điều kiện match guard sẽ áp dụng cho tất cả các mẫu. Listing 19-28
hiển thị sự ưu tiên khi kết hợp một mẫu sử dụng `|` với một match guard. Phần
quan trọng của ví dụ này là match guard `if y` áp dụng cho `4`, `5`, _và_ `6`,
mặc dù nó có vẻ như `if y` chỉ áp dụng cho `6`.

<Listing number="19-28" caption="Kết hợp nhiều mẫu với match guard">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-28/src/main.rs:here}}
```

</Listing>

Điều kiện match nêu rõ rằng arm chỉ khớp nếu giá trị của `x` bằng `4`, `5`, hoặc
`6` _và_ nếu `y` là `true`. Khi đoạn mã này chạy, mẫu của arm đầu tiên khớp vì
`x` là `4`, nhưng match guard `if y` là `false`, vì vậy arm đầu tiên không được
chọn. Đoạn mã chuyển sang arm thứ hai, khớp, và chương trình này in ra `no`. Lý
do là điều kiện `if` áp dụng cho toàn bộ mẫu `4 | 5 | 6`, không chỉ cho giá trị
cuối cùng `6`. Nói cách khác, ưu tiên của một match guard trong mối quan hệ với
một mẫu hoạt động như thế này:

```text
(4 | 5 | 6) if y => ...
```

chứ không phải như thế này:

```text
4 | 5 | (6 if y) => ...
```

Sau khi chạy đoạn mã, hành vi ưu tiên là rõ ràng: nếu match guard chỉ được áp
dụng cho giá trị cuối cùng trong danh sách các giá trị được chỉ định bằng toán
tử `|`, thì arm đã khớp và chương trình đã in ra `yes`.

### Ràng Buộc `@`

Toán tử _at_ `@` cho phép chúng ta tạo một biến giữ một giá trị đồng thời kiểm
tra giá trị đó để khớp với một mẫu. Trong Listing 19-29, chúng ta muốn kiểm tra
rằng trường `id` của `Message::Hello` nằm trong phạm vi `3..=7`. Chúng ta cũng
muốn ràng buộc giá trị với biến `id_variable` để chúng ta có thể sử dụng nó
trong mã liên kết với arm. Chúng ta có thể đặt tên biến này là `id`, giống như
tên trường, nhưng trong ví dụ này chúng ta sẽ sử dụng một tên khác.

<Listing number="19-29" caption="Sử dụng `@` để ràng buộc với một giá trị trong một mẫu trong khi cũng kiểm tra nó">

```rust
{{#rustdoc_include ../listings/ch19-patterns-and-matching/listing-19-29/src/main.rs:here}}
```

</Listing>

Ví dụ này sẽ in ra `Found an id in range: 5`. Bằng cách chỉ định `id_variable @`
trước phạm vi `3..=7`, chúng ta đang bắt bất kỳ giá trị nào khớp với phạm vi
đồng thời kiểm tra rằng giá trị đó khớp với mẫu phạm vi.

Trong arm thứ hai, nơi chúng ta chỉ có một phạm vi được chỉ định trong mẫu, mã
liên kết với arm không có biến chứa giá trị thực tế của trường `id`. Giá trị của
trường `id` có thể là 10, 11, hoặc 12, nhưng mã đi kèm với mẫu đó không biết nó
là gì. Mã mẫu không thể sử dụng giá trị từ trường `id`, vì chúng ta chưa lưu giá
trị `id` trong một biến.

Trong arm cuối cùng, nơi chúng ta đã chỉ định một biến mà không có phạm vi,
chúng ta có giá trị có sẵn để sử dụng trong mã của arm trong một biến có tên là
`id`. Lý do là chúng ta đã sử dụng cú pháp rút gọn trường struct. Nhưng chúng ta
chưa áp dụng bất kỳ kiểm tra nào cho giá trị trong trường `id` trong arm này,
như chúng ta đã làm với hai arm đầu tiên: bất kỳ giá trị nào cũng sẽ khớp với
mẫu này.

Sử dụng `@` cho phép chúng ta kiểm tra một giá trị và lưu nó trong một biến
trong một mẫu.

## Tóm Tắt

Các mẫu của Rust rất hữu ích trong việc phân biệt giữa các loại dữ liệu khác
nhau. Khi được sử dụng trong biểu thức `match`, Rust đảm bảo rằng các mẫu của
bạn bao gồm mọi giá trị có thể có, nếu không chương trình của bạn sẽ không biên
dịch. Các mẫu trong câu lệnh `let` và tham số hàm làm cho các cấu trúc đó hữu
ích hơn, cho phép phá vỡ các giá trị thành các phần nhỏ hơn đồng thời gán các
phần đó cho các biến. Chúng ta có thể tạo các mẫu đơn giản hoặc phức tạp để phù
hợp với nhu cầu của chúng ta.

Tiếp theo, ở chương gần cuối của cuốn sách, chúng ta sẽ xem xét một số khía cạnh
nâng cao của nhiều tính năng của Rust.
