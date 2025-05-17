## Kiểu Dữ Liệu

Mỗi giá trị trong Rust thuộc về một _kiểu dữ liệu_ nhất định, điều này cho Rust
biết loại dữ liệu đang được chỉ định để nó biết cách làm việc với dữ liệu đó.
Chúng ta sẽ xem xét hai tập con kiểu dữ liệu: kiểu vô hướng (scalar) và kiểu
phức hợp (compound).

Hãy nhớ rằng Rust là một ngôn ngữ _kiểu tĩnh_, nghĩa là nó phải biết kiểu của
tất cả các biến tại thời điểm biên dịch. Trình biên dịch thường có thể suy ra
kiểu mà chúng ta muốn sử dụng dựa trên giá trị và cách chúng ta sử dụng nó.
Trong các trường hợp có nhiều kiểu có thể xảy ra, chẳng hạn như khi chúng ta
chuyển đổi một `String` thành một kiểu số bằng cách sử dụng `parse` trong phần
["So sánh số đoán với số bí
mật"][comparing-the-guess-to-the-secret-number]<!-- ignore --> ở Chương 2, chúng
ta phải thêm chú thích kiểu, như sau:

```rust
let guess: u32 = "42".parse().expect("Not a number!");
```

Nếu chúng ta không thêm chú thích kiểu `: u32` như trong đoạn mã trên, Rust sẽ
hiển thị lỗi sau, có nghĩa là trình biên dịch cần thêm thông tin từ chúng ta để
biết kiểu nào chúng ta muốn sử dụng:

```console
{{#include ../listings/ch03-common-programming-concepts/output-only-01-no-type-annotations/output.txt}}
```

Bạn sẽ thấy các chú thích kiểu khác nhau cho các kiểu dữ liệu khác.

### Kiểu Vô Hướng

Một _kiểu vô hướng_ biểu diễn một giá trị đơn lẻ. Rust có bốn kiểu vô hướng cơ
bản: số nguyên, số thực dấu phẩy động, Boolean và ký tự. Có thể bạn đã quen
thuộc với những kiểu này từ các ngôn ngữ lập trình khác. Chúng ta hãy xem cách
chúng hoạt động trong Rust.

#### Kiểu Số Nguyên

_Số nguyên_ là một số không có phần thập phân. Chúng ta đã sử dụng một kiểu số
nguyên trong Chương 2, kiểu `u32`. Khai báo kiểu này cho biết rằng giá trị được
liên kết với nó phải là một số nguyên không dấu (các kiểu số nguyên có dấu bắt
đầu bằng `i` thay vì `u`) chiếm 32 bit không gian lưu trữ. Bảng 3-1 cho thấy các
kiểu số nguyên tích hợp trong Rust. Chúng ta có thể sử dụng bất kỳ biến thể nào
trong số này để khai báo kiểu của một giá trị số nguyên.

<span class="caption">Bảng 3-1: Các Kiểu Số Nguyên trong Rust</span>

| Độ dài                  | Có dấu  | Không dấu |
| ----------------------- | ------- | --------- |
| 8-bit                   | `i8`    | `u8`      |
| 16-bit                  | `i16`   | `u16`     |
| 32-bit                  | `i32`   | `u32`     |
| 64-bit                  | `i64`   | `u64`     |
| 128-bit                 | `i128`  | `u128`    |
| phụ thuộc vào kiến trúc | `isize` | `usize`   |

Mỗi biến thể có thể có dấu hoặc không dấu và có một kích thước rõ ràng. _Có dấu_
và _không dấu_ đề cập đến việc liệu số đó có thể âm hay không—nói cách khác,
liệu số đó cần có dấu kèm theo (có dấu) hay chỉ bao giờ cũng dương và do đó có
thể được biểu diễn mà không cần dấu (không dấu). Giống như viết số trên giấy:
khi dấu quan trọng, số được hiển thị với dấu cộng hoặc dấu trừ; tuy nhiên, khi
có thể giả định rằng số đó là dương, nó được hiển thị không có dấu. Số có dấu
được lưu trữ bằng cách sử dụng biểu diễn [bù
hai][twos-complement]<!-- ignore -->.

Mỗi biến thể có dấu có thể lưu trữ số từ −(2<sup>n − 1</sup>) đến 2<sup>n −
1</sup> − 1 bao gồm cả hai đầu, trong đó _n_ là số bit mà biến thể đó sử dụng.
Vì vậy, `i8` có thể lưu trữ số từ −(2<sup>7</sup>) đến 2<sup>7</sup> − 1, tương
đương từ −128 đến 127. Các biến thể không dấu có thể lưu trữ số từ 0 đến
2<sup>n</sup> − 1, nên `u8` có thể lưu trữ số từ 0 đến 2<sup>8</sup> − 1, tương
đương từ 0 đến 255.

Ngoài ra, các kiểu `isize` và `usize` phụ thuộc vào kiến trúc của máy tính mà
chương trình của bạn đang chạy: 64 bit nếu bạn đang sử dụng kiến trúc 64 bit và
32 bit nếu bạn đang sử dụng kiến trúc 32 bit.

Bạn có thể viết số nguyên theo bất kỳ dạng nào được hiển thị trong Bảng 3-2. Lưu
ý rằng các số nguyên có thể thuộc nhiều kiểu số khác nhau cho phép hậu tố kiểu,
chẳng hạn như `57u8`, để chỉ định kiểu. Số nguyên cũng có thể sử dụng `_` như
một dấu phân cách trực quan để làm cho số dễ đọc hơn, chẳng hạn như `1_000`, sẽ
có giá trị giống như khi bạn chỉ định `1000`.

<span class="caption">Bảng 3-2: Các Số Nguyên trong Rust</span>

| Dạng số         | Ví dụ         |
| --------------- | ------------- |
| Thập phân       | `98_222`      |
| Thập lục phân   | `0xff`        |
| Bát phân        | `0o77`        |
| Nhị phân        | `0b1111_0000` |
| Byte (`u8` chỉ) | `b'A'`        |

Vậy làm thế nào để biết kiểu số nguyên nào để sử dụng? Nếu bạn không chắc chắn,
mặc định của Rust thường là nơi tốt để bắt đầu: các kiểu số nguyên mặc định là
`i32`. Trường hợp chính mà bạn sẽ sử dụng `isize` hoặc `usize` là khi đánh chỉ
số cho một số loại bộ sưu tập.

> ##### Tràn số nguyên
>
> Giả sử bạn có một biến kiểu `u8` có thể lưu giá trị từ 0 đến 255. Nếu bạn cố
> gắng thay đổi biến thành một giá trị ngoài phạm vi đó, chẳng hạn như 256,
> _tràn số nguyên_ sẽ xảy ra, có thể dẫn đến một trong hai hành vi. Khi bạn biên
> dịch ở chế độ debug, Rust bao gồm các kiểm tra tràn số nguyên khiến chương
> trình của bạn _panic_ khi chạy nếu hành vi này xảy ra. Rust sử dụng thuật ngữ
> _panicking_ khi một chương trình kết thúc với lỗi; chúng ta sẽ thảo luận sâu
> hơn về panic trong phần ["Lỗi không thể khôi phục với
> `panic!`"][unrecoverable-errors-with-panic]<!-- ignore --> trong Chương 9.
>
> Khi bạn biên dịch ở chế độ phát hành với cờ `--release`, Rust _không_ bao gồm
> các kiểm tra tràn số nguyên gây ra panic. Thay vào đó, nếu xảy ra tràn, Rust
> thực hiện _bao quanh bù hai_. Nói ngắn gọn, các giá trị lớn hơn giá trị tối đa
> mà kiểu có thể lưu trữ "bao quanh" về giá trị tối thiểu mà kiểu có thể lưu
> trữ. Trong trường hợp của `u8`, giá trị 256 trở thành 0, giá trị 257 trở thành
> 1, và cứ thế. Chương trình sẽ không panic, nhưng biến sẽ có giá trị có lẽ
> không phải là giá trị mà bạn mong đợi nó phải có. Việc dựa vào hành vi bao
> quanh của tràn số nguyên được coi là một lỗi.
>
> Để xử lý rõ ràng khả năng tràn, bạn có thể sử dụng các họ phương thức sau được
> cung cấp bởi thư viện chuẩn cho các kiểu số nguyên nguyên thủy:
>
> - Bao quanh trong mọi chế độ với các phương thức `wrapping_*`, như
>   `wrapping_add`.
> - Trả về giá trị `None` nếu có tràn với các phương thức `checked_*`.
> - Trả về giá trị và một Boolean cho biết liệu có tràn hay không với các phương
>   thức `overflowing_*`.
> - Bão hòa ở giá trị tối thiểu hoặc tối đa của giá trị với các phương thức
>   `saturating_*`.

#### Kiểu Số Thực Dấu Phẩy Động

Rust cũng có hai kiểu nguyên thủy cho _số thực dấu phẩy động_, là số có dấu thập
phân. Kiểu số thực dấu phẩy động của Rust là `f32` và `f64`, có kích thước lần
lượt là 32 bit và 64 bit. Kiểu mặc định là `f64` vì trên CPU hiện đại, nó có tốc
độ gần tương đương với `f32` nhưng có khả năng chính xác cao hơn. Tất cả các
kiểu số thực dấu phẩy động đều có dấu.

Đây là một ví dụ thể hiện số thực dấu phẩy động trong hành động:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-06-floating-point/src/main.rs}}
```

Số thực dấu phẩy động được biểu diễn theo tiêu chuẩn IEEE-754.

#### Phép Toán Số Học

Rust hỗ trợ các phép toán cơ bản mà bạn mong đợi cho tất cả các loại số: cộng,
trừ, nhân, chia và lấy phần dư. Phép chia số nguyên cắt cụt về phía số không đến
số nguyên gần nhất. Đoạn mã sau cho thấy cách bạn sử dụng mỗi phép toán trong
một câu lệnh `let`:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-07-numeric-operations/src/main.rs}}
```

