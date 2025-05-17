## Các lỗi có thể khôi phục với `Result`

Hầu hết các lỗi không đủ nghiêm trọng để yêu cầu chương trình dừng hoàn toàn.
Đôi khi khi một hàm thất bại, đó là vì lý do mà bạn có thể dễ dàng hiểu và phản
hồi. Ví dụ, nếu bạn cố mở một tập tin và thao tác đó thất bại vì tập tin không
tồn tại, bạn có thể muốn tạo tập tin đó thay vì kết thúc tiến trình.

Nhắc lại từ ["Xử lý thất bại tiềm ẩn với `Result`"][handle_failure]<!--
ignore --> trong Chương 2 rằng enum `Result` được định nghĩa với hai biến thể, `Ok`
và `Err`, như sau:

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

`T` và `E` là các tham số kiểu generic: chúng ta sẽ thảo luận chi tiết về
generics trong Chương 10. Điều bạn cần biết ngay bây giờ là `T` đại diện cho
kiểu của giá trị sẽ được trả về trong trường hợp thành công bên trong biến thể
`Ok`, và `E` đại diện cho kiểu của lỗi sẽ được trả về trong trường hợp thất bại
bên trong biến thể `Err`. Bởi vì `Result` có các tham số kiểu generic này, chúng
ta có thể sử dụng kiểu `Result` và các hàm được định nghĩa trên nó trong nhiều
tình huống khác nhau, nơi giá trị thành công và giá trị lỗi mà chúng ta muốn trả
về có thể khác nhau.

Hãy gọi một hàm trả về giá trị `Result` vì hàm có thể thất bại. Trong Listing
9-3, chúng ta thử mở một tập tin.

<Listing number="9-3" file-name="src/main.rs" caption="Mở một tập tin">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-03/src/main.rs}}
```

</Listing>

Kiểu trả về của `File::open` là `Result<T, E>`. Tham số generic `T` đã được điền
bởi quá trình triển khai của `File::open` với kiểu của giá trị thành công,
`std::fs::File`, là một handle của tập tin. Kiểu của `E` được sử dụng trong giá
trị lỗi là `std::io::Error`. Kiểu trả về này có nghĩa là lời gọi đến
`File::open` có thể thành công và trả về một handle của tập tin mà chúng ta có
thể đọc hoặc ghi. Lời gọi hàm cũng có thể thất bại: ví dụ, tập tin có thể không
tồn tại, hoặc chúng ta có thể không có quyền truy cập vào tập tin. Hàm
`File::open` cần có cách để nói cho chúng ta biết liệu nó đã thành công hay thất
bại và đồng thời cung cấp cho chúng ta handle của tập tin hoặc thông tin lỗi.
Thông tin này chính xác là những gì mà enum `Result` truyền đạt.

Trong trường hợp `File::open` thành công, giá trị trong biến
`greeting_file_result` sẽ là một thực thể của `Ok` chứa một handle của tập tin.
Trong trường hợp thất bại, giá trị trong `greeting_file_result` sẽ là một thực
thể của `Err` chứa thêm thông tin về loại lỗi đã xảy ra.

Chúng ta cần thêm vào mã trong Listing 9-3 để thực hiện các hành động khác nhau
tùy thuộc vào giá trị mà `File::open` trả về. Listing 9-4 cho thấy một cách để
xử lý `Result` bằng cách sử dụng công cụ cơ bản, biểu thức `match` mà chúng ta
đã thảo luận trong Chương 6.

<Listing number="9-4" file-name="src/main.rs" caption="Sử dụng biểu thức `match` để xử lý các biến thể `Result` có thể được trả về">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-04/src/main.rs}}
```

</Listing>

Lưu ý rằng, giống như enum `Option`, enum `Result` và các biến thể của nó đã
được đưa vào phạm vi bởi prelude, vì vậy chúng ta không cần chỉ định `Result::`
trước các biến thể `Ok` và `Err` trong các nhánh của `match`.

