# Lập Trình Một Trò Chơi Đoán Số

Hãy cùng bắt đầu làm quen với Rust thông qua một dự án thực tế! Chương này sẽ
giới thiệu cho bạn một số khái niệm phổ biến trong Rust bằng cách hướng dẫn bạn
sử dụng chúng trong một chương trình thực tế. Bạn sẽ học về `let`, `match`,
phương thức, hàm liên kết, crate bên ngoài và nhiều hơn nữa! Trong các chương
tiếp theo, chúng ta sẽ khám phá những ý tưởng này chi tiết hơn. Trong chương
này, bạn chỉ cần thực hành những kiến thức cơ bản.

Chúng ta sẽ triển khai một bài tập lập trình cổ điển dành cho người mới bắt đầu:
trò chơi đoán số. Trò chơi hoạt động như sau: chương trình sẽ tạo ra một số
nguyên ngẫu nhiên từ 1 đến 100. Sau đó, chương trình sẽ yêu cầu người chơi nhập
vào một số để đoán. Sau khi nhập số đoán, chương trình sẽ cho biết số đoán đó là
quá nhỏ hay quá lớn. Nếu số đoán chính xác, trò chơi sẽ in ra thông báo chúc
mừng và kết thúc.

## Thiết Lập Một Dự Án Mới

Để thiết lập một dự án mới, hãy đi đến thư mục _projects_ mà bạn đã tạo trong
Chương 1 và tạo một dự án mới bằng Cargo, như sau:

```console
$ cargo new guessing_game
$ cd guessing_game
```

Lệnh đầu tiên, `cargo new`, lấy tên của dự án (`guessing_game`) làm đối số đầu
tiên. Lệnh thứ hai sẽ chuyển đến thư mục của dự án mới.

Hãy xem file _Cargo.toml_ đã được tạo ra:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial
rm -rf no-listing-01-cargo-new
cargo new no-listing-01-cargo-new --name guessing_game
cd no-listing-01-cargo-new
cargo run > output.txt 2>&1
cd ../../..
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/Cargo.toml}}
```

Như bạn đã thấy trong Chương 1, `cargo new` tạo ra một chương trình “Hello,
world!” cho bạn. Hãy xem file _src/main.rs_:

<span class="filename">Filename: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/src/main.rs}}
```

Bây giờ hãy biên dịch chương trình “Hello, world!” này và chạy nó trong cùng một
bước bằng cách sử dụng lệnh `cargo run`:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-01-cargo-new/output.txt}}
```

Lệnh `run` rất hữu ích khi bạn cần lặp lại nhanh chóng trên một dự án, như chúng
ta sẽ làm trong trò chơi này, nhanh chóng kiểm tra từng lần lặp trước khi chuyển
sang lần tiếp theo.

Mở lại file _src/main.rs_. Bạn sẽ viết tất cả mã trong file này.

## Xử Lý Một Số Đoán

Phần đầu tiên của chương trình trò chơi đoán số sẽ yêu cầu người dùng nhập vào,
xử lý đầu vào đó và kiểm tra xem đầu vào có đúng dạng mong đợi hay không. Để bắt
đầu, chúng ta sẽ cho phép người chơi nhập vào một số đoán. Nhập mã trong Listing
2-1 vào _src/main.rs_.

<Listing number="2-1" file-name="src/main.rs" caption="Mã lấy số đoán từ người dùng và in ra">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:all}}
```

</Listing>

Mã này chứa rất nhiều thông tin, vì vậy hãy đi qua từng dòng một. Để lấy đầu vào
từ người dùng và sau đó in kết quả ra màn hình, chúng ta cần đưa thư viện đầu
vào/đầu ra `io` vào phạm vi. Thư viện `io` đến từ thư viện tiêu chuẩn, được gọi
là `std`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:io}}
```

Theo mặc định, Rust có một tập hợp các mục được định nghĩa trong thư viện tiêu
chuẩn mà nó đưa vào phạm vi của mọi chương trình. Tập hợp này được gọi là
_prelude_, và bạn có thể xem tất cả trong [tài liệu thư viện tiêu
chuẩn][prelude].

Nếu một kiểu bạn muốn sử dụng không có trong prelude, bạn phải đưa kiểu đó vào
phạm vi rõ ràng bằng một câu lệnh `use`. Sử dụng thư viện `std::io` cung cấp cho
bạn một số tính năng hữu ích, bao gồm khả năng chấp nhận đầu vào từ người dùng.

Như bạn đã thấy trong Chương 1, hàm `main` là điểm vào của chương trình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:main}}
```

Cú pháp `fn` khai báo một hàm mới; dấu ngoặc đơn, `()`, chỉ ra rằng không có
tham số; và dấu ngoặc nhọn, `{`, bắt đầu thân của hàm.

Như bạn cũng đã học trong Chương 1, `println!` là một macro in một chuỗi ra màn
hình:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print}}
```

Mã này đang in ra một lời nhắc cho biết trò chơi là gì và yêu cầu đầu vào từ
người dùng.

### Lưu Trữ Giá Trị Với Biến

Tiếp theo, chúng ta sẽ tạo một _biến_ để lưu trữ đầu vào từ người dùng, như sau:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:string}}
```

Bây giờ chương trình đang trở nên thú vị! Có rất nhiều điều đang diễn ra trong
dòng nhỏ này. Chúng ta sử dụng câu lệnh `let` để tạo biến. Đây là một ví dụ
khác:

```rust,ignore
let apples = 5;
```

Dòng này tạo ra một biến mới có tên là `apples` và gán nó với giá trị 5. Trong
Rust, các biến mặc định là không thay đổi, có nghĩa là một khi chúng ta gán giá
trị cho biến, giá trị đó sẽ không thay đổi. Chúng ta sẽ thảo luận chi tiết về
khái niệm này trong phần [“Biến và Tính Không Thay
Đổi”][variables-and-mutability]<!-- ignore --> trong Chương 3. Để làm cho một
biến có thể thay đổi, chúng ta thêm `mut` trước tên biến:

```rust,ignore
let apples = 5; // không thay đổi
let mut bananas = 5; // có thể thay đổi
```