Mỗi biểu thức trong các câu lệnh này sử dụng một toán tử toán học và đánh giá
thành một giá trị duy nhất, sau đó được gắn với một biến. [Phụ lục
B][appendix_b]<!-- ignore --> chứa danh sách tất cả các toán tử mà Rust cung
cấp.

#### Kiểu Boolean

Cũng giống như trong hầu hết các ngôn ngữ lập trình khác, kiểu Boolean trong
Rust có hai giá trị có thể có: `true` và `false`. Boolean có kích thước một
byte. Kiểu Boolean trong Rust được chỉ định bằng `bool`. Ví dụ:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-08-boolean/src/main.rs}}
```

Cách chính để sử dụng giá trị Boolean là thông qua các điều kiện, chẳng hạn như
biểu thức `if`. Chúng ta sẽ tìm hiểu cách biểu thức `if` hoạt động trong Rust
trong phần ["Luồng Điều Khiển"][control-flow]<!-- ignore -->.

#### Kiểu Ký Tự

Kiểu `char` của Rust là kiểu bảng chữ cái nguyên thủy nhất của ngôn ngữ. Dưới
đây là một số ví dụ về khai báo giá trị `char`:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-09-char/src/main.rs}}
```

Lưu ý rằng chúng ta chỉ định các ký tự bằng dấu nháy đơn, khác với chuỗi, sử
dụng dấu nháy kép. Kiểu `char` của Rust có kích thước bốn byte và đại diện cho
một giá trị scalar của Unicode, nghĩa là nó có thể đại diện cho nhiều hơn chỉ là
ASCII. Các chữ cái có dấu; ký tự tiếng Trung, tiếng Nhật và tiếng Hàn; biểu
tượng cảm xúc; và khoảng trắng có độ rộng bằng không đều là giá trị `char` hợp
lệ trong Rust. Giá trị scalar của Unicode nằm trong khoảng từ `U+0000` đến
`U+D7FF` và `U+E000` đến `U+10FFFF` bao gồm. Tuy nhiên, "ký tự" không thực sự là
một khái niệm trong Unicode, vì vậy trực giác của con người về "ký tự" có thể
không khớp với khái niệm `char` trong Rust. Chúng ta sẽ thảo luận chi tiết về
chủ đề này trong ["Lưu Trữ Văn Bản Mã Hóa UTF-8 với
Chuỗi"][strings]<!-- ignore --> ở Chương 8.

### Kiểu Phức Hợp

_Kiểu phức hợp_ có thể nhóm nhiều giá trị thành một kiểu. Rust có hai kiểu phức
hợp nguyên thủy: bộ giá trị (tuple) và mảng.

#### Kiểu Bộ Giá Trị

Một _bộ giá trị_ là một cách chung để nhóm một số giá trị có nhiều kiểu khác
nhau vào một kiểu phức hợp. Bộ giá trị có độ dài cố định: một khi được khai báo,
chúng không thể tăng hoặc giảm kích thước.

Chúng ta tạo một bộ giá trị bằng cách viết một danh sách các giá trị được phân
tách bằng dấu phẩy bên trong dấu ngoặc đơn. Mỗi vị trí trong bộ giá trị có một
kiểu, và các kiểu của các giá trị khác nhau trong bộ giá trị không nhất thiết
phải giống nhau. Chúng tôi đã thêm chú thích kiểu tùy chọn trong ví dụ này:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-10-tuples/src/main.rs}}
```

Biến `tup` gắn với toàn bộ bộ giá trị vì bộ giá trị được coi là một phần tử phức
hợp đơn lẻ. Để lấy các giá trị riêng lẻ từ một bộ giá trị, chúng ta có thể sử
dụng pattern matching để phá hủy cấu trúc một giá trị bộ giá trị, như sau:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-11-destructuring-tuples/src/main.rs}}
```

