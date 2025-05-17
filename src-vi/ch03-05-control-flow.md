## Luồng Điều Khiển

Khả năng chạy một số mã tùy thuộc vào việc một điều kiện là `true` và chạy một
số mã lặp đi lặp lại trong khi một điều kiện là `true` là những khối xây dựng cơ
bản trong hầu hết các ngôn ngữ lập trình. Các cấu trúc phổ biến nhất cho phép
bạn kiểm soát luồng thực thi của mã Rust là biểu thức `if` và vòng lặp.

### Biểu Thức `if`

Biểu thức `if` cho phép bạn phân nhánh mã của mình dựa trên các điều kiện. Bạn
cung cấp một điều kiện và sau đó nói, "Nếu điều kiện này được đáp ứng, hãy chạy
khối mã này. Nếu điều kiện không được đáp ứng, đừng chạy khối mã này."

Tạo một dự án mới có tên _branches_ trong thư mục _projects_ của bạn để khám phá
biểu thức `if`. Trong tệp _src/main.rs_, nhập nội dung sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/src/main.rs}}
```

Tất cả các biểu thức `if` bắt đầu bằng từ khóa `if`, theo sau là một điều kiện.
Trong trường hợp này, điều kiện kiểm tra xem biến `number` có giá trị nhỏ hơn 5
hay không. Chúng ta đặt khối mã để thực thi nếu điều kiện là `true` ngay sau
điều kiện bên trong dấu ngoặc nhọn. Các khối mã liên kết với các điều kiện trong
biểu thức `if` đôi khi được gọi là _nhánh_, giống như các nhánh trong biểu thức
`match` mà chúng ta đã thảo luận trong phần ["So Sánh Dự Đoán với Số Bí
Mật"][comparing-the-guess-to-the-secret-number]<!-- ignore --> của Chương 2.

Tùy chọn, chúng ta cũng có thể bao gồm một biểu thức `else`, mà chúng ta đã chọn
ở đây, để cung cấp cho chương trình một khối mã thay thế để thực thi nếu điều
kiện đánh giá thành `false`. Nếu bạn không cung cấp một biểu thức `else` và điều
kiện là `false`, chương trình sẽ bỏ qua khối `if` và chuyển sang đoạn mã tiếp
theo.

Hãy thử chạy mã này; bạn sẽ thấy đầu ra sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-26-if-true/output.txt}}
```

Hãy thử thay đổi giá trị của `number` thành một giá trị khiến điều kiện thành
`false` để xem điều gì xảy ra:

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/src/main.rs:here}}
```

Chạy chương trình lần nữa và xem đầu ra:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-27-if-false/output.txt}}
```

Cũng cần lưu ý rằng điều kiện trong mã này _phải_ là một `bool`. Nếu điều kiện
không phải là `bool`, chúng ta sẽ gặp lỗi. Ví dụ, hãy thử chạy mã sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/src/main.rs}}
```

Điều kiện `if` đánh giá thành một giá trị `3` lần này, và Rust báo lỗi:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-28-if-condition-must-be-bool/output.txt}}
```

Lỗi chỉ ra rằng Rust mong đợi một `bool` nhưng nhận được một số nguyên. Không
giống như các ngôn ngữ như Ruby và JavaScript, Rust sẽ không tự động chuyển đổi
các kiểu không phải Boolean thành Boolean. Bạn phải rõ ràng và luôn cung cấp cho
`if` một Boolean làm điều kiện của nó. Nếu chúng ta muốn khối mã `if` chạy chỉ
khi một số không bằng `0`, ví dụ, chúng ta có thể thay đổi biểu thức `if` thành
như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-29-if-not-equal-0/src/main.rs}}
```

Chạy mã này sẽ in ra `number was something other than zero`.

#### Xử Lý Nhiều Điều Kiện với `else if`

Bạn có thể sử dụng nhiều điều kiện bằng cách kết hợp `if` và `else` trong một
biểu thức `else if`. Ví dụ:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/src/main.rs}}
```