> Lưu ý: Cú pháp `//` bắt đầu một bình luận kéo dài đến cuối dòng. Rust bỏ qua
> mọi thứ trong bình luận. Chúng ta sẽ thảo luận chi tiết về bình luận trong
> [Chương 3][comments]<!-- ignore -->.

Quay lại chương trình trò chơi đoán số, bây giờ bạn đã biết rằng `let mut guess`
sẽ giới thiệu một biến có thể thay đổi có tên là `guess`. Dấu bằng (`=`) cho
Rust biết chúng ta muốn gán một cái gì đó cho biến ngay bây giờ. Ở bên phải của
dấu bằng là giá trị mà `guess` được gán, đó là kết quả của việc gọi
`String::new`, một hàm trả về một phiên bản mới của một `String`.
[`String`][string]<!-- ignore --> là một kiểu chuỗi được cung cấp bởi thư viện
tiêu chuẩn, là một đoạn văn bản có thể mở rộng, được mã hóa UTF-8.

Cú pháp `::` trong dòng `::new` chỉ ra rằng `new` là một hàm liên kết của kiểu
`String`. Một _hàm liên kết_ là một hàm được triển khai trên một kiểu, trong
trường hợp này là `String`. Hàm `new` này tạo ra một chuỗi mới, trống. Bạn sẽ
tìm thấy một hàm `new` trên nhiều kiểu vì nó là một tên phổ biến cho một hàm tạo
ra một giá trị mới của một loại nào đó.

Tóm lại, dòng `let mut guess = String::new();` đã tạo ra một biến có thể thay
đổi hiện đang được gán với một phiên bản mới, trống của một `String`. Whew!

### Nhận Đầu Vào Từ Người Dùng

Nhớ lại rằng chúng ta đã bao gồm chức năng đầu vào/đầu ra từ thư viện tiêu chuẩn
với `use std::io;` ở dòng đầu tiên của chương trình. Bây giờ chúng ta sẽ gọi hàm
`stdin` từ module `io`, điều này sẽ cho phép chúng ta xử lý đầu vào từ người
dùng:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:read}}
```

Nếu chúng ta không nhập module `io` với `use std::io;` ở đầu chương trình, chúng
ta vẫn có thể sử dụng hàm này bằng cách viết lời gọi hàm này là
`std::io::stdin`. Hàm `stdin` trả về một phiên bản của
[`std::io::Stdin`][iostdin]<!-- ignore -->, là một kiểu đại diện cho một tay cầm
đến đầu vào tiêu chuẩn cho terminal của bạn.

Tiếp theo, dòng `.read_line(&mut guess)` gọi phương thức
[`read_line`][read_line]<!-- ignore --> trên tay cầm đầu vào tiêu chuẩn để lấy
đầu vào từ người dùng. Chúng ta cũng đang truyền `&mut guess` làm đối số cho
`read_line` để cho nó biết chuỗi nào để lưu trữ đầu vào từ người dùng. Công việc
đầy đủ của `read_line` là lấy bất cứ thứ gì người dùng nhập vào đầu vào tiêu
chuẩn và thêm vào một chuỗi (mà không ghi đè nội dung của nó), vì vậy chúng ta
truyền chuỗi đó làm đối số. Đối số chuỗi cần phải có thể thay đổi để phương thức
có thể thay đổi nội dung của chuỗi.

Dấu `&` chỉ ra rằng đối số này là một _tham chiếu_, điều này cho bạn một cách để
cho nhiều phần của mã truy cập vào một phần dữ liệu mà không cần phải sao chép
dữ liệu đó vào bộ nhớ nhiều lần. Tham chiếu là một tính năng phức tạp, và một
trong những lợi thế lớn của Rust là cách sử dụng tham chiếu an toàn và dễ dàng.
Bạn không cần phải biết nhiều chi tiết đó để hoàn thành chương trình này. Hiện
tại, tất cả những gì bạn cần biết là, giống như các biến, các tham chiếu mặc
định là không thay đổi. Do đó, bạn cần viết `&mut guess` thay vì `&guess` để làm
cho nó có thể thay đổi. (Chương 4 sẽ giải thích tham chiếu chi tiết hơn.)

<!-- Old heading. Do not remove or links may break. -->

<a id="handling-potential-failure-with-the-result-type"></a>

### Xử Lý Khả Năng Thất Bại Với `Result`

Chúng ta vẫn đang làm việc trên dòng mã này. Chúng ta hiện đang thảo luận về
dòng văn bản thứ ba, nhưng lưu ý rằng nó vẫn là một phần của một dòng mã logic
duy nhất. Phần tiếp theo là phương thức này:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:expect}}
```

Chúng ta có thể đã viết mã này như sau:

```rust,ignore
io::stdin().read_line(&mut guess).expect("Failed to read line");
```

Tuy nhiên, một dòng dài rất khó đọc, vì vậy tốt nhất là chia nó ra. Thường thì
nên giới thiệu một dòng mới và các khoảng trắng khác để giúp chia nhỏ các dòng
dài khi bạn gọi một phương thức với cú pháp `.method_name()`. Bây giờ hãy thảo
luận về những gì dòng này làm.

Như đã đề cập trước đó, `read_line` đặt bất cứ thứ gì người dùng nhập vào chuỗi
mà chúng ta truyền cho nó, nhưng nó cũng trả về một giá trị `Result`.
[`Result`][result]<!-- ignore --> là một [_liệt kê_][enums]<!-- ignore -->,
thường được gọi là _enum_, là một kiểu có thể ở một trong nhiều trạng thái có
thể. Chúng ta gọi mỗi trạng thái có thể là một _biến thể_.

[Chương 6][enums]<!-- ignore --> sẽ bao gồm các enum chi tiết hơn. Mục đích của
các kiểu `Result` này là để mã hóa thông tin xử lý lỗi.

Các biến thể của `Result` là `Ok` và `Err`. Biến thể `Ok` chỉ ra rằng hoạt động
đã thành công, và nó chứa giá trị được tạo ra thành công. Biến thể `Err` có
nghĩa là hoạt động đã thất bại, và nó chứa thông tin về cách hoặc lý do tại sao
hoạt động thất bại.

