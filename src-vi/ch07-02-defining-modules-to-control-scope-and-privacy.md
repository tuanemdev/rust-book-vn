## Định nghĩa Modules để Kiểm soát Phạm vi và Quyền riêng tư

Trong phần này, chúng ta sẽ nói về modules và các phần khác của hệ thống module,
cụ thể là _paths_ (đường dẫn), cho phép bạn đặt tên cho các item; từ khóa `use`
đưa một đường dẫn vào phạm vi; và từ khóa `pub` để công khai các item. Chúng ta
cũng sẽ thảo luận về từ khóa `as`, các gói bên ngoài, và toán tử glob.

### Bảng tóm tắt về Modules

Trước khi đi vào chi tiết về modules và đường dẫn, ở đây chúng tôi cung cấp một
tài liệu tham khảo nhanh về cách modules, đường dẫn, từ khóa `use` và từ khóa
`pub` hoạt động trong trình biên dịch, và cách mà hầu hết các nhà phát triển tổ
chức mã của họ. Chúng ta sẽ đi qua các ví dụ cho từng quy tắc này trong suốt
chương này, nhưng đây là một nơi tuyệt vời để tham khảo như một lời nhắc về cách
modules hoạt động.

- **Bắt đầu từ crate root**: Khi biên dịch một crate, trình biên dịch đầu tiên
  tìm kiếm trong tệp crate root (thường là _src/lib.rs_ cho library crate hoặc
  _src/main.rs_ cho binary crate) để biên dịch mã.
- **Khai báo modules**: Trong tệp crate root, bạn có thể khai báo modules mới;
  giả sử bạn khai báo một module "garden" với `mod garden;`. Trình biên dịch sẽ
  tìm kiếm mã của module ở những nơi sau:
  - Trực tiếp, trong dấu ngoặc nhọn thay thế dấu chấm phẩy sau `mod garden`
  - Trong tệp _src/garden.rs_
  - Trong tệp _src/garden/mod.rs_
- **Khai báo submodules**: Trong bất kỳ tệp nào khác ngoài crate root, bạn có
  thể khai báo submodules. Ví dụ, bạn có thể khai báo `mod vegetables;` trong
  _src/garden.rs_. Trình biên dịch sẽ tìm kiếm mã của submodule trong thư mục có
  tên của module cha ở những nơi sau:
  - Trực tiếp, ngay sau `mod vegetables`, trong dấu ngoặc nhọn thay vì dấu chấm
    phẩy
  - Trong tệp _src/garden/vegetables.rs_
  - Trong tệp _src/garden/vegetables/mod.rs_
- **Đường dẫn đến mã trong modules**: Khi một module là một phần của crate của
  bạn, bạn có thể tham chiếu đến mã trong module đó từ bất kỳ nơi nào khác trong
  cùng crate đó, miễn là các quy tắc quyền riêng tư cho phép, sử dụng đường dẫn
  đến mã. Ví dụ, một kiểu `Asparagus` trong module vegetables của garden sẽ được
  tìm thấy tại `crate::garden::vegetables::Asparagus`.
- **Riêng tư vs. công khai**: Mã trong một module mặc định là riêng tư từ các
  module cha của nó. Để làm cho một module công khai, khai báo nó với `pub mod`
  thay vì `mod`. Để làm cho các item trong một module công khai cũng công khai,
  sử dụng `pub` trước khai báo của chúng.
- **Từ khóa `use`**: Trong một phạm vi, từ khóa `use` tạo ra các lối tắt đến các
  item để giảm lặp lại các đường dẫn dài. Trong bất kỳ phạm vi nào có thể tham
  chiếu đến `crate::garden::vegetables::Asparagus`, bạn có thể tạo một lối tắt
  với `use crate::garden::vegetables::Asparagus;` và từ đó trở đi bạn chỉ cần
  viết `Asparagus` để sử dụng kiểu đó trong phạm vi.

Ở đây, chúng ta tạo một binary crate có tên `backyard` minh họa các quy tắc này.
Thư mục của crate, cũng có tên là `backyard`, chứa các tệp và thư mục sau:

```text
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs
```

Tệp crate root trong trường hợp này là _src/main.rs_, và nó chứa:

<Listing file-name="src/main.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/main.rs}}
```

</Listing>

Dòng `pub mod garden;` cho trình biên dịch biết để bao gồm mã mà nó tìm thấy
trong _src/garden.rs_, đó là:

<Listing file-name="src/garden.rs">

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden.rs}}
```

</Listing>

Ở đây, `pub mod vegetables;` có nghĩa là mã trong _src/garden/vegetables.rs_
cũng được bao gồm. Mã đó là:

```rust,noplayground,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/quick-reference-example/src/garden/vegetables.rs}}
```

