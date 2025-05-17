## Xác thực tham chiếu với Lifetimes

Lifetimes (thời gian sống) là một loại generic khác mà chúng ta đã sử dụng. Thay
vì đảm bảo rằng một kiểu có hành vi chúng ta mong muốn, lifetimes đảm bảo rằng
các tham chiếu hợp lệ miễn là chúng ta cần chúng.

Một chi tiết chúng ta chưa thảo luận trong phần ["Tham chiếu và
Mượn"][references-and-borrowing]<!-- ignore --> ở Chương 4 là mỗi tham chiếu
trong Rust có một _lifetime_, là phạm vi mà tham chiếu đó có hiệu lực. Hầu hết
thời gian, lifetimes được ngầm định và suy luận, giống như hầu hết thời gian,
các kiểu được suy luận. Chúng ta chỉ bị yêu cầu chú thích kiểu khi nhiều kiểu có
thể áp dụng. Tương tự, chúng ta phải chú thích lifetimes khi lifetimes của các
tham chiếu có thể liên quan theo một số cách khác nhau. Rust yêu cầu chúng ta
chú thích các mối quan hệ bằng các tham số lifetime generic để đảm bảo các tham
chiếu thực tế được sử dụng trong thời gian chạy chắc chắn sẽ hợp lệ.

Việc chú thích lifetimes thậm chí không phải là một khái niệm mà hầu hết các
ngôn ngữ lập trình khác có, vì vậy điều này sẽ cảm thấy không quen thuộc. Mặc dù
chúng ta sẽ không bao quát lifetimes một cách đầy đủ trong chương này, chúng ta
sẽ thảo luận về các cách phổ biến mà bạn có thể gặp cú pháp lifetime để bạn có
thể làm quen với khái niệm này.

### Ngăn chặn Dangling References (Tham chiếu treo) với Lifetimes

Mục đích chính của lifetimes là ngăn chặn _dangling references_, gây ra cho
chương trình tham chiếu đến dữ liệu khác với dữ liệu mà nó dự định tham chiếu.
Hãy xem xét chương trình trong Listing 10-16, có một phạm vi ngoài và một phạm
vi trong.

<Listing number="10-16" caption="Một nỗ lực sử dụng một tham chiếu có giá trị đã ra khỏi phạm vi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/src/main.rs}}
```

</Listing>

> Lưu ý: Các ví dụ trong Listings 10-16, 10-17, và 10-23 khai báo các biến mà
> không cung cấp giá trị ban đầu, vì vậy tên biến tồn tại trong phạm vi ngoài.
> Thoạt nhìn, điều này có thể xuất hiện mâu thuẫn với việc Rust không có giá trị
> null. Tuy nhiên, nếu chúng ta cố gắng sử dụng một biến trước khi gán giá trị
> cho nó, chúng ta sẽ nhận được lỗi biên dịch, điều này cho thấy rằng Rust thực
> sự không cho phép giá trị null.

Phạm vi ngoài khai báo một biến tên `r` không có giá trị ban đầu, và phạm vi
trong khai báo một biến tên `x` với giá trị ban đầu là `5`. Bên trong phạm vi
trong, chúng ta cố gắng đặt giá trị của `r` là một tham chiếu đến `x`. Sau đó
phạm vi trong kết thúc, và chúng ta cố gắng in giá trị trong `r`. Mã này sẽ
không biên dịch vì giá trị mà `r` đang tham chiếu đã ra khỏi phạm vi trước khi
chúng ta cố gắng sử dụng nó. Đây là thông báo lỗi:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-16/output.txt}}
```

Thông báo lỗi nói rằng biến `x` "không sống đủ lâu." Lý do là `x` sẽ ra khỏi
phạm vi khi phạm vi trong kết thúc ở dòng 7. Nhưng `r` vẫn hợp lệ cho phạm vi
ngoài; vì phạm vi của nó lớn hơn, chúng ta nói rằng nó "sống lâu hơn." Nếu Rust
cho phép mã này hoạt động, `r` sẽ tham chiếu đến bộ nhớ đã bị giải phóng khi `x`
ra khỏi phạm vi, và bất cứ điều gì chúng ta cố gắng làm với `r` sẽ không hoạt
động đúng. Vậy làm thế nào Rust xác định rằng mã này không hợp lệ? Nó sử dụng
một borrow checker.

### Borrow Checker (Kiểm tra Mượn)

Trình biên dịch Rust có một _borrow checker_ so sánh phạm vi để xác định liệu
tất cả các mượn có hợp lệ hay không. Listing 10-17 hiển thị cùng một mã như
Listing 10-16 nhưng có chú thích hiển thị lifetimes của các biến.

<Listing number="10-17" caption="Chú thích lifetimes của `r` và `x`, lần lượt được đặt tên là `'a` và `'b`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-17/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đã chú thích lifetime của `r` với `'a` và lifetime của `x` với
`'b`. Như bạn có thể thấy, khối `'b` bên trong nhỏ hơn nhiều so với khối
lifetime `'a` bên ngoài. Tại thời điểm biên dịch, Rust so sánh kích thước của
hai lifetimes và thấy rằng `r` có lifetime là `'a` nhưng nó tham chiếu đến bộ
nhớ với lifetime là `'b`. Chương trình bị từ chối vì `'b` ngắn hơn `'a`: chủ thể
của tham chiếu không sống lâu bằng tham chiếu.

Listing 10-18 sửa mã để nó không có dangling reference và nó biên dịch mà không
có bất kỳ lỗi nào.

<Listing number="10-18" caption="Một tham chiếu hợp lệ vì dữ liệu có lifetime dài hơn tham chiếu">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-18/src/main.rs}}
```

</Listing>

Ở đây, `x` có lifetime `'b`, trong trường hợp này lớn hơn `'a`. Điều này có
nghĩa là `r` có thể tham chiếu `x` vì Rust biết rằng tham chiếu trong `r` sẽ
luôn hợp lệ trong khi `x` hợp lệ.

Bây giờ bạn đã biết lifetimes của tham chiếu ở đâu và cách Rust phân tích
lifetimes để đảm bảo tham chiếu sẽ luôn hợp lệ, hãy khám phá generic lifetimes
của tham số và giá trị trả về trong bối cảnh của các hàm.

### Generic Lifetimes trong Hàm

Chúng ta sẽ viết một hàm trả về slice chuỗi dài hơn trong hai slice chuỗi. Hàm
này sẽ nhận hai slice chuỗi và trả về một slice chuỗi. Sau khi chúng ta đã triển
khai hàm `longest`, mã trong Listing 10-19 sẽ in ra
`The longest string is abcd`.

<Listing number="10-19" file-name="src/main.rs" caption="Một hàm `main` gọi hàm `longest` để tìm slice chuỗi dài hơn trong hai slice chuỗi">

```rust,ignore
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-19/src/main.rs}}
```

</Listing>

Lưu ý rằng chúng ta muốn hàm nhận slice chuỗi, là tham chiếu, thay vì chuỗi, vì
chúng ta không muốn hàm `longest` lấy quyền sở hữu các tham số của nó. Tham khảo
["String Slices as Parameters"][string-slices-as-parameters]<!-- ignore -->
trong Chương 4 để biết thêm thảo luận về lý do tại sao các tham số chúng ta sử
dụng trong Listing 10-19 là những tham số mà chúng ta muốn.

Nếu chúng ta cố gắng triển khai hàm `longest` như trong Listing 10-20, nó sẽ
không biên dịch.

<Listing number="10-20" file-name="src/main.rs" caption="Một triển khai của hàm `longest` trả về slice chuỗi dài hơn trong hai slice chuỗi nhưng chưa biên dịch">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/src/main.rs:here}}
```