Các giá trị của kiểu `Result`, giống như các giá trị của bất kỳ kiểu nào, có các
phương thức được định nghĩa trên chúng. Một phiên bản của `Result` có một phương
thức [`expect`][expect]<!-- ignore --> mà bạn có thể gọi. Nếu phiên bản này của
`Result` là một giá trị `Err`, `expect` sẽ làm cho chương trình bị sập và hiển
thị thông báo mà bạn đã truyền làm đối số cho `expect`. Nếu phương thức
`read_line` trả về một `Err`, nó có thể là kết quả của một lỗi đến từ hệ điều
hành cơ bản. Nếu phiên bản này của `Result` là một giá trị `Ok`, `expect` sẽ lấy
giá trị trả về mà `Ok` đang giữ và trả về chỉ giá trị đó cho bạn để bạn có thể
sử dụng nó. Trong trường hợp này, giá trị đó là số byte trong đầu vào của người
dùng.

Nếu bạn không gọi `expect`, chương trình sẽ biên dịch, nhưng bạn sẽ nhận được
một cảnh báo:

```console
{{#include ../listings/ch02-guessing-game-tutorial/no-listing-02-without-expect/output.txt}}
```

Rust cảnh báo rằng bạn chưa sử dụng giá trị `Result` trả về từ `read_line`, chỉ
ra rằng chương trình chưa xử lý một lỗi có thể xảy ra.

Cách đúng để loại bỏ cảnh báo là thực sự viết mã xử lý lỗi, nhưng trong trường
hợp của chúng ta, chúng ta chỉ muốn làm cho chương trình này bị sập khi có vấn
đề xảy ra, vì vậy chúng ta có thể sử dụng `expect`. Bạn sẽ học về cách khôi phục
từ lỗi trong [Chương 9][recover]<!-- ignore -->.

### In Giá Trị Với `println!` Placeholder

Ngoài dấu ngoặc nhọn đóng, chỉ còn một dòng nữa để thảo luận trong mã cho đến
nay:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-01/src/main.rs:print_guess}}
```

Dòng này in ra chuỗi hiện chứa đầu vào của người dùng. Bộ `{}` của dấu ngoặc
nhọn là một placeholder: hãy nghĩ về `{}` như những cái kẹp nhỏ giữ một giá trị
tại chỗ. Khi in giá trị của một biến, tên biến có thể được đặt bên trong dấu
ngoặc nhọn. Khi in kết quả của việc đánh giá một biểu thức, đặt dấu ngoặc nhọn
trống trong chuỗi định dạng, sau đó theo sau chuỗi định dạng với một danh sách
các biểu thức được phân tách bằng dấu phẩy để in trong mỗi placeholder dấu ngoặc
nhọn trống theo cùng thứ tự. In một biến và kết quả của một biểu thức trong một
lần gọi `println!` sẽ trông như thế này:

```rust
let x = 5;
let y = 10;

println!("x = {x} and y + 2 = {}", y + 2);
```

Mã này sẽ in `x = 5 and y + 2 = 12`.

### Kiểm Tra Phần Đầu Tiên

Hãy kiểm tra phần đầu tiên của trò chơi đoán số. Chạy nó bằng cách sử dụng
`cargo run`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-01/
cargo clean
cargo run
input 6 -->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 6.44s
     Running `target/debug/guessing_game`
Guess the number!
Please input your guess.
6
You guessed: 6
```

Tại thời điểm này, phần đầu tiên của trò chơi đã hoàn thành: chúng ta đang nhận
đầu vào từ bàn phím và sau đó in nó ra.

## Tạo Một Số Bí Mật

Tiếp theo, chúng ta cần tạo ra một số bí mật mà người dùng sẽ cố gắng đoán. Số
bí mật nên khác nhau mỗi lần để trò chơi thú vị hơn khi chơi nhiều lần. Chúng ta
sẽ sử dụng một số ngẫu nhiên từ 1 đến 100 để trò chơi không quá khó. Rust chưa
bao gồm chức năng tạo số ngẫu nhiên trong thư viện tiêu chuẩn của nó. Tuy nhiên,
nhóm Rust cung cấp một crate [`rand`][randcrate] với chức năng này.

### Sử Dụng Một Crate Để Có Thêm Chức Năng

Nhớ rằng một crate là một tập hợp các file mã nguồn Rust. Dự án mà chúng ta đã
xây dựng là một _binary crate_, là một tệp thực thi. Crate `rand` là một
_library crate_, chứa mã được sử dụng trong các chương trình khác và không thể
thực thi độc lập.

Sự phối hợp của Cargo với các crate bên ngoài là nơi Cargo thực sự tỏa sáng.
Trước khi chúng ta có thể viết mã sử dụng `rand`, chúng ta cần sửa đổi file
_Cargo.toml_ để bao gồm crate `rand` làm phụ thuộc. Mở file đó ngay bây giờ và
thêm dòng sau vào cuối, bên dưới phần tiêu đề `[dependencies]` mà Cargo đã tạo
cho bạn. Hãy chắc chắn chỉ định `rand` chính xác như chúng tôi đã làm ở đây, với
số phiên bản này, hoặc các ví dụ mã trong hướng dẫn này có thể không hoạt động:

<!-- Khi cập nhật phiên bản của `rand` được sử dụng, cũng cập nhật phiên bản của
`rand` được sử dụng trong các file sau để chúng tất cả khớp:
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
* ch14-03-cargo-workspaces.md
-->

