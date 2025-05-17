## Xin chào, Thế giới!

Bây giờ bạn đã cài đặt Rust, đã đến lúc viết chương trình Rust đầu tiên của bạn.
Theo truyền thống khi học một ngôn ngữ mới, chúng ta thường viết một chương
trình nhỏ in ra dòng chữ `Hello, world!` lên màn hình, vì vậy chúng ta sẽ làm
điều tương tự ở đây!

> Lưu ý: Cuốn sách này giả định bạn đã có kiến thức cơ bản về dòng lệnh. Rust
> không đưa ra yêu cầu cụ thể nào về công cụ soạn thảo hoặc nơi lưu trữ mã của
> bạn, vì vậy nếu bạn thích sử dụng IDE thay vì dòng lệnh, hãy tự nhiên sử dụng
> IDE yêu thích của bạn. Nhiều IDE hiện nay có một số mức độ hỗ trợ Rust; hãy
> kiểm tra tài liệu của IDE để biết chi tiết. Đội ngũ Rust đã tập trung vào việc
> hỗ trợ IDE thông qua `rust-analyzer`. Xem [Phụ lục D][devtools]<!-- ignore -->
> để biết thêm chi tiết.

<!-- Old headings. Do not remove or links may break. -->
<a id="creating-a-project-directory"></a>

### Thiết lập thư mục dự án

Bạn sẽ bắt đầu bằng việc tạo một thư mục để lưu trữ mã Rust của bạn. Đối với
Rust, nơi lưu trữ mã của bạn không quan trọng, nhưng đối với các bài tập và dự
án trong sách này, chúng tôi gợi ý tạo một thư mục _projects_ trong thư mục home
của bạn và lưu giữ tất cả các dự án của bạn ở đó.

Mở một terminal và nhập các lệnh sau để tạo một thư mục _projects_ và một thư
mục cho dự án “Hello, world!” trong thư mục _projects_.

Đối với Linux, macOS và PowerShell trên Windows, nhập:

```console
$ mkdir ~/projects
$ cd ~/projects
$ mkdir hello_world
$ cd hello_world
```

Đối với Windows CMD, nhập:

```cmd
> mkdir "%USERPROFILE%\projects"
> cd /d "%USERPROFILE%\projects"
> mkdir hello_world
> cd hello_world
```

<!-- Old headings. Do not remove or links may break. -->
<a id="writing-and-running-a-rust-program"></a>

### Chương trình Rust cơ bản

Tiếp theo, tạo một tệp nguồn mới và gọi nó là _main.rs_. Các tệp Rust luôn kết
thúc bằng phần mở rộng _.rs_. Nếu bạn sử dụng nhiều hơn một từ trong tên tệp của
mình, quy ước là sử dụng dấu gạch dưới để tách chúng. Ví dụ, sử dụng
_hello_world.rs_ thay vì _helloworld.rs_.

Bây giờ mở tệp _main.rs_ mà bạn vừa tạo và nhập mã trong Listing 1-1.

<Listing number="1-1" file-name="main.rs" caption="Một chương trình in ra `Hello, world!`">

```rust
fn main() {
    println!("Hello, world!");
}
```

</Listing>

Lưu tệp và quay lại cửa sổ terminal của bạn trong thư mục
_~/projects/hello_world_. Trên Linux hoặc macOS, nhập các lệnh sau để biên dịch
và chạy tệp:

```console
$ rustc main.rs
$ ./main
Hello, world!
```

Trên Windows, nhập lệnh `.\main` thay vì `./main`:

```powershell
> rustc main.rs
> .\main
Hello, world!
```

Bất kể hệ điều hành của bạn là gì, chuỗi `Hello, world!` sẽ được in ra terminal.
Nếu bạn không thấy đầu ra này, hãy tham khảo lại phần [“Khắc phục sự
cố”][troubleshooting]<!-- ignore --> của phần Cài đặt để biết cách nhận trợ
giúp.

Nếu `Hello, world!` đã được in ra, chúc mừng bạn! Bạn đã chính thức viết một
chương trình Rust. Điều đó làm cho bạn trở thành một lập trình viên Rust — chào
mừng!

<!-- Old headings. Do not remove or links may break. -->
<a id="anatomy-of-a-rust-program"></a>

### Cấu trúc của một chương trình Rust

Hãy xem xét chi tiết chương trình “Hello, world!” này. Đây là mảnh ghép đầu
tiên:

```rust
fn main() {

}
```

Những dòng này định nghĩa một hàm tên là `main`. Hàm `main` là một hàm đặc biệt:
Nó luôn là mã đầu tiên chạy trong mọi chương trình Rust thực thi. Ở đây, dòng
đầu tiên khai báo một hàm tên là `main` không có tham số và không trả về gì. Nếu
có tham số, chúng sẽ nằm trong dấu ngoặc đơn (`()`).

Thân hàm được bao bọc trong `{}`. Rust yêu cầu dấu ngoặc nhọn xung quanh tất cả
thân hàm. Đó là phong cách tốt để đặt dấu ngoặc nhọn mở trên cùng một dòng với
khai báo hàm, thêm một khoảng trắng ở giữa.

