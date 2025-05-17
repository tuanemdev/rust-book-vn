## Xây Dựng Một Web Server Đơn Luồng

Chúng ta sẽ bắt đầu bằng việc xây dựng một web server chạy trên một luồng duy
nhất. Trước khi bắt đầu, hãy xem qua tổng quan nhanh về các giao thức liên quan
đến việc xây dựng web server. Chi tiết của các giao thức này nằm ngoài phạm vi
của cuốn sách này, nhưng một tổng quan ngắn gọn sẽ cung cấp cho bạn thông tin
cần thiết.

Hai giao thức chính liên quan đến web server là _Hypertext Transfer Protocol_
_(HTTP)_ và _Transmission Control Protocol_ _(TCP)_. Cả hai giao thức đều là các
giao thức _yêu cầu-phản hồi_, nghĩa là _khách hàng_ (client) khởi tạo yêu cầu và
_máy chủ_ (server) lắng nghe các yêu cầu và cung cấp phản hồi cho khách hàng.
Nội dung của các yêu cầu và phản hồi đó được định nghĩa bởi các giao thức.

TCP là một giao thức cấp thấp hơn mô tả chi tiết về cách thông tin được truyền
từ một máy chủ sang máy chủ khác nhưng không chỉ định thông tin đó là gì. HTTP
xây dựng trên TCP bằng cách định nghĩa nội dung của các yêu cầu và phản hồi. Về
mặt kỹ thuật, có thể sử dụng HTTP với các giao thức khác, nhưng trong phần lớn
trường hợp, HTTP gửi dữ liệu qua TCP. Chúng ta sẽ làm việc với các byte thô của
yêu cầu và phản hồi TCP và HTTP.

### Lắng Nghe Kết Nối TCP

Web server của chúng ta cần lắng nghe kết nối TCP, vì vậy đó là phần đầu tiên mà
chúng ta sẽ xử lý. Thư viện chuẩn cung cấp một module `std::net` cho phép chúng
ta làm điều này. Hãy tạo một dự án mới theo cách thông thường với `cargo new`:

```console
$ cargo new hello
     Created binary (application) `hello` project
$ cd hello
```

Bây giờ nhập mã trong Listing 21-1 vào _src/main.rs_ để bắt đầu. Mã này sẽ lắng
nghe tại địa chỉ cục bộ `127.0.0.1:7878` để tìm các luồng TCP đến. Khi nó nhận
được một luồng đến, nó sẽ in ra `Connection established!`.

> Lưu ý: nếu bạn đang chạy chương trình này trên một máy từ xa thay vì cục bộ,
> trình duyệt web của bạn sẽ không thể kết nối. Để cho phép kết nối, hãy tìm địa
> chỉ IP công cộng của máy từ xa của bạn và thay thế `127.0.0.1` bằng địa chỉ IP
> công cộng của bạn trong cả _src/main.rs_ và URL bạn sử dụng trong trình duyệt.