</Listing>

Thay vào đó, chúng ta nhận được lỗi sau nói về lifetimes:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-20/output.txt}}
```

Văn bản trợ giúp tiết lộ rằng kiểu trả về cần một tham số lifetime generic vì
Rust không thể biết liệu tham chiếu được trả về tham chiếu đến `x` hay `y`. Thực
tế, chúng ta cũng không biết, vì khối `if` trong thân của hàm này trả về một
tham chiếu đến `x` và khối `else` trả về một tham chiếu đến `y`!

Khi chúng ta định nghĩa hàm này, chúng ta không biết các giá trị cụ thể sẽ được
truyền vào hàm này, vì vậy chúng ta không biết liệu trường hợp `if` hay trường
hợp `else` sẽ được thực thi. Chúng ta cũng không biết lifetimes cụ thể của các
tham chiếu sẽ được truyền vào, vì vậy chúng ta không thể xem xét phạm vi như
chúng ta đã làm trong Listing 10-17 và 10-18 để xác định liệu tham chiếu chúng
ta trả về sẽ luôn hợp lệ hay không. Borrow checker cũng không thể xác định điều
này, bởi vì nó không biết lifetimes của `x` và `y` liên quan đến lifetime của
giá trị trả về như thế nào. Để sửa lỗi này, chúng ta sẽ thêm tham số lifetime
generic xác định mối quan hệ giữa các tham chiếu để borrow checker có thể thực
hiện phân tích của nó.

### Cú pháp Chú thích Lifetime

Chú thích lifetime không thay đổi thời gian tồn tại của bất kỳ tham chiếu nào.
Thay vào đó, chúng mô tả mối quan hệ của lifetimes của nhiều tham chiếu với nhau
mà không ảnh hưởng đến lifetimes. Giống như các hàm có thể chấp nhận bất kỳ kiểu
nào khi chữ ký chỉ định một tham số kiểu generic, các hàm có thể chấp nhận tham
chiếu với bất kỳ lifetime nào bằng cách chỉ định một tham số lifetime generic.

Chú thích lifetime có cú pháp hơi khác thường: tên của tham số lifetime phải bắt
đầu bằng dấu nháy đơn (`'`) và thường đều viết thường và rất ngắn, giống như các
kiểu generic. Hầu hết mọi người sử dụng tên `'a` cho chú thích lifetime đầu
tiên. Chúng ta đặt chú thích tham số lifetime sau dấu `&` của tham chiếu, sử
dụng khoảng cách để phân tách chú thích khỏi kiểu của tham chiếu.