Chương trình này đầu tiên tạo ra một bộ giá trị và gắn nó với biến `tup`. Sau
đó, nó sử dụng một pattern với `let` để lấy `tup` và biến nó thành ba biến riêng
biệt, `x`, `y` và `z`. Điều này được gọi là _phá hủy cấu trúc_ vì nó chia một bộ
giá trị thành ba phần. Cuối cùng, chương trình in giá trị của `y`, là `6.4`.

Chúng ta cũng có thể truy cập trực tiếp một phần tử của bộ giá trị bằng cách sử
dụng dấu chấm (`.`) theo sau là chỉ số của giá trị mà chúng ta muốn truy cập. Ví
dụ:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-12-tuple-indexing/src/main.rs}}
```

Chương trình này tạo ra bộ giá trị `x` và sau đó truy cập vào mỗi phần tử của bộ
giá trị bằng cách sử dụng các chỉ số tương ứng của chúng. Như với hầu hết các
ngôn ngữ lập trình, chỉ số đầu tiên trong bộ giá trị là 0.

Bộ giá trị không có giá trị nào có một tên đặc biệt, _unit_. Giá trị này và kiểu
tương ứng của nó đều được viết là `()` và đại diện cho một giá trị trống hoặc
một kiểu trả về trống. Biểu thức ngầm định trả về giá trị unit nếu chúng không
trả về bất kỳ giá trị nào khác.

#### Kiểu Mảng

Một cách khác để có một tập hợp gồm nhiều giá trị là với một _mảng_. Không giống
như bộ giá trị, mọi phần tử của mảng phải có cùng kiểu. Không giống như mảng
trong một số ngôn ngữ khác, mảng trong Rust có độ dài cố định.

Chúng ta viết các giá trị trong một mảng như một danh sách được phân tách bằng
dấu phẩy bên trong dấu ngoặc vuông:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-13-arrays/src/main.rs}}
```