Khi kết quả là `Ok`, mã này sẽ trả về giá trị `file` bên trong từ biến thể `Ok`,
và sau đó chúng ta gán giá trị handle của tập tin đó cho biến `greeting_file`.
Sau `match`, chúng ta có thể sử dụng handle của tập tin để đọc hoặc ghi.

Nhánh khác của `match` xử lý trường hợp chúng ta nhận được giá trị `Err` từ
`File::open`. Trong ví dụ này, chúng ta đã chọn gọi macro `panic!`. Nếu không có
tập tin nào có tên _hello.txt_ trong thư mục hiện tại của chúng ta và chúng ta
chạy mã này, chúng ta sẽ thấy đầu ra sau đây từ macro `panic!`:

```console
{{#include ../listings/ch09-error-handling/listing-09-04/output.txt}}
```

Như thường lệ, đầu ra này cho chúng ta biết chính xác điều gì đã sai.

### Xử lý các lỗi khác nhau

Mã trong Listing 9-4 sẽ gọi `panic!` bất kể vì sao `File::open` thất bại. Tuy
nhiên, chúng ta muốn thực hiện các hành động khác nhau với các lý do thất bại
khác nhau. Nếu `File::open` thất bại vì tập tin không tồn tại, chúng ta muốn tạo
tập tin và trả về handle cho tập tin mới. Nếu `File::open` thất bại vì bất kỳ lý
do nào khác—ví dụ, vì chúng ta không có quyền mở tập tin—chúng ta vẫn muốn mã
gọi `panic!` theo cách giống như trong Listing 9-4. Để làm điều này, chúng ta
thêm một biểu thức `match` bên trong, như trong Listing 9-5.

<Listing number="9-5" file-name="src/main.rs" caption="Xử lý các loại lỗi khác nhau theo những cách khác nhau">

<!-- ignore this test because otherwise it creates hello.txt which causes other
tests to fail lol -->

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-05/src/main.rs}}
```

</Listing>

Kiểu của giá trị mà `File::open` trả về bên trong biến thể `Err` là `io::Error`,
đây là một struct được cung cấp bởi thư viện chuẩn. Struct này có một phương
thức `kind` mà chúng ta có thể gọi để lấy một giá trị `io::ErrorKind`. Enum
`io::ErrorKind` được cung cấp bởi thư viện chuẩn và có các biến thể đại diện cho
các loại lỗi khác nhau có thể phát sinh từ một thao tác `io`. Biến thể mà chúng
ta muốn sử dụng là `ErrorKind::NotFound`, cho biết tập tin mà chúng ta đang cố
gắng mở không tồn tại. Vì vậy, chúng ta khớp với `greeting_file_result`, nhưng
chúng ta cũng có một match bên trong trên `error.kind()`.

Điều kiện mà chúng ta muốn kiểm tra trong match bên trong là liệu giá trị trả về
bởi `error.kind()` có phải là biến thể `NotFound` của enum `ErrorKind` hay
không. Nếu đúng, chúng ta cố gắng tạo tập tin với `File::create`. Tuy nhiên, vì
`File::create` cũng có thể thất bại, chúng ta cần một nhánh thứ hai trong biểu
thức `match` bên trong. Khi tập tin không thể được tạo, một thông báo lỗi khác
được in ra. Nhánh thứ hai của match bên ngoài không đổi, vì vậy chương trình sẽ
panic khi có bất kỳ lỗi nào khác ngoài lỗi tập tin không tồn tại.

> #### Các lựa chọn thay thế cho việc sử dụng `match` với `Result<T, E>`
>
> `match` thật là nhiều! Biểu thức `match` rất hữu ích nhưng cũng rất nguyên
> thủy. Trong Chương 13, bạn sẽ học về closures, được sử dụng với nhiều phương
> thức được định nghĩa trên `Result<T, E>`. Các phương thức này có thể ngắn gọn
> hơn việc sử dụng `match` khi xử lý các giá trị `Result<T, E>` trong mã của
> bạn.
>
> Ví dụ, đây là một cách khác để viết logic tương tự như đã hiển thị trong
> Listing 9-5, lần này sử dụng closures và phương thức `unwrap_or_else`:
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore
> use std::fs::File;
> use std::io::ErrorKind;
>
> fn main() {
>     let greeting_file = File::open("hello.txt").unwrap_or_else(|error| {
>         if error.kind() == ErrorKind::NotFound {
>             File::create("hello.txt").unwrap_or_else(|error| {
>                 panic!("Có vấn đề khi tạo tập tin: {error:?}");
>             })
>         } else {
>             panic!("Có vấn đề khi mở tập tin: {error:?}");
>         }
>     });
> }
> ```
>
> Mặc dù mã này có cùng hành vi với Listing 9-5, nhưng nó không chứa bất kỳ biểu
> thức `match` nào và dễ đọc hơn. Quay lại ví dụ này sau khi bạn đã đọc Chương
> 13, và tìm kiếm phương thức `unwrap_or_else` trong tài liệu của thư viện
> chuẩn. Nhiều phương thức khác có thể làm gọn các biểu thức `match` lồng nhau
> lớn khi bạn xử lý lỗi.