Đây là một số ví dụ: một tham chiếu đến `i32` không có tham số lifetime, một
tham chiếu đến `i32` có tham số lifetime tên `'a`, và một tham chiếu có thể thay
đổi đến `i32` cũng có lifetime `'a`.

```rust,ignore
&i32        // một tham chiếu
&'a i32     // một tham chiếu với lifetime rõ ràng
&'a mut i32 // một tham chiếu có thể thay đổi với lifetime rõ ràng
```

Một chú thích lifetime tự nó không có nhiều ý nghĩa vì các chú thích được dùng
để nói với Rust cách các tham số lifetime generic của nhiều tham chiếu liên quan
đến nhau. Hãy xem xét cách chú thích lifetime liên quan đến nhau trong bối cảnh
của hàm `longest`.

### Chú thích Lifetime trong Chữ ký Hàm

Để sử dụng chú thích lifetime trong chữ ký hàm, chúng ta cần khai báo các tham
số _lifetime_ generic bên trong dấu ngoặc nhọn giữa tên hàm và danh sách tham
số, giống như chúng ta đã làm với tham số _kiểu_ generic.

Chúng ta muốn chữ ký thể hiện ràng buộc sau: tham chiếu trả về sẽ hợp lệ miễn là
cả hai tham số đều hợp lệ. Đây là mối quan hệ giữa lifetimes của tham số và giá
trị trả về. Chúng ta sẽ đặt tên lifetime là `'a` và sau đó thêm nó vào mỗi tham
chiếu, như trong Listing 10-21.

<Listing number="10-21" file-name="src/main.rs" caption="Định nghĩa hàm `longest` chỉ định rằng tất cả các tham chiếu trong chữ ký phải có cùng lifetime `'a`">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-21/src/main.rs:here}}
```

</Listing>

Mã này nên biên dịch và tạo ra kết quả chúng ta muốn khi chúng ta sử dụng nó với
hàm `main` trong Listing 10-19.

Chữ ký hàm bây giờ nói với Rust rằng đối với một lifetime `'a` nào đó, hàm nhận
hai tham số, cả hai đều là slice chuỗi sống ít nhất là lifetime `'a`. Chữ ký hàm
cũng nói với Rust rằng slice chuỗi được trả về từ hàm sẽ sống ít nhất là
lifetime `'a`. Trong thực tế, điều này có nghĩa là lifetime của tham chiếu được
trả về bởi hàm `longest` giống như lifetime nhỏ hơn của các giá trị được tham
chiếu bởi các đối số hàm. Đây là những mối quan hệ mà chúng ta muốn Rust sử dụng
khi phân tích mã này.

Hãy nhớ rằng, khi chúng ta chỉ định tham số lifetime trong chữ ký hàm này, chúng
ta không thay đổi lifetimes của bất kỳ giá trị nào được truyền vào hoặc trả về.
Thay vào đó, chúng ta đang chỉ định rằng borrow checker nên từ chối bất kỳ giá
trị nào không tuân theo các ràng buộc này. Lưu ý rằng hàm `longest` không cần
biết chính xác `x` và `y` sẽ sống bao lâu, chỉ cần một phạm vi có thể được thay
thế cho `'a` sẽ thỏa mãn chữ ký này.