<Listing number="21-1" file-name="src/main.rs" caption="Lắng nghe các luồng đến và in một thông báo khi chúng ta nhận được một luồng">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-01/src/main.rs}}
```

</Listing>

Sử dụng `TcpListener`, chúng ta có thể lắng nghe các kết nối TCP tại địa chỉ
`127.0.0.1:7878`. Trong địa chỉ, phần trước dấu hai chấm là địa chỉ IP đại diện
cho máy tính của bạn (điều này giống nhau trên mọi máy tính và không đại diện cụ
thể cho máy tính của tác giả), và `7878` là cổng. Chúng ta đã chọn cổng này vì
hai lý do: HTTP thường không được chấp nhận trên cổng này nên server của chúng
ta khó có thể xung đột với bất kỳ web server nào khác mà bạn có thể đang chạy
trên máy của mình, và 7878 là _rust_ được gõ trên điện thoại.

Hàm `bind` trong tình huống này hoạt động giống như hàm `new` vì nó sẽ trả về
một thể hiện `TcpListener` mới. Hàm này được gọi là `bind` vì trong mạng, việc
kết nối với một cổng để lắng nghe được gọi là "binding to a port".

Hàm `bind` trả về một `Result<T, E>`, cho thấy rằng việc liên kết có thể thất
bại. Ví dụ, kết nối với cổng 80 yêu cầu quyền quản trị viên (người dùng không
phải quản trị viên chỉ có thể lắng nghe trên các cổng cao hơn 1023), vì vậy nếu
chúng ta cố kết nối với cổng 80 mà không phải là quản trị viên, việc liên kết sẽ
không hoạt động. Việc liên kết cũng sẽ không hoạt động, ví dụ, nếu chúng ta chạy
hai thể hiện của chương trình và do đó có hai chương trình lắng nghe cùng một
cổng. Bởi vì chúng ta đang viết một server cơ bản chỉ để học, chúng ta sẽ không
lo lắng về việc xử lý các loại lỗi này; thay vào đó, chúng ta sử dụng `unwrap`
để dừng chương trình nếu có lỗi xảy ra.

Phương thức `incoming` trên `TcpListener` trả về một iterator cung cấp cho chúng
ta một chuỗi các luồng (cụ thể hơn, các luồng thuộc loại `TcpStream`). Một
_luồng_ (stream) đơn lẻ đại diện cho một kết nối mở giữa khách hàng và máy chủ.
Một _kết nối_ (connection) là tên cho toàn bộ quá trình yêu cầu và phản hồi
trong đó khách hàng kết nối với máy chủ, máy chủ tạo ra phản hồi, và máy chủ
đóng kết nối. Do đó, `TcpStream` cho phép chúng ta đọc từ nó để xem khách hàng
đã gửi gì và sau đó viết phản hồi của chúng ta vào luồng để gửi dữ liệu trở lại
cho khách hàng. Nhìn chung, vòng lặp `for` này sẽ xử lý từng kết nối lần lượt và
tạo ra một chuỗi các luồng để chúng ta xử lý.

Hiện tại, việc xử lý luồng của chúng ta bao gồm việc gọi `unwrap` để chấm dứt
chương trình của chúng ta nếu luồng có bất kỳ lỗi nào; nếu không có lỗi nào,
chương trình sẽ in ra một thông báo. Chúng ta sẽ thêm chức năng cho trường hợp
thành công trong các ví dụ sau. Lý do chúng ta có thể nhận được lỗi từ phương
thức `incoming` khi khách hàng kết nối với máy chủ là vì chúng ta không thực sự
lặp qua các kết nối. Thay vào đó, chúng ta đang lặp qua _các nỗ lực kết nối_.
Kết nối có thể không thành công vì nhiều lý do, nhiều lý do phụ thuộc vào hệ
điều hành. Ví dụ, nhiều hệ điều hành có giới hạn về số lượng kết nối đồng thời
mở mà chúng có thể hỗ trợ; các nỗ lực kết nối mới vượt quá số đó sẽ tạo ra lỗi
cho đến khi một số kết nối mở được đóng lại.

Hãy thử chạy mã này! Gọi `cargo run` trong terminal và sau đó tải
_127.0.0.1:7878_ trong trình duyệt web. Trình duyệt sẽ hiển thị một thông báo
lỗi như "Connection reset" vì máy chủ hiện không gửi lại bất kỳ dữ liệu nào.
Nhưng khi bạn nhìn vào terminal của mình, bạn sẽ thấy một số thông báo đã được
in ra khi trình duyệt kết nối với máy chủ!

```text
     Running `target/debug/hello`
