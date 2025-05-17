## Tổ Chức Kiểm Thử

Như đã đề cập ở đầu chương, kiểm thử là một lĩnh vực phức tạp, và nhiều người sử
dụng thuật ngữ và tổ chức khác nhau. Cộng đồng Rust nghĩ về kiểm thử theo hai
loại chính: kiểm thử đơn vị và kiểm thử tích hợp. _Kiểm thử đơn vị_ nhỏ và tập
trung hơn, kiểm thử một module riêng biệt tại một thời điểm, và có thể kiểm thử
các giao diện riêng tư. _Kiểm thử tích hợp_ hoàn toàn bên ngoài thư viện của bạn
và sử dụng code của bạn theo cách mà bất kỳ mã bên ngoài nào khác sẽ sử dụng,
chỉ sử dụng giao diện công cộng và có thể thực hiện nhiều module trong mỗi bài
kiểm thử.

Viết cả hai loại kiểm thử là quan trọng để đảm bảo rằng các thành phần của thư
viện của bạn đang hoạt động như bạn mong đợi, cả riêng biệt và cùng nhau.

### Kiểm Thử Đơn Vị

Mục đích của kiểm thử đơn vị là kiểm thử từng đơn vị code riêng biệt với phần
còn lại của code để nhanh chóng xác định nơi code đang hoạt động và không hoạt
động như mong đợi. Bạn sẽ đặt các kiểm thử đơn vị trong thư mục _src_ trong mỗi
tệp với code mà chúng đang kiểm thử. Quy ước là tạo một module có tên `tests`
trong mỗi tệp để chứa các hàm kiểm thử và chú thích module bằng `cfg(test)`.

#### Module Tests và `#[cfg(test)]`

Chú thích `#[cfg(test)]` trên module `tests` nói với Rust biên dịch và chạy code
kiểm thử chỉ khi bạn chạy `cargo test`, không phải khi bạn chạy `cargo build`.
Điều này tiết kiệm thời gian biên dịch khi bạn chỉ muốn xây dựng thư viện và
tiết kiệm không gian trong kết quả biên dịch vì các kiểm thử không được bao gồm.
Bạn sẽ thấy rằng vì kiểm thử tích hợp nằm trong một thư mục khác, chúng không
cần chú thích `#[cfg(test)]`. Tuy nhiên, vì kiểm thử đơn vị nằm trong cùng tệp
với mã, bạn sẽ sử dụng `#[cfg(test)]` để chỉ định rằng chúng không nên được bao
gồm trong kết quả biên dịch.

Nhớ lại rằng khi chúng ta tạo dự án mới `adder` trong phần đầu tiên của chương
này, Cargo đã tạo mã này cho chúng ta:

<span class="filename">Tên tệp: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

Trên module `tests` được tự động tạo ra, thuộc tính `cfg` là viết tắt của _cấu
hình_ và nói với Rust rằng mục tiếp theo chỉ nên được bao gồm khi có một tùy
chọn cấu hình nhất định. Trong trường hợp này, tùy chọn cấu hình là `test`, được
Rust cung cấp để biên dịch và chạy các bài kiểm thử. Bằng cách sử dụng thuộc
tính `cfg`, Cargo chỉ biên dịch code kiểm thử của chúng ta nếu chúng ta chủ động
chạy các bài kiểm thử với `cargo test`. Điều này bao gồm bất kỳ hàm trợ giúp nào
có thể nằm trong module này, ngoài các hàm được chú thích với `#[test]`.

#### Kiểm Thử Các Hàm Riêng Tư

Có tranh luận trong cộng đồng kiểm thử về việc liệu các hàm riêng tư có nên được
kiểm thử trực tiếp hay không, và các ngôn ngữ khác làm cho việc kiểm thử các hàm
riêng tư trở nên khó khăn hoặc không thể. Bất kể bạn tuân theo ý thức hệ kiểm
thử nào, các quy tắc riêng tư của Rust cho phép bạn kiểm thử các hàm riêng tư.
Hãy xem xét mã trong Listing 11-12 với hàm riêng tư `internal_adder`.