> Lưu ý: Nếu bạn muốn tuân theo một phong cách chuẩn trong các dự án Rust, bạn
> có thể sử dụng công cụ định dạng tự động gọi là `rustfmt` để định dạng mã của
> bạn theo một phong cách cụ thể (đọc thêm về `rustfmt` trong [Phụ lục
> D][devtools]<!-- ignore -->). Đội ngũ phát triển Rust đã bao gồm công cụ này
> trong bản phân phối Rust tiêu chuẩn, giống như `rustc`, vì vậy nó có thể đã
> được cài đặt trên máy tính của bạn!

Thân hàm `main` chứa mã sau:

```rust
println!("Hello, world!");
```

Dòng này thực hiện tất cả công việc trong chương trình nhỏ này: Nó in văn bản
lên màn hình. Có ba chi tiết quan trọng cần chú ý ở đây.

Đầu tiên, `println!` là một macro của Rust. Thay vì nếu nó được gọi là một hàm,
nó sẽ được nhập là `println` (không có `!`). Các macro của Rust là một cách để
viết mã tạo ra mã để mở rộng cú pháp Rust, và chúng ta sẽ thảo luận chi tiết hơn
về chúng trong [Chương 20][ch20-macros]<!-- ignore -->. Hiện tại, bạn chỉ cần
biết rằng việc sử dụng `!` có nghĩa là bạn đang gọi một macro thay vì một hàm
bình thường và các macro không luôn tuân theo các quy tắc giống như các hàm.

Thứ hai, bạn thấy chuỗi `"Hello, world!"`. Chúng ta truyền chuỗi này làm đối số
cho `println!`, và chuỗi được in ra màn hình.

Thứ ba, chúng ta kết thúc dòng bằng dấu chấm phẩy (`;`), điều này cho biết rằng
biểu thức này đã kết thúc, và biểu thức tiếp theo đã sẵn sàng để bắt đầu. Hầu
hết các dòng mã Rust kết thúc bằng dấu chấm phẩy.

<!-- Old headings. Do not remove or links may break. -->
<a id="compiling-and-running-are-separate-steps"></a>

### Biên dịch và Chạy

Bạn vừa chạy một chương trình mới tạo, vì vậy hãy xem xét từng bước trong quá
trình.

Trước khi chạy một chương trình Rust, bạn phải biên dịch nó bằng trình biên dịch
Rust bằng cách nhập lệnh `rustc` và truyền cho nó tên tệp nguồn của bạn, như thế
này:

```console
$ rustc main.rs
```

Nếu bạn có nền tảng C hoặc C++, bạn sẽ nhận thấy điều này tương tự như `gcc`
hoặc `clang`. Sau khi biên dịch thành công, Rust xuất ra một tệp thực thi nhị
phân.

Trên Linux, macOS và PowerShell trên Windows, bạn có thể thấy tệp thực thi bằng
cách nhập lệnh `ls` trong shell của bạn:

```console
$ ls
main  main.rs
```

Trên Linux và macOS, bạn sẽ thấy hai tệp. Với PowerShell trên Windows, bạn sẽ
thấy ba tệp giống như bạn sẽ thấy khi sử dụng CMD. Với CMD trên Windows, bạn sẽ
nhập lệnh sau:

```cmd
> dir /B %= tùy chọn /B chỉ hiển thị tên tệp =%
main.exe
main.pdb
main.rs
```

Điều này hiển thị tệp mã nguồn với phần mở rộng _.rs_, tệp thực thi (_main.exe_
trên Windows, nhưng _main_ trên tất cả các nền tảng khác), và, khi sử dụng
Windows, một tệp chứa thông tin gỡ lỗi với phần mở rộng _.pdb_. Từ đây, bạn chạy
tệp _main_ hoặc _main.exe_, như sau:

```console
$ ./main # hoặc .\main trên Windows
```

Nếu _main.rs_ của bạn là chương trình “Hello, world!” của bạn, dòng này sẽ in
`Hello, world!` lên terminal của bạn.

Nếu bạn quen thuộc hơn với một ngôn ngữ động, chẳng hạn như Ruby, Python hoặc
JavaScript, bạn có thể không quen với việc biên dịch và chạy một chương trình
như các bước riêng biệt. Rust là một ngôn ngữ _biên dịch trước_ (ahead-of-time
compiled), có nghĩa là bạn có thể biên dịch một chương trình và đưa tệp thực thi
cho người khác, và họ có thể chạy nó ngay cả khi không cài đặt Rust. Nếu bạn đưa
cho ai đó một tệp _.rb_, _.py_ hoặc _.js_, họ cần phải có một triển khai Ruby,
Python hoặc JavaScript tương ứng được cài đặt. Nhưng trong những ngôn ngữ đó,
bạn chỉ cần một lệnh để biên dịch và chạy chương trình của bạn. Mọi thứ đều là
sự đánh đổi trong thiết kế ngôn ngữ.

Chỉ biên dịch với `rustc` là đủ cho các chương trình đơn giản, nhưng khi dự án
của bạn phát triển, bạn sẽ muốn quản lý tất cả các tùy chọn và làm cho việc chia
sẻ mã của bạn trở nên dễ dàng. Tiếp theo, chúng tôi sẽ giới thiệu cho bạn công
cụ Cargo, công cụ sẽ giúp bạn viết các chương trình Rust thực tế.

[troubleshooting]: ch01-01-installation.html#troubleshooting
[devtools]: appendix-04-useful-development-tools.html
[ch20-macros]: ch20-05-macros.html