Chương trình này có bốn đường dẫn có thể thực hiện. Sau khi chạy nó, bạn sẽ thấy
đầu ra sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-30-else-if/output.txt}}
```

Khi chương trình này thực thi, nó kiểm tra từng biểu thức `if` lần lượt và thực
thi thân đầu tiên mà điều kiện đánh giá thành `true`. Lưu ý rằng mặc dù 6 chia
hết cho 2, chúng ta không thấy đầu ra `number is divisible by 2`, cũng không
thấy văn bản `number is not divisible by 4, 3, or 2` từ khối `else`. Đó là vì
Rust chỉ thực thi khối cho điều kiện `true` đầu tiên, và khi tìm thấy một điều
kiện, nó thậm chí không kiểm tra phần còn lại.

Sử dụng quá nhiều biểu thức `else if` có thể làm rối mã của bạn, vì vậy nếu bạn
có nhiều hơn một, bạn có thể muốn cấu trúc lại mã của mình. Chương 6 mô tả một
cấu trúc phân nhánh mạnh mẽ của Rust gọi là `match` cho những trường hợp này.

#### Sử dụng `if` trong một câu lệnh `let`

Vì `if` là một biểu thức, chúng ta có thể sử dụng nó ở phía bên phải của câu
lệnh `let` để gán kết quả cho một biến, như trong Listing 3-2.

<Listing number="3-2" file-name="src/main.rs" caption="Gán kết quả của một biểu thức `if` cho một biến">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-02/src/main.rs}}
```

</Listing>

Biến `number` sẽ được gắn với một giá trị dựa trên kết quả của biểu thức `if`.
Chạy mã này để xem điều gì xảy ra:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-02/output.txt}}
```

Hãy nhớ rằng các khối mã đánh giá thành biểu thức cuối cùng trong chúng, và các
số tự chúng cũng là biểu thức. Trong trường hợp này, giá trị của toàn bộ biểu
thức `if` phụ thuộc vào khối mã nào được thực thi. Điều này có nghĩa là các giá
trị có khả năng là kết quả từ mỗi nhánh của `if` phải cùng một loại; trong
Listing 3-2, kết quả của cả nhánh `if` và nhánh `else` đều là số nguyên `i32`.
Nếu các loại không khớp, như trong ví dụ sau, chúng ta sẽ gặp lỗi:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/src/main.rs}}
```

Khi cố gắng biên dịch mã này, chúng ta sẽ gặp lỗi. Các nhánh `if` và `else` có
kiểu giá trị không tương thích, và Rust chỉ ra chính xác nơi tìm thấy vấn đề
trong chương trình:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-31-arms-must-return-same-type/output.txt}}
```

Biểu thức trong khối `if` đánh giá thành một số nguyên, và biểu thức trong khối
`else` đánh giá thành một chuỗi. Điều này sẽ không hoạt động vì các biến phải có
một kiểu duy nhất, và Rust cần biết tại thời điểm biên dịch kiểu biến `number`
là gì, một cách chắc chắn. Biết kiểu của `number` cho phép trình biên dịch xác
minh kiểu là hợp lệ ở mọi nơi chúng ta sử dụng `number`. Rust sẽ không thể làm
điều đó nếu kiểu của `number` chỉ được xác định trong thời gian chạy; trình biên
dịch sẽ phức tạp hơn và sẽ cung cấp ít đảm bảo hơn về mã nếu phải theo dõi nhiều
kiểu giả định cho bất kỳ biến nào.

### Lặp Lại với Vòng Lặp

Thường rất hữu ích để thực thi một khối mã nhiều lần. Cho nhiệm vụ này, Rust
cung cấp một số _vòng lặp_, sẽ chạy qua mã bên trong thân vòng lặp đến cuối và
sau đó bắt đầu lại từ đầu ngay lập tức. Để thử nghiệm với vòng lặp, hãy tạo một
dự án mới có tên _loops_.

Rust có ba loại vòng lặp: `loop`, `while`, và `for`. Hãy thử từng loại.

#### Lặp Lại Mã với `loop`

Từ khóa `loop` yêu cầu Rust thực thi một khối mã lặp đi lặp lại mãi mãi hoặc cho
đến khi bạn yêu cầu nó dừng lại.

Ví dụ, hãy thay đổi tệp _src/main.rs_ trong thư mục _loops_ của bạn để trông như
thế này:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-loop/src/main.rs}}
```

