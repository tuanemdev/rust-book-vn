## Phát triển chức năng của thư viện với phát triển hướng kiểm thử

Giờ đây khi chúng ta đã trích xuất logic vào _src/lib.rs_ và để lại phần thu
thập đối số và xử lý lỗi trong _src/main.rs_, việc viết các bài kiểm thử cho
chức năng cốt lõi của mã của chúng ta trở nên dễ dàng hơn nhiều. Chúng ta có thể
gọi các hàm trực tiếp với các đối số khác nhau và kiểm tra giá trị trả về mà
không cần phải gọi nhị phân của chúng ta từ dòng lệnh.

Trong phần này, chúng ta sẽ thêm logic tìm kiếm vào chương trình `minigrep` bằng
cách sử dụng quy trình phát triển hướng kiểm thử (TDD) với các bước sau:

1. Viết một bài kiểm thử thất bại và chạy nó để đảm bảo rằng nó thất bại vì lý
   do bạn mong đợi.
2. Viết hoặc chỉnh sửa đủ mã để làm cho bài kiểm thử mới vượt qua.
3. Cải tiến mã bạn vừa thêm hoặc thay đổi và đảm bảo các bài kiểm thử tiếp tục
   vượt qua.
4. Lặp lại từ bước 1!

Mặc dù đây chỉ là một trong nhiều cách để viết phần mềm, TDD có thể giúp định
hướng thiết kế mã. Viết bài kiểm thử trước khi bạn viết mã làm cho bài kiểm thử
vượt qua giúp duy trì độ phủ kiểm thử cao trong suốt quá trình.

Chúng ta sẽ kiểm tra-hướng việc triển khai chức năng thực sự tìm kiếm chuỗi truy
vấn trong nội dung tệp và tạo ra danh sách các dòng phù hợp với truy vấn. Chúng
ta sẽ thêm chức năng này trong một hàm gọi là `search`.

### Viết một bài kiểm thử thất bại

Bởi vì chúng ta không còn cần chúng nữa, hãy xóa các câu lệnh `println!` từ
_src/lib.rs_ và _src/main.rs_ mà chúng ta đã sử dụng để kiểm tra hành vi của
chương trình. Sau đó, trong _src/lib.rs_, chúng ta sẽ thêm một mô-đun `tests`
với một hàm kiểm thử, như chúng ta đã làm trong [Chương
11][ch11-anatomy]<!-- ignore -->. Hàm kiểm thử chỉ định hành vi mà chúng ta muốn
hàm `search` có: nó sẽ lấy một truy vấn và văn bản để tìm kiếm, và nó sẽ chỉ trả
về các dòng từ văn bản mà chứa truy vấn. Listing 12-15 hiển thị bài kiểm thử
này, mà chưa thể biên dịch được.

<Listing number="12-15" file-name="src/lib.rs" caption="Tạo một bài kiểm thử thất bại cho hàm `search` mà chúng ta mong muốn có">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-15/src/lib.rs:here}}
```

</Listing>

Bài kiểm thử này tìm kiếm chuỗi `"duct"`. Văn bản chúng ta đang tìm kiếm có ba
dòng, chỉ có một dòng chứa `"duct"` (lưu ý rằng dấu gạch chéo ngược sau dấu
ngoặc kép mở báo với Rust không đặt ký tự xuống dòng vào đầu nội dung của chuỗi
chữ này). Chúng ta khẳng định rằng giá trị trả về từ hàm `search` chỉ chứa dòng
mà chúng ta mong đợi.

Chúng ta chưa thể chạy bài kiểm thử này và xem nó thất bại vì bài kiểm thử thậm
chí không biên dịch được: hàm `search` chưa tồn tại! Theo các nguyên tắc TDD,
chúng ta sẽ thêm đủ mã để bài kiểm thử biên dịch và chạy bằng cách thêm một định
nghĩa của hàm `search` luôn trả về một vector rỗng, như được hiển thị trong
Listing 12-16. Sau đó bài kiểm thử sẽ biên dịch và thất bại vì một vector rỗng
không khớp với vector chứa dòng `"safe, fast, productive."`

<Listing number="12-16" file-name="src/lib.rs" caption="Định nghĩa đủ của hàm `search` để bài kiểm thử của chúng ta sẽ biên dịch">

```rust,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-16/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cần định nghĩa một lifetime rõ ràng `'a` trong chữ ký của
`search` và sử dụng lifetime đó với đối số `contents` và giá trị trả về. Nhớ lại
trong [Chương 10][ch10-lifetimes]<!-- ignore --> rằng tham số lifetime chỉ định
đối số lifetime nào được kết nối với lifetime của giá trị trả về. Trong trường
hợp này, chúng ta chỉ ra rằng vector được trả về sẽ chứa slice chuỗi tham chiếu
đến các slice của đối số `contents` (thay vì đối số `query`).