Khi chú thích lifetimes trong hàm, các chú thích được đặt trong chữ ký hàm,
không phải trong thân hàm. Các chú thích lifetime trở thành một phần của hợp
đồng của hàm, giống như các kiểu trong chữ ký. Việc có chữ ký hàm chứa hợp đồng
lifetime có nghĩa là việc phân tích mà trình biên dịch Rust thực hiện có thể đơn
giản hơn. Nếu có vấn đề với cách một hàm được chú thích hoặc cách nó được gọi,
lỗi của trình biên dịch có thể chỉ ra phần mã của chúng ta và các ràng buộc
chính xác hơn. Nếu, thay vào đó, trình biên dịch Rust đưa ra nhiều suy luận hơn
về những gì chúng ta dự định mối quan hệ của lifetimes sẽ như thế nào, trình
biên dịch có thể chỉ có khả năng chỉ ra việc sử dụng mã của chúng ta nhiều bước
cách xa nguyên nhân của vấn đề.

Khi chúng ta truyền các tham chiếu cụ thể cho `longest`, lifetime cụ thể được
thay thế cho `'a` là phần của phạm vi của `x` trùng với phạm vi của `y`. Nói
cách khác, lifetime generic `'a` sẽ nhận lifetime cụ thể bằng với lifetime nhỏ
hơn của `x` và `y`. Bởi vì chúng ta đã chú thích tham chiếu được trả về với cùng
tham số lifetime `'a`, tham chiếu được trả về cũng sẽ hợp lệ cho độ dài của
lifetime nhỏ hơn của `x` và `y`.

Hãy xem cách các chú thích lifetime hạn chế hàm `longest` bằng cách truyền vào
các tham chiếu có lifetimes cụ thể khác nhau. Listing 10-22 là một ví dụ đơn
giản.

<Listing number="10-22" file-name="src/main.rs" caption="Sử dụng hàm `longest` với các tham chiếu đến giá trị `String` có lifetimes cụ thể khác nhau">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-22/src/main.rs:here}}
```

</Listing>

Trong ví dụ này, `string1` hợp lệ cho đến khi kết thúc phạm vi ngoài, `string2`
hợp lệ cho đến khi kết thúc phạm vi trong, và `result` tham chiếu đến một thứ
hợp lệ cho đến khi kết thúc phạm vi trong. Chạy mã này và bạn sẽ thấy rằng
borrow checker chấp thuận; nó sẽ biên dịch và in
`The longest string is long string is long`.

Tiếp theo, hãy thử một ví dụ cho thấy rằng lifetime của tham chiếu trong
`result` phải là lifetime nhỏ hơn của hai đối số. Chúng ta sẽ di chuyển khai báo
biến `result` ra ngoài phạm vi trong nhưng để việc gán giá trị cho biến `result`
bên trong phạm vi với `string2`. Sau đó, chúng ta sẽ di chuyển `println!` sử
dụng `result` ra ngoài phạm vi trong, sau khi phạm vi trong đã kết thúc. Mã
trong Listing 10-23 sẽ không biên dịch.

<Listing number="10-23" file-name="src/main.rs" caption="Cố gắng sử dụng `result` sau khi `string2` đã ra khỏi phạm vi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/src/main.rs:here}}
```

</Listing>

Khi chúng ta cố gắng biên dịch mã này, chúng ta nhận được lỗi này:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-23/output.txt}}
```

Lỗi cho thấy rằng để `result` hợp lệ cho câu lệnh `println!`, `string2` cần hợp
lệ cho đến khi kết thúc phạm vi ngoài. Rust biết điều này vì chúng ta đã chú
thích lifetimes của tham số hàm và giá trị trả về bằng cùng một tham số lifetime
`'a`.

Với tư cách là con người, chúng ta có thể nhìn vào mã này và thấy rằng `string1`
dài hơn `string2`, và do đó `result` sẽ chứa một tham chiếu đến `string1`. Bởi
vì `string1` chưa ra khỏi phạm vi, một tham chiếu đến `string1` vẫn sẽ hợp lệ
cho câu lệnh `println!`. Tuy nhiên, trình biên dịch không thể thấy rằng tham
chiếu hợp lệ trong trường hợp này. Chúng ta đã nói với Rust rằng lifetime của
tham chiếu được trả về bởi hàm `longest` giống như lifetime nhỏ hơn của các tham
chiếu được truyền vào. Do đó, borrow checker không cho phép mã trong Listing
10-23 vì nó có thể có một tham chiếu không hợp lệ.