<span class="filename">Filename: Cargo.toml</span>

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:8:}}
```

Trong file _Cargo.toml_, mọi thứ theo sau một tiêu đề là một phần của phần đó
tiếp tục cho đến khi một phần khác bắt đầu. Trong `[dependencies]` bạn nói với
Cargo những crate bên ngoài mà dự án của bạn phụ thuộc vào và các phiên bản của
các crate đó mà bạn yêu cầu. Trong trường hợp này, chúng ta chỉ định crate
`rand` với chỉ định phiên bản ngữ nghĩa `0.8.5`. Cargo hiểu [Semantic
Versioning][semver]<!-- ignore --> (đôi khi được gọi là _SemVer_), là một tiêu
chuẩn để viết số phiên bản. Chỉ định `0.8.5` thực sự là viết tắt của `^0.8.5`,
có nghĩa là bất kỳ phiên bản nào ít nhất là 0.8.5 nhưng dưới 0.9.0.

Cargo coi các phiên bản này có API công khai tương thích với phiên bản 0.8.5, và
chỉ định này đảm bảo bạn sẽ nhận được bản phát hành vá lỗi mới nhất mà vẫn biên
dịch được với mã trong chương này. Bất kỳ phiên bản nào từ 0.9.0 trở lên không
được đảm bảo có cùng API như những gì các ví dụ sau đây sử dụng.

Bây giờ, mà không thay đổi bất kỳ mã nào, hãy xây dựng dự án, như được hiển thị
trong Listing 2-2.

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
rm Cargo.lock
cargo clean
cargo build -->

<Listing number="2-2" caption="Kết quả từ việc chạy `cargo build` sau khi thêm crate rand làm phụ thuộc">

```console
$ cargo build
  Updating crates.io index
   Locking 15 packages to latest Rust 1.85.0 compatible versions
    Adding rand v0.8.5 (available: v0.9.0)
 Compiling proc-macro2 v1.0.93
 Compiling unicode-ident v1.0.17
 Compiling libc v0.2.170
 Compiling cfg-if v1.0.0
 Compiling byteorder v1.5.0
 Compiling getrandom v0.2.15
 Compiling rand_core v0.6.4
 Compiling quote v1.0.38
 Compiling syn v2.0.98
 Compiling zerocopy-derive v0.7.35
 Compiling zerocopy v0.7.35
 Compiling ppv-lite86 v0.2.20
 Compiling rand_chacha v0.3.1
 Compiling rand v0.8.5
 Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
  Finished `dev` profile [unoptimized + debuginfo] target(s) in 2.48s
```

</Listing>

Bạn có thể thấy các số phiên bản khác nhau (nhưng tất cả sẽ tương thích với mã,
nhờ SemVer!) và các dòng khác nhau (tùy thuộc vào hệ điều hành), và các dòng có
thể ở một thứ tự khác.

Khi chúng ta bao gồm một phụ thuộc bên ngoài, Cargo sẽ lấy các phiên bản mới
nhất của mọi thứ mà phụ thuộc đó cần từ _registry_, là một bản sao của dữ liệu
từ [Crates.io][cratesio]. Crates.io là nơi mọi người trong hệ sinh thái Rust
đăng các dự án Rust mã nguồn mở của họ để người khác sử dụng.

Sau khi cập nhật registry, Cargo kiểm tra phần `[dependencies]` và tải xuống bất
kỳ crate nào được liệt kê mà chưa được tải xuống. Trong trường hợp này, mặc dù
chúng ta chỉ liệt kê `rand` làm phụ thuộc, Cargo cũng đã lấy các crate khác mà
`rand` phụ thuộc vào để hoạt động. Sau khi tải xuống các crate, Rust biên dịch
chúng và sau đó biên dịch dự án với các phụ thuộc có sẵn.

Nếu bạn ngay lập tức chạy `cargo build` lại mà không thực hiện bất kỳ thay đổi
nào, bạn sẽ không nhận được bất kỳ kết quả nào ngoài dòng `Finished`. Cargo biết
rằng nó đã tải xuống và biên dịch các phụ thuộc, và bạn chưa thay đổi bất kỳ
điều gì về chúng trong file _Cargo.toml_ của bạn. Cargo cũng biết rằng bạn chưa
thay đổi bất kỳ điều gì về mã của bạn, vì vậy nó không biên dịch lại mã đó.
Không có gì để làm, nó chỉ đơn giản thoát ra.

Nếu bạn mở file _src/main.rs_, thực hiện một thay đổi nhỏ và sau đó lưu nó và
xây dựng lại, bạn sẽ chỉ thấy hai dòng kết quả:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
touch src/main.rs
cargo build -->

```console
$ cargo build
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
```

Những dòng này cho thấy rằng Cargo chỉ cập nhật bản dựng với thay đổi nhỏ của
bạn đối với file _src/main.rs_. Các phụ thuộc của bạn không thay đổi, vì vậy
Cargo biết rằng nó có thể tái sử dụng những gì nó đã tải xuống và biên dịch cho
những phụ thuộc đó.

#### Đảm Bảo Các Bản Dựng Có Thể Tái Tạo Với File _Cargo.lock_

Cargo có một cơ chế đảm bảo bạn có thể tái tạo cùng một artifact mỗi khi bạn
hoặc bất kỳ ai khác xây dựng mã của bạn: Cargo sẽ chỉ sử dụng các phiên bản của
các phụ thuộc mà bạn đã chỉ định cho đến khi bạn chỉ định khác. Ví dụ, giả sử
rằng tuần tới phiên bản 0.8.6 của crate `rand` ra mắt, và phiên bản đó chứa một
bản sửa lỗi quan trọng, nhưng nó cũng chứa một lỗi hồi quy sẽ làm hỏng mã của
bạn. Để xử lý điều này, Rust tạo file _Cargo.lock_ lần đầu tiên bạn chạy
`cargo build`, vì vậy bây giờ chúng ta có file này trong thư mục
_guessing_game_.

Khi bạn xây dựng một dự án lần đầu tiên, Cargo sẽ tìm ra tất cả các phiên bản
của các phụ thuộc phù hợp với tiêu chí và sau đó ghi chúng vào file
_Cargo.lock_. Khi bạn xây dựng dự án của mình trong tương lai, Cargo sẽ thấy
rằng file _Cargo.lock_ tồn tại và sẽ sử dụng các phiên bản được chỉ định ở đó
thay vì làm tất cả công việc tìm ra các phiên bản lại. Điều này cho phép bạn có
một bản dựng có thể tái tạo tự động. Nói cách khác, dự án của bạn sẽ vẫn ở phiên
bản 0.8.5 cho đến khi bạn nâng cấp rõ ràng, nhờ vào file _Cargo.lock_. Vì file
_Cargo.lock_ quan trọng đối với các bản dựng có thể tái tạo, nó thường được kiểm
tra vào kiểm soát nguồn cùng với phần còn lại của mã trong dự án của bạn.

#### Cập Nhật Một Crate Để Nhận Phiên Bản Mới

Khi bạn _muốn_ cập nhật một crate, Cargo cung cấp lệnh `update`, lệnh này sẽ bỏ
qua file _Cargo.lock_ và tìm ra tất cả các phiên bản mới nhất phù hợp với các
chỉ định của bạn trong _Cargo.toml_. Cargo sau đó sẽ ghi các phiên bản đó vào
file _Cargo.lock_. Trong trường hợp này, Cargo sẽ chỉ tìm các phiên bản lớn hơn
0.8.5 và nhỏ hơn 0.9.0. Nếu crate `rand` đã phát hành hai phiên bản mới 0.8.6 và
0.9.0, bạn sẽ thấy điều sau nếu bạn chạy `cargo update`:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-02/
cargo update
assuming there is a new 0.8.x version of rand; otherwise use another update
as a guide to creating the hypothetical output shown here -->

```console
$ cargo update
    Updating crates.io index
     Locking 1 package to latest Rust 1.85.0 compatible version
    Updating rand v0.8.5 -> v0.8.6 (available: v0.9.0)
