## Xuất bản một Crate lên Crates.io

Chúng ta đã sử dụng các gói từ [crates.io](https://crates.io/)<!-- ignore -->
làm các phụ thuộc của dự án, nhưng bạn cũng có thể chia sẻ mã của mình với người
khác bằng cách xuất bản các gói của riêng bạn. Sổ đăng ký crate tại
[crates.io](https://crates.io/)<!-- ignore --> phân phối mã nguồn của các gói
của bạn, vì vậy nó chủ yếu lưu trữ mã nguồn mở.

Rust và Cargo có các tính năng giúp gói đã xuất bản của bạn dễ dàng được tìm
thấy và sử dụng hơn. Chúng ta sẽ nói về một số tính năng này tiếp theo và sau đó
giải thích cách xuất bản một gói.

### Tạo Bình luận Tài liệu Hữu ích

Việc tài liệu hóa chính xác các gói sẽ giúp người dùng khác biết cách và khi nào
sử dụng chúng, vì vậy đáng để đầu tư thời gian viết tài liệu. Trong Chương 3,
chúng ta đã thảo luận về cách bình luận mã Rust bằng hai dấu gạch chéo, `//`.
Rust cũng có một loại bình luận đặc biệt cho tài liệu, được gọi một cách thuận
tiện là _bình luận tài liệu_, sẽ tạo ra tài liệu HTML. HTML hiển thị nội dung
của các bình luận tài liệu cho các mục API công khai dành cho các lập trình viên
quan tâm đến việc biết cách _sử dụng_ crate của bạn thay vì cách crate của bạn
được _triển khai_.

Bình luận tài liệu sử dụng ba dấu gạch chéo, `///`, thay vì hai và hỗ trợ ký
hiệu Markdown để định dạng văn bản. Đặt bình luận tài liệu ngay trước mục mà
chúng tài liệu hóa. Listing 14-1 hiển thị bình luận tài liệu cho một hàm
`add_one` trong một crate có tên `my_crate`.

<Listing number="14-1" file-name="src/lib.rs" caption="Một bình luận tài liệu cho một hàm">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-01/src/lib.rs}}
```

</Listing>

Ở đây, chúng ta cung cấp mô tả về những gì hàm `add_one` làm, bắt đầu một phần
với tiêu đề `Examples`, và sau đó cung cấp mã minh họa cách sử dụng hàm
`add_one`. Chúng ta có thể tạo tài liệu HTML từ bình luận tài liệu này bằng cách
chạy `cargo doc`. Lệnh này chạy công cụ `rustdoc` được phân phối với Rust và đặt
tài liệu HTML được tạo ra trong thư mục _target/doc_.

Để thuận tiện, chạy `cargo doc --open` sẽ xây dựng HTML cho tài liệu của crate
hiện tại (cũng như tài liệu cho tất cả các phụ thuộc của crate) và mở kết quả
trong trình duyệt web. Điều hướng đến hàm `add_one` và bạn sẽ thấy cách văn bản
trong bình luận tài liệu được hiển thị, như trong Hình 14-1.

<img alt="Tài liệu HTML được tạo ra cho hàm `add_one` của `my_crate`" src="img/trpl14-01.png" class="center" />

<span class="caption">Hình 14-1: Tài liệu HTML cho hàm `add_one`</span>

#### Các Phần Thường Được Sử dụng

Chúng ta đã sử dụng tiêu đề Markdown `# Examples` trong Listing 14-1 để tạo một
phần trong HTML với tiêu đề "Examples." Dưới đây là một số phần khác mà tác giả
crate thường sử dụng trong tài liệu của họ:

- **Panics**: Các tình huống mà hàm đang được tài liệu hóa có thể gây panic.
  Người gọi hàm không muốn chương trình của họ bị panic nên đảm bảo rằng họ
  không gọi hàm trong những tình huống này.
- **Errors**: Nếu hàm trả về một `Result`, mô tả các loại lỗi có thể xảy ra và
  điều kiện nào có thể gây ra những lỗi đó được trả về có thể hữu ích cho người
  gọi để họ có thể viết mã xử lý các loại lỗi khác nhau theo các cách khác nhau.
- **Safety**: Nếu hàm là `unsafe` khi gọi (chúng ta thảo luận về tính không an
  toàn trong Chương 20), nên có một phần giải thích tại sao hàm không an toàn và
  đề cập đến các bất biến mà hàm mong đợi người gọi duy trì.