Hãy thử thiết kế thêm các thử nghiệm thay đổi giá trị và lifetimes của các tham
chiếu được truyền vào hàm `longest` và cách tham chiếu trả về được sử dụng. Đưa
ra các giả thuyết về việc các thử nghiệm của bạn sẽ vượt qua borrow checker
trước khi bạn biên dịch; sau đó kiểm tra xem bạn có đúng không!

### Suy nghĩ theo Lifetimes

Cách bạn cần chỉ định tham số lifetime phụ thuộc vào những gì hàm của bạn đang
làm. Ví dụ, nếu chúng ta thay đổi triển khai của hàm `longest` để luôn trả về
tham số đầu tiên thay vì slice chuỗi dài nhất, chúng ta sẽ không cần chỉ định
một lifetime cho tham số `y`. Mã sau sẽ biên dịch:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-08-only-one-reference-with-lifetime/src/main.rs:here}}
```

</Listing>

Chúng ta đã chỉ định một tham số lifetime `'a` cho tham số `x` và kiểu trả về,
nhưng không cho tham số `y`, bởi vì lifetime của `y` không có bất kỳ mối quan hệ
nào với lifetime của `x` hoặc giá trị trả về.

Khi trả về một tham chiếu từ một hàm, tham số lifetime cho kiểu trả về cần phải
khớp với tham số lifetime cho một trong các tham số. Nếu tham chiếu được trả về
_không_ tham chiếu đến một trong các tham số, thì nó phải tham chiếu đến một giá
trị được tạo bên trong hàm này. Tuy nhiên, đây sẽ là một dangling reference vì
giá trị sẽ ra khỏi phạm vi khi kết thúc hàm. Hãy xem xét nỗ lực triển khai hàm
`longest` sau đây mà không biên dịch:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/src/main.rs:here}}
```

</Listing>

Ở đây, mặc dù chúng ta đã chỉ định một tham số lifetime `'a` cho kiểu trả về,
nhưng triển khai này sẽ không biên dịch vì lifetime của giá trị trả về không
liên quan gì đến lifetime của các tham số. Đây là thông báo lỗi chúng ta nhận
được:

```console
{{#include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-09-unrelated-lifetime/output.txt}}
```

Vấn đề là `result` ra khỏi phạm vi và bị dọn dẹp vào cuối hàm `longest`. Chúng
ta cũng đang cố gắng trả về một tham chiếu đến `result` từ hàm. Không có cách
nào chúng ta có thể chỉ định tham số lifetime để thay đổi dangling reference, và
Rust sẽ không cho phép chúng ta tạo một dangling reference. Trong trường hợp
này, giải pháp tốt nhất là trả về một kiểu dữ liệu sở hữu thay vì một tham chiếu
để hàm gọi sau đó có trách nhiệm dọn dẹp giá trị.

Cuối cùng, cú pháp lifetime là về việc kết nối lifetimes của các tham số và giá
trị trả về của các hàm. Một khi chúng được kết nối, Rust có đủ thông tin để cho
phép các hoạt động an toàn bộ nhớ và ngăn chặn các hoạt động có thể tạo ra con
trỏ treo hoặc vi phạm an toàn bộ nhớ.

### Chú thích Lifetime trong Định nghĩa Struct

Cho đến nay, các struct chúng ta đã định nghĩa đều chứa các kiểu sở hữu. Chúng
ta có thể định nghĩa các struct để giữ các tham chiếu, nhưng trong trường hợp đó
chúng ta cần thêm một chú thích lifetime trên mỗi tham chiếu trong định nghĩa
struct. Listing 10-24 có một struct được gọi là `ImportantExcerpt` giữ một slice
chuỗi.

<Listing number="10-24" file-name="src/main.rs" caption="Một struct giữ một tham chiếu, yêu cầu chú thích lifetime">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-24/src/main.rs}}
```

</Listing>

Struct này có trường duy nhất `part` giữ một slice chuỗi, là một tham chiếu.
Cũng giống như với các kiểu dữ liệu generic, chúng ta khai báo tên của tham số
lifetime generic bên trong dấu ngoặc nhọn sau tên của struct để chúng ta có thể
sử dụng tham số lifetime trong thân định nghĩa struct. Chú thích này có nghĩa là
một thể hiện của `ImportantExcerpt` không thể tồn tại lâu hơn tham chiếu mà nó
giữ trong trường `part`.

Hàm `main` ở đây tạo một thể hiện của struct `ImportantExcerpt` giữ một tham
chiếu đến câu đầu tiên của `String` được sở hữu bởi biến `novel`. Dữ liệu trong
`novel` tồn tại trước khi thể hiện `ImportantExcerpt` được tạo. Ngoài ra,
`novel` không ra khỏi phạm vi cho đến sau khi thể hiện `ImportantExcerpt` ra
khỏi phạm vi, vì vậy tham chiếu trong thể hiện `ImportantExcerpt` hợp lệ.

### Lifetime Elision (Loại bỏ Lifetime)

Bạn đã học rằng mỗi tham chiếu có một lifetime và bạn cần chỉ định tham số
lifetime cho các hàm hoặc struct sử dụng tham chiếu. Tuy nhiên, chúng ta đã có
một hàm trong Listing 4-9, được hiển thị lại trong Listing 10-25, đã biên dịch
mà không có chú thích lifetime.

<Listing number="10-25" file-name="src/lib.rs" caption="Hàm chúng ta đã định nghĩa trong Listing 4-9 đã biên dịch mà không có chú thích lifetime, mặc dù tham số và kiểu trả về đều là tham chiếu">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-25/src/main.rs:here}}
```