<Listing number="11-12" file-name="src/lib.rs" caption="Kiểm thử một hàm riêng tư">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-12/src/lib.rs}}
```

</Listing>

Lưu ý rằng hàm `internal_adder` không được đánh dấu là `pub`. Các bài kiểm thử
chỉ là mã Rust, và module `tests` chỉ là một module khác. Như chúng ta đã thảo
luận trong ["Đường Dẫn cho Việc Tham Chiếu đến Một Mục trong Cây
Module"][paths]<!-- ignore -->, các mục trong module con có thể sử dụng các mục
trong module tổ tiên của chúng. Trong bài kiểm thử này, chúng ta đưa tất cả các
mục của module cha của module `tests` vào phạm vi với `use super::*`, và sau đó
bài kiểm thử có thể gọi `internal_adder`. Nếu bạn không nghĩ rằng các hàm riêng
tư nên được kiểm thử, không có gì trong Rust buộc bạn phải làm như vậy.

### Kiểm Thử Tích Hợp

Trong Rust, kiểm thử tích hợp hoàn toàn bên ngoài thư viện của bạn. Chúng sử
dụng thư viện của bạn theo cách mà bất kỳ mã nào khác sẽ sử dụng, có nghĩa là
chúng chỉ có thể gọi các hàm là một phần của API công khai của thư viện của bạn.
Mục đích của chúng là kiểm tra liệu nhiều phần của thư viện của bạn có hoạt động
cùng nhau một cách chính xác hay không. Các đơn vị mã hoạt động chính xác độc
lập có thể gặp vấn đề khi được tích hợp, vì vậy việc kiểm thử bao phủ mã tích
hợp cũng rất quan trọng. Để tạo kiểm thử tích hợp, trước tiên bạn cần một thư
mục _tests_.

#### Thư Mục _tests_

Chúng ta tạo một thư mục _tests_ ở cấp trên cùng của thư mục dự án, bên cạnh
_src_. Cargo biết cách tìm các tệp kiểm thử tích hợp trong thư mục này. Sau đó,
chúng ta có thể tạo nhiều tệp kiểm thử tùy thích, và Cargo sẽ biên dịch mỗi tệp
dưới dạng một crate riêng biệt.

Hãy tạo một kiểm thử tích hợp. Với mã trong Listing 11-12 vẫn nằm trong tệp
_src/lib.rs_, hãy tạo một thư mục _tests_, và tạo một tệp mới có tên
_tests/integration_test.rs_. Cấu trúc thư mục của bạn nên trông như thế này:

```text
adder
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    └── integration_test.rs
```

Nhập mã trong Listing 11-13 vào tệp _tests/integration_test.rs_.

<Listing number="11-13" file-name="tests/integration_test.rs" caption="Một kiểm thử tích hợp của một hàm trong crate `adder`">

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-13/tests/integration_test.rs}}
```

</Listing>

Mỗi tệp trong thư mục _tests_ là một crate riêng biệt, vì vậy chúng ta cần đưa
thư viện của mình vào phạm vi của mỗi crate kiểm thử. Vì lý do đó, chúng ta thêm
`use adder::add_two;` ở đầu mã, điều mà chúng ta không cần trong các kiểm thử
đơn vị.