Khi chúng ta chạy chương trình này, chúng ta sẽ thấy `again!` được in liên tục
cho đến khi chúng ta dừng chương trình bằng tay. Hầu hết các terminal hỗ trợ
phím tắt <kbd>ctrl</kbd>-<kbd>c</kbd> để ngắt một chương trình bị kẹt trong một
vòng lặp liên tục. Hãy thử:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-32-loop
cargo run
CTRL-C
-->

```console
$ cargo run
   Compiling loops v0.1.0 (file:///projects/loops)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.08s
     Running `target/debug/loops`
again!
again!
again!
again!
^Cagain!
```

Ký hiệu `^C` biểu thị nơi bạn đã nhấn <kbd>ctrl</kbd>-<kbd>c</kbd>.

Bạn có thể sẽ thấy hoặc không thấy từ `again!` được in sau `^C`, tùy thuộc vào
vị trí mã trong vòng lặp khi nó nhận được tín hiệu ngắt.

May mắn thay, Rust cũng cung cấp một cách để thoát khỏi vòng lặp bằng mã. Bạn có
thể đặt từ khóa `break` trong vòng lặp để yêu cầu chương trình dừng thực thi
vòng lặp. Hãy nhớ rằng chúng ta đã làm điều này trong trò chơi đoán số ở phần
["Thoát Sau Khi Đoán Đúng"][quitting-after-a-correct-guess]<!-- ignore --> của
Chương 2 để thoát chương trình khi người dùng thắng trò chơi bằng cách đoán đúng
số.

Chúng ta cũng đã sử dụng `continue` trong trò chơi đoán số, trong một vòng lặp
nó bảo chương trình bỏ qua bất kỳ mã còn lại nào trong lần lặp này của vòng lặp
và chuyển sang lần lặp tiếp theo.

#### Trả Về Giá Trị từ Vòng Lặp

Một trong những cách sử dụng của `loop` là thử lại một hoạt động mà bạn biết có
thể thất bại, chẳng hạn như kiểm tra xem một luồng đã hoàn thành công việc của
nó hay chưa. Bạn cũng có thể cần truyền kết quả của hoạt động đó ra khỏi vòng
lặp đến phần còn lại của mã của bạn. Để làm điều này, bạn có thể thêm giá trị
bạn muốn trả về sau biểu thức `break` bạn sử dụng để dừng vòng lặp; giá trị đó
sẽ được trả về từ vòng lặp để bạn có thể sử dụng nó, như được hiển thị ở đây:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-33-return-value-from-loop/src/main.rs}}
```

Trước vòng lặp, chúng ta khai báo một biến có tên `counter` và khởi tạo nó thành
`0`. Sau đó, chúng ta khai báo một biến có tên `result` để giữ giá trị trả về từ
vòng lặp. Trong mỗi lần lặp của vòng lặp, chúng ta thêm `1` vào biến `counter`,
và sau đó kiểm tra xem `counter` có bằng `10` không. Khi nó bằng `10`, chúng ta
sử dụng từ khóa `break` với giá trị `counter * 2`. Sau vòng lặp, chúng ta sử
dụng dấu chấm phẩy để kết thúc câu lệnh gán giá trị cho `result`. Cuối cùng,
chúng ta in giá trị trong `result`, trong trường hợp này là `20`.

Bạn cũng có thể `return` từ bên trong một vòng lặp. Trong khi `break` chỉ thoát
khỏi vòng lặp hiện tại, `return` luôn thoát khỏi hàm hiện tại.

#### Nhãn Vòng Lặp để Phân Biệt Giữa Nhiều Vòng Lặp

Nếu bạn có vòng lặp trong vòng lặp, `break` và `continue` áp dụng cho vòng lặp
bên trong nhất tại thời điểm đó. Bạn có thể tùy chọn chỉ định một _nhãn vòng
lặp_ trên một vòng lặp mà bạn có thể sử dụng với `break` hoặc `continue` để chỉ
định rằng những từ khóa đó áp dụng cho vòng lặp được gắn nhãn thay vì vòng lặp
bên trong nhất. Nhãn vòng lặp phải bắt đầu bằng một dấu nháy đơn. Đây là một ví
dụ với hai vòng lặp lồng nhau:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/src/main.rs}}
```