Bây giờ chúng ta hãy đi vào chi tiết của các quy tắc này và minh họa chúng trong
hành động!

### Nhóm Mã Liên quan trong Modules

_Modules_ cho phép chúng ta tổ chức mã trong một crate để dễ đọc và tái sử dụng.
Modules cũng cho phép chúng ta kiểm soát _quyền riêng tư_ của các item vì mã
trong một module mặc định là riêng tư. Các item riêng tư là chi tiết triển khai
nội bộ không có sẵn cho việc sử dụng bên ngoài. Chúng ta có thể chọn làm cho các
modules và các item bên trong chúng công khai, điều này làm cho chúng hiện ra để
cho phép mã bên ngoài sử dụng và phụ thuộc vào chúng.

Làm ví dụ, chúng ta hãy viết một library crate cung cấp chức năng của một nhà
hàng. Chúng ta sẽ định nghĩa các chữ ký của các hàm nhưng để phần thân của chúng
trống để tập trung vào việc tổ chức mã hơn là triển khai của một nhà hàng.

Trong ngành công nghiệp nhà hàng, một số phần của nhà hàng được gọi là _front of
house_ (tiền sảnh) và những phần khác là _back of house_ (hậu trường). Front of
house là nơi khách hàng đến; điều này bao gồm nơi các chủ nhà đón tiếp khách
hàng, nhân viên phục vụ ghi nhận đơn hàng và thanh toán, và người pha chế pha đồ
uống. Back of house là nơi các đầu bếp và người nấu ăn làm việc trong nhà bếp,
người rửa bát dọn dẹp, và người quản lý làm công việc hành chính.

Để cấu trúc crate của chúng ta theo cách này, chúng ta có thể tổ chức các hàm
của nó thành các modules lồng nhau. Tạo một thư viện mới có tên `restaurant`
bằng cách chạy `cargo new restaurant --lib`. Sau đó nhập mã trong Listing 7-1
vào _src/lib.rs_ để định nghĩa một số modules và chữ ký hàm; mã này là phần
front of house.

<Listing number="7-1" file-name="src/lib.rs" caption="Module `front_of_house` chứa các module khác mà sau đó chứa các hàm">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-01/src/lib.rs}}
```

</Listing>

Chúng ta định nghĩa một module với từ khóa `mod` theo sau là tên của module
(trong trường hợp này, `front_of_house`). Phần thân của module sau đó đi vào
trong dấu ngoặc nhọn. Bên trong modules, chúng ta có thể đặt các module khác,
như trong trường hợp này với các module `hosting` và `serving`. Modules cũng có
thể chứa các định nghĩa cho các item khác, như structs, enums, constants,
traits, và như trong Listing 7-1, các hàm.

Bằng cách sử dụng modules, chúng ta có thể nhóm các định nghĩa liên quan với
nhau và đặt tên tại sao chúng liên quan. Các lập trình viên sử dụng mã này có
thể điều hướng mã dựa trên các nhóm thay vì phải đọc qua tất cả các định nghĩa,
làm cho việc tìm kiếm các định nghĩa liên quan đến họ trở nên dễ dàng hơn. Các
lập trình viên thêm chức năng mới vào mã này sẽ biết nơi để đặt mã để giữ cho
chương trình được tổ chức.

Trước đây, chúng ta đã đề cập rằng _src/main.rs_ và _src/lib.rs_ được gọi là
_crate roots_. Lý do cho tên của chúng là vì nội dung của một trong hai tệp này
tạo thành một module có tên `crate` ở gốc của cấu trúc module của crate, được
gọi là _cây module_.

Listing 7-2 hiển thị cây module cho cấu trúc trong Listing 7-1.

<Listing number="7-2" caption="Cây module cho mã trong Listing 7-1">

```text
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

</Listing>

Cây này cho thấy cách một số modules lồng bên trong các module khác; ví dụ,
`hosting` lồng bên trong `front_of_house`. Cây cũng cho thấy rằng một số modules
là _siblings_ (anh chị em), có nghĩa là chúng được định nghĩa trong cùng một
module; `hosting` và `serving` là anh chị em được định nghĩa trong
`front_of_house`. Nếu module A được chứa bên trong module B, chúng ta nói rằng
module A là _con_ của module B và module B là _cha_ của module A. Lưu ý rằng
toàn bộ cây module được gốc dưới module ẩn có tên `crate`.

Cây module có thể nhắc bạn nhớ đến cây thư mục của hệ thống tệp trên máy tính
của bạn; đây là một so sánh rất đúng! Giống như các thư mục trong hệ thống tệp,
bạn sử dụng modules để tổ chức mã của mình. Và giống như các tệp trong một thư
mục, chúng ta cần một cách để tìm thấy các module của mình.