Chúng ta không cần chú thích bất kỳ mã nào trong _tests/integration_test.rs_ với
`#[cfg(test)]`. Cargo xử lý thư mục _tests_ đặc biệt và chỉ biên dịch các tệp
trong thư mục này khi chúng ta chạy `cargo test`. Chạy `cargo test` ngay bây
giờ:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-13/output.txt}}
```

Ba phần của kết quả bao gồm các kiểm thử đơn vị, kiểm thử tích hợp và kiểm thử
tài liệu. Lưu ý rằng nếu bất kỳ bài kiểm thử nào trong một phần không thành
công, các phần tiếp theo sẽ không được chạy. Ví dụ, nếu một kiểm thử đơn vị
không thành công, sẽ không có kết quả nào cho kiểm thử tích hợp và kiểm thử tài
liệu vì những kiểm thử đó chỉ được chạy nếu tất cả các kiểm thử đơn vị đều thành
công.

Phần đầu tiên cho các kiểm thử đơn vị giống như những gì chúng ta đã thấy: một
dòng cho mỗi kiểm thử đơn vị (một có tên `internal` mà chúng ta đã thêm trong
Listing 11-12) và sau đó là một dòng tóm tắt cho các kiểm thử đơn vị.

Phần kiểm thử tích hợp bắt đầu bằng dòng `Running tests/integration_test.rs`.
Tiếp theo, có một dòng cho mỗi hàm kiểm thử trong kiểm thử tích hợp đó và một
dòng tóm tắt cho kết quả của kiểm thử tích hợp ngay trước khi phần
`Doc-tests adder` bắt đầu.

Mỗi tệp kiểm thử tích hợp có phần riêng của nó, vì vậy nếu chúng ta thêm nhiều
tệp hơn trong thư mục _tests_, sẽ có nhiều phần kiểm thử tích hợp hơn.

Chúng ta vẫn có thể chạy một hàm kiểm thử tích hợp cụ thể bằng cách chỉ định tên
của hàm kiểm thử làm đối số cho `cargo test`. Để chạy tất cả các kiểm thử trong
một tệp kiểm thử tích hợp cụ thể, hãy sử dụng đối số `--test` của `cargo test`
theo sau là tên của tệp:

```console
{{#include ../listings/ch11-writing-automated-tests/output-only-05-single-integration/output.txt}}
```

Lệnh này chỉ chạy các kiểm thử trong tệp _tests/integration_test.rs_.

#### Các Module Con trong Kiểm Thử Tích Hợp

Khi bạn thêm nhiều kiểm thử tích hợp, bạn có thể muốn tạo nhiều tệp trong thư
mục _tests_ để giúp tổ chức chúng; ví dụ, bạn có thể nhóm các hàm kiểm thử theo
chức năng mà chúng đang kiểm thử. Như đã đề cập trước đây, mỗi tệp trong thư mục
_tests_ được biên dịch như là crate riêng biệt của nó, điều này hữu ích để tạo
các phạm vi riêng biệt để bắt chước chặt chẽ hơn cách người dùng cuối sẽ sử dụng
crate của bạn. Tuy nhiên, điều này có nghĩa là các tệp trong thư mục _tests_
không chia sẻ cùng một hành vi như các tệp trong _src_, như bạn đã học trong
Chương 7 về cách tách mã thành các module và tệp.

Hành vi khác nhau của các tệp thư mục _tests_ là rõ ràng nhất khi bạn có một tập
hợp các hàm trợ giúp để sử dụng trong nhiều tệp kiểm thử tích hợp và bạn cố gắng
làm theo các bước trong phần ["Tách Module thành Các Tệp Khác
Nhau"][separating-modules-into-files]<!-- ignore --> của Chương 7 để trích xuất
chúng vào một module chung. Ví dụ, nếu chúng ta tạo _tests/common.rs_ và đặt một
hàm có tên `setup` trong đó, chúng ta có thể thêm một số mã vào `setup` mà chúng
ta muốn gọi từ nhiều hàm kiểm thử trong nhiều tệp kiểm thử:

<span class="filename">Tên tệp: tests/common.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/tests/common.rs}}
```

Khi chúng ta chạy các bài kiểm thử một lần nữa, chúng ta sẽ thấy một phần mới
trong kết quả kiểm thử cho tệp _common.rs_, ngay cả khi tệp này không chứa bất
kỳ hàm kiểm thử nào và chúng ta cũng không gọi hàm `setup` từ bất cứ đâu:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-12-shared-test-code-problem/output.txt}}
```

Việc `common` xuất hiện trong kết quả kiểm thử với `running 0 tests` hiển thị
cho nó không phải là điều chúng ta muốn. Chúng ta chỉ muốn chia sẻ một số mã với
các tệp kiểm thử tích hợp khác. Để tránh làm cho `common` xuất hiện trong kết
quả kiểm thử, thay vì tạo _tests/common.rs_, chúng ta sẽ tạo
_tests/common/mod.rs_. Thư mục dự án bây giờ trông như thế này:

```text
├── Cargo.lock
├── Cargo.toml
├── src
│   └── lib.rs
└── tests
    ├── common
    │   └── mod.rs
    └── integration_test.rs