```

Cargo bỏ qua phiên bản 0.9.0. Tại thời điểm này, bạn cũng sẽ nhận thấy một thay
đổi trong file _Cargo.lock_ ghi nhận rằng phiên bản của crate `rand` mà bạn đang
sử dụng bây giờ là 0.8.6. Để sử dụng phiên bản 0.9.0 của `rand` hoặc bất kỳ
phiên bản nào trong loạt 0.9._x_, bạn sẽ phải cập nhật file _Cargo.toml_ để
trông như thế này:

```toml
[dependencies]
rand = "0.9.0"
```

Lần tiếp theo bạn chạy `cargo build`, Cargo sẽ cập nhật registry của các crate
có sẵn và đánh giá lại các yêu cầu của bạn về `rand` theo phiên bản mới mà bạn
đã chỉ định.

Có rất nhiều điều để nói về [Cargo][doccargo]<!-- ignore --> và [hệ sinh thái
của nó][doccratesio]<!-- ignore -->, mà chúng ta sẽ thảo luận trong Chương 14,
nhưng hiện tại, đó là tất cả những gì bạn cần biết. Cargo làm cho việc tái sử
dụng các thư viện rất dễ dàng, vì vậy các Rustaceans có thể viết các dự án nhỏ
hơn được lắp ráp từ một số gói.

### Tạo Một Số Ngẫu Nhiên

Hãy bắt đầu sử dụng `rand` để tạo ra một số để đoán. Bước tiếp theo là cập nhật
_src/main.rs_, như được hiển thị trong Listing 2-3.

<Listing number="2-3" file-name="src/main.rs" caption="Thêm mã để tạo ra một số ngẫu nhiên">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:all}}
```

</Listing>

Đầu tiên chúng ta thêm dòng `use rand::Rng;`. Trait `Rng` định nghĩa các phương
thức mà các bộ tạo số ngẫu nhiên triển khai, và trait này phải nằm trong phạm vi
để chúng ta sử dụng các phương thức đó. Chương 10 sẽ bao gồm các trait chi tiết.

Tiếp theo, chúng ta thêm hai dòng ở giữa. Trong dòng đầu tiên, chúng ta gọi hàm
`rand::thread_rng` để lấy bộ tạo số ngẫu nhiên cụ thể mà chúng ta sẽ sử dụng:
một bộ tạo số ngẫu nhiên cục bộ cho luồng thực thi hiện tại và được khởi tạo bởi
hệ điều hành. Sau đó, chúng ta gọi phương thức `gen_range` trên bộ tạo số ngẫu
nhiên. Phương thức này được định nghĩa bởi trait `Rng` mà chúng ta đã đưa vào
phạm vi với câu lệnh `use rand::Rng;`. Phương thức `gen_range` lấy một biểu thức
phạm vi làm đối số và tạo ra một số ngẫu nhiên trong phạm vi đó. Loại biểu thức
phạm vi mà chúng ta đang sử dụng ở đây có dạng `start..=end` và bao gồm cả giới
hạn dưới và giới hạn trên, vì vậy chúng ta cần chỉ định `1..=100` để yêu cầu một
số từ 1 đến 100.

> Lưu ý: Bạn sẽ không chỉ biết các trait nào để sử dụng và các phương thức và
> hàm nào để gọi từ một crate, vì vậy mỗi crate có tài liệu với hướng dẫn sử
> dụng nó. Một tính năng thú vị khác của Cargo là chạy lệnh `cargo doc --open`
> sẽ xây dựng tài liệu được cung cấp bởi tất cả các phụ thuộc của bạn cục bộ và
> mở nó trong trình duyệt của bạn. Nếu bạn quan tâm đến các chức năng khác trong
> crate `rand`, ví dụ, hãy chạy `cargo doc --open` và nhấp vào `rand` trong
> thanh bên trái.

Dòng mới thứ hai in ra số bí mật. Điều này hữu ích trong khi chúng ta đang phát
triển chương trình để có thể kiểm tra nó, nhưng chúng ta sẽ xóa nó khỏi phiên
bản cuối cùng. Nó không phải là một trò chơi nếu chương trình in ra câu trả lời
ngay khi nó bắt đầu!

Hãy thử chạy chương trình một vài lần:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-03/
cargo run
4
cargo run
5
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 7
Please input your guess.
4
You guessed: 4

$ cargo run
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.02s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 83
Please input your guess.
5
You guessed: 5
```

Bạn nên nhận được các số ngẫu nhiên khác nhau, và tất cả chúng nên là các số từ
1 đến 100. Làm tốt lắm!

## So Sánh Số Đoán Với Số Bí Mật

Bây giờ chúng ta đã có đầu vào từ người dùng và một số ngẫu nhiên, chúng ta có
thể so sánh chúng. Bước đó được hiển thị trong Listing 2-4. Lưu ý rằng mã này sẽ
không biên dịch ngay lập tức, như chúng ta sẽ giải thích.

<Listing number="2-4" file-name="src/main.rs" caption="Xử lý các giá trị trả về có thể của việc so sánh hai số">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-04/src/main.rs:here}}
```

</Listing>