</Listing>

Lý do hàm này biên dịch mà không có chú thích lifetime là lịch sử: trong các
phiên bản sớm (trước 1.0) của Rust, mã này sẽ không biên dịch vì mỗi tham chiếu
cần một lifetime rõ ràng. Vào thời điểm đó, chữ ký hàm sẽ được viết như sau:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Sau khi viết nhiều mã Rust, nhóm Rust thấy rằng các lập trình viên Rust đang
nhập các chú thích lifetime giống nhau lặp đi lặp lại trong các tình huống cụ
thể. Các tình huống này có thể dự đoán được và tuân theo một số mẫu xác định.
Các nhà phát triển đã lập trình các mẫu này vào mã của trình biên dịch để borrow
checker có thể suy ra lifetimes trong các tình huống này và không cần các chú
thích rõ ràng.

Phần lịch sử Rust này có liên quan vì có khả năng nhiều mẫu xác định hơn sẽ xuất
hiện và được thêm vào trình biên dịch. Trong tương lai, thậm chí có thể cần ít
chú thích lifetime hơn.

Các mẫu được lập trình vào phân tích tham chiếu của Rust được gọi là _luật loại
bỏ lifetime_. Đây không phải là luật cho lập trình viên tuân theo; chúng là một
tập hợp các trường hợp cụ thể mà trình biên dịch sẽ xem xét, và nếu mã của bạn
phù hợp với các trường hợp này, bạn không cần viết lifetimes một cách rõ ràng.

Các luật loại bỏ không cung cấp suy luận hoàn toàn. Nếu vẫn còn sự không rõ ràng
về lifetimes mà các tham chiếu có sau khi Rust áp dụng các luật, thì trình biên
dịch sẽ không đoán xem lifetime của các tham chiếu còn lại là gì. Thay vì đoán,
trình biên dịch sẽ đưa ra một lỗi mà bạn có thể giải quyết bằng cách thêm các
chú thích lifetime.

Lifetimes trên tham số hàm hoặc phương thức được gọi là _lifetimes đầu vào_, và
lifetimes trên giá trị trả về được gọi là _lifetimes đầu ra_.

Trình biên dịch sử dụng ba luật để xác định lifetimes của các tham chiếu khi
không có chú thích rõ ràng. Luật đầu tiên áp dụng cho lifetimes đầu vào, và luật
thứ hai và thứ ba áp dụng cho lifetimes đầu ra. Nếu trình biên dịch đến cuối ba
luật và vẫn còn tham chiếu mà nó không thể xác định lifetimes, trình biên dịch
sẽ dừng lại với một lỗi. Các luật này áp dụng cho các định nghĩa `fn` cũng như
các khối `impl`.

Luật đầu tiên là trình biên dịch gán một tham số lifetime cho mỗi tham số là một
tham chiếu. Nói cách khác, một hàm với một tham số nhận một tham số lifetime:
`fn foo<'a>(x: &'a i32)`; một hàm với hai tham số nhận hai tham số lifetime
riêng biệt: `fn foo<'a, 'b>(x: &'a i32, y: &'b i32)`; và vân vân.

Luật thứ hai là, nếu chỉ có một tham số lifetime đầu vào, thì lifetime đó được
gán cho tất cả các tham số lifetime đầu ra: `fn foo<'a>(x: &'a i32) -> &'a i32`.

Luật thứ ba là, nếu có nhiều tham số lifetime đầu vào, nhưng một trong số đó là
`&self` hoặc `&mut self` vì đây là một phương thức, thì lifetime của `self` được
gán cho tất cả các tham số lifetime đầu ra. Luật thứ ba này làm cho các phương
thức dễ đọc và viết hơn nhiều vì cần ít ký hiệu hơn.