Hầu hết các bình luận tài liệu không cần tất cả các phần này, nhưng đây là một
danh sách kiểm tra tốt để nhắc nhở bạn về các khía cạnh của mã mà người dùng sẽ
quan tâm.

#### Bình luận Tài liệu như Các Bài Kiểm thử

Thêm các khối mã ví dụ trong bình luận tài liệu của bạn có thể giúp minh họa
cách sử dụng thư viện của bạn, và làm như vậy có một lợi ích bổ sung: chạy
`cargo test` sẽ chạy các ví dụ mã trong tài liệu của bạn như các bài kiểm thử!
Không có gì tốt hơn tài liệu với các ví dụ. Nhưng không có gì tệ hơn các ví dụ
không hoạt động vì mã đã thay đổi kể từ khi tài liệu được viết. Nếu chúng ta
chạy `cargo test` với tài liệu cho hàm `add_one` từ Listing 14-1, chúng ta sẽ
thấy một phần trong kết quả kiểm thử trông như thế này:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-01/
cargo test
copy just the doc-tests section below
-->

```text
   Doc-tests my_crate

running 1 test
test src/lib.rs - add_one (line 5) ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.27s
```

Bây giờ, nếu chúng ta thay đổi hàm hoặc ví dụ để `assert_eq!` trong ví dụ gây
panic, và chạy `cargo test` một lần nữa, chúng ta sẽ thấy rằng các bài kiểm thử
tài liệu phát hiện ra ví dụ và mã không còn đồng bộ với nhau!

#### Bình luận về Các Mục Chứa

Kiểu bình luận tài liệu `//!` thêm tài liệu vào mục _chứa_ bình luận thay vì vào
các mục _theo sau_ bình luận. Chúng ta thường sử dụng các bình luận tài liệu này
bên trong tệp gốc crate (_src/lib.rs_ theo quy ước) hoặc bên trong một module để
tài liệu hóa crate hoặc module nói chung.

Ví dụ, để thêm tài liệu mô tả mục đích của crate `my_crate` chứa hàm `add_one`,
chúng ta thêm bình luận tài liệu bắt đầu bằng `//!` vào đầu tệp _src/lib.rs_,
như trong Listing 14-2.

<Listing number="14-2" file-name="src/lib.rs" caption="Tài liệu cho toàn bộ crate `my_crate`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-02/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng không có bất kỳ mã nào sau dòng cuối cùng bắt đầu bằng `//!`. Bởi vì
chúng ta bắt đầu bình luận bằng `//!` thay vì `///`, chúng ta đang tài liệu hóa
mục chứa bình luận này thay vì một mục theo sau bình luận này. Trong trường hợp
này, mục đó là tệp _src/lib.rs_, là gốc crate. Các bình luận này mô tả toàn bộ
crate.

Khi chúng ta chạy `cargo doc --open`, các bình luận này sẽ hiển thị trên trang
đầu của tài liệu cho `my_crate` phía trên danh sách các mục công khai trong
crate, như trong Hình 14-2.

<img alt="Tài liệu HTML được tạo ra với một bình luận cho toàn bộ crate" src="img/trpl14-02.png" class="center" />

<span class="caption">Hình 14-2: Tài liệu được tạo ra cho `my_crate`, bao gồm
bình luận mô tả toàn bộ crate</span>

Bình luận tài liệu trong các mục rất hữu ích để mô tả crates và modules đặc
biệt. Sử dụng chúng để giải thích mục đích tổng thể của container để giúp người
dùng hiểu tổ chức của crate.

### Xuất một API Công khai Thuận tiện với `pub use`

Cấu trúc của API công khai của bạn là một cân nhắc quan trọng khi xuất bản một
crate. Những người sử dụng crate của bạn ít quen thuộc với cấu trúc hơn bạn và
có thể gặp khó khăn khi tìm các phần họ muốn sử dụng nếu crate của bạn có một
cấu trúc phân cấp module lớn.

