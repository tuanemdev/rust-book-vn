## Chấp nhận đối số dòng lệnh

Hãy tạo một dự án mới với, như thường lệ, `cargo new`. Chúng ta sẽ gọi dự án của
mình là `minigrep` để phân biệt nó với công cụ `grep` mà có thể bạn đã có trên
hệ thống của mình.

```console
$ cargo new minigrep
     Created binary (application) `minigrep` project
$ cd minigrep
```

Nhiệm vụ đầu tiên là làm cho `minigrep` chấp nhận hai đối số dòng lệnh của nó:
đường dẫn tệp và một chuỗi để tìm kiếm. Nghĩa là, chúng ta muốn có thể chạy
chương trình của mình với `cargo run`, hai dấu gạch ngang để chỉ ra rằng các đối
số tiếp theo là dành cho chương trình của chúng ta chứ không phải cho `cargo`,
một chuỗi để tìm kiếm, và một đường dẫn đến một tệp để tìm kiếm, như sau:

```console
$ cargo run -- searchstring example-filename.txt
```

Hiện tại, chương trình được tạo ra bởi `cargo new` không thể xử lý các đối số mà
chúng ta cung cấp. Một số thư viện hiện có trên [crates.io](https://crates.io/)
có thể giúp viết một chương trình chấp nhận đối số dòng lệnh, nhưng bởi vì bạn
đang học khái niệm này, hãy tự triển khai khả năng này.

### Đọc giá trị đối số

Để cho phép `minigrep` đọc các giá trị của đối số dòng lệnh mà chúng ta truyền
vào nó, chúng ta sẽ cần hàm `std::env::args` được cung cấp trong thư viện chuẩn
của Rust. Hàm này trả về một iterator của các đối số dòng lệnh được truyền vào
`minigrep`. Chúng ta sẽ thảo luận đầy đủ về iterator trong [Chương
13][ch13]<!-- ignore
-->. Hiện tại, bạn chỉ cần biết hai chi tiết về iterator: iterator tạo ra một chuỗi
giá trị, và chúng ta có thể gọi phương thức `collect` trên một iterator để chuyển
nó thành một bộ sưu tập, chẳng hạn như một vector, chứa tất cả các phần tử mà iterator
tạo ra.

Mã trong Listing 12-1 cho phép chương trình `minigrep` của bạn đọc bất kỳ đối số
dòng lệnh nào được truyền vào nó, và sau đó thu thập các giá trị vào một vector.

<Listing number="12-1" file-name="src/main.rs" caption="Thu thập các đối số dòng lệnh vào một vector và in chúng">

```rust
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-01/src/main.rs}}
```

</Listing>

Đầu tiên chúng ta đưa mô-đun `std::env` vào phạm vi với câu lệnh `use` để chúng
ta có thể sử dụng hàm `args` của nó. Lưu ý rằng hàm `std::env::args` được lồng
trong hai cấp độ mô-đun. Như chúng ta đã thảo luận trong [Chương
7][ch7-idiomatic-use]<!-- ignore -->, trong trường hợp mà hàm mong muốn được
lồng trong nhiều hơn một mô-đun, chúng ta đã chọn đưa mô-đun cha vào phạm vi
thay vì hàm. Bằng cách này, chúng ta có thể dễ dàng sử dụng các hàm khác từ
`std::env`. Nó cũng ít gây nhầm lẫn hơn việc thêm `use std::env::args` và sau đó
gọi hàm chỉ với `args`, bởi vì `args` có thể dễ dàng bị nhầm lẫn với một hàm
được định nghĩa trong mô-đun hiện tại.

> ### Hàm `args` và Unicode không hợp lệ
>
> Lưu ý rằng `std::env::args` sẽ panic nếu bất kỳ đối số nào chứa Unicode không
> hợp lệ. Nếu chương trình của bạn cần chấp nhận các đối số chứa Unicode không
> hợp lệ, hãy sử dụng `std::env::args_os` thay thế. Hàm đó trả về một iterator
> tạo ra giá trị `OsString` thay vì giá trị `String`. Chúng ta đã chọn sử dụng
> `std::env::args` ở đây để đơn giản hóa bởi vì giá trị `OsString` khác nhau
> trên mỗi nền tảng và phức tạp hơn để làm việc so với giá trị `String`.

Ở dòng đầu tiên của `main`, chúng ta gọi `env::args`, và ngay lập tức sử dụng
`collect` để chuyển iterator thành một vector chứa tất cả các giá trị được tạo
ra bởi iterator. Chúng ta có thể sử dụng hàm `collect` để tạo nhiều loại bộ sưu
tập khác nhau, vì vậy chúng ta chú thích rõ ràng kiểu của `args` để chỉ định
rằng chúng ta muốn một vector của các chuỗi. Mặc dù bạn rất hiếm khi cần phải
chú thích kiểu trong Rust, `collect` là một hàm mà bạn thường cần phải chú thích
bởi vì Rust không thể suy luận được loại bộ sưu tập mà bạn muốn.

Cuối cùng, chúng ta in vector sử dụng macro debug. Hãy thử chạy mã này đầu tiên
không có đối số và sau đó với hai đối số:

```console
{{#include ../listings/ch12-an-io-project/listing-12-01/output.txt}}
```

```console
{{#include ../listings/ch12-an-io-project/output-only-01-with-args/output.txt}}
```

Lưu ý rằng giá trị đầu tiên trong vector là `"target/debug/minigrep"`, đó là tên
của binary của chúng ta. Điều này phù hợp với hành vi của danh sách đối số trong
C, cho phép các chương trình sử dụng tên mà chúng được gọi trong quá trình thực
thi. Thường rất thuận tiện để có quyền truy cập vào tên chương trình trong
trường hợp bạn muốn in nó trong các thông báo hoặc thay đổi hành vi của chương
trình dựa trên alias dòng lệnh nào được sử dụng để gọi chương trình. Nhưng cho
mục đích của chương này, chúng ta sẽ bỏ qua nó và chỉ lưu hai đối số mà chúng ta
cần.

### Lưu giá trị đối số trong các biến

Chương trình hiện tại có thể truy cập các giá trị được chỉ định làm đối số dòng
lệnh. Bây giờ chúng ta cần lưu giá trị của hai đối số trong các biến để chúng ta
có thể sử dụng các giá trị đó trong phần còn lại của chương trình. Chúng ta làm
điều đó trong Listing 12-2.

<Listing number="12-2" file-name="src/main.rs" caption="Tạo biến để giữ đối số truy vấn và đối số đường dẫn tệp">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-02/src/main.rs}}
```

</Listing>

Như chúng ta đã thấy khi in vector, tên của chương trình chiếm vị trí giá trị
đầu tiên trong vector tại `args[0]`, vì vậy chúng ta bắt đầu đối số từ chỉ
mục 1. Đối số đầu tiên mà `minigrep` nhận là chuỗi chúng ta đang tìm kiếm, vì
vậy chúng ta đặt một tham chiếu đến đối số đầu tiên trong biến `query`. Đối số
thứ hai sẽ là đường dẫn tệp, vì vậy chúng ta đặt một tham chiếu đến đối số thứ
hai trong biến `file_path`.

Chúng ta tạm thời in các giá trị của những biến này để chứng minh rằng mã đang
hoạt động như chúng ta mong muốn. Hãy chạy chương trình này một lần nữa với các
đối số `test` và `sample.txt`:

```console
{{#include ../listings/ch12-an-io-project/listing-12-02/output.txt}}
```

Tuyệt vời, chương trình đang hoạt động! Giá trị của các đối số mà chúng ta cần
đang được lưu vào các biến đúng. Sau này, chúng ta sẽ thêm xử lý lỗi để đối phó
với một số tình huống lỗi tiềm ẩn, chẳng hạn như khi người dùng không cung cấp
đối số; hiện tại, chúng ta sẽ bỏ qua tình huống đó và làm việc để thêm khả năng
đọc tệp thay vào đó.

[ch13]: ch13-00-functional-features.html
[ch7-idiomatic-use]:
  ch07-04-bringing-paths-into-scope-with-the-use-keyword.html#creating-idiomatic-use-paths