#### Các cách viết tắt cho Panic khi có lỗi: `unwrap` và `expect`

Sử dụng `match` hoạt động khá tốt, nhưng có thể hơi dài dòng và không phải lúc
nào cũng truyền đạt ý định rõ ràng. Kiểu `Result<T, E>` có nhiều phương thức trợ
giúp được định nghĩa trên nó để thực hiện các tác vụ khác nhau, cụ thể hơn.
Phương thức `unwrap` là một phương thức viết tắt được triển khai giống như biểu
thức `match` mà chúng ta đã viết trong Listing 9-4. Nếu giá trị `Result` là biến
thể `Ok`, `unwrap` sẽ trả về giá trị bên trong `Ok`. Nếu `Result` là biến thể
`Err`, `unwrap` sẽ gọi macro `panic!` cho chúng ta. Đây là một ví dụ về `unwrap`
đang hoạt động:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-04-unwrap/src/main.rs}}
```

</Listing>

Nếu chúng ta chạy mã này mà không có tập tin _hello.txt_, chúng ta sẽ thấy một
thông báo lỗi từ lời gọi `panic!` mà phương thức `unwrap` thực hiện:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-04-unwrap
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:4:49:
called `Result::unwrap()` on an `Err` value: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Tương tự, phương thức `expect` cho phép chúng ta chọn thông báo lỗi `panic!`. Sử
dụng `expect` thay vì `unwrap` và cung cấp thông báo lỗi tốt có thể truyền đạt ý
định của bạn và làm cho việc theo dõi nguồn gốc của một panic dễ dàng hơn. Cú
pháp của `expect` trông như thế này:

<Listing file-name="src/main.rs">

```rust,should_panic
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-05-expect/src/main.rs}}
```

</Listing>

Chúng ta sử dụng `expect` theo cách tương tự như `unwrap`: để trả về handle tập
tin hoặc gọi macro `panic!`. Thông báo lỗi được sử dụng bởi `expect` trong lời
gọi của nó đến `panic!` sẽ là tham số mà chúng ta truyền cho `expect`, thay vì
thông báo `panic!` mặc định mà `unwrap` sử dụng. Đây là cách nó trông:

<!-- manual-regeneration
cd listings/ch09-error-handling/no-listing-05-expect
cargo run
copy and paste relevant text
-->

```text
thread 'main' panicked at src/main.rs:5:10:
hello.txt should be included in this project: Os { code: 2, kind: NotFound, message: "No such file or directory" }
```

Trong mã chất lượng sản xuất, hầu hết các Rustacean chọn `expect` thay vì
`unwrap` và cung cấp thêm ngữ cảnh về lý do tại sao thao tác được mong đợi sẽ
luôn thành công. Bằng cách đó, nếu giả định của bạn được chứng minh là sai, bạn
có thêm thông tin để sử dụng trong việc gỡ lỗi.

### Lan truyền lỗi

Khi triển khai của một hàm gọi thứ gì đó có thể thất bại, thay vì xử lý lỗi bên
trong chính hàm, bạn có thể trả về lỗi cho mã gọi hàm đó để nó có thể quyết định
phải làm gì. Điều này được gọi là _lan truyền_ lỗi và cung cấp nhiều quyền kiểm
soát hơn cho mã gọi hàm, nơi có thể có nhiều thông tin hoặc logic hơn để quyết
định cách xử lý lỗi so với những gì bạn có sẵn trong ngữ cảnh của mã của bạn.

Ví dụ, Listing 9-6 hiển thị một hàm đọc tên người dùng từ một tập tin. Nếu tập
tin không tồn tại hoặc không thể đọc được, hàm này sẽ trả về những lỗi đó cho mã
đã gọi hàm.

<Listing number="9-6" file-name="src/main.rs" caption="Một hàm trả về lỗi cho mã gọi hàm bằng cách sử dụng `match`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-06/src/main.rs:here}}
```