Mảng hữu ích khi bạn muốn dữ liệu của mình được phân bổ trên stack, giống như
các kiểu khác mà chúng ta đã thấy cho đến nay, thay vì trên heap (chúng ta sẽ
thảo luận thêm về stack và heap trong [Chương 4][stack-and-heap]<!-- ignore -->)
hoặc khi bạn muốn đảm bảo rằng bạn luôn có một số lượng phần tử cố định. Mảng
không linh hoạt như kiểu vector, tuy nhiên. Một _vector_ là một kiểu tập hợp
tương tự được cung cấp bởi thư viện chuẩn _được_ phép tăng hoặc giảm kích thước
vì nội dung của nó nằm trên heap. Nếu bạn không chắc liệu nên sử dụng một mảng
hay một vector, khả năng là bạn nên sử dụng một vector. [Chương
8][vectors]<!-- ignore --> thảo luận chi tiết hơn về vector.

Tuy nhiên, mảng hữu ích hơn khi bạn biết số lượng phần tử sẽ không cần thay đổi.
Ví dụ, nếu bạn đang sử dụng tên của các tháng trong một chương trình, bạn có lẽ
sẽ sử dụng một mảng thay vì một vector vì bạn biết nó sẽ luôn chứa 12 phần tử:

```rust
let months = ["January", "February", "March", "April", "May", "June", "July",
              "August", "September", "October", "November", "December"];
```

Bạn viết kiểu của một mảng bằng cách sử dụng dấu ngoặc vuông với kiểu của mỗi
phần tử, dấu chấm phẩy, và sau đó là số phần tử trong mảng, như sau:

```rust
let a: [i32; 5] = [1, 2, 3, 4, 5];
```

Ở đây, `i32` là kiểu của mỗi phần tử. Sau dấu chấm phẩy, số `5` cho biết mảng
chứa năm phần tử.