Connection established!
Connection established!
Connection established!
```

Đôi khi bạn sẽ thấy nhiều thông báo được in ra cho một yêu cầu trình duyệt; lý
do có thể là trình duyệt đang gửi yêu cầu cho trang cũng như yêu cầu cho các tài
nguyên khác, như biểu tượng _favicon.ico_ xuất hiện trong tab trình duyệt.

Cũng có thể là trình duyệt đang cố gắng kết nối với máy chủ nhiều lần vì máy chủ
không phản hồi với bất kỳ dữ liệu nào. Khi `stream` ra khỏi phạm vi và bị drop
vào cuối vòng lặp, kết nối sẽ bị đóng như một phần của việc triển khai `drop`.
Trình duyệt đôi khi xử lý các kết nối bị đóng bằng cách thử lại, vì vấn đề có
thể là tạm thời. Điều quan trọng là chúng ta đã thành công trong việc có được
một handle cho kết nối TCP!

Hãy nhớ dừng chương trình bằng cách nhấn <kbd>ctrl</kbd>-<kbd>c</kbd> khi bạn đã
chạy xong một phiên bản cụ thể của mã. Sau đó khởi động lại chương trình bằng
cách gọi lệnh `cargo run` sau khi bạn đã thực hiện mỗi bộ thay đổi mã để đảm bảo
bạn đang chạy mã mới nhất.

### Đọc Yêu Cầu

Hãy triển khai chức năng để đọc yêu cầu từ trình duyệt! Để tách biệt các mối
quan tâm về việc đầu tiên là nhận được kết nối và sau đó thực hiện một số hành
động với kết nối, chúng ta sẽ bắt đầu một hàm mới để xử lý các kết nối. Trong
hàm `handle_connection` mới này, chúng ta sẽ đọc dữ liệu từ luồng TCP và in nó
ra để chúng ta có thể thấy dữ liệu được gửi từ trình duyệt. Thay đổi mã để trông
giống như Listing 21-2.

<Listing number="21-2" file-name="src/main.rs" caption="Đọc từ `TcpStream` và in dữ liệu">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-02/src/main.rs}}
```

</Listing>

Chúng ta đưa `std::io::prelude` và `std::io::BufReader` vào phạm vi để có quyền
truy cập vào các traits và kiểu cho phép chúng ta đọc từ và viết vào luồng.
Trong vòng lặp `for` trong hàm `main`, thay vì in một thông báo cho biết chúng
ta đã thiết lập kết nối, bây giờ chúng ta gọi hàm `handle_connection` mới và
truyền `stream` vào nó.

Trong hàm `handle_connection`, chúng ta tạo một thể hiện `BufReader` mới bao bọc
một tham chiếu đến `stream`. `BufReader` thêm bộ đệm bằng cách quản lý các cuộc
gọi đến các phương thức của trait `std::io::Read` cho chúng ta.

Chúng ta tạo một biến có tên `http_request` để thu thập các dòng của yêu cầu mà
trình duyệt gửi đến máy chủ của chúng ta. Chúng ta chỉ ra rằng chúng ta muốn thu
thập các dòng này trong một vector bằng cách thêm chú thích kiểu `Vec<_>`.

`BufReader` triển khai trait `std::io::BufRead`, cung cấp phương thức `lines`.
Phương thức `lines` trả về một iterator của `Result<String, std::io::Error>`
bằng cách chia luồng dữ liệu bất cứ khi nào nó thấy một byte xuống dòng. Để lấy
mỗi `String`, chúng ta map và `unwrap` mỗi `Result`. `Result` có thể là một lỗi
nếu dữ liệu không phải là UTF-8 hợp lệ hoặc nếu có vấn đề khi đọc từ luồng. Một
lần nữa, một chương trình sản xuất nên xử lý các lỗi này một cách thanh lịch
hơn, nhưng chúng ta chọn dừng chương trình trong trường hợp lỗi để đơn giản.

Trình duyệt báo hiệu kết thúc của một yêu cầu HTTP bằng cách gửi hai ký tự xuống
dòng liên tiếp, vì vậy để nhận một yêu cầu từ luồng, chúng ta lấy các dòng cho
đến khi chúng ta nhận được một dòng là chuỗi rỗng. Một khi chúng ta đã thu thập
các dòng vào vector, chúng ta in chúng ra bằng cách sử dụng định dạng gỡ lỗi đẹp
để chúng ta có thể xem xét các hướng dẫn mà trình duyệt web đang gửi đến máy chủ
của chúng ta.

Hãy thử mã này! Khởi động chương trình và tạo yêu cầu trong trình duyệt web một
lần nữa. Lưu ý rằng chúng ta vẫn sẽ nhận được một trang lỗi trong trình duyệt,
nhưng đầu ra của chương trình của chúng ta trong terminal bây giờ sẽ trông giống
như thế này:

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished dev [unoptimized + debuginfo] target(s) in 0.42s
     Running `target/debug/hello`
