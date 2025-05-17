## Các Hàm

Hàm xuất hiện phổ biến trong mã Rust. Bạn đã thấy một trong những hàm quan trọng
nhất trong ngôn ngữ này: hàm `main`, đó là điểm khởi đầu của nhiều chương trình.
Bạn cũng đã thấy từ khóa `fn`, cho phép bạn khai báo các hàm mới.

Mã Rust sử dụng _snake case_ là quy ước phong cách cho tên hàm và tên biến,
trong đó tất cả các chữ cái đều là chữ thường và dấu gạch dưới phân tách các từ.
Đây là một chương trình chứa ví dụ về định nghĩa hàm:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-16-functions/src/main.rs}}
```

Chúng ta định nghĩa một hàm trong Rust bằng cách nhập `fn` theo sau là tên hàm
và một tập hợp dấu ngoặc đơn. Dấu ngoặc nhọn cho trình biên dịch biết nơi bắt
đầu và kết thúc thân hàm.

Chúng ta có thể gọi bất kỳ hàm nào mà chúng ta đã định nghĩa bằng cách nhập tên
hàm theo sau bởi tập hợp dấu ngoặc đơn. Vì `another_function` được định nghĩa
trong chương trình, nó có thể được gọi từ bên trong hàm `main`. Lưu ý rằng chúng
ta đã định nghĩa `another_function` _sau_ hàm `main` trong mã nguồn; chúng ta
cũng có thể đã định nghĩa nó trước. Rust không quan tâm bạn định nghĩa hàm của
mình ở đâu, chỉ cần chúng được định nghĩa ở đâu đó trong một phạm vi mà người
gọi có thể nhìn thấy.

Hãy bắt đầu một dự án nhị phân mới có tên là _functions_ để khám phá hàm sâu
hơn. Đặt ví dụ `another_function` trong _src/main.rs_ và chạy nó. Bạn sẽ thấy
kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-16-functions/output.txt}}
```

Các dòng thực thi theo thứ tự chúng xuất hiện trong hàm `main`. Đầu tiên, thông
báo "Hello, world!" được in ra, và sau đó `another_function` được gọi và thông
báo của nó được in ra.

### Tham số

Chúng ta có thể định nghĩa hàm có _tham số_, đó là các biến đặc biệt là một phần
của chữ ký hàm. Khi một hàm có tham số, bạn có thể cung cấp cho nó các giá trị
cụ thể cho các tham số đó. Về mặt kỹ thuật, các giá trị cụ thể được gọi là _đối
số_, nhưng trong cuộc trò chuyện thông thường, mọi người thường sử dụng từ _tham
số_ và _đối số_ thay thế cho nhau cho cả biến trong định nghĩa một hàm hoặc các
giá trị cụ thể được truyền vào khi bạn gọi một hàm.

Trong phiên bản này của `another_function`, chúng ta thêm một tham số:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/src/main.rs}}
```

Hãy thử chạy chương trình này; bạn sẽ nhận được kết quả sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-17-functions-with-parameters/output.txt}}
```

Khai báo của `another_function` có một tham số có tên là `x`. Kiểu của `x` được
xác định là `i32`. Khi chúng ta truyền `5` vào `another_function`, macro
`println!` đặt `5` vào nơi có cặp dấu ngoặc nhọn chứa `x` trong chuỗi định dạng.

Trong chữ ký hàm, bạn _phải_ khai báo kiểu của mỗi tham số. Đây là một quyết
định có chủ đích trong thiết kế của Rust: yêu cầu chú thích kiểu trong định
nghĩa hàm có nghĩa là trình biên dịch hầu như không cần bạn sử dụng chúng ở nơi
khác trong mã để xác định kiểu bạn muốn. Trình biên dịch cũng có thể đưa ra
thông báo lỗi hữu ích hơn nếu nó biết kiểu nào mà hàm mong đợi.

