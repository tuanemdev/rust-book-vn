## Làm việc với Biến Môi trường

Chúng ta sẽ cải thiện `minigrep` bằng cách thêm một tính năng bổ sung: một tùy
chọn cho tìm kiếm không phân biệt chữ hoa chữ thường mà người dùng có thể bật
thông qua một biến môi trường. Chúng ta có thể biến tính năng này thành một tùy
chọn dòng lệnh và yêu cầu người dùng nhập nó mỗi khi họ muốn áp dụng, nhưng thay
vào đó, bằng cách sử dụng biến môi trường, chúng ta cho phép người dùng đặt biến
môi trường một lần và thực hiện tất cả các tìm kiếm của họ không phân biệt chữ
hoa chữ thường trong phiên terminal đó.

### Viết một Test Thất bại cho Hàm `search` Không Phân biệt Chữ hoa Chữ thường

Đầu tiên, chúng ta sẽ thêm một hàm mới là `search_case_insensitive` vào thư viện
`minigrep`. Hàm này sẽ được gọi khi biến môi trường có giá trị. Chúng ta sẽ tiếp
tục tuân thủ quy trình TDD, vì vậy bước đầu tiên vẫn là viết một bài kiểm thử
thất bại. Chúng ta sẽ thêm một bài kiểm thử mới cho hàm
`search_case_insensitive` vừa rồi và đổi tên bài kiểm thử cũ từ `one_result`
thành `case_sensitive` để làm rõ sự khác biệt giữa hai bài kiểm thử, như thể
hiện trong Listing 12-20.

<Listing number="12-20" file-name="src/lib.rs" caption="Thêm một bài kiểm tra thất bại mới cho chức năng không phân biệt chữ hoa chữ thường mà chúng ta sắp thêm">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-20/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cũng đã chỉnh sửa `contents` của test cũ. Chúng ta đã thêm
một dòng mới với nội dung `"Duct tape."` sử dụng chữ _D_ viết hoa, nó không nên
khớp với truy vấn `"duct"` khi chúng ta tìm kiếm phân biệt chữ hoa chữ thường.
Việc thay đổi test cũ theo cách này giúp đảm bảo chúng ta không vô tình phá vỡ
chức năng tìm kiếm phân biệt chữ hoa chữ thường mà chúng ta đã triển khai. Test
này bây giờ nên pass và nên tiếp tục pass khi chúng ta làm việc trên tìm kiếm
không phân biệt chữ hoa chữ thường.

Test mới cho tìm kiếm _không phân biệt_ chữ hoa chữ thường sử dụng `"rUsT"` làm
truy vấn. Trong hàm `search_case_insensitive` mà chúng ta sắp thêm, truy vấn
`"rUsT"` nên khớp với dòng chứa `"Rust:"` với chữ _R_ viết hoa và khớp với dòng
`"Trust me."` mặc dù cả hai đều có cách viết hoa thường khác với truy vấn. Đây
là test thất bại của chúng ta, và nó sẽ không biên dịch được vì chúng ta chưa
định nghĩa hàm `search_case_insensitive`. Hãy thoải mái thêm một cài đặt khung
luôn trả về một vector rỗng, tương tự như cách chúng ta đã làm cho hàm `search`
trong Listing 12-16 để thấy test biên dịch và thất bại.

### Triển khai Hàm `search_case_insensitive`

Hàm `search_case_insensitive`, được hiển thị trong Listing 12-21, sẽ gần giống
với hàm `search`. Sự khác biệt duy nhất là chúng ta sẽ chuyển đổi `query` và mỗi
`line` thành chữ thường để dù đầu vào có chữ hoa hay chữ thường như thế nào,
chúng sẽ giống nhau khi chúng ta kiểm tra xem dòng có chứa truy vấn hay không.