Request: [
    "GET / HTTP/1.1",
    "Host: 127.0.0.1:7878",
    "User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:99.0) Gecko/20100101 Firefox/99.0",
    "Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8",
    "Accept-Language: en-US,en;q=0.5",
    "Accept-Encoding: gzip, deflate, br",
    "Connection: keep-alive",
    "Upgrade-Insecure-Requests: 1",
    "Sec-Fetch-Dest: document",
    "Sec-Fetch-Mode: navigate",
    "Sec-Fetch-Site: none",
    "Sec-Fetch-User: ?1",
]
```

Tùy thuộc vào trình duyệt của bạn, bạn có thể nhận được đầu ra hơi khác. Bây giờ
chúng ta đang in dữ liệu yêu cầu, chúng ta có thể thấy tại sao chúng ta nhận
được nhiều kết nối từ một yêu cầu trình duyệt bằng cách nhìn vào đường dẫn sau
`GET` trong dòng đầu tiên của yêu cầu. Nếu các kết nối lặp lại đều yêu cầu _/_,
chúng ta biết rằng trình duyệt đang cố gắng tìm nạp _/_ nhiều lần vì nó không
nhận được phản hồi từ chương trình của chúng ta.

Hãy phân tích dữ liệu yêu cầu này để hiểu những gì trình duyệt đang yêu cầu từ
chương trình của chúng ta.

### Xem Xét Kỹ Hơn về Yêu Cầu HTTP

HTTP là một giao thức dựa trên văn bản, và một yêu cầu có định dạng này:

```text
Method Request-URI HTTP-Version CRLF
headers CRLF
message-body
```

Dòng đầu tiên là _dòng yêu cầu_ (request line) chứa thông tin về những gì khách
hàng đang yêu cầu. Phần đầu tiên của dòng yêu cầu chỉ ra _phương thức_ (method)
được sử dụng, chẳng hạn như `GET` hoặc `POST`, mô tả cách khách hàng đang thực
hiện yêu cầu này. Khách hàng của chúng ta sử dụng yêu cầu `GET`, có nghĩa là nó
đang yêu cầu thông tin.

Phần tiếp theo của dòng yêu cầu là _/_, chỉ ra _bộ nhận dạng tài nguyên thống
nhất_ _(URI)_ mà khách hàng đang yêu cầu: một URI gần như, nhưng không hoàn
toàn, giống như _định vị tài nguyên thống nhất_ _(URL)_. Sự khác biệt giữa URI
và URL không quan trọng cho mục đích của chúng ta trong chương này, nhưng đặc tả
HTTP sử dụng thuật ngữ URI, vì vậy chúng ta có thể chỉ thay thế _URL_ cho _URI_
ở đây.

Phần cuối cùng là phiên bản HTTP mà khách hàng sử dụng, và sau đó dòng yêu cầu
kết thúc bằng một chuỗi CRLF. (CRLF là viết tắt của _carriage return_ và _line
feed_, là các thuật ngữ từ thời máy đánh chữ!) Chuỗi CRLF cũng có thể được viết
là `\r\n`, trong đó `\r` là carriage return và `\n` là line feed. _Chuỗi CRLF_
phân tách dòng yêu cầu khỏi phần còn lại của dữ liệu yêu cầu. Lưu ý rằng khi
CRLF được in, chúng ta thấy một dòng mới bắt đầu thay vì `\r\n`.

Nhìn vào dữ liệu dòng yêu cầu mà chúng ta đã nhận được từ việc chạy chương trình
của mình cho đến nay, chúng ta thấy rằng `GET` là phương thức, _/_ là URI yêu
cầu, và `HTTP/1.1` là phiên bản.

Sau dòng yêu cầu, các dòng còn lại bắt đầu từ `Host:` trở đi là các header. Yêu
cầu `GET` không có phần thân.

Hãy thử tạo một yêu cầu từ một trình duyệt khác hoặc yêu cầu một địa chỉ khác,
chẳng hạn như _127.0.0.1:7878/test_, để xem dữ liệu yêu cầu thay đổi như thế
nào.

Bây giờ chúng ta biết trình duyệt đang yêu cầu gì, hãy gửi lại một số dữ liệu!

### Viết Phản Hồi

Chúng ta sẽ triển khai việc gửi dữ liệu để đáp ứng yêu cầu của khách hàng. Các
phản hồi có định dạng sau:

```text
HTTP-Version Status-Code Reason-Phrase CRLF
headers CRLF
message-body
```

Dòng đầu tiên là _dòng trạng thái_ (status line) chứa phiên bản HTTP được sử
dụng trong phản hồi, một mã trạng thái số tóm tắt kết quả của yêu cầu, và một
cụm từ lý do cung cấp mô tả văn bản của mã trạng thái. Sau chuỗi CRLF là bất kỳ
header nào, một chuỗi CRLF khác, và thân của phản hồi.

Dưới đây là một ví dụ về phản hồi sử dụng phiên bản HTTP 1.1, có mã trạng thái
200, cụm từ lý do OK, không có header, và không có thân:

```text
HTTP/1.1 200 OK\r\n\r\n
```

Mã trạng thái 200 là phản hồi thành công tiêu chuẩn. Văn bản là một phản hồi
HTTP thành công nhỏ. Hãy viết điều này vào luồng như phản hồi của chúng ta cho
một yêu cầu thành công! Từ hàm `handle_connection`, xóa `println!` đã in dữ liệu
yêu cầu và thay thế nó bằng mã trong Listing 21-3.

<Listing number="21-3" file-name="src/main.rs" caption="Viết một phản hồi HTTP thành công nhỏ vào luồng">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-03/src/main.rs:here}}
```

