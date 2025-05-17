## Tách Module Thành Nhiều File Khác Nhau

Cho đến nay, tất cả các ví dụ trong chương này đều định nghĩa nhiều module trong
một file. Khi các module trở nên lớn, bạn có thể muốn di chuyển các định nghĩa
của chúng vào một file riêng biệt để làm cho mã dễ điều hướng hơn.

Ví dụ, hãy bắt đầu từ mã trong Listing 7-17 với nhiều module nhà hàng. Chúng ta
sẽ trích xuất module vào các file thay vì định nghĩa tất cả module trong file
gốc của crate. Trong trường hợp này, file gốc của crate là _src/lib.rs_, nhưng
quy trình này cũng hoạt động với các crate nhị phân mà file gốc của crate là
_src/main.rs_.

Đầu tiên, chúng ta sẽ trích xuất module `front_of_house` vào file riêng của nó.
Xóa mã trong dấu ngoặc nhọn của module `front_of_house`, chỉ để lại khai báo
`mod front_of_house;`, để _src/lib.rs_ chứa mã hiển thị trong Listing 7-21. Lưu
ý rằng mã này sẽ không biên dịch được cho đến khi chúng ta tạo file
_src/front_of_house.rs_ trong Listing 7-22.

<Listing number="7-21" file-name="src/lib.rs" caption="Khai báo module `front_of_house` mà nội dung sẽ nằm trong *src/front_of_house.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/lib.rs}}
```

</Listing>

Tiếp theo, đặt mã đã nằm trong dấu ngoặc nhọn vào một file mới có tên
_src/front_of_house.rs_, như trong Listing 7-22. Trình biên dịch biết phải tìm
file này vì nó đã gặp khai báo module trong gốc của crate với tên
`front_of_house`.

<Listing number="7-22" file-name="src/front_of_house.rs" caption="Các định nghĩa bên trong module `front_of_house` trong *src/front_of_house.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-21-and-22/src/front_of_house.rs}}
```

</Listing>

Lưu ý rằng bạn chỉ cần tải một file bằng khai báo `mod` _một lần_ trong cây
module của bạn. Khi trình biên dịch biết file đó là một phần của dự án (và biết
vị trí của mã trong cây module dựa vào nơi bạn đặt câu lệnh `mod`), các file
khác trong dự án của bạn nên tham chiếu đến mã của file đã được tải bằng đường
dẫn đến nơi nó được khai báo, như đã đề cập trong phần ["Đường dẫn để tham chiếu
đến một mục trong cây module"][paths]<!-- ignore -->. Nói cách khác, `mod`
_không phải_ là một thao tác "include" mà bạn có thể đã thấy trong các ngôn ngữ
lập trình khác.

Tiếp theo, chúng ta sẽ trích xuất module `hosting` vào file riêng của nó. Quy
trình này hơi khác vì `hosting` là một module con của `front_of_house`, không
phải của module gốc. Chúng ta sẽ đặt file cho `hosting` trong một thư mục mới
được đặt tên theo tổ tiên của nó trong cây module, trong trường hợp này là
_src/front_of_house_.

Để bắt đầu di chuyển `hosting`, chúng ta thay đổi _src/front_of_house.rs_ để chỉ
chứa khai báo của module `hosting`:

<Listing file-name="src/front_of_house.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house.rs}}
```

</Listing>

Sau đó chúng ta tạo một thư mục _src/front_of_house_ và một file _hosting.rs_ để
chứa các định nghĩa được tạo trong module `hosting`:

<Listing file-name="src/front_of_house/hosting.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-02-extracting-hosting/src/front_of_house/hosting.rs}}
```

</Listing>

Nếu thay vào đó chúng ta đặt _hosting.rs_ trong thư mục _src_, trình biên dịch
sẽ mong đợi mã _hosting.rs_ nằm trong một module `hosting` được khai báo trong
gốc của crate, chứ không phải được khai báo là con của module `front_of_house`.
Các quy tắc của trình biên dịch về việc kiểm tra file nào cho mã của module nào
có nghĩa là các thư mục và file khớp chặt chẽ hơn với cây module.

> ### Đường dẫn file thay thế
>
> Cho đến nay chúng ta đã đề cập đến các đường dẫn file kiểu mẫu nhất mà trình
> biên dịch Rust sử dụng, nhưng Rust cũng hỗ trợ một kiểu đường dẫn file cũ hơn.
> Đối với module có tên `front_of_house` được khai báo trong gốc của crate,
> trình biên dịch sẽ tìm kiếm mã của module trong:
>
> - _src/front_of_house.rs_ (những gì chúng ta đã đề cập)
> - _src/front_of_house/mod.rs_ (kiểu cũ hơn, vẫn được hỗ trợ)
>
> Đối với module có tên `hosting` là một module con của `front_of_house`, trình
> biên dịch sẽ tìm kiếm mã của module trong:
>
> - _src/front_of_house/hosting.rs_ (những gì chúng ta đã đề cập)
> - _src/front_of_house/hosting/mod.rs_ (kiểu cũ hơn, vẫn được hỗ trợ)
>
> Nếu bạn sử dụng cả hai kiểu cho cùng một module, bạn sẽ gặp lỗi biên dịch. Sử
> dụng kết hợp cả hai kiểu cho các module khác nhau trong cùng một dự án là được
> phép, nhưng có thể gây nhầm lẫn cho những người điều hướng dự án của bạn.
>
> Nhược điểm chính của kiểu sử dụng các file có tên _mod.rs_ là dự án của bạn có
> thể kết thúc với nhiều file có tên _mod.rs_, điều này có thể gây nhầm lẫn khi
> bạn mở chúng cùng lúc trong trình soạn thảo.

Chúng ta đã di chuyển mã của mỗi module vào một file riêng biệt, và cây module
vẫn giữ nguyên. Các lệnh gọi hàm trong `eat_at_restaurant` sẽ hoạt động mà không
cần bất kỳ sự sửa đổi nào, mặc dù các định nghĩa nằm trong các file khác nhau.
Kỹ thuật này cho phép bạn di chuyển các module vào các file mới khi chúng tăng
kích thước.

Lưu ý rằng câu lệnh `pub use crate::front_of_house::hosting` trong _src/lib.rs_
cũng không thay đổi, và `use` cũng không có bất kỳ tác động nào đến việc những
file nào được biên dịch như một phần của crate. Từ khóa `mod` khai báo các
module, và Rust tìm kiếm trong một file có cùng tên với module để lấy mã đi vào
module đó.

## Tóm tắt

Rust cho phép bạn chia một gói thành nhiều crate và một crate thành nhiều module
để bạn có thể tham chiếu đến các mục được định nghĩa trong một module từ một
module khác. Bạn có thể làm điều này bằng cách chỉ định đường dẫn tuyệt đối hoặc
đường dẫn tương đối. Các đường dẫn này có thể được đưa vào phạm vi với một câu
lệnh `use` để bạn có thể sử dụng một đường dẫn ngắn hơn cho nhiều lần sử dụng
mục đó trong phạm vi đó. Mã module là riêng tư theo mặc định, nhưng bạn có thể
công khai các định nghĩa bằng cách thêm từ khóa `pub`.

Trong chương tiếp theo, chúng ta sẽ xem xét một số cấu trúc dữ liệu tập hợp
trong thư viện chuẩn mà bạn có thể sử dụng trong mã được tổ chức gọn gàng của
mình.

[paths]: ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
