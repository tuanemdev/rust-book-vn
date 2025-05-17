## Kiểm Soát Cách Chạy Các Bài Kiểm Thử

Giống như `cargo run` biên dịch code của bạn và sau đó chạy tệp nhị phân kết
quả, `cargo test` biên dịch code của bạn ở chế độ kiểm thử và chạy tệp nhị phân
kiểm thử kết quả. Hành vi mặc định của tệp nhị phân được tạo ra bởi `cargo test`
là chạy tất cả các bài kiểm thử song song và thu thập kết quả được tạo ra trong
quá trình chạy kiểm thử, ngăn chặn kết quả được hiển thị và làm cho việc đọc kết
quả liên quan đến kết quả kiểm thử dễ dàng hơn. Tuy nhiên, bạn có thể chỉ định
các tùy chọn dòng lệnh để thay đổi hành vi mặc định này.

Một số tùy chọn dòng lệnh được chuyển tới `cargo test`, và một số được chuyển
tới tệp nhị phân kiểm thử kết quả. Để phân tách hai loại đối số này, bạn liệt kê
các đối số chuyển tới `cargo test` theo sau là dấu phân tách `--` và sau đó là
các đối số chuyển tới tệp nhị phân kiểm thử. Chạy `cargo test --help` hiển thị
các tùy chọn bạn có thể sử dụng với `cargo test`, và chạy `cargo test -- --help`
hiển thị các tùy chọn bạn có thể sử dụng sau dấu phân tách. Những tùy chọn đó
cũng được ghi lại trong [phần "Tests"][tests] của [sách rustc][rustc].

[tests]: https://doc.rust-lang.org/rustc/tests/index.html
[rustc]: https://doc.rust-lang.org/rustc/index.html

### Chạy Các Bài Kiểm Thử Song Song hoặc Tuần Tự

Khi bạn chạy nhiều bài kiểm thử, theo mặc định chúng chạy song song sử dụng các
thread, có nghĩa là chúng chạy xong nhanh hơn và bạn nhận được phản hồi nhanh
hơn. Vì các bài kiểm thử đang chạy cùng lúc, bạn phải đảm bảo rằng các bài kiểm
thử của bạn không phụ thuộc vào nhau hoặc vào bất kỳ trạng thái chung nào, bao
gồm cả môi trường chung, chẳng hạn như thư mục làm việc hiện tại hoặc biến môi
trường.

Ví dụ, giả sử mỗi bài kiểm thử của bạn chạy một số code tạo ra một tệp trên đĩa
có tên _test-output.txt_ và viết một số dữ liệu vào tệp đó. Sau đó, mỗi bài kiểm
thử đọc dữ liệu trong tệp đó và khẳng định rằng tệp chứa một giá trị cụ thể,
khác nhau trong mỗi bài kiểm thử. Vì các bài kiểm thử chạy cùng lúc, một bài
kiểm thử có thể ghi đè lên tệp trong khoảng thời gian giữa một bài kiểm thử khác
đang viết và đọc tệp. Bài kiểm thử thứ hai sẽ thất bại, không phải vì code không
đúng mà vì các bài kiểm thử đã can thiệp vào nhau trong quá trình chạy song
song. Một giải pháp là đảm bảo mỗi bài kiểm thử ghi vào một tệp khác nhau; giải
pháp khác là chạy các bài kiểm thử từng cái một.

Nếu bạn không muốn chạy các bài kiểm thử song song hoặc nếu bạn muốn kiểm soát
chi tiết hơn về số lượng thread được sử dụng, bạn có thể gửi cờ `--test-threads`
và số thread bạn muốn sử dụng cho tệp nhị phân kiểm thử. Hãy xem ví dụ sau:

```console
$ cargo test -- --test-threads=1
```

Chúng ta đặt số thread kiểm thử thành `1`, báo cho chương trình không sử dụng
bất kỳ tính năng song song nào. Chạy các bài kiểm thử bằng một thread sẽ mất
nhiều thời gian hơn so với chạy chúng song song, nhưng các bài kiểm thử sẽ không
can thiệp vào nhau nếu chúng chia sẻ trạng thái.

### Hiển Thị Kết Quả Hàm

Theo mặc định, nếu một bài kiểm thử thành công, thư viện kiểm thử của Rust sẽ
thu thập bất kỳ thứ gì được in ra đầu ra tiêu chuẩn. Ví dụ, nếu chúng ta gọi
`println!` trong một bài kiểm thử và bài kiểm thử thành công, chúng ta sẽ không
thấy kết quả `println!` trong terminal; chúng ta sẽ chỉ thấy dòng cho biết bài
kiểm thử đã thành công. Nếu một bài kiểm thử thất bại, chúng ta sẽ thấy bất cứ
thứ gì đã được in ra đầu ra tiêu chuẩn cùng với phần còn lại của thông báo lỗi.

Ví dụ, Listing 11-10 có một hàm ngớ ngẩn in ra giá trị của tham số và trả về 10,
cũng như một bài kiểm thử thành công và một bài kiểm thử thất bại.

<Listing number="11-10" file-name="src/lib.rs" caption="Các bài kiểm thử cho một hàm gọi `println!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-10/src/lib.rs}}
```

</Listing>

Khi chúng ta chạy các bài kiểm thử này với `cargo test`, chúng ta sẽ thấy kết
quả sau:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-10/output.txt}}
```