</Listing>

Dòng mới đầu tiên định nghĩa biến `response` giữ dữ liệu của thông báo thành
công. Sau đó chúng ta gọi `as_bytes` trên `response` để chuyển đổi dữ liệu chuỗi
thành byte. Phương thức `write_all` trên `stream` nhận một `&[u8]` và gửi các
byte đó trực tiếp qua kết nối. Vì thao tác `write_all` có thể thất bại, chúng ta
sử dụng `unwrap` trên bất kỳ kết quả lỗi nào như trước đây. Một lần nữa, trong
một ứng dụng thực tế, bạn sẽ thêm xử lý lỗi ở đây.

Với những thay đổi này, hãy chạy mã của chúng ta và tạo một yêu cầu. Chúng ta
không còn in bất kỳ dữ liệu nào vào terminal, vì vậy chúng ta sẽ không thấy bất
kỳ đầu ra nào ngoài đầu ra từ Cargo. Khi bạn tải _127.0.0.1:7878_ trong trình
duyệt web, bạn sẽ nhận được một trang trống thay vì lỗi. Bạn vừa viết mã bằng
tay để nhận yêu cầu HTTP và gửi phản hồi!

### Trả về HTML Thực Sự

Hãy triển khai chức năng để trả về nhiều hơn một trang trống. Tạo file mới
_hello.html_ trong thư mục gốc của dự án của bạn, không phải trong thư mục
_src_. Bạn có thể nhập bất kỳ HTML nào bạn muốn; Listing 21-4 hiển thị một khả
năng.

<Listing number="21-4" file-name="hello.html" caption="Một file HTML mẫu để trả về trong phản hồi">

```html
{{#include ../listings/ch21-web-server/listing-21-05/hello.html}}
```

</Listing>

Đây là một tài liệu HTML5 tối thiểu với một tiêu đề và một số văn bản. Để trả về
từ máy chủ khi một yêu cầu được nhận, chúng ta sẽ sửa đổi `handle_connection`
như trong Listing 21-5 để đọc file HTML, thêm nó vào phản hồi dưới dạng thân, và
gửi nó.