<Listing number="12-21" file-name="src/lib.rs" caption="Định nghĩa hàm `search_case_insensitive` để chuyển truy vấn và dòng thành chữ thường trước khi so sánh chúng">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-21/src/lib.rs:here}}
```

</Listing>

Đầu tiên, chúng ta chuyển đổi chuỗi `query` thành chữ thường và lưu trữ nó trong
một biến mới có cùng tên, ghi đè biến `query` gốc. Gọi `to_lowercase` trên truy
vấn là cần thiết để dù truy vấn của người dùng là `"rust"`, `"RUST"`, `"Rust"`,
hay ` "``rUsT``" `, chúng ta sẽ xử lý truy vấn như thể nó là `"rust"` và không
phân biệt chữ hoa chữ thường. Mặc dù `to_lowercase` sẽ xử lý Unicode cơ bản, nó
sẽ không chính xác 100%. Nếu chúng ta đang viết một ứng dụng thực, chúng ta sẽ
muốn làm thêm một chút công việc ở đây, nhưng phần này là về biến môi trường,
không phải về Unicode, vì vậy chúng ta sẽ để nó như vậy ở đây.

Lưu ý rằng `query` bây giờ là một `String` thay vì một string slice vì gọi
`to_lowercase` tạo ra dữ liệu mới thay vì tham chiếu đến dữ liệu hiện có. Ví dụ,
nếu truy vấn là `"rUsT"`: string slice đó không chứa chữ `u` hoặc `t` viết
thường để chúng ta sử dụng, vì vậy chúng ta phải cấp phát một `String` mới chứa
`"rust"`. Khi chúng ta truyền `query` làm đối số cho phương thức `contains` bây
giờ, chúng ta cần thêm dấu và (ampersand) vì chữ ký của `contains` được định
nghĩa để nhận một string slice.

Tiếp theo, chúng ta thêm một lệnh gọi `to_lowercase` cho mỗi `line` để chuyển
tất cả các ký tự thành chữ thường. Bây giờ sau khi chúng ta đã chuyển đổi `line`
và `query` thành chữ thường, chúng ta sẽ tìm thấy các kết quả phù hợp bất kể
truy vấn có chữ hoa hay chữ thường.

Hãy xem liệu cài đặt này có vượt qua các bài kiểm tra không:

```console
{{#include ../listings/ch12-an-io-project/listing-12-21/output.txt}}
```

Tuyệt vời! Chúng đều đã pass. Bây giờ, chúng ta hãy gọi hàm
`search_case_insensitive` mới từ hàm `run`. Đầu tiên, chúng ta sẽ thêm một tùy
chọn cấu hình vào struct `Config` để chuyển đổi giữa tìm kiếm phân biệt chữ hoa
chữ thường và không phân biệt chữ hoa chữ thường. Việc thêm trường này sẽ gây ra
lỗi biên dịch vì chúng ta chưa khởi tạo trường này ở bất kỳ đâu:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:here}}
```

Chúng ta đã thêm trường `ignore_case` chứa một giá trị Boolean. Tiếp theo, chúng
ta cần hàm `run` kiểm tra giá trị của trường `ignore_case` và sử dụng nó để
quyết định gọi hàm `search` hay hàm `search_case_insensitive`, như được hiển thị
trong Listing 12-22. Điều này vẫn chưa thể biên dịch.

<Listing number="12-22" file-name="src/main.rs" caption="Gọi `search` hoặc `search_case_insensitive` dựa trên giá trị trong `config.ignore_case`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-22/src/main.rs:there}}
```

</Listing>

Cuối cùng, chúng ta cần kiểm tra biến môi trường. Các hàm để làm việc với biến
môi trường nằm trong module `env` trong thư viện tiêu chuẩn, Điều này đã nằm
trong phạm vi ở đầu file src/main.rs. Chúng ta sẽ sử dụng hàm `var` từ module
`env` để kiểm tra xem liệu có bất kỳ giá trị nào đã được đặt cho một biến môi
trường tên là `IGNORE_CASE` hay không, như được trình bày trong Listing 12-23.

<Listing number="12-23" file-name="src/main.rs" caption="Kiểm tra bất kỳ giá trị nào trong biến môi trường có tên `IGNORE_CASE`">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-23/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta tạo một biến mới, `ignore_case`. Để đặt giá trị của nó, chúng ta
gọi hàm `env::var` và truyền tên của biến môi trường `IGNORE_CASE`. Hàm
`env::var` trả về một `Result` sẽ là biến thể `Ok` thành công chứa giá trị của
biến môi trường nếu biến môi trường được đặt thành bất kỳ giá trị nào. Nó sẽ trả
về biến thể `Err` nếu biến môi trường không được đặt.

Chúng ta đang sử dụng phương thức `is_ok` trên `Result` để kiểm tra xem biến môi
trường có được đặt hay không, nghĩa là chương trình nên thực hiện tìm kiếm không
phân biệt chữ hoa chữ thường. Nếu biến môi trường `IGNORE_CASE` không được đặt
thành bất kỳ giá trị nào, `is_ok` sẽ trả về `false` và chương trình sẽ thực hiện
tìm kiếm phân biệt chữ hoa chữ thường. Chúng ta không quan tâm đến _giá trị_ của
biến môi trường, chỉ quan tâm xem nó được đặt hay không, vì vậy chúng ta đang
kiểm tra `is_ok` thay vì sử dụng `unwrap`, `expect`, hoặc bất kỳ phương thức nào
khác chúng ta đã thấy trên `Result`.

Chúng ta truyền giá trị trong biến `ignore_case` cho đối tượng `Config` để hàm
`run` có thể đọc giá trị đó và quyết định gọi `search_case_insensitive` hay
`search`, như chúng ta đã triển khai trong Listing 12-22.

Hãy thử nó! Đầu tiên, chúng ta sẽ chạy chương trình mà không đặt biến môi trường
và với truy vấn `to`, sẽ khớp với bất kỳ dòng nào chứa từ _to_ viết thường:

```console
{{#include ../listings/ch12-an-io-project/listing-12-23/output.txt}}
```

Có vẻ như điều đó vẫn hoạt động! Bây giờ chúng ta hãy chạy chương trình với
`IGNORE_CASE` được đặt thành `1` nhưng với cùng truy vấn _to_:

```console
$ IGNORE_CASE=1 cargo run -- to poem.txt
```

Nếu bạn đang sử dụng PowerShell, bạn sẽ cần đặt biến môi trường và chạy chương
trình như các lệnh riêng biệt:

```console
PS> $Env:IGNORE_CASE=1; cargo run -- to poem.txt
```

Điều này sẽ làm cho `IGNORE_CASE` tồn tại trong phiên shell còn lại của bạn. Có
thể bỏ đặt nó bằng lệnh `Remove-Item`:

```console
PS> Remove-Item Env:IGNORE_CASE
```

Chúng ta nên nhận được các dòng chứa _to_ có thể có chữ viết hoa:

<!-- manual-regeneration
cd listings/ch12-an-io-project/listing-12-23
IGNORE_CASE=1 cargo run -- to poem.txt
can't extract because of the environment variable
-->

```console
Are you nobody, too?
How dreary to be somebody!
To tell your name the livelong day
To an admiring bog!
```

Tuyệt vời, chúng ta cũng nhận được các dòng chứa _To_! Chương trình `minigrep`
của chúng ta bây giờ có thể thực hiện tìm kiếm không phân biệt chữ hoa chữ
thường được điều khiển bởi một biến môi trường. Bây giờ bạn biết cách quản lý
các tùy chọn được đặt bằng cả đối số dòng lệnh hoặc biến môi trường.

Một số chương trình cho phép đối số _và_ biến môi trường cho cùng một cấu hình.
Trong những trường hợp đó, các chương trình quyết định rằng một trong hai có
quyền ưu tiên. Đối với một bài tập khác tự thực hiện, hãy thử kiểm soát độ nhạy
chữ hoa chữ thường thông qua một đối số dòng lệnh hoặc một biến môi trường.
Quyết định xem đối số dòng lệnh hay biến môi trường nên có quyền ưu tiên nếu
chương trình được chạy với một cái được đặt thành phân biệt chữ hoa chữ thường
và một cái được đặt để bỏ qua chữ hoa chữ thường.

Module `std::env` chứa nhiều tính năng hữu ích hơn để xử lý biến môi trường: hãy
xem tài liệu của nó để biết những gì có sẵn.