</Listing>

Hàm này có thể được viết theo cách ngắn gọn hơn nhiều, nhưng chúng ta sẽ bắt đầu
bằng cách làm nhiều thao tác thủ công để khám phá việc xử lý lỗi; cuối cùng,
chúng ta sẽ hiển thị cách viết ngắn gọn hơn. Hãy xem xét kiểu trả về của hàm
trước: `Result<String, io::Error>`. Điều này có nghĩa là hàm đang trả về một giá
trị thuộc kiểu `Result<T, E>`, trong đó tham số generic `T` đã được điền với
kiểu cụ thể `String` và kiểu generic `E` đã được điền với kiểu cụ thể
`io::Error`.

Nếu hàm này thành công mà không có bất kỳ vấn đề nào, mã gọi hàm này sẽ nhận
được một giá trị `Ok` chứa một `String`—tên người dùng mà hàm này đã đọc từ tập
tin. Nếu hàm này gặp bất kỳ vấn đề nào, mã gọi sẽ nhận được một giá trị `Err`
chứa một instance của `io::Error` có chứa thêm thông tin về những vấn đề đã xảy
ra. Chúng ta đã chọn `io::Error` làm kiểu trả về của hàm này vì đó là kiểu của
giá trị lỗi được trả về từ cả hai thao tác mà chúng ta đang gọi trong thân hàm
có thể thất bại: hàm `File::open` và phương thức `read_to_string`.

Thân hàm bắt đầu bằng cách gọi hàm `File::open`. Sau đó, chúng ta xử lý giá trị
`Result` với một `match` tương tự như `match` trong Listing 9-4. Nếu
`File::open` thành công, handle tập tin trong biến mẫu `file` sẽ trở thành giá
trị trong biến có thể thay đổi `username_file` và hàm tiếp tục. Trong trường hợp
`Err`, thay vì gọi `panic!`, chúng ta sử dụng từ khóa `return` để thoát sớm khỏi
hàm hoàn toàn và truyền giá trị lỗi từ `File::open`, hiện đang nằm trong biến
mẫu `e`, trở lại cho mã gọi như là giá trị lỗi của hàm này.

Vì vậy, nếu chúng ta có một handle tập tin trong `username_file`, hàm tiếp theo
sẽ tạo một `String` mới trong biến `username` và gọi phương thức
`read_to_string` trên handle tập tin trong `username_file` để đọc nội dung của
tập tin vào `username`. Phương thức `read_to_string` cũng trả về một `Result` vì
nó có thể thất bại, ngay cả khi `File::open` đã thành công. Vì vậy, chúng ta cần
một `match` khác để xử lý `Result` đó: nếu `read_to_string` thành công, thì hàm
của chúng ta đã thành công, và chúng ta trả về tên người dùng từ tập tin hiện
đang nằm trong `username` được bọc trong một `Ok`. Nếu `read_to_string` thất
bại, chúng ta trả về giá trị lỗi theo cách tương tự như cách chúng ta trả về giá
trị lỗi trong `match` đã xử lý giá trị trả về của `File::open`. Tuy nhiên, chúng
ta không cần nói `return` một cách rõ ràng, vì đây là biểu thức cuối cùng trong
hàm.