<Listing number="21-5" file-name="src/main.rs" caption="Gửi nội dung của *hello.html* như là thân của phản hồi">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-05/src/main.rs:here}}
```

</Listing>

Chúng ta đã thêm `fs` vào câu lệnh `use` để đưa module filesystem của thư viện
chuẩn vào phạm vi. Mã để đọc nội dung của một file vào một chuỗi sẽ trông quen
thuộc; chúng ta đã sử dụng nó khi chúng ta đọc nội dung của một file cho dự án
I/O của chúng ta trong Listing 12-4.

Tiếp theo, chúng ta sử dụng `format!` để thêm nội dung của file dưới dạng thân
của phản hồi thành công. Để đảm bảo một phản hồi HTTP hợp lệ, chúng ta thêm
header `Content-Length` được thiết lập bằng kích thước của thân phản hồi, trong
trường hợp này là kích thước của `contents`.

Chạy mã này với `cargo run` và tải _127.0.0.1:7878_ trong trình duyệt của bạn;
bạn sẽ thấy HTML của bạn được hiển thị!

Hiện tại, chúng ta đang bỏ qua dữ liệu yêu cầu trong `http_request` và chỉ đơn
giản gửi lại nội dung của file HTML một cách vô điều kiện. Điều đó có nghĩa là
nếu bạn thử yêu cầu _127.0.0.1:7878/something-else_ trong trình duyệt của bạn,
bạn vẫn sẽ nhận lại cùng một phản hồi HTML. Ở thời điểm hiện tại, máy chủ của
chúng ta rất hạn chế và không làm những gì hầu hết các máy chủ web làm. Chúng ta
muốn tùy chỉnh các phản hồi của mình tùy thuộc vào yêu cầu và chỉ gửi lại file
HTML cho một yêu cầu được định dạng tốt tới _/_.

### Xác Thực Yêu Cầu và Phản Hồi Có Chọn Lọc

Hiện tại, máy chủ web của chúng ta sẽ trả về HTML trong file bất kể khách hàng
yêu cầu gì. Hãy thêm chức năng để kiểm tra xem trình duyệt có yêu cầu _/_ trước
khi trả về file HTML và trả về lỗi nếu trình duyệt yêu cầu bất cứ thứ gì khác.
Để làm điều này, chúng ta cần sửa đổi `handle_connection`, như trong Listing
21-6. Mã mới này kiểm tra nội dung của yêu cầu nhận được so với những gì chúng
ta biết một yêu cầu cho _/_ trông như thế nào và thêm các khối `if` và `else` để
xử lý các yêu cầu khác nhau.

<Listing number="21-6" file-name="src/main.rs" caption="Xử lý các yêu cầu đến */* khác với các yêu cầu khác">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-06/src/main.rs:here}}
```

</Listing>

Chúng ta chỉ xem xét dòng đầu tiên của yêu cầu HTTP, vì vậy thay vì đọc toàn bộ
yêu cầu vào vector, chúng ta đang gọi `next` để lấy mục đầu tiên từ iterator.
`unwrap` đầu tiên xử lý `Option` và dừng chương trình nếu iterator không có mục
nào. `unwrap` thứ hai xử lý `Result` và có cùng tác dụng với `unwrap` đã được
thêm vào `map` trong Listing 21-2.

Tiếp theo, chúng ta kiểm tra `request_line` để xem nó có bằng với dòng yêu cầu
của một yêu cầu GET đến đường dẫn _/_ không. Nếu có, khối `if` trả về nội dung
của file HTML của chúng ta.

Nếu `request_line` _không_ bằng với yêu cầu GET đến đường dẫn _/_, có nghĩa là
chúng ta đã nhận được một số yêu cầu khác. Chúng ta sẽ thêm mã vào khối `else`
ngay bây giờ để phản hồi tất cả các yêu cầu khác.

Chạy mã này bây giờ và yêu cầu _127.0.0.1:7878_; bạn sẽ nhận được HTML trong
_hello.html_. Nếu bạn thực hiện bất kỳ yêu cầu nào khác, chẳng hạn như
_127.0.0.1:7878/something-else_, bạn sẽ nhận được lỗi kết nối giống như những
lỗi bạn đã thấy khi chạy mã trong Listing 21-1 và Listing 21-2.