Hãy giả vờ chúng ta là trình biên dịch. Chúng ta sẽ áp dụng các luật này để tìm
ra lifetimes của các tham chiếu trong chữ ký của hàm `first_word` trong Listing
10-25. Chữ ký bắt đầu mà không có bất kỳ lifetime nào được liên kết với các tham
chiếu:

```rust,ignore
fn first_word(s: &str) -> &str {
```

Sau đó trình biên dịch áp dụng luật đầu tiên, chỉ định rằng mỗi tham số nhận
lifetime của riêng nó. Chúng ta sẽ gọi nó là `'a` như thường lệ, vì vậy bây giờ
chữ ký là như sau:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &str {
```

Luật thứ hai áp dụng vì có chính xác một lifetime đầu vào. Luật thứ hai chỉ định
rằng lifetime của tham số đầu vào duy nhất được gán cho lifetime đầu ra, vì vậy
chữ ký bây giờ là như sau:

```rust,ignore
fn first_word<'a>(s: &'a str) -> &'a str {
```

Bây giờ tất cả các tham chiếu trong chữ ký hàm này có lifetimes, và trình biên
dịch có thể tiếp tục phân tích mà không cần lập trình viên chú thích lifetimes
trong chữ ký hàm này.

Hãy xem một ví dụ khác, lần này sử dụng hàm `longest` đã không có tham số
lifetime khi chúng ta bắt đầu làm việc với nó trong Listing 10-20:

```rust,ignore
fn longest(x: &str, y: &str) -> &str {
```

Hãy áp dụng luật đầu tiên: mỗi tham số nhận lifetime của riêng nó. Lần này chúng
ta có hai tham số thay vì một, vì vậy chúng ta có hai lifetimes:

```rust,ignore
fn longest<'a, 'b>(x: &'a str, y: &'b str) -> &str {
```

Bạn có thể thấy rằng luật thứ hai không áp dụng vì có nhiều hơn một lifetime đầu
vào. Luật thứ ba cũng không áp dụng, vì `longest` là một hàm chứ không phải là
một phương thức, vì vậy không có tham số nào là `self`. Sau khi làm việc qua tất
cả ba luật, chúng ta vẫn chưa tìm ra lifetime của kiểu trả về là gì. Đây là lý
do tại sao chúng ta nhận được lỗi khi cố gắng biên dịch mã trong Listing 10-20:
trình biên dịch đã làm việc thông qua các luật loại bỏ lifetime nhưng vẫn không
thể tìm ra tất cả lifetimes của các tham chiếu trong chữ ký.

Bởi vì luật thứ ba thực sự chỉ áp dụng trong chữ ký phương thức, chúng ta sẽ xem
xét lifetimes trong ngữ cảnh đó tiếp theo để thấy tại sao luật thứ ba có nghĩa
là chúng ta không phải chú thích lifetimes trong chữ ký phương thức thường
xuyên.

### Chú thích Lifetime trong Định nghĩa Phương thức

Khi chúng ta triển khai các phương thức trên một struct với lifetimes, chúng ta
sử dụng cùng cú pháp như của tham số kiểu generic, như được hiển thị trong
Listing 10-11. Nơi chúng ta khai báo và sử dụng các tham số lifetime phụ thuộc
vào việc chúng có liên quan đến các trường của struct hay các tham số phương
thức và giá trị trả về.

Tên lifetime cho các trường struct luôn cần được khai báo sau từ khóa `impl` và
sau đó được sử dụng sau tên của struct vì những lifetimes đó là một phần của
kiểu của struct.

Trong chữ ký phương thức bên trong khối `impl`, các tham chiếu có thể bị ràng
buộc với lifetime của các tham chiếu trong các trường của struct, hoặc chúng có
thể độc lập. Ngoài ra, các luật loại bỏ lifetime thường làm cho chú thích
lifetime không cần thiết trong chữ ký phương thức. Hãy xem một số ví dụ sử dụng
struct có tên `ImportantExcerpt` mà chúng ta đã định nghĩa trong Listing 10-24.

Đầu tiên, chúng ta sẽ sử dụng một phương thức có tên `level` có tham số duy nhất
là tham chiếu đến `self` và giá trị trả về là `i32`, không phải là tham chiếu
đến bất kỳ thứ gì:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:1st}}
```

Khai báo tham số lifetime sau `impl` và việc sử dụng nó sau tên kiểu là cần
thiết, nhưng chúng ta không bắt buộc phải chú thích lifetime của tham chiếu đến
`self` vì luật loại bỏ đầu tiên.

Đây là một ví dụ nơi luật loại bỏ lifetime thứ ba áp dụng:

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-10-lifetimes-on-methods/src/main.rs:3rd}}
```

Có hai lifetime đầu vào, vì vậy Rust áp dụng luật loại bỏ lifetime đầu tiên và
gán cho cả `&self` và `announcement` lifetimes riêng của chúng. Sau đó, bởi vì
một trong các tham số là `&self`, kiểu trả về nhận lifetime của `&self`, và tất
cả lifetimes đã được tính đến.

### Static Lifetime

Một lifetime đặc biệt mà chúng ta cần thảo luận là `'static`, nó biểu thị rằng
tham chiếu bị ảnh hưởng _có thể_ tồn tại trong toàn bộ thời gian của chương
trình. Tất cả các literal chuỗi có lifetime `'static`, mà chúng ta có thể chú
thích như sau:

```rust
let s: &'static str = "I have a static lifetime.";
```

Văn bản của chuỗi này được lưu trữ trực tiếp trong tệp nhị phân của chương
trình, mà luôn có sẵn. Do đó, lifetime của tất cả các literal chuỗi là
`'static`.

Bạn có thể thấy các gợi ý trong thông báo lỗi để sử dụng lifetime `'static`.
Nhưng trước khi chỉ định `'static` làm lifetime cho một tham chiếu, hãy suy nghĩ
về việc liệu tham chiếu bạn có thực sự tồn tại trong toàn bộ lifetime của chương
trình hay không, và liệu bạn có muốn nó như vậy hay không. Hầu hết thời gian,
một thông báo lỗi gợi ý lifetime `'static` là kết quả của việc cố gắng tạo một
dangling reference hoặc không khớp của các lifetimes có sẵn. Trong những trường
hợp như vậy, giải pháp là sửa những vấn đề đó, không phải chỉ định lifetime
`'static`.

## Tham số kiểu Generic, Trait Bounds và Lifetimes cùng nhau

Hãy xem qua cú pháp của việc chỉ định tham số kiểu generic, trait bounds và
lifetimes tất cả trong một hàm!

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/no-listing-11-generics-traits-and-lifetimes/src/main.rs:here}}
```

Đây là hàm `longest` từ Listing 10-21 trả về cái dài hơn trong hai slice chuỗi.
Nhưng bây giờ nó có một tham số bổ sung có tên `ann` của kiểu generic `T`, có
thể được điền bởi bất kỳ kiểu nào triển khai trait `Display` như được chỉ định
bởi mệnh đề `where`. Tham số bổ sung này sẽ được in bằng `{}`, đó là lý do tại
sao ràng buộc trait `Display` là cần thiết. Bởi vì lifetimes là một loại
generic, khai báo của tham số lifetime `'a` và tham số kiểu generic `T` được đặt
trong cùng một danh sách bên trong dấu ngoặc nhọn sau tên hàm.

## Tóm tắt

Chúng ta đã bao quát nhiều trong chương này! Bây giờ bạn đã biết về tham số kiểu
generic, traits và trait bounds, và tham số lifetime generic, bạn đã sẵn sàng để
viết mã không trùng lặp hoạt động trong nhiều tình huống khác nhau. Tham số kiểu
generic cho phép bạn áp dụng mã cho các kiểu khác nhau. Traits và trait bounds
đảm bảo rằng mặc dù các kiểu là generic, chúng sẽ có hành vi mà mã cần. Bạn đã
học cách sử dụng chú thích lifetime để đảm bảo rằng mã linh hoạt này sẽ không có
bất kỳ dangling reference nào. Và tất cả những phân tích này xảy ra tại thời
điểm biên dịch, không ảnh hưởng đến hiệu suất thời gian chạy!

Tin hay không, có còn nhiều điều để học về các chủ đề chúng ta đã thảo luận
trong chương này: Chương 18 thảo luận về trait objects, là một cách khác để sử
dụng traits. Cũng có các kịch bản phức tạp hơn liên quan đến chú thích lifetime
mà bạn sẽ chỉ cần trong các kịch bản rất nâng cao; cho những điều đó, bạn nên
đọc [Rust Reference][reference]. Nhưng tiếp theo, bạn sẽ học cách viết tests
trong Rust để đảm bảo mã của bạn hoạt động theo cách mà nó nên làm.

[references-and-borrowing]:
  ch04-02-references-and-borrowing.html#references-and-borrowing
[string-slices-as-parameters]: ch04-03-slices.html#string-slices-as-parameters
[reference]: ../reference/index.html