Trong Chương 7, chúng ta đã đề cập đến cách làm cho các mục công khai bằng cách
sử dụng từ khóa `pub`, và cách đưa các mục vào phạm vi với từ khóa `use`. Tuy
nhiên, cấu trúc có ý nghĩa đối với bạn trong khi phát triển một crate có thể
không thuận tiện cho người dùng của bạn. Bạn có thể muốn tổ chức các struct của
mình trong một hệ thống phân cấp chứa nhiều cấp, nhưng sau đó những người muốn
sử dụng một kiểu bạn đã định nghĩa sâu trong hệ thống phân cấp có thể gặp khó
khăn khi tìm ra kiểu đó tồn tại. Họ cũng có thể bực mình khi phải nhập
`use my_crate::some_module::another_module::UsefulType;` thay vì
`use my_crate::UsefulType;`.

Tin tốt là nếu cấu trúc _không_ thuận tiện cho người khác sử dụng từ một thư
viện khác, bạn không cần phải sắp xếp lại tổ chức nội bộ của mình: thay vào đó,
bạn có thể tái xuất các mục để tạo một cấu trúc công khai khác với cấu trúc
riêng tư của bạn bằng cách sử dụng `pub use`. _Tái xuất_ lấy một mục công khai ở
một vị trí và làm cho nó công khai ở một vị trí khác, như thể nó được định nghĩa
ở vị trí khác đó.

Ví dụ, giả sử chúng ta đã tạo một thư viện có tên `art` để mô hình hóa các khái
niệm nghệ thuật. Trong thư viện này có hai module: module `kinds` chứa hai enum
có tên `PrimaryColor` và `SecondaryColor` và module `utils` chứa một hàm có tên
`mix`, như trong Listing 14-3:

<Listing number="14-3" file-name="src/lib.rs" caption="Một thư viện `art` với các mục được tổ chức thành các module `kinds` và `utils`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-03/src/lib.rs:here}}
```

</Listing>

Hình 14-3 hiển thị trang đầu tiên của tài liệu cho crate này được tạo ra bởi
`cargo doc` sẽ trông như thế nào:

<img alt="Tài liệu được tạo ra cho thư viện `art` liệt kê các module `kinds` và `utils`" src="img/trpl14-03.png" class="center" />

<span class="caption">Hình 14-3: Trang đầu tiên của tài liệu cho `art` liệt kê
các module `kinds` và `utils`</span>

Lưu ý rằng các kiểu `PrimaryColor` và `SecondaryColor` không được liệt kê trên
trang đầu, cũng như hàm `mix`. Chúng ta phải nhấp vào `kinds` và `utils` để xem
chúng.

Một crate khác phụ thuộc vào thư viện này sẽ cần các câu lệnh `use` đưa các mục
từ `art` vào phạm vi, chỉ định cấu trúc module hiện đang được định nghĩa.
Listing 14-4 hiển thị một ví dụ về crate sử dụng các mục `PrimaryColor` và `mix`
từ crate `art`:

<Listing number="14-4" file-name="src/main.rs" caption="Một crate sử dụng các mục của crate `art` với cấu trúc nội bộ được xuất">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-04/src/main.rs}}
```

</Listing>

Tác giả của mã trong Listing 14-4, sử dụng crate `art`, đã phải tìm ra rằng
`PrimaryColor` nằm trong module `kinds` và `mix` nằm trong module `utils`. Cấu
trúc module của crate `art` liên quan hơn đến các nhà phát triển làm việc trên
crate `art` hơn là những người sử dụng nó. Cấu trúc nội bộ không chứa bất kỳ
thông tin hữu ích nào cho ai đó đang cố gắng hiểu cách sử dụng crate `art`, mà
thay vào đó gây ra nhầm lẫn vì các nhà phát triển sử dụng nó phải tìm ra nơi để
tìm, và phải chỉ định tên module trong các câu lệnh `use`.

Để loại bỏ tổ chức nội bộ khỏi API công khai, chúng ta có thể sửa đổi mã crate
`art` trong Listing 14-3 để thêm các câu lệnh `pub use` để tái xuất các mục ở
cấp cao nhất, như trong Listing 14-5:

<Listing number="14-5" file-name="src/lib.rs" caption="Thêm các câu lệnh `pub use` để tái xuất các mục">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-05/src/lib.rs:here}}
```

</Listing>

Tài liệu API mà `cargo doc` tạo ra cho crate này bây giờ sẽ liệt kê và liên kết
các tái xuất trên trang đầu, như trong Hình 14-4, làm cho các kiểu
`PrimaryColor` và `SecondaryColor` và hàm `mix` dễ tìm hơn.

<img alt="Tài liệu được tạo ra cho thư viện `art` với các tái xuất trên trang đầu" src="img/trpl14-04.png" class="center" />

<span class="caption">Hình 14-4: Trang đầu tiên của tài liệu cho `art` liệt kê
các tái xuất</span>

Người dùng crate `art` vẫn có thể thấy và sử dụng cấu trúc nội bộ từ Listing
14-3 như được minh họa trong Listing 14-4, hoặc họ có thể sử dụng cấu trúc thuận
tiện hơn trong Listing 14-5, như được hiển thị trong Listing 14-6:

<Listing number="14-6" file-name="src/main.rs" caption="Một chương trình sử dụng các mục được tái xuất từ crate `art`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-06/src/main.rs:here}}
```

</Listing>

Trong các trường hợp có nhiều module lồng nhau, tái xuất các kiểu ở cấp cao nhất
với `pub use` có thể tạo ra sự khác biệt đáng kể trong trải nghiệm của những
người sử dụng crate. Một cách sử dụng phổ biến khác của `pub use` là tái xuất
các định nghĩa của một phụ thuộc trong crate hiện tại để làm cho các định nghĩa
của crate đó trở thành một phần của API công khai của crate của bạn.

Tạo một cấu trúc API công khai hữu ích là một nghệ thuật hơn là một khoa học, và
bạn có thể lặp lại để tìm API hoạt động tốt nhất cho người dùng của bạn. Chọn
`pub use` cho bạn sự linh hoạt trong cách bạn cấu trúc crate của mình nội bộ và
tách biệt cấu trúc nội bộ đó khỏi những gì bạn trình bày cho người dùng của bạn.
Xem xét một số mã của crates bạn đã cài đặt để xem liệu cấu trúc nội bộ của
chúng có khác với API công khai của chúng hay không.

### Thiết lập Tài khoản Crates.io