Đầu tiên chúng ta thêm một câu lệnh `use` khác, đưa một kiểu gọi là
`std::cmp::Ordering` vào phạm vi từ thư viện tiêu chuẩn. Kiểu `Ordering` là một
enum khác và có các biến thể `Less`, `Greater` và `Equal`. Đây là ba kết quả có
thể xảy ra khi bạn so sánh hai giá trị.

Sau đó, chúng ta thêm năm dòng mới ở cuối sử dụng kiểu `Ordering`. Phương thức
`cmp` so sánh hai giá trị và có thể được gọi trên bất kỳ thứ gì có thể so sánh.
Nó lấy một tham chiếu đến bất kỳ thứ gì bạn muốn so sánh: ở đây nó đang so sánh
`guess` với `secret_number`. Sau đó, nó trả về một biến thể của enum `Ordering`
mà chúng ta đã đưa vào phạm vi với câu lệnh `use`. Chúng ta sử dụng một biểu
thức [`match`][match]<!-- ignore --> để quyết định làm gì tiếp theo dựa trên
biến thể nào của `Ordering` được trả về từ lời gọi đến `cmp` với các giá trị
trong `guess` và `secret_number`.

Một biểu thức `match` được tạo thành từ các _nhánh_. Một nhánh bao gồm một _mẫu_
để so khớp, và mã sẽ được chạy nếu giá trị được đưa vào `match` phù hợp với mẫu
của nhánh đó. Rust lấy giá trị được đưa vào `match` và xem qua mẫu của từng
nhánh lần lượt. Các mẫu và cấu trúc `match` là các tính năng mạnh mẽ của Rust:
chúng cho phép bạn biểu đạt nhiều tình huống mà mã của bạn có thể gặp phải và
chúng đảm bảo bạn xử lý tất cả chúng. Các tính năng này sẽ được bao gồm chi tiết
trong Chương 6 và Chương 19, tương ứng.

Hãy đi qua một ví dụ với biểu thức `match` mà chúng ta sử dụng ở đây. Giả sử
rằng người dùng đã đoán 50 và số bí mật được tạo ngẫu nhiên lần này là 38.

Khi mã so sánh 50 với 38, phương thức `cmp` sẽ trả về `Ordering::Greater` vì 50
lớn hơn 38. Biểu thức `match` nhận giá trị `Ordering::Greater` và bắt đầu kiểm
tra mẫu của từng nhánh. Nó xem mẫu của nhánh đầu tiên, `Ordering::Less`, và thấy
rằng giá trị `Ordering::Greater` không phù hợp với `Ordering::Less`, vì vậy nó
bỏ qua mã trong nhánh đó và chuyển sang nhánh tiếp theo. Mẫu của nhánh tiếp theo
là `Ordering::Greater`, điều này _phù hợp_ với `Ordering::Greater`! Mã liên kết
trong nhánh đó sẽ được thực thi và in ra `Too big!` trên màn hình. Biểu thức
`match` kết thúc sau khi khớp thành công đầu tiên, vì vậy nó sẽ không xem nhánh
cuối cùng trong tình huống này.

Tuy nhiên, mã trong Listing 2-4 sẽ không biên dịch ngay lập tức. Hãy thử nó:

<!--
Các số lỗi trong kết quả này nên là của mã **KHÔNG CÓ** các
anchor hoặc snip comments
-->

```console
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-04/output.txt}}
```

Lỗi cốt lõi cho biết rằng có _các kiểu không khớp_. Rust có một hệ thống kiểu
mạnh mẽ và tĩnh. Tuy nhiên, nó cũng có suy luận kiểu. Khi chúng ta viết
`let mut guess = String::new()`, Rust có thể suy luận rằng `guess` nên là một
`String` và không bắt chúng ta phải viết kiểu. Mặt khác, `secret_number` là một
kiểu số. Một vài kiểu số của Rust có thể có giá trị từ 1 đến 100: `i32`, một số
32-bit; `u32`, một số 32-bit không dấu; `i64`, một số 64-bit; cũng như các kiểu
khác. Trừ khi được chỉ định khác, Rust mặc định là một `i32`, đó là kiểu của
`secret_number` trừ khi bạn thêm thông tin kiểu ở nơi khác để Rust suy luận một
kiểu số khác. Lý do cho lỗi là Rust không thể so sánh một chuỗi và một kiểu số.