Mã gọi mã này sẽ xử lý việc nhận giá trị `Ok` chứa tên người dùng hoặc giá trị
`Err` chứa một `io::Error`. Mã gọi quyết định phải làm gì với những giá trị này.
Nếu mã gọi nhận được giá trị `Err`, nó có thể gọi `panic!` và làm crash chương
trình, sử dụng một tên người dùng mặc định, hoặc tìm kiếm tên người dùng từ một
nơi khác ngoài tập tin, ví dụ vậy. Chúng ta không có đủ thông tin về những gì mã
gọi thực sự đang cố gắng làm, vì vậy chúng ta lan truyền tất cả thông tin thành
công hoặc lỗi lên trên để nó xử lý một cách phù hợp.

Mẫu lan truyền lỗi này rất phổ biến trong Rust đến nỗi Rust cung cấp toán tử dấu
hỏi `?` để làm điều này dễ dàng hơn.

#### Một cách viết tắt để lan truyền lỗi: Toán tử `?`

Listing 9-7 hiển thị một triển khai của `read_username_from_file` có cùng chức
năng như trong Listing 9-6, nhưng triển khai này sử dụng toán tử `?`.

<Listing number="9-7" file-name="src/main.rs" caption="Một hàm trả về lỗi cho mã gọi bằng cách sử dụng toán tử `?`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-07/src/main.rs:here}}
```

</Listing>

Dấu `?` được đặt sau một giá trị `Result` được định nghĩa để hoạt động theo cách
gần như giống như các biểu thức `match` mà chúng ta đã định nghĩa để xử lý các
giá trị `Result` trong Listing 9-6. Nếu giá trị của `Result` là `Ok`, giá trị
bên trong `Ok` sẽ được trả về từ biểu thức này, và chương trình sẽ tiếp tục. Nếu
giá trị là `Err`, `Err` sẽ được trả về từ toàn bộ hàm như thể chúng ta đã sử
dụng từ khóa `return` để giá trị lỗi được lan truyền đến mã gọi.

Có một sự khác biệt giữa những gì mà biểu thức `match` từ Listing 9-6 làm và
những gì toán tử `?` làm: các giá trị lỗi mà toán tử `?` được gọi trên chúng đi
qua hàm `from`, được định nghĩa trong trait `From` trong thư viện chuẩn, được sử
dụng để chuyển đổi giá trị từ một kiểu sang một kiểu khác. Khi toán tử `?` gọi
hàm `from`, kiểu lỗi nhận được được chuyển đổi thành kiểu lỗi được định nghĩa
trong kiểu trả về của hàm hiện tại. Điều này hữu ích khi một hàm trả về một kiểu
lỗi để đại diện cho tất cả các cách mà một hàm có thể thất bại, ngay cả khi các
phần có thể thất bại vì nhiều lý do khác nhau.

Ví dụ, chúng ta có thể thay đổi hàm `read_username_from_file` trong Listing 9-7
để trả về một kiểu lỗi tùy chỉnh có tên `OurError` mà chúng ta định nghĩa. Nếu
chúng ta cũng định nghĩa `impl From<io::Error> for OurError` để xây dựng một
instance của `OurError` từ một `io::Error`, thì các lời gọi toán tử `?` trong
thân của `read_username_from_file` sẽ gọi `from` và chuyển đổi các kiểu lỗi mà
không cần thêm bất kỳ mã nào vào hàm.

Trong ngữ cảnh của Listing 9-7, dấu `?` ở cuối của lời gọi `File::open` sẽ trả
về giá trị bên trong `Ok` cho biến `username_file`. Nếu xảy ra lỗi, toán tử `?`
sẽ thoát sớm khỏi toàn bộ hàm và đưa bất kỳ giá trị `Err` nào cho mã gọi. Điều
tương tự cũng áp dụng cho dấu `?` ở cuối lời gọi `read_to_string`.

Toán tử `?` loại bỏ rất nhiều mã soạn sẵn và làm cho triển khai của hàm này đơn
giản hơn. Chúng ta thậm chí có thể làm cho mã này ngắn gọn hơn nữa bằng cách nối
các lời gọi phương thức ngay sau `?`, như trong Listing 9-8.

<Listing number="9-8" file-name="src/main.rs" caption="Nối các lời gọi phương thức sau toán tử `?`">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-08/src/main.rs:here}}
```