Lưu ý rằng không có chỗ nào trong kết quả này chúng ta thấy `I got the value 4`,
được in ra khi bài kiểm thử thành công chạy. Kết quả đó đã bị thu thập. Kết quả
từ bài kiểm thử thất bại, `I got the value 8`, xuất hiện trong phần tóm tắt kết
quả kiểm thử, phần này cũng hiển thị nguyên nhân của việc thất bại kiểm thử.

Nếu chúng ta muốn thấy các giá trị được in ra cho các bài kiểm thử thành công,
chúng ta có thể báo cho Rust hiển thị kết quả của các bài kiểm thử thành công
bằng `--show-output`:

```console
$ cargo test -- --show-output
```

Khi chúng ta chạy lại các bài kiểm thử trong Listing 11-10 với cờ
`--show-output`, chúng ta thấy kết quả sau:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-01-show-output/output.txt}}
```

### Chạy Một Tập Con Các Bài Kiểm Thử Theo Tên

Đôi khi, việc chạy một bộ kiểm thử đầy đủ có thể mất nhiều thời gian. Nếu bạn
đang làm việc trên code trong một khu vực cụ thể, bạn có thể muốn chỉ chạy các
bài kiểm thử liên quan đến code đó. Bạn có thể chọn các bài kiểm thử để chạy
bằng cách truyền cho `cargo test` tên hoặc các tên của (các) bài kiểm thử bạn
muốn chạy làm đối số.

Để minh họa cách chạy một tập con các bài kiểm thử, trước tiên chúng ta sẽ tạo
ba bài kiểm thử cho hàm `add_two` của chúng ta, như được hiển thị trong Listing
11-11, và chọn những bài nào để chạy.

<Listing number="11-11" file-name="src/lib.rs" caption="Ba bài kiểm thử với ba tên khác nhau">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-11/src/lib.rs}}
```

</Listing>

Nếu chúng ta chạy các bài kiểm thử mà không truyền bất kỳ đối số nào, như chúng
ta đã thấy trước đây, tất cả các bài kiểm thử sẽ chạy song song:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-11/output.txt}}
```

#### Chạy Các Bài Kiểm Thử Đơn Lẻ

Chúng ta có thể truyền tên của bất kỳ hàm kiểm thử nào cho `cargo test` để chỉ
chạy bài kiểm thử đó:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-02-single-test/output.txt}}
```

Chỉ có bài kiểm thử có tên `one_hundred` được chạy; hai bài kiểm thử còn lại
không khớp với tên đó. Kết quả kiểm thử cho chúng ta biết rằng chúng ta có nhiều
bài kiểm thử không được chạy bằng cách hiển thị `2 filtered out` ở cuối.

Chúng ta không thể chỉ định tên của nhiều bài kiểm thử theo cách này; chỉ có giá
trị đầu tiên được cung cấp cho `cargo test` sẽ được sử dụng. Nhưng có một cách
để chạy nhiều bài kiểm thử.

#### Lọc Để Chạy Nhiều Bài Kiểm Thử

Chúng ta có thể chỉ định một phần của tên bài kiểm thử, và bất kỳ bài kiểm thử
nào có tên khớp với giá trị đó sẽ được chạy. Ví dụ, vì hai trong số các bài kiểm
thử của chúng ta có tên chứa `add`, chúng ta có thể chạy hai bài đó bằng cách
chạy `cargo test add`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-03-multiple-tests/output.txt}}
```

Lệnh này đã chạy tất cả các bài kiểm thử có `add` trong tên và lọc ra bài kiểm
thử có tên `one_hundred`. Cũng lưu ý rằng module mà bài kiểm thử xuất hiện trở
thành một phần của tên bài kiểm thử, vì vậy chúng ta có thể chạy tất cả các bài
kiểm thử trong một module bằng cách lọc theo tên của module.

### Bỏ Qua Một Số Bài Kiểm Thử Trừ Khi Có Yêu Cầu Cụ Thể

Đôi khi một số bài kiểm thử cụ thể có thể tốn rất nhiều thời gian để thực hiện,
vì vậy bạn có thể muốn loại trừ chúng trong hầu hết các lần chạy `cargo test`.
Thay vì liệt kê làm đối số tất cả các bài kiểm thử bạn muốn chạy, bạn có thể
thay vào đó chú thích các bài kiểm thử tốn thời gian bằng thuộc tính `ignore` để
loại trừ chúng, như được hiển thị ở đây:

<span class="filename">Tên tệp: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/src/lib.rs:here}}
```

Sau `#[test]`, chúng ta thêm dòng `#[ignore]` cho bài kiểm thử mà chúng ta muốn
loại trừ. Bây giờ khi chúng ta chạy các bài kiểm thử, `it_works` chạy, nhưng
`expensive_test` thì không:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-11-ignore-a-test/output.txt}}
```

Hàm `expensive_test` được liệt kê là `ignored` (bị bỏ qua). Nếu chúng ta chỉ
muốn chạy các bài kiểm thử bị bỏ qua, chúng ta có thể sử dụng
`cargo test -- --ignored`:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-04-running-ignored/output.txt}}
```

Bằng cách kiểm soát các bài kiểm thử được chạy, bạn có thể đảm bảo rằng kết quả
`cargo test` của bạn sẽ được trả về nhanh chóng. Khi bạn ở một điểm mà việc kiểm
tra kết quả của các bài kiểm thử `ignored` có ý nghĩa và bạn có thời gian để chờ
đợi kết quả, bạn có thể chạy `cargo test -- --ignored` thay thế. Nếu bạn muốn
chạy tất cả các bài kiểm thử cho dù chúng có bị bỏ qua hay không, bạn có thể
chạy `cargo test -- --include-ignored`.