Cuối cùng, chúng ta muốn chuyển đổi `String` mà chương trình đọc làm đầu vào
thành một kiểu số để chúng ta có thể so sánh nó về mặt số học với số bí mật.
Chúng ta làm điều đó bằng cách thêm dòng này vào thân hàm `main`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/src/main.rs:here}}
```

Dòng này là:

```rust,ignore
let guess: u32 = guess.trim().parse().expect("Please type a number!");
```

Chúng ta tạo một biến có tên là `guess`. Nhưng chờ đã, chương trình đã có một
biến có tên là `guess` chưa? Đúng vậy, nhưng may mắn thay Rust cho phép chúng ta
che bóng giá trị trước đó của `guess` bằng một giá trị mới. _Che bóng_ cho phép
chúng ta tái sử dụng tên biến `guess` thay vì buộc chúng ta phải tạo hai biến
duy nhất, chẳng hạn như `guess_str` và `guess`, ví dụ. Chúng ta sẽ bao gồm điều
này chi tiết hơn trong [Chương 3][shadowing]<!-- ignore -->, nhưng hiện tại, hãy
biết rằng tính năng này thường được sử dụng khi bạn muốn chuyển đổi một giá trị
từ một kiểu sang một kiểu khác.

Chúng ta gán biến mới này với biểu thức `guess.trim().parse()`. `guess` trong
biểu thức đề cập đến biến `guess` ban đầu chứa đầu vào dưới dạng chuỗi. Phương
thức `trim` trên một phiên bản `String` sẽ loại bỏ bất kỳ khoảng trắng nào ở đầu
và cuối, điều mà chúng ta phải làm trước khi có thể chuyển đổi chuỗi thành một
`u32`, chỉ có thể chứa dữ liệu số. Người dùng phải nhấn <kbd>enter</kbd> để thỏa
mãn `read_line` và nhập số đoán của họ, điều này thêm một ký tự xuống dòng vào
chuỗi. Ví dụ, nếu người dùng nhập <kbd>5</kbd> và nhấn <kbd>enter</kbd>, `guess`
trông như thế này: `5\n`. `\n` đại diện cho “xuống dòng.” (Trên Windows, nhấn
<kbd>enter</kbd> sẽ dẫn đến một ký tự xuống dòng và một ký tự trở về, `\r\n`.)
Phương thức `trim` loại bỏ `\n` hoặc `\r\n`, chỉ để lại `5`.

Phương thức [`parse` trên chuỗi][parse]<!-- ignore --> chuyển đổi một chuỗi
thành một kiểu khác. Ở đây, chúng ta sử dụng nó để chuyển đổi từ một chuỗi thành
một số. Chúng ta cần nói với Rust kiểu số chính xác mà chúng ta muốn bằng cách
sử dụng `let guess: u32`. Dấu hai chấm (`:`) sau `guess` cho Rust biết rằng
chúng ta sẽ chú thích kiểu của biến. Rust có một vài kiểu số tích hợp; `u32`
được thấy ở đây là một số nguyên 32-bit không dấu. Đó là một lựa chọn mặc định
tốt cho một số dương nhỏ. Bạn sẽ học về các kiểu số khác trong [Chương
3][integers]<!-- ignore -->.

Ngoài ra, chú thích `u32` trong chương trình ví dụ này và so sánh với
`secret_number` có nghĩa là Rust sẽ suy luận rằng `secret_number` cũng nên là
một `u32`. Vì vậy, bây giờ so sánh sẽ là giữa hai giá trị cùng kiểu!

Phương thức `parse` sẽ chỉ hoạt động trên các ký tự có thể chuyển đổi hợp lý
thành số và do đó có thể dễ dàng gây ra lỗi. Nếu, ví dụ, chuỗi chứa `A👍%`, sẽ
không có cách nào để chuyển đổi điều đó thành một số. Vì nó có thể thất bại,
phương thức `parse` trả về một kiểu `Result`, giống như phương thức `read_line`
(được thảo luận trước đó trong
[“Xử Lý Khả Năng Thất Bại Với `Result`”](#handling-potential-failure-with-result)<!-- ignore-->).
Chúng ta sẽ xử lý `Result` này theo cùng cách bằng cách sử dụng phương thức
`expect` một lần nữa. Nếu `parse` trả về một biến thể `Err` của `Result` vì nó
không thể tạo ra một số từ chuỗi, lời gọi `expect` sẽ làm cho trò chơi bị sập và
in ra thông báo mà chúng ta đưa ra. Nếu `parse` có thể chuyển đổi thành công
chuỗi thành một số, nó sẽ trả về biến thể `Ok` của `Result`, và `expect` sẽ trả
về số mà chúng ta muốn từ giá trị `Ok`.

Hãy chạy chương trình ngay bây giờ:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-03-convert-string-to-number/
touch src/main.rs
cargo run
  76
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.26s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 58
Please input your guess.
  76
You guessed: 76
Too big!
```

Tuyệt vời! Mặc dù đã thêm các khoảng trắng trước số đoán, chương trình vẫn nhận
ra rằng người dùng đã đoán 76. Chạy chương trình một vài lần để xác minh hành vi
khác nhau với các loại đầu vào khác nhau: đoán số chính xác, đoán một số quá lớn
và đoán một số quá nhỏ.

Chúng ta đã có hầu hết trò chơi hoạt động, nhưng người dùng chỉ có thể đoán một
lần. Hãy thay đổi điều đó bằng cách thêm một vòng lặp!

## Cho Phép Nhiều Lần Đoán Với Vòng Lặp

Từ khóa `loop` tạo ra một vòng lặp vô hạn. Chúng ta sẽ thêm một vòng lặp để cho
phép người dùng có nhiều cơ hội đoán số hơn:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-04-looping/src/main.rs:here}}
```

Như bạn có thể thấy, chúng ta đã di chuyển mọi thứ từ lời nhắc nhập số đoán trở
đi vào một vòng lặp. Hãy chắc chắn thụt lề các dòng bên trong vòng lặp thêm bốn
khoảng trắng mỗi dòng và chạy chương trình lại. Chương trình bây giờ sẽ yêu cầu
một số đoán khác mãi mãi, điều này thực sự giới thiệu một vấn đề mới. Dường như
người dùng không thể thoát ra!

Người dùng luôn có thể ngắt chương trình bằng cách sử dụng phím tắt
<kbd>ctrl</kbd>-<kbd>c</kbd>. Nhưng có một cách khác để thoát khỏi con quái vật
không thể thỏa mãn này, như đã đề cập trong phần thảo luận về `parse` trong
[“So Sánh Số Đoán Với Số Bí Mật”](#comparing-the-guess-to-the-secret-number)<!-- ignore -->:
nếu người dùng nhập một câu trả lời không phải là số, chương trình sẽ bị sập.
Chúng ta có thể tận dụng điều đó để cho phép người dùng thoát ra, như được hiển
thị ở đây:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/no-listing-04-looping/
touch src/main.rs
cargo run
(too small guess)
(too big guess)
(correct guess)
quit
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.23s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 59
Please input your guess.
45
You guessed: 45
Too small!
Please input your guess.
60
You guessed: 60
Too big!
Please input your guess.
59
You guessed: 59
You win!
Please input your guess.
quit

thread 'main' panicked at src/main.rs:28:47:
Please type a number!: ParseIntError { kind: InvalidDigit }
note: run with `RUST_BACKTRACE=1` environment variable to display a backtrace
```

Nhập `quit` sẽ thoát khỏi trò chơi, nhưng như bạn sẽ nhận thấy, việc nhập bất kỳ
đầu vào không phải là số nào khác cũng sẽ thoát ra. Điều này là không tối ưu, ít
nhất là; chúng ta muốn trò chơi cũng dừng lại khi số đoán chính xác.

### Thoát Sau Khi Đoán Đúng