Khi định nghĩa nhiều tham số, hãy phân tách khai báo tham số bằng dấu phẩy, như
thế này:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/src/main.rs}}
```

Ví dụ này tạo ra một hàm có tên là `print_labeled_measurement` với hai tham số.
Tham số đầu tiên có tên là `value` và là một `i32`. Tham số thứ hai có tên là
`unit_label` và có kiểu `char`. Sau đó hàm in ra văn bản chứa cả `value` và
`unit_label`.

Hãy thử chạy mã này. Thay thế chương trình hiện có trong tập tin _src/main.rs_
của dự án _functions_ bằng ví dụ trước đó và chạy nó bằng `cargo run`:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-18-functions-with-multiple-parameters/output.txt}}
```

Vì chúng ta đã gọi hàm với `5` là giá trị cho `value` và `'h'` là giá trị cho
`unit_label`, kết quả chương trình chứa những giá trị đó.

### Câu lệnh và Biểu thức

Phần thân hàm được cấu thành từ một chuỗi các câu lệnh, tùy chọn kết thúc bằng
một biểu thức. Cho đến nay, các hàm chúng ta đã đề cập chưa bao gồm một biểu
thức kết thúc, nhưng bạn đã thấy một biểu thức như một phần của một câu lệnh.
Bởi vì Rust là một ngôn ngữ dựa trên biểu thức, đây là một sự phân biệt quan
trọng cần hiểu. Các ngôn ngữ khác không có sự phân biệt giống nhau, vì vậy hãy
xem xét câu lệnh và biểu thức là gì và cách sự khác biệt của chúng ảnh hưởng đến
phần thân của các hàm.

- Câu lệnh là những chỉ thị thực hiện một số hành động và không trả về giá trị.
- Biểu thức đánh giá thành một giá trị kết quả.

Hãy xem một số ví dụ.

Chúng ta thực sự đã sử dụng câu lệnh và biểu thức rồi. Việc tạo ra một biến và
gán giá trị cho nó với từ khóa `let` là một câu lệnh. Trong Listing 3-1,
`let y = 6;` là một câu lệnh.

<Listing number="3-1" file-name="src/main.rs" caption="Khai báo hàm `main` chứa một câu lệnh">

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/listing-03-01/src/main.rs}}
```

</Listing>

Định nghĩa hàm cũng là câu lệnh; toàn bộ ví dụ trước đó là một câu lệnh tự nó.
(Như chúng ta sẽ thấy dưới đây, _gọi_ một hàm không phải là một câu lệnh, tuy
nhiên.)

Câu lệnh không trả về giá trị. Do đó, bạn không thể gán một câu lệnh `let` cho
một biến khác, như mã sau cố gắng làm; bạn sẽ gặp lỗi:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/src/main.rs}}
```

Khi bạn chạy chương trình này, lỗi bạn sẽ nhận được trông như thế này:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-19-statements-vs-expressions/output.txt}}
```

Câu lệnh `let y = 6` không trả về giá trị, vì vậy không có gì để `x` ràng buộc.
Điều này khác với những gì xảy ra trong các ngôn ngữ khác, chẳng hạn như C và
Ruby, nơi phép gán trả về giá trị của phép gán. Trong các ngôn ngữ đó, bạn có
thể viết `x = y = 6` và cả `x` và `y` đều có giá trị `6`; điều đó không đúng
trong Rust.

Biểu thức đánh giá thành một giá trị và tạo nên phần lớn phần còn lại của mã mà
bạn sẽ viết trong Rust. Hãy xem xét một phép toán toán học, chẳng hạn như
`5 + 6`, đó là một biểu thức đánh giá thành giá trị `11`. Biểu thức có thể là
một phần của câu lệnh: trong Listing 3-1, `6` trong câu lệnh `let y = 6;` là một
biểu thức đánh giá thành giá trị `6`. Gọi một hàm là một biểu thức. Gọi một
macro là một biểu thức. Một khối phạm vi mới được tạo ra với dấu ngoặc nhọn là
một biểu thức, ví dụ:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-20-blocks-are-expressions/src/main.rs}}
```