Trước khi bạn có thể xuất bản bất kỳ crate nào, bạn cần tạo một tài khoản trên
[crates.io](https://crates.io/)<!-- ignore --> và nhận một token API. Để làm
điều này, hãy truy cập trang chủ tại
[crates.io](https://crates.io/)<!-- ignore --> và đăng nhập thông qua tài khoản
GitHub. (Tài khoản GitHub hiện là một yêu cầu, nhưng trang web có thể hỗ trợ các
cách khác để tạo tài khoản trong tương lai.) Sau khi đăng nhập, hãy truy cập cài
đặt tài khoản của bạn tại
[https://crates.io/me/](https://crates.io/me/)<!-- ignore --> và lấy khóa API
của bạn. Sau đó chạy lệnh `cargo login` và dán khóa API của bạn khi được nhắc,
như thế này:

```console
$ cargo login
abcdefghijklmnopqrstuvwxyz012345
```

Lệnh này sẽ thông báo cho Cargo về token API của bạn và lưu trữ nó cục bộ trong
_~/.cargo/credentials.toml_. Lưu ý rằng token này là một _bí mật_: không chia sẻ
nó với bất kỳ ai khác. Nếu bạn chia sẻ nó với bất kỳ ai vì bất kỳ lý do gì, bạn
nên thu hồi nó và tạo một token mới trên
[crates.io](https://crates.io/)<!-- ignore -->.

### Thêm Metadata cho một Crate Mới

Giả sử bạn có một crate bạn muốn xuất bản. Trước khi xuất bản, bạn sẽ cần thêm
một số metadata trong phần `[package]` của tệp _Cargo.toml_ của crate.

Crate của bạn sẽ cần một tên duy nhất. Trong khi bạn đang làm việc trên một
crate cục bộ, bạn có thể đặt tên crate bất cứ điều gì bạn thích. Tuy nhiên, tên
crate trên [crates.io](https://crates.io/)<!-- ignore --> được phân bổ theo
nguyên tắc ai đến trước được phục vụ trước. Một khi tên crate được lấy, không ai
khác có thể xuất bản một crate với tên đó. Trước khi cố gắng xuất bản một crate,
hãy tìm kiếm tên bạn muốn sử dụng. Nếu tên đã được sử dụng, bạn sẽ cần tìm một
tên khác và chỉnh sửa trường `name` trong tệp _Cargo.toml_ dưới phần `[package]`
để sử dụng tên mới cho việc xuất bản, như sau:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
```

Ngay cả khi bạn đã chọn một tên duy nhất, khi bạn chạy `cargo publish` để xuất
bản crate tại thời điểm này, bạn sẽ nhận được cảnh báo và sau đó là lỗi:

<!-- manual-regeneration
Create a new package with an unregistered name, making no further modifications
  to the generated package, so it is missing the description and license fields.
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
warning: manifest has no description, license, license-file, documentation, homepage or repository.
See https://doc.rust-lang.org/cargo/reference/manifest.html#package-metadata for more info.
--snip--
error: failed to publish to registry at https://crates.io

Caused by:
  the remote server responded with an error (status 400 Bad Request): missing or empty metadata fields: description, license. Please see https://doc.rust-lang.org/cargo/reference/manifest.html for more information on configuring these fields
```

Điều này dẫn đến lỗi vì bạn thiếu một số thông tin quan trọng: một mô tả và giấy
phép là bắt buộc để mọi người biết crate của bạn làm gì và theo những điều khoản
nào họ có thể sử dụng nó. Trong _Cargo.toml_, thêm một mô tả chỉ là một câu hoặc
hai, vì nó sẽ xuất hiện với crate của bạn trong kết quả tìm kiếm. Đối với trường
`license`, bạn cần cung cấp một _giá trị định danh giấy phép_. [Trao đổi Dữ liệu
Gói Phần mềm (SPDX) của Quỹ Linux][spdx] liệt kê các định danh bạn có thể sử
dụng cho giá trị này. Ví dụ, để chỉ định rằng bạn đã cấp phép crate của mình
bằng Giấy phép MIT, hãy thêm định danh `MIT`:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
license = "MIT"
```

Nếu bạn muốn sử dụng giấy phép không xuất hiện trong SPDX, bạn cần đặt văn bản
của giấy phép đó vào một tệp, bao gồm tệp đó trong dự án của bạn, và sau đó sử
dụng `license-file` để chỉ định tên của tệp đó thay vì sử dụng khóa `license`.

Hướng dẫn về giấy phép nào phù hợp cho dự án của bạn nằm ngoài phạm vi của cuốn
sách này. Nhiều người trong cộng đồng Rust cấp phép cho các dự án của họ theo
cách giống như Rust bằng cách sử dụng giấy phép kép là `MIT OR Apache-2.0`. Thực
hành này cho thấy bạn cũng có thể chỉ định nhiều định danh giấy phép được phân
tách bằng `OR` để có nhiều giấy phép cho dự án của bạn.

Với một tên duy nhất, phiên bản, mô tả của bạn, và giấy phép được thêm vào, tệp
_Cargo.toml_ cho một dự án sẵn sàng xuất bản có thể trông như thế này:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
[package]
name = "guessing_game"
version = "0.1.0"
edition = "2024"
description = "Một trò chơi vui nhộn nơi bạn đoán số mà máy tính đã chọn."
license = "MIT OR Apache-2.0"

[dependencies]
```

[Tài liệu của Cargo](https://doc.rust-lang.org/cargo/) mô tả các metadata khác
bạn có thể chỉ định để đảm bảo người khác có thể khám phá và sử dụng crate của
bạn dễ dàng hơn.

### Xuất bản lên Crates.io

Bây giờ bạn đã tạo một tài khoản, lưu token API của bạn, chọn một tên cho crate
của bạn, và chỉ định các metadata bắt buộc, bạn đã sẵn sàng để xuất bản! Xuất
bản một crate tải lên một phiên bản cụ thể lên
[crates.io](https://crates.io/)<!-- ignore --> để người khác sử dụng.

Hãy cẩn thận, vì một lần xuất bản là _vĩnh viễn_. Phiên bản không bao giờ có thể
được ghi đè, và mã không thể bị xóa trừ trong một số trường hợp nhất định. Một
mục tiêu chính của [crates.io](https://crates.io/)<!-- ignore --> là hoạt động
như một kho lưu trữ vĩnh viễn của mã để các bản dựng của tất cả các dự án phụ
thuộc vào crates từ [crates.io](https://crates.io/)<!-- ignore --> sẽ tiếp tục
hoạt động. Cho phép xóa phiên bản sẽ làm cho việc đạt được mục tiêu đó là không
thể. Tuy nhiên, không có giới hạn về số lượng phiên bản crate bạn có thể xuất
bản.

Chạy lệnh `cargo publish` một lần nữa. Lần này nó sẽ thành công:

<!-- manual-regeneration
go to some valid crate, publish a new version
cargo publish
copy just the relevant lines below
-->

```console
$ cargo publish
    Updating crates.io index
   Packaging guessing_game v0.1.0 (file:///projects/guessing_game)
    Packaged 6 files, 1.2KiB (895.0B compressed)
   Verifying guessing_game v0.1.0 (file:///projects/guessing_game)
   Compiling guessing_game v0.1.0
(file:///projects/guessing_game/target/package/guessing_game-0.1.0)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.19s
   Uploading guessing_game v0.1.0 (file:///projects/guessing_game)
    Uploaded guessing_game v0.1.0 to registry `crates-io`
note: waiting for `guessing_game v0.1.0` to be available at registry
`crates-io`.
You may press ctrl-c to skip waiting; the crate should be available shortly.
   Published guessing_game v0.1.0 at registry `crates-io`
```

Chúc mừng! Bây giờ bạn đã chia sẻ mã của mình với cộng đồng Rust, và bất kỳ ai
cũng có thể dễ dàng thêm crate của bạn làm phụ thuộc của dự án của họ.

### Xuất bản Một Phiên bản Mới của Crate Hiện có

Khi bạn đã thực hiện các thay đổi đối với crate của mình và sẵn sàng phát hành
một phiên bản mới, bạn thay đổi giá trị `version` được chỉ định trong tệp
_Cargo.toml_ của bạn và xuất bản lại. Sử dụng các quy tắc [Semantic
Versioning][semver] để quyết định một số phiên bản tiếp theo phù hợp dựa trên
các loại thay đổi bạn đã thực hiện. Sau đó chạy `cargo publish` để tải lên phiên
bản mới.

<!-- Old link, do not remove -->

<a id="removing-versions-from-cratesio-with-cargo-yank"></a>

### Không dùng nữa Các Phiên bản từ Crates.io với `cargo yank`

Mặc dù bạn không thể xóa các phiên bản trước đó của một crate, bạn có thể ngăn
bất kỳ dự án tương lai nào thêm chúng làm phụ thuộc mới. Điều này hữu ích khi
một phiên bản crate bị lỗi vì lý do nào đó. Trong những tình huống như vậy,
Cargo hỗ trợ _yanking_ (rút lại) một phiên bản crate.

_Yanking_ một phiên bản ngăn các dự án mới phụ thuộc vào phiên bản đó trong khi
cho phép tất cả các dự án hiện có phụ thuộc vào nó tiếp tục. Về cơ bản, rút lại
có nghĩa là tất cả các dự án với _Cargo.lock_ sẽ không bị hỏng, và bất kỳ tệp
_Cargo.lock_ nào được tạo ra trong tương lai sẽ không sử dụng phiên bản đã bị
rút lại.

Để rút lại một phiên bản của crate, trong thư mục của crate mà bạn đã xuất bản
trước đó, chạy `cargo yank` và chỉ định phiên bản bạn muốn rút lại. Ví dụ, nếu
chúng ta đã xuất bản một crate có tên `guessing_game` phiên bản 1.0.1 và chúng
ta muốn rút lại nó, trong thư mục dự án cho `guessing_game` chúng ta sẽ chạy:

<!-- manual-regeneration:
cargo yank carol-test --version 2.1.0
cargo yank carol-test --version 2.1.0 --undo
-->

```console
$ cargo yank --vers 1.0.1
    Updating crates.io index
        Yank guessing_game@1.0.1
```

Bằng cách thêm `--undo` vào lệnh, bạn cũng có thể hoàn tác việc rút lại và cho
phép các dự án bắt đầu phụ thuộc vào một phiên bản một lần nữa:

```console
$ cargo yank --vers 1.0.1 --undo
    Updating crates.io index
      Unyank guessing_game@1.0.1
```

Việc rút lại _không_ xóa bất kỳ mã nào. Ví dụ, nó không thể xóa các bí mật vô
tình được tải lên. Nếu điều đó xảy ra, bạn phải đặt lại các bí mật đó ngay lập
tức.

[spdx]: http://spdx.org/licenses/
[semver]: http://semver.org/