Bạn cũng có thể khởi tạo một mảng để chứa cùng một giá trị cho mỗi phần tử bằng
cách chỉ định giá trị ban đầu, theo sau là dấu chấm phẩy, và sau đó là độ dài
của mảng trong dấu ngoặc vuông, như được hiển thị ở đây:

```rust
let a = [3; 5];
```

Mảng có tên `a` sẽ chứa `5` phần tử mà tất cả sẽ được thiết lập ban đầu với giá
trị `3`. Điều này tương đương với việc viết `let a = [3, 3, 3, 3, 3];` nhưng
theo cách ngắn gọn hơn.

##### Truy Cập Phần Tử Mảng

Một mảng là một khối bộ nhớ đơn với kích thước đã biết, cố định có thể được cấp
phát trên stack. Bạn có thể truy cập các phần tử của mảng bằng cách sử dụng chỉ
số, như sau:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-14-array-indexing/src/main.rs}}
```

Trong ví dụ này, biến có tên `first` sẽ nhận giá trị `1` vì đó là giá trị tại
chỉ số `[0]` trong mảng. Biến có tên `second` sẽ nhận giá trị `2` từ chỉ số
`[1]` trong mảng.

##### Truy Cập Phần Tử Mảng Không Hợp Lệ

Hãy xem điều gì xảy ra nếu bạn cố gắng truy cập một phần tử của mảng ở ngoài
cuối mảng. Giả sử bạn chạy mã này, tương tự như trò chơi đoán số ở Chương 2, để
nhận chỉ số mảng từ người dùng:

<span class="filename">Tên tập tin: src/main.rs</span>

```rust,ignore,panics
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access/src/main.rs}}
```

Mã này biên dịch thành công. Nếu bạn chạy mã này bằng `cargo run` và nhập `0`,
`1`, `2`, `3`, hoặc `4`, chương trình sẽ in ra giá trị tương ứng tại chỉ số đó
trong mảng. Nếu thay vào đó bạn nhập một số ngoài cuối mảng, chẳng hạn như `10`,
bạn sẽ thấy đầu ra như sau:

<!-- manual-regeneration
cd listings/ch03-common-programming-concepts/no-listing-15-invalid-array-access
cargo run
10
-->

```console
thread 'main' panicked at src/main.rs:19:19:
index out of bounds: the len is 5 but the index is 10
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Chương trình dẫn đến lỗi _thời gian chạy_ ở điểm sử dụng một giá trị không hợp
lệ trong phép toán đánh chỉ số. Chương trình kết thúc với một thông báo lỗi và
không thực thi câu lệnh `println!` cuối cùng. Khi bạn cố gắng truy cập một phần
tử bằng cách sử dụng chỉ số, Rust sẽ kiểm tra xem chỉ số mà bạn đã chỉ định có
nhỏ hơn độ dài mảng hay không. Nếu chỉ số lớn hơn hoặc bằng độ dài, Rust sẽ
panic. Kiểm tra này phải được thực hiện ở thời gian chạy, đặc biệt là trong
trường hợp này, vì trình biên dịch không thể nào biết được người dùng sẽ nhập
giá trị nào khi họ chạy mã sau này.

Đây là một ví dụ về nguyên tắc an toàn bộ nhớ của Rust trong hành động. Trong
nhiều ngôn ngữ bậc thấp, loại kiểm tra này không được thực hiện, và khi bạn cung
cấp một chỉ số không chính xác, bộ nhớ không hợp lệ có thể bị truy cập. Rust bảo
vệ bạn khỏi loại lỗi này bằng cách thoát ngay lập tức thay vì cho phép truy cập
bộ nhớ và tiếp tục. Chương 9 thảo luận thêm về xử lý lỗi của Rust và cách bạn có
thể viết mã an toàn, dễ đọc mà không panic cũng không cho phép truy cập bộ nhớ
không hợp lệ.

[comparing-the-guess-to-the-secret-number]:
  ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[twos-complement]: https://en.wikipedia.org/wiki/Two%27s_complement
[control-flow]: ch03-05-control-flow.html#control-flow
[strings]: ch08-02-strings.html#storing-utf-8-encoded-text-with-strings
[stack-and-heap]: ch04-01-what-is-ownership.html#the-stack-and-the-heap
[vectors]: ch08-01-vectors.html
[unrecoverable-errors-with-panic]: ch09-01-unrecoverable-errors-with-panic.html
[appendix_b]: appendix-02-operators.md