Biểu thức này:

```rust,ignore
{
    let x = 3;
    x + 1
}
```

là một khối mà trong trường hợp này, đánh giá thành `4`. Giá trị đó được ràng
buộc với `y` như một phần của câu lệnh `let`. Lưu ý rằng dòng `x + 1` không có
dấu chấm phẩy ở cuối, khác với hầu hết các dòng mà bạn đã thấy cho đến nay. Biểu
thức không bao gồm dấu chấm phẩy kết thúc. Nếu bạn thêm dấu chấm phẩy vào cuối
một biểu thức, bạn biến nó thành một câu lệnh, và nó sẽ không trả về giá trị
nữa. Hãy ghi nhớ điều này khi bạn khám phá giá trị trả về của hàm và biểu thức
tiếp theo.

### Hàm với Giá trị Trả về

Hàm có thể trả về giá trị cho mã gọi chúng. Chúng ta không đặt tên cho các giá
trị trả về, nhưng chúng ta phải khai báo kiểu của chúng sau một mũi tên (`->`).
Trong Rust, giá trị trả về của hàm đồng nghĩa với giá trị của biểu thức cuối
cùng trong khối của phần thân hàm. Bạn có thể trả về sớm từ một hàm bằng cách sử
dụng từ khóa `return` và chỉ định một giá trị, nhưng hầu hết các hàm đều trả về
biểu thức cuối cùng một cách ngầm định. Đây là một ví dụ về một hàm trả về một
giá trị:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/src/main.rs}}
```

Không có lời gọi hàm, macro, hay thậm chí câu lệnh `let` trong hàm `five`—chỉ có
số `5` đứng một mình. Đó là một hàm hoàn toàn hợp lệ trong Rust. Lưu ý rằng kiểu
trả về của hàm cũng được chỉ định, là `-> i32`. Hãy thử chạy mã này; kết quả sẽ
trông như thế này:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-21-function-return-values/output.txt}}
```

Số `5` trong `five` là giá trị trả về của hàm, đó là lý do tại sao kiểu trả về
là `i32`. Hãy xem xét điều này chi tiết hơn. Có hai phần quan trọng: đầu tiên,
dòng `let x = five();` cho thấy rằng chúng ta đang sử dụng giá trị trả về của
một hàm để khởi tạo một biến. Bởi vì hàm `five` trả về `5`, dòng đó giống như
sau:

```rust
let x = 5;
```

Thứ hai, hàm `five` không có tham số và định nghĩa kiểu của giá trị trả về,
nhưng phần thân của hàm chỉ là một `5` cô đơn không có dấu chấm phẩy vì đó là
một biểu thức mà chúng ta muốn trả về giá trị.

Hãy xem một ví dụ khác:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-22-function-parameter-and-return/src/main.rs}}
```

Chạy mã này sẽ in ra `The value of x is: 6`. Nhưng nếu chúng ta đặt một dấu chấm
phẩy ở cuối dòng chứa `x + 1`, biến nó từ một biểu thức thành một câu lệnh,
chúng ta sẽ nhận được lỗi:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/src/main.rs}}
```

Biên dịch mã này tạo ra lỗi như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-23-statements-dont-return-values/output.txt}}
```

Thông báo lỗi chính, `mismatched types` (không khớp kiểu), tiết lộ vấn đề cốt
lõi với mã này. Định nghĩa của hàm `plus_one` nói rằng nó sẽ trả về một `i32`,
nhưng câu lệnh không đánh giá thành một giá trị, được biểu thị bởi `()`, kiểu
đơn vị. Do đó, không có gì được trả về, mâu thuẫn với định nghĩa hàm và dẫn đến
lỗi. Trong kết quả này, Rust cung cấp một thông báo để có thể giúp khắc phục vấn
đề này: nó đề xuất loại bỏ dấu chấm phẩy, điều đó sẽ khắc phục lỗi.