Vòng lặp bên ngoài có nhãn `'counting_up`, và nó sẽ đếm lên từ 0 đến 2. Vòng lặp
bên trong không có nhãn đếm ngược từ 10 đến 9. `break` đầu tiên không chỉ định
nhãn sẽ chỉ thoát khỏi vòng lặp bên trong. Câu lệnh `break 'counting_up;` sẽ
thoát khỏi vòng lặp bên ngoài. Mã này in:

```console
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-32-5-loop-labels/output.txt}}
```

#### Vòng Lặp Có Điều Kiện với `while`

Một chương trình thường cần đánh giá một điều kiện trong một vòng lặp. Trong khi
điều kiện là `true`, vòng lặp chạy. Khi điều kiện không còn là `true`, chương
trình gọi `break`, dừng vòng lặp. Có thể thực hiện hành vi như thế này bằng cách
kết hợp `loop`, `if`, `else`, và `break`; bạn có thể thử điều đó bây giờ trong
một chương trình, nếu bạn muốn. Tuy nhiên, mẫu này rất phổ biến nên Rust có một
cấu trúc ngôn ngữ tích hợp cho nó, gọi là vòng lặp `while`. Trong Listing 3-3,
chúng ta sử dụng `while` để chạy chương trình ba lần, đếm ngược mỗi lần, và sau
đó, sau vòng lặp, in một thông báo và thoát.

<Listing number="3-3" file-name="src/main.rs" caption="Sử dụng vòng lặp `while` để chạy mã trong khi một điều kiện đánh giá thành `true`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-03/src/main.rs}}
```

</Listing>

Cấu trúc này loại bỏ rất nhiều việc lồng nhau sẽ cần thiết nếu bạn sử dụng
`loop`, `if`, `else`, và `break`, và nó rõ ràng hơn. Trong khi một điều kiện
đánh giá thành `true`, mã chạy; nếu không, nó thoát khỏi vòng lặp.

#### Lặp Qua một Tập Hợp với `for`

Bạn có thể chọn sử dụng cấu trúc `while` để lặp qua các phần tử của một tập hợp,
chẳng hạn như một mảng. Ví dụ, vòng lặp trong Listing 3-4 in ra từng phần tử
trong mảng `a`.

<Listing number="3-4" file-name="src/main.rs" caption="Lặp qua từng phần tử của một tập hợp bằng vòng lặp `while`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-04/src/main.rs}}
```

</Listing>

Ở đây, mã đếm qua các phần tử trong mảng. Nó bắt đầu từ chỉ mục `0`, và sau đó
lặp cho đến khi đạt đến chỉ mục cuối cùng trong mảng (đó là khi `index < 5`
không còn `true` nữa). Chạy mã này sẽ in mọi phần tử trong mảng:

```console
{{#include ../listings/ch03-common-programming-concepts/listing-03-04/output.txt}}
```

Tất cả năm giá trị mảng xuất hiện trong terminal, như mong đợi. Mặc dù `index`
sẽ đạt đến giá trị `5` tại một thời điểm nào đó, vòng lặp dừng thực thi trước
khi cố gắng lấy giá trị thứ sáu từ mảng.