</Listing>

Chúng ta đã di chuyển việc tạo `String` mới trong `username` lên đầu hàm; phần
đó không thay đổi. Thay vì tạo biến `username_file`, chúng ta đã nối lời gọi đến
`read_to_string` trực tiếp vào kết quả của `File::open("hello.txt")?`. Chúng ta
vẫn có một dấu `?` ở cuối lời gọi `read_to_string`, và chúng ta vẫn trả về một
giá trị `Ok` chứa `username` khi cả `File::open` và `read_to_string` đều thành
công thay vì trả về lỗi. Chức năng một lần nữa giống như trong Listing 9-6 và
Listing 9-7; đây chỉ là một cách viết khác, phong cách hơn.

Listing 9-9 hiển thị một cách để làm cho nó thậm chí còn ngắn gọn hơn bằng cách
sử dụng `fs::read_to_string`.

<Listing number="9-9" file-name="src/main.rs" caption="Sử dụng `fs::read_to_string` thay vì mở và sau đó đọc tập tin">

<!-- Deliberately not using rustdoc_include here; the `main` function in the
file panics. We do want to include it for reader experimentation purposes, but
don't want to include it for rustdoc testing purposes. -->

```rust
{{#include ../listings/ch09-error-handling/listing-09-09/src/main.rs:here}}
```

</Listing>

Đọc một tập tin vào một chuỗi là một hoạt động khá phổ biến, vì vậy thư viện
chuẩn cung cấp hàm `fs::read_to_string` thuận tiện để mở tập tin, tạo một
`String` mới, đọc nội dung của tập tin, đưa nội dung vào `String` đó, và trả về
nó. Tất nhiên, sử dụng `fs::read_to_string` không cho chúng ta cơ hội để giải
thích tất cả việc xử lý lỗi, vì vậy chúng ta đã làm nó theo cách dài dòng hơn
trước.

#### Toán tử `?` có thể được sử dụng ở đâu

Toán tử `?` chỉ có thể được sử dụng trong các hàm có kiểu trả về tương thích với
giá trị mà `?` được sử dụng trên đó. Điều này là vì toán tử `?` được định nghĩa
để thực hiện việc trả về sớm một giá trị từ hàm, theo cùng cách như biểu thức
`match` mà chúng ta đã định nghĩa trong Listing 9-6. Trong Listing 9-6, `match`
đang sử dụng một giá trị `Result`, và nhánh trả về sớm trả về một giá trị
`Err(e)`. Kiểu trả về của hàm phải là một `Result` để nó tương thích với
`return` này.

Trong Listing 9-10, hãy xem lỗi mà chúng ta sẽ nhận được nếu chúng ta sử dụng
toán tử `?` trong một hàm `main` với kiểu trả về không tương thích với kiểu của
giá trị mà chúng ta sử dụng `?` trên đó:

<Listing number="9-10" file-name="src/main.rs" caption="Cố gắng sử dụng toán tử `?` trong hàm `main` trả về `()` sẽ không biên dịch">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-10/src/main.rs}}
```

</Listing>

Mã này mở một tập tin, điều này có thể thất bại. Toán tử `?` theo sau giá trị
`Result` được trả về bởi `File::open`, nhưng hàm `main` này có kiểu trả về là
`()`, không phải `Result`. Khi chúng ta biên dịch mã này, chúng ta nhận được
thông báo lỗi sau:

```console
{{#include ../listings/ch09-error-handling/listing-09-10/output.txt}}
```

Lỗi này chỉ ra rằng chúng ta chỉ được phép sử dụng toán tử `?` trong một hàm trả
về `Result`, `Option`, hoặc một kiểu khác mà có triển khai `FromResidual`.

Để sửa lỗi, bạn có hai lựa chọn. Một lựa chọn là thay đổi kiểu trả về của hàm để
tương thích với giá trị mà bạn đang sử dụng toán tử `?` trên đó miễn là bạn
không có hạn chế nào ngăn bạn làm điều đó. Lựa chọn khác là sử dụng `match` hoặc
một trong các phương thức của `Result<T, E>` để xử lý `Result<T, E>` theo cách
phù hợp.

Thông báo lỗi cũng đề cập rằng `?` có thể được sử dụng với các giá trị
`Option<T>` cũng như. Như với việc sử dụng `?` trên `Result`, bạn chỉ có thể sử
dụng `?` trên `Option` trong một hàm trả về một `Option`. Hành vi của toán tử
`?` khi được gọi trên một `Option<T>` tương tự như hành vi của nó khi được gọi
trên một `Result<T, E>`: nếu giá trị là `None`, `None` sẽ được trả về sớm từ hàm
tại điểm đó. Nếu giá trị là `Some`, giá trị bên trong `Some` là giá trị kết quả
của biểu thức, và hàm tiếp tục. Listing 9-11 có một ví dụ về một hàm tìm ký tự
cuối cùng của dòng đầu tiên trong văn bản đã cho.

<Listing number="9-11" caption="Sử dụng toán tử `?` trên một giá trị `Option<T>`">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-11/src/main.rs:here}}
```

</Listing>

Hàm này trả về `Option<char>` vì có thể có một ký tự ở đó, nhưng cũng có thể
không có. Mã này nhận tham số slice chuỗi `text` và gọi phương thức `lines` trên
nó, trả về một iterator trên các dòng trong chuỗi. Vì hàm này muốn kiểm tra dòng
đầu tiên, nó gọi `next` trên iterator để lấy giá trị đầu tiên từ iterator. Nếu
`text` là chuỗi rỗng, lời gọi này đến `next` sẽ trả về `None`, trong trường hợp
đó chúng ta sử dụng `?` để dừng và trả về `None` từ `last_char_of_first_line`.
Nếu `text` không phải là chuỗi rỗng, `next` sẽ trả về một giá trị `Some` chứa
một slice chuỗi của dòng đầu tiên trong `text`.

Dấu `?` trích xuất slice chuỗi, và chúng ta có thể gọi `chars` trên slice chuỗi
đó để lấy một iterator của các ký tự của nó. Chúng ta quan tâm đến ký tự cuối
cùng trong dòng đầu tiên này, vì vậy chúng ta gọi `last` để trả về mục cuối cùng
trong iterator. Đây là một `Option` vì có thể dòng đầu tiên là chuỗi rỗng; ví
dụ, nếu `text` bắt đầu bằng một dòng trống nhưng có các ký tự trên các dòng
khác, như trong `"\nhi"`. Tuy nhiên, nếu có một ký tự cuối cùng trên dòng đầu
tiên, nó sẽ được trả về trong biến thể `Some`. Toán tử `?` ở giữa cung cấp cho
chúng ta một cách súc tích để biểu thị logic này, cho phép chúng ta triển khai
hàm trong một dòng. Nếu chúng ta không thể sử dụng toán tử `?` trên `Option`,
chúng ta sẽ phải triển khai logic này bằng cách sử dụng nhiều lời gọi phương
thức hơn hoặc một biểu thức `match`.

Lưu ý rằng bạn có thể sử dụng toán tử `?` trên một `Result` trong một hàm trả về
`Result`, và bạn có thể sử dụng toán tử `?` trên một `Option` trong một hàm trả
về `Option`, nhưng bạn không thể kết hợp và phù hợp. Toán tử `?` sẽ không tự
động chuyển đổi một `Result` thành một `Option` hoặc ngược lại; trong những
trường hợp đó, bạn có thể sử dụng các phương thức như phương thức `ok` trên
`Result` hoặc phương thức `ok_or` trên `Option` để thực hiện chuyển đổi một cách
rõ ràng.

Cho đến nay, tất cả các hàm `main` mà chúng ta đã sử dụng đều trả về `()`. Hàm
`main` rất đặc biệt vì nó là điểm vào và điểm ra của một chương trình thực thi,
và có các hạn chế về kiểu trả về của nó để chương trình hoạt động như mong đợi.

May mắn thay, `main` cũng có thể trả về một `Result<(), E>`. Listing 9-12 có mã
từ Listing 9-10, nhưng chúng ta đã thay đổi kiểu trả về của `main` thành
`Result<(), Box<dyn Error>>` và đã thêm một giá trị trả về `Ok(())` vào cuối. Mã
này sẽ biên dịch bây giờ.

<Listing number="9-12" file-name="src/main.rs" caption="Thay đổi `main` để trả về `Result<(), E>` cho phép sử dụng toán tử `?` trên các giá trị `Result`.">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-12/src/main.rs}}
```

</Listing>

Kiểu `Box<dyn Error>` là một _trait object_, chúng ta sẽ nói về nó trong ["Sử
dụng các Trait Object cho phép các giá trị của các kiểu khác
nhau"][trait-objects]<!--
ignore --> trong Chương 18. Hiện tại, bạn có thể đọc `Box<dyn Error>` như là "bất
kỳ loại lỗi nào." Sử dụng `?` trên một giá trị `Result` trong một hàm `main` với
kiểu lỗi `Box<dyn Error>` được cho phép vì nó cho phép bất kỳ giá trị `Err` nào được
trả về sớm. Mặc dù thân của hàm `main` này sẽ chỉ trả về các lỗi của kiểu `std::io::Error`,
nhưng bằng cách chỉ định `Box<dyn Error>`, chữ ký này sẽ tiếp tục chính xác ngay
cả khi thêm mã trả về các lỗi khác được thêm vào thân của `main`.

Khi một hàm `main` trả về một `Result<(), E>`, chương trình thực thi sẽ thoát
với giá trị `0` nếu `main` trả về `Ok(())` và sẽ thoát với giá trị khác 0 nếu
`main` trả về một giá trị `Err`. Các chương trình thực thi được viết bằng C trả
về số nguyên khi chúng thoát: các chương trình thoát thành công trả về số nguyên
`0`, và các chương trình bị lỗi trả về một số nguyên khác 0. Rust cũng trả về số
nguyên từ các chương trình thực thi để tương thích với quy ước này.

Hàm `main` có thể trả về bất kỳ kiểu nào triển khai [trait
`std::process::Termination`][termination]<!-- ignore -->, chứa một hàm `report`
trả về một `ExitCode`. Tham khảo tài liệu của thư viện chuẩn để biết thêm thông
tin về việc triển khai trait `Termination` cho các kiểu của riêng bạn.

Bây giờ chúng ta đã thảo luận về chi tiết của việc gọi `panic!` hoặc trả về
`Result`, hãy quay lại chủ đề về cách quyết định sử dụng cái nào trong trường
hợp nào.

[handle_failure]:
  ch02-00-guessing-game-tutorial.html#handling-potential-failure-with-result
[trait-objects]:
  ch18-02-trait-objects.html#using-trait-objects-that-allow-for-values-of-different-types
[termination]: ../std/process/trait.Termination.html