```

Đây là quy ước đặt tên cũ mà Rust cũng hiểu mà chúng ta đã đề cập trong ["Đường
Dẫn Tệp Thay Thế"][alt-paths]<!-- ignore --> ở Chương 7. Đặt tên tệp theo cách
này nói với Rust không xử lý module `common` như một tệp kiểm thử tích hợp. Khi
chúng ta di chuyển mã hàm `setup` vào _tests/common/mod.rs_ và xóa tệp
_tests/common.rs_, phần trong kết quả kiểm thử sẽ không còn xuất hiện nữa. Các
tệp trong thư mục con của thư mục _tests_ không được biên dịch như các crate
riêng biệt hoặc có phần trong kết quả kiểm thử.

Sau khi chúng ta đã tạo _tests/common/mod.rs_, chúng ta có thể sử dụng nó từ bất
kỳ tệp kiểm thử tích hợp nào như một module. Đây là một ví dụ về việc gọi hàm
`setup` từ kiểm thử `it_adds_two` trong _tests/integration_test.rs_:

<span class="filename">Tên tệp: tests/integration_test.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-13-fix-shared-test-code-problem/tests/integration_test.rs}}
```

Lưu ý rằng khai báo `mod common;` giống như khai báo module mà chúng ta đã minh
họa trong Listing 7-21. Sau đó, trong hàm kiểm thử, chúng ta có thể gọi hàm
`common::setup()`.

#### Kiểm Thử Tích Hợp cho Các Crate Nhị Phân

Nếu dự án của chúng ta là một crate nhị phân chỉ chứa một tệp _src/main.rs_ và
không có tệp _src/lib.rs_, chúng ta không thể tạo kiểm thử tích hợp trong thư
mục _tests_ và đưa các hàm được định nghĩa trong tệp _src/main.rs_ vào phạm vi
với một câu lệnh `use`. Chỉ có các crate thư viện mới để lộ các hàm mà các crate
khác có thể sử dụng; các crate nhị phân được thiết kế để chạy độc lập.

Đây là một trong những lý do các dự án Rust cung cấp một tệp nhị phân có một tệp
_src/main.rs_ đơn giản gọi logic nằm trong tệp _src/lib.rs_. Sử dụng cấu trúc
đó, kiểm thử tích hợp _có thể_ kiểm thử crate thư viện với `use` để làm cho chức
năng quan trọng có sẵn. Nếu chức năng quan trọng hoạt động, lượng mã nhỏ trong
tệp _src/main.rs_ cũng sẽ hoạt động, và lượng mã nhỏ đó không cần phải được kiểm
thử.

## Tóm Tắt

Các tính năng kiểm thử của Rust cung cấp một cách để chỉ định cách mã nên hoạt
động để đảm bảo nó tiếp tục hoạt động như bạn mong đợi, ngay cả khi bạn thực
hiện các thay đổi. Kiểm thử đơn vị kiểm tra các phần khác nhau của thư viện một
cách riêng biệt và có thể kiểm tra các chi tiết triển khai riêng tư. Kiểm thử
tích hợp kiểm tra xem nhiều phần của thư viện có hoạt động cùng nhau một cách
chính xác hay không, và chúng sử dụng API công khai của thư viện để kiểm tra mã
theo cùng cách mà mã bên ngoài sẽ sử dụng nó. Mặc dù hệ thống kiểu và quy tắc sở
hữu của Rust giúp ngăn chặn một số loại lỗi, các kiểm thử vẫn quan trọng để giảm
các lỗi logic liên quan đến cách mã của bạn được mong đợi sẽ hoạt động.

Hãy kết hợp kiến thức bạn đã học trong chương này và trong các chương trước để
làm việc trên một dự án!

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[separating-modules-into-files]:
  ch07-05-separating-modules-into-different-files.html
[alt-paths]:
  ch07-05-separating-modules-into-different-files.html#alternate-file-paths