Tuy nhiên, cách tiếp cận này dễ xảy ra lỗi; chúng ta có thể khiến chương trình
hoảng sợ nếu giá trị chỉ mục hoặc điều kiện kiểm tra không chính xác. Ví dụ, nếu
bạn thay đổi định nghĩa của mảng `a` để có bốn phần tử nhưng quên cập nhật điều
kiện thành `while index < 4`, mã sẽ hoảng sợ. Nó cũng chậm, vì trình biên dịch
thêm mã thời gian chạy để thực hiện kiểm tra điều kiện xem chỉ mục có nằm trong
giới hạn của mảng trong mỗi lần lặp qua vòng lặp không.

Là một lựa chọn thay thế ngắn gọn hơn, bạn có thể sử dụng vòng lặp `for` và thực
thi một số mã cho mỗi phần tử trong một tập hợp. Một vòng lặp `for` trông giống
như mã trong Listing 3-5.

<Listing number="3-5" file-name="src/main.rs" caption="Lặp qua từng phần tử của một tập hợp bằng vòng lặp `for`">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-05/src/main.rs}}
```

</Listing>

Khi chúng ta chạy mã này, chúng ta sẽ thấy đầu ra giống như trong Listing 3-4.
Quan trọng hơn, bây giờ chúng ta đã tăng độ an toàn của mã và loại bỏ khả năng
xảy ra lỗi có thể dẫn đến vượt quá cuối mảng hoặc không đi đủ xa và bỏ qua một
số phần tử. Mã máy được tạo từ vòng lặp `for` cũng có thể hiệu quả hơn, vì chỉ
mục không cần phải so sánh với độ dài của mảng ở mỗi lần lặp.

Sử dụng vòng lặp `for`, bạn sẽ không cần phải nhớ thay đổi bất kỳ mã nào khác
nếu bạn thay đổi số lượng giá trị trong mảng, như bạn sẽ làm với phương pháp
được sử dụng trong Listing 3-4.

Sự an toàn và ngắn gọn của vòng lặp `for` làm cho chúng trở thành cấu trúc vòng
lặp được sử dụng phổ biến nhất trong Rust. Ngay cả trong những tình huống mà bạn
muốn chạy một số mã một số lần nhất định, như trong ví dụ đếm ngược sử dụng vòng
lặp `while` trong Listing 3-3, hầu hết người dùng Rust sẽ sử dụng vòng lặp
`for`. Cách để làm điều đó sẽ là sử dụng một `Range`, được cung cấp bởi thư viện
tiêu chuẩn, tạo ra tất cả các số theo thứ tự bắt đầu từ một số và kết thúc trước
một số khác.

Đây là cách đếm ngược sẽ trông như thế nào khi sử dụng vòng lặp `for` và một
phương pháp khác mà chúng ta chưa nói đến, `rev`, để đảo ngược phạm vi:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-34-for-range/src/main.rs}}
```

Mã này tốt hơn một chút, phải không?

## Tóm Tắt

Bạn đã làm được! Đây là một chương đáng kể: bạn đã học về biến, kiểu dữ liệu vô
hướng và phức hợp, hàm, nhận xét, biểu thức `if`, và vòng lặp! Để thực hành với
các khái niệm được thảo luận trong chương này, hãy thử xây dựng các chương trình
để thực hiện các việc sau:

- Chuyển đổi nhiệt độ giữa Fahrenheit và Celsius.
- Tạo số Fibonacci thứ _n_.
- In lời bài hát Giáng sinh "The Twelve Days of Christmas", tận dụng sự lặp lại
  trong bài hát.

Khi bạn sẵn sàng tiếp tục, chúng ta sẽ nói về một khái niệm trong Rust mà
_không_ thường tồn tại trong các ngôn ngữ lập trình khác: quyền sở hữu.

[comparing-the-guess-to-the-secret-number]:
  ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[quitting-after-a-correct-guess]:
  ch02-00-guessing-game-tutorial.html#quitting-after-a-correct-guess