Bây giờ hãy thêm mã trong Listing 21-7 vào khối `else` để trả về phản hồi với mã
trạng thái 404, báo hiệu rằng nội dung cho yêu cầu không được tìm thấy. Chúng ta
cũng sẽ trả về một số HTML cho một trang để hiển thị trong trình duyệt cho biết
phản hồi cho người dùng cuối.

<Listing number="21-7" file-name="src/main.rs" caption="Phản hồi với mã trạng thái 404 và trang lỗi nếu bất cứ thứ gì khác ngoài */* được yêu cầu">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-07/src/main.rs:here}}
```

</Listing>

Ở đây, phản hồi của chúng ta có một dòng trạng thái với mã trạng thái 404 và cụm
từ lý do `NOT FOUND`. Thân của phản hồi sẽ là HTML trong file _404.html_. Bạn sẽ
cần tạo một file _404.html_ cạnh _hello.html_ cho trang lỗi; một lần nữa, hãy tự
do sử dụng bất kỳ HTML nào bạn muốn hoặc sử dụng HTML mẫu trong Listing 21-8.

<Listing number="21-8" file-name="404.html" caption="Nội dung mẫu cho trang cần gửi lại với bất kỳ phản hồi 404 nào">

```html
{{#include ../listings/ch21-web-server/listing-21-07/404.html}}
```

</Listing>

Với những thay đổi này, hãy chạy lại máy chủ của bạn. Yêu cầu _127.0.0.1:7878_
sẽ trả về nội dung của _hello.html_, và bất kỳ yêu cầu nào khác, như
_127.0.0.1:7878/foo_, sẽ trả về HTML lỗi từ _404.html_.

### Một Chút Tái Cấu Trúc

Tại thời điểm này, các khối `if` và `else` có nhiều sự lặp lại: cả hai đều đang
đọc các file và viết nội dung của các file vào luồng. Những khác biệt duy nhất
là dòng trạng thái và tên file. Hãy làm cho mã ngắn gọn hơn bằng cách rút ra
những khác biệt đó thành các dòng `if` và `else` riêng biệt sẽ gán giá trị của
dòng trạng thái và tên file cho các biến; sau đó chúng ta có thể sử dụng vô điều
kiện các biến đó trong mã để đọc file và viết phản hồi. Listing 21-9 hiển thị mã
kết quả sau khi thay thế các khối `if` và `else` lớn.

<Listing number="21-9" file-name="src/main.rs" caption="Tái cấu trúc các khối `if` và `else` để chỉ chứa mã khác nhau giữa hai trường hợp">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-09/src/main.rs:here}}
```

</Listing>

Bây giờ các khối `if` và `else` chỉ trả về các giá trị thích hợp cho dòng trạng
thái và tên file trong một tuple; sau đó chúng ta sử dụng destructuring để gán
hai giá trị này cho `status_line` và `filename` bằng cách sử dụng một mẫu trong
câu lệnh `let`, như đã thảo luận trong Chương 19.

Mã đã được lặp lại trước đây hiện nằm ngoài các khối `if` và `else` và sử dụng
các biến `status_line` và `filename`. Điều này làm cho nó dễ dàng hơn để thấy sự
khác biệt giữa hai trường hợp, và có nghĩa là chúng ta chỉ có một nơi để cập
nhật mã nếu chúng ta muốn thay đổi cách đọc file và viết phản hồi. Hành vi của
mã trong Listing 21-9 sẽ giống với mã trong Listing 21-7.

Tuyệt vời! Bây giờ chúng ta có một máy chủ web đơn giản trong khoảng 40 dòng mã
Rust phản hồi một yêu cầu với một trang nội dung và phản hồi tất cả các yêu cầu
khác với phản hồi 404.

Hiện tại, máy chủ của chúng ta chạy trong một luồng duy nhất, nghĩa là nó chỉ có
thể phục vụ một yêu cầu tại một thời điểm. Hãy xem xét làm thế nào điều đó có
thể là một vấn đề bằng cách mô phỏng một số yêu cầu chậm. Sau đó, chúng ta sẽ
khắc phục nó để máy chủ của chúng ta có thể xử lý nhiều yêu cầu cùng một lúc.