Hãy lập trình trò chơi để thoát khi người dùng thắng bằng cách thêm một câu lệnh
`break`:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/no-listing-05-quitting/src/main.rs:here}}
```

Thêm dòng `break` sau `You win!` làm cho chương trình thoát khỏi vòng lặp khi
người dùng đoán đúng số bí mật. Thoát khỏi vòng lặp cũng có nghĩa là thoát khỏi
chương trình, vì vòng lặp là phần cuối cùng của `main`.

### Xử Lý Đầu Vào Không Hợp Lệ

Để tinh chỉnh thêm hành vi của trò chơi, thay vì làm cho chương trình bị sập khi
người dùng nhập vào một số không phải là số, hãy làm cho trò chơi bỏ qua số
không phải là số để người dùng có thể tiếp tục đoán. Chúng ta có thể làm điều đó
bằng cách thay đổi dòng mà `guess` được chuyển đổi từ một `String` thành một
`u32`, như được hiển thị trong Listing 2-5.

<Listing number="2-5" file-name="src/main.rs" caption="Bỏ qua một số đoán không phải là số và yêu cầu một số đoán khác thay vì làm cho chương trình bị sập">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-05/src/main.rs:here}}
```

</Listing>

Chúng ta chuyển từ một lời gọi `expect` sang một biểu thức `match` để chuyển từ
việc làm cho chương trình bị sập khi có lỗi sang xử lý lỗi. Nhớ rằng `parse` trả
về một kiểu `Result` và `Result` là một enum có các biến thể `Ok` và `Err`.
Chúng ta đang sử dụng một biểu thức `match` ở đây, như chúng ta đã làm với kết
quả `Ordering` của phương thức `cmp`.

Nếu `parse` có thể chuyển đổi thành công chuỗi thành một số, nó sẽ trả về một
giá trị `Ok` chứa số kết quả. Giá trị `Ok` đó sẽ khớp với mẫu của nhánh đầu
tiên, và biểu thức `match` sẽ chỉ trả về giá trị `num` mà `parse` đã tạo ra và
đặt bên trong giá trị `Ok`. Số đó sẽ kết thúc ngay tại nơi chúng ta muốn trong
biến `guess` mới mà chúng ta đang tạo.

Nếu `parse` _không_ thể chuyển đổi chuỗi thành một số, nó sẽ trả về một giá trị
`Err` chứa thêm thông tin về lỗi. Giá trị `Err` không khớp với mẫu `Ok(num)`
trong nhánh đầu tiên của `match`, nhưng nó khớp với mẫu `Err(_)` trong nhánh thứ
hai. Dấu gạch dưới, `_`, là một giá trị bắt tất cả; trong ví dụ này, chúng ta
đang nói rằng chúng ta muốn khớp với tất cả các giá trị `Err`, bất kể thông tin
nào chúng có bên trong. Vì vậy, chương trình sẽ thực thi mã của nhánh thứ hai,
`continue`, điều này cho chương trình biết đi đến lần lặp tiếp theo của `loop`
và yêu cầu một số đoán khác. Vì vậy, về cơ bản, chương trình bỏ qua tất cả các
lỗi mà `parse` có thể gặp phải!

Bây giờ mọi thứ trong chương trình nên hoạt động như mong đợi. Hãy thử nó:

<!-- manual-regeneration
cd listings/ch02-guessing-game-tutorial/listing-02-05/
cargo run
(too small guess)
(too big guess)
foo
(correct guess)
-->

```console
$ cargo run
   Compiling guessing_game v0.1.0 (file:///projects/guessing_game)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.13s
     Running `target/debug/guessing_game`
Guess the number!
The secret number is: 61
Please input your guess.
10
You guessed: 10
Too small!
Please input your guess.
99
You guessed: 99
Too big!
Please input your guess.
foo
Please input your guess.
61
You guessed: 61
You win!
```

Tuyệt vời! Với một tinh chỉnh nhỏ cuối cùng, chúng ta sẽ hoàn thành trò chơi
đoán số. Nhớ rằng chương trình vẫn đang in ra số bí mật. Điều đó hoạt động tốt
cho việc kiểm tra, nhưng nó làm hỏng trò chơi. Hãy xóa `println!` in ra số bí
mật. Listing 2-6 hiển thị mã hoàn chỉnh.

<Listing number="2-6" file-name="src/main.rs" caption="Mã hoàn chỉnh của trò chơi đoán số">

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-06/src/main.rs}}
```

</Listing>

Tại thời điểm này, bạn đã xây dựng thành công trò chơi đoán số. Chúc mừng!

## Tóm Tắt

Dự án này là một cách thực hành để giới thiệu cho bạn nhiều khái niệm mới trong
Rust: `let`, `match`, hàm, việc sử dụng các crate bên ngoài và nhiều hơn nữa.
Trong các chương tiếp theo, bạn sẽ học về các khái niệm này chi tiết hơn. Chương
3 bao gồm các khái niệm mà hầu hết các ngôn ngữ lập trình đều có, chẳng hạn như
biến, kiểu dữ liệu và hàm, và chỉ cho bạn cách sử dụng chúng trong Rust. Chương
4 khám phá quyền sở hữu, một tính năng làm cho Rust khác biệt so với các ngôn
ngữ khác. Chương 5 thảo luận về cấu trúc và cú pháp phương thức, và Chương 6
giải thích cách hoạt động của các enum.

[prelude]: ../std/prelude/index.html
[variables-and-mutability]:
  ch03-01-variables-and-mutability.html#variables-and-mutability
[comments]: ch03-04-comments.html
[string]: ../std/string/struct.String.html
[iostdin]: ../std/io/struct.Stdin.html
[read_line]: ../std/io/struct.Stdin.html#method.read_line
[result]: ../std/result/enum.Result.html
[enums]: ch06-00-enums.html
[expect]: ../std/result/enum.Result.html#method.expect
[recover]: ch09-02-recoverable-errors-with-result.html
[randcrate]: https://crates.io/crates/rand
[semver]: http://semver.org
[cratesio]: https://crates.io/
[doccargo]: https://doc.rust-lang.org/cargo/
[doccratesio]: https://doc.rust-lang.org/cargo/reference/publishing.html
[match]: ch06-02-match.html
[shadowing]: ch03-01-variables-and-mutability.html#shadowing
[parse]: ../std/primitive.str.html#method.parse
[integers]: ch03-02-data-types.html#integer-types