Nói cách khác, chúng ta nói với Rust rằng dữ liệu được trả về bởi hàm `search`
sẽ tồn tại lâu như dữ liệu được truyền vào hàm `search` trong đối số `contents`.
Điều này quan trọng! Dữ liệu được tham chiếu _bởi_ một slice cần phải hợp lệ để
tham chiếu hợp lệ; nếu trình biên dịch giả định chúng ta đang tạo slice chuỗi
của `query` thay vì `contents`, nó sẽ thực hiện kiểm tra an toàn của nó không
chính xác.

Nếu chúng ta quên chú thích lifetime và cố gắng biên dịch hàm này, chúng ta sẽ
gặp lỗi này:

```console
{{#include ../listings/ch12-an-io-project/output-only-02-missing-lifetimes/output.txt}}
```

Rust không thể biết được chúng ta cần đối số nào trong hai đối số, vì vậy chúng
ta cần phải cho nó biết rõ ràng. Bởi vì `contents` là đối số chứa tất cả văn bản
của chúng ta và chúng ta muốn trả về các phần của văn bản đó phù hợp, chúng ta
biết `contents` là đối số nên được kết nối với giá trị trả về bằng cách sử dụng
cú pháp lifetime.

Các ngôn ngữ lập trình khác không yêu cầu bạn kết nối các đối số với giá trị trả
về trong chữ ký, nhưng thực hành này sẽ trở nên dễ dàng hơn theo thời gian. Bạn
có thể muốn so sánh ví dụ này với các ví dụ trong phần ["Xác thực tham chiếu với
Lifetime"][validating-references-with-lifetimes]<!-- ignore --> trong Chương 10.

Bây giờ hãy chạy bài kiểm thử:

```console
{{#include ../listings/ch12-an-io-project/listing-12-16/output.txt}}
```

Tuyệt, bài kiểm thử thất bại, chính xác như chúng ta mong đợi. Hãy làm cho bài
kiểm thử vượt qua!

### Viết mã để vượt qua bài kiểm thử

Hiện tại, bài kiểm thử của chúng ta đang thất bại vì chúng ta luôn trả về một
vector rỗng. Để sửa điều đó và triển khai `search`, chương trình của chúng ta
cần tuân theo các bước sau:

1. Lặp qua từng dòng của nội dung.
2. Kiểm tra xem dòng đó có chứa chuỗi truy vấn của chúng ta hay không.
3. Nếu có, thêm nó vào danh sách các giá trị mà chúng ta đang trả về.
4. Nếu không, không làm gì cả.
5. Trả về danh sách các kết quả phù hợp.

Hãy làm việc qua từng bước, bắt đầu với việc lặp qua các dòng.

#### Lặp qua các dòng với phương thức `lines`

Rust có một phương thức hữu ích để xử lý lặp từng dòng của chuỗi, tiện lợi được
đặt tên là `lines`, hoạt động như được hiển thị trong Listing 12-17. Lưu ý rằng
điều này chưa thể biên dịch được.

<Listing number="12-17" file-name="src/lib.rs" caption="Lặp qua từng dòng trong `contents`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-17/src/lib.rs:here}}
```

</Listing>

Phương thức `lines` trả về một iterator. Chúng ta sẽ nói về iterator một cách
chi tiết trong [Chương 13][ch13-iterators]<!-- ignore -->, nhưng hãy nhớ lại
rằng bạn đã thấy cách sử dụng iterator này trong [Listing
3-5][ch3-iter]<!-- ignore -->, nơi chúng ta đã sử dụng vòng lặp `for` với một
iterator để chạy một đoạn mã trên mỗi phần tử trong một bộ sưu tập.

#### Tìm kiếm từng dòng cho truy vấn

Tiếp theo, chúng ta sẽ kiểm tra xem dòng hiện tại có chứa chuỗi truy vấn của
chúng ta hay không. May mắn thay, chuỗi có một phương thức hữu ích có tên là
`contains` làm điều này cho chúng ta! Thêm một lệnh gọi đến phương thức
`contains` trong hàm `search`, như được hiển thị trong Listing 12-18. Lưu ý rằng
điều này vẫn chưa biên dịch được.

<Listing number="12-18" file-name="src/lib.rs" caption="Thêm chức năng để xem liệu dòng có chứa chuỗi trong `query` hay không">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-18/src/lib.rs:here}}
```

</Listing>

Hiện tại, chúng ta đang xây dựng chức năng. Để mã biên dịch, chúng ta cần trả về
một giá trị từ thân hàm như chúng ta đã chỉ ra trong chữ ký hàm.

#### Lưu trữ các dòng phù hợp

Để hoàn thành hàm này, chúng ta cần một cách để lưu trữ các dòng phù hợp mà
chúng ta muốn trả về. Để làm điều đó, chúng ta có thể tạo một vector có thể thay
đổi trước vòng lặp `for` và gọi phương thức `push` để lưu trữ một `line` trong
vector. Sau vòng lặp `for`, chúng ta trả về vector, như được hiển thị trong
Listing 12-19.

<Listing number="12-19" file-name="src/lib.rs" caption="Lưu trữ các dòng phù hợp để chúng ta có thể trả về chúng">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:here}}
```

</Listing>

Bây giờ hàm `search` chỉ nên trả về các dòng chứa `query`, và bài kiểm thử của
chúng ta sẽ vượt qua. Hãy chạy bài kiểm thử:

```console
{{#include ../listings/ch12-an-io-project/listing-12-19/output.txt}}
```

Bài kiểm thử của chúng ta đã vượt qua, vì vậy chúng ta biết nó hoạt động!

Tại thời điểm này, chúng ta có thể xem xét các cơ hội để cải tiến việc triển
khai hàm tìm kiếm trong khi giữ cho các bài kiểm thử vượt qua để duy trì chức
năng tương tự. Mã trong hàm tìm kiếm không quá tệ, nhưng nó không tận dụng một
số tính năng hữu ích của iterator. Chúng ta sẽ quay lại ví dụ này trong [Chương
13][ch13-iterators]<!-- ignore -->, nơi chúng ta sẽ khám phá iterator một cách
chi tiết và xem xét cách cải thiện nó.

#### Sử dụng hàm `search` trong hàm `run`

Bây giờ hàm `search` đang hoạt động và được kiểm thử, chúng ta cần gọi `search`
từ hàm `run` của chúng ta. Chúng ta cần truyền giá trị `config.query` và
`contents` mà `run` đọc từ tệp cho hàm `search`. Sau đó `run` sẽ in mỗi dòng
được trả về từ `search`:

<span class="filename">Tên tệp: src/lib.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/src/lib.rs:here}}
```

Chúng ta vẫn đang sử dụng vòng lặp `for` để trả về mỗi dòng từ `search` và in
nó.

Bây giờ toàn bộ chương trình sẽ hoạt động! Hãy thử nó, đầu tiên với một từ nên
trả về chính xác một dòng từ bài thơ của Emily Dickinson: _frog_.

```console
{{#include ../listings/ch12-an-io-project/no-listing-02-using-search-in-run/output.txt}}
```

Tuyệt! Bây giờ hãy thử một từ sẽ khớp với nhiều dòng, như _body_:

```console
{{#include ../listings/ch12-an-io-project/output-only-03-multiple-matches/output.txt}}
```

Và cuối cùng, hãy đảm bảo rằng chúng ta không nhận được bất kỳ dòng nào khi
chúng ta tìm kiếm một từ không có ở bất kỳ đâu trong bài thơ, ví dụ như
_monomorphization_:

```console
{{#include ../listings/ch12-an-io-project/output-only-04-no-matches/output.txt}}
```

Xuất sắc! Chúng ta đã xây dựng phiên bản mini của riêng mình cho một công cụ cổ
điển và đã học được rất nhiều về cách cấu trúc ứng dụng. Chúng ta cũng đã học
được một chút về đầu vào và đầu ra tệp, lifetime, kiểm thử và phân tích dòng
lệnh.

Để hoàn thành dự án này, chúng ta sẽ trình bày ngắn gọn cách làm việc với biến
môi trường và cách in ra lỗi tiêu chuẩn, cả hai đều hữu ích khi bạn đang viết
các chương trình dòng lệnh.

[validating-references-with-lifetimes]:
  ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[ch11-anatomy]: ch11-01-writing-tests.html#the-anatomy-of-a-test-function
[ch10-lifetimes]: ch10-03-lifetime-syntax.html
[ch3-iter]: ch03-05-control-flow.html#looping-through-a-collection-with-for
[ch13-iterators]: ch13-02-iterators.html
