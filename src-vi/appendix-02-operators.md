## Phụ lục B: Các toán tử và ký hiệu

Phụ lục này chứa danh mục các cú pháp của Rust, bao gồm các toán tử và các ký
hiệu khác xuất hiện độc lập hoặc trong ngữ cảnh của đường dẫn, generics, ràng
buộc trait, macro, thuộc tính, bình luận, tuple và dấu ngoặc.

### Các toán tử

Bảng B-1 chứa các toán tử trong Rust, một ví dụ về cách toán tử xuất hiện trong
ngữ cảnh, một lời giải thích ngắn gọn và liệu toán tử đó có thể được nạp chồng
hay không. Nếu một toán tử có thể được nạp chồng, trait có liên quan được sử
dụng để nạp chồng toán tử đó được liệt kê.

<span class="caption">Bảng B-1: Các toán tử</span>

| Toán tử                   | Ví dụ                                                   | Giải thích                                                          | Nạp chồng?     |
| ------------------------- | ------------------------------------------------------- | ------------------------------------------------------------------- | -------------- |
| `!`                       | `ident!(...)`, `ident!{...}`, `ident![...]`             | Mở rộng macro                                                       |                |
| `!`                       | `!expr`                                                 | Phủ định bit hoặc logic                                             | `Not`          |
| `!=`                      | `expr != expr`                                          | So sánh không bằng                                                  | `PartialEq`    |
| `%`                       | `expr % expr`                                           | Phép chia lấy dư                                                    | `Rem`          |
| `%=`                      | `var %= expr`                                           | Phép chia lấy dư và gán                                             | `RemAssign`    |
| `&`                       | `&expr`, `&mut expr`                                    | Mượn (borrow)                                                       |                |
| `&`                       | `&type`, `&mut type`, `&'a type`, `&'a mut type`        | Kiểu con trỏ mượn                                                   |                |
| `&`                       | `expr & expr`                                           | Phép AND bit                                                        | `BitAnd`       |
| `&=`                      | `var &= expr`                                           | Phép AND bit và gán                                                 | `BitAndAssign` |
| `&&`                      | `expr && expr`                                          | Phép AND logic ngắn mạch                                            |                |
| `*`                       | `expr * expr`                                           | Phép nhân số học                                                    | `Mul`          |
| `*=`                      | `var *= expr`                                           | Phép nhân số học và gán                                             | `MulAssign`    |
| `*`                       | `*expr`                                                 | Dereference (giải tham chiếu)                                       | `Deref`        |
| `*`                       | `*const type`, `*mut type`                              | Con trỏ thô                                                         |                |
| `+`                       | `trait + trait`, `'a + trait`                           | Ràng buộc kiểu phức hợp                                             |                |
| `+`                       | `expr + expr`                                           | Phép cộng số học                                                    | `Add`          |
| `+=`                      | `var += expr`                                           | Phép cộng số học và gán                                             | `AddAssign`    |
| `,`                       | `expr, expr`                                            | Phân cách đối số và phần tử                                         |                |
| `-`                       | `- expr`                                                | Phủ định số học (đổi dấu)                                           | `Neg`          |
| `-`                       | `expr - expr`                                           | Phép trừ số học                                                     | `Sub`          |
| `-=`                      | `var -= expr`                                           | Phép trừ số học và gán                                              | `SubAssign`    |
| `->`                      | `fn(...) -> type`, <code>&vert;...&vert; -> type</code> | Kiểu trả về của hàm và closure                                      |                |
| `.`                       | `expr.ident`                                            | Truy cập trường                                                     |                |
| `.`                       | `expr.ident(expr, ...)`                                 | Gọi phương thức                                                     |                |
| `.`                       | `expr.0`, `expr.1`, v.v.                                | Truy cập phần tử của tuple theo chỉ mục                             |                |
| `..`                      | `..`, `expr..`, `..expr`, `expr..expr`                  | Khoảng (range) loại trừ bên phải                                    | `PartialOrd`   |
| `..=`                     | `..=expr`, `expr..=expr`                                | Khoảng (range) bao gồm bên phải                                     | `PartialOrd`   |
| `..`                      | `..expr`                                                | Cú pháp cập nhật biểu thức struct                                   |                |
| `..`                      | `variant(x, ..)`, `struct_type { x, .. }`               | Gán mẫu "và phần còn lại"                                           |                |
| `...`                     | `expr...expr`                                           | (Đã lỗi thời, sử dụng `..=` thay thế) Trong mẫu: mẫu khoảng bao gồm |                |
| `/`                       | `expr / expr`                                           | Phép chia số học                                                    | `Div`          |
| `/=`                      | `var /= expr`                                           | Phép chia số học và gán                                             | `DivAssign`    |
| `:`                       | `pat: type`, `ident: type`                              | Ràng buộc                                                           |                |
| `:`                       | `ident: expr`                                           | Khởi tạo trường struct                                              |                |
| `:`                       | `'a: loop {...}`                                        | Nhãn vòng lặp                                                       |                |
| `;`                       | `expr;`                                                 | Kết thúc câu lệnh và mục                                            |                |
| `;`                       | `[...; len]`                                            | Một phần của cú pháp mảng kích thước cố định                        |                |
| `<<`                      | `expr << expr`                                          | Dịch trái                                                           | `Shl`          |
| `<<=`                     | `var <<= expr`                                          | Dịch trái và gán                                                    | `ShlAssign`    |
| `<`                       | `expr < expr`                                           | So sánh nhỏ hơn                                                     | `PartialOrd`   |
| `<=`                      | `expr <= expr`                                          | So sánh nhỏ hơn hoặc bằng                                           | `PartialOrd`   |
| `=`                       | `var = expr`, `ident = type`                            | Gán/tương đương                                                     |                |
| `==`                      | `expr == expr`                                          | So sánh bằng                                                        | `PartialEq`    |
| `=>`                      | `pat => expr`                                           | Một phần của cú pháp nhánh match                                    |                |
| `>`                       | `expr > expr`                                           | So sánh lớn hơn                                                     | `PartialOrd`   |
| `>=`                      | `expr >= expr`                                          | So sánh lớn hơn hoặc bằng                                           | `PartialOrd`   |
| `>>`                      | `expr >> expr`                                          | Dịch phải                                                           | `Shr`          |
| `>>=`                     | `var >>= expr`                                          | Dịch phải và gán                                                    | `ShrAssign`    |
| `@`                       | `ident @ pat`                                           | Gán mẫu                                                             |                |
| `^`                       | `expr ^ expr`                                           | Phép XOR bit                                                        | `BitXor`       |
| `^=`                      | `var ^= expr`                                           | Phép XOR bit và gán                                                 | `BitXorAssign` |
| <code>&vert;</code>       | <code>pat &vert; pat</code>                             | Các mẫu thay thế                                                    |                |
| <code>&vert;</code>       | <code>expr &vert; expr</code>                           | Phép OR bit                                                         | `BitOr`        |
| <code>&vert;=</code>      | <code>var &vert;= expr</code>                           | Phép OR bit và gán                                                  | `BitOrAssign`  |
| <code>&vert;&vert;</code> | <code>expr &vert;&vert; expr</code>                     | Phép OR logic ngắn mạch                                             |                |
| `?`                       | `expr?`                                                 | Truyền lỗi                                                          |                |

### Các ký hiệu không phải toán tử

Các bảng sau đây chứa tất cả các ký hiệu không hoạt động như các toán tử; nghĩa
là, chúng không hoạt động như một lời gọi hàm hoặc phương thức.

Bảng B-2 hiển thị các ký hiệu xuất hiện độc lập và hợp lệ trong nhiều vị trí
khác nhau.

<span class="caption">Bảng B-2: Cú pháp độc lập</span>

| Ký hiệu                                                    | Giải thích                                                             |
| ---------------------------------------------------------- | ---------------------------------------------------------------------- |
| `'ident`                                                   | Tên Lifetime hoặc nhãn vòng lặp                                        |
| Chữ số theo ngay sau bởi `u8`, `i32`, `f64`, `usize`, v.v. | Giá trị số có kiểu cụ thể                                              |
| `"..."`                                                    | Chuỗi ký tự                                                            |
| `r"..."`, `r#"..."#`, `r##"..."##`, v.v.                   | Chuỗi ký tự thô; các ký tự thoát không được xử lý                      |
| `b"..."`                                                   | Chuỗi byte; tạo mảng byte thay vì chuỗi                                |
| `br"..."`, `br#"..."#`, `br##"..."##`, v.v.                | Chuỗi byte thô; kết hợp của chuỗi thô và chuỗi byte                    |
| `'...'`                                                    | Ký tự                                                                  |
| `b'...'`                                                   | Byte ASCII                                                             |
| <code>&vert;...&vert; expr</code>                          | Closure                                                                |
| `!`                                                        | Kiểu bottom luôn trống cho các hàm phân kỳ                             |
| `_`                                                        | Gán mẫu "bị bỏ qua"; cũng được sử dụng để làm cho số nguyên dễ đọc hơn |

Bảng B-3 hiển thị các ký hiệu xuất hiện trong ngữ cảnh của đường dẫn thông qua
hệ thống phân cấp module đến một mục.

<span class="caption">Bảng B-3: Cú pháp đường dẫn liên kết</span>

| Ký hiệu                                 | Giải thích                                                                                          |
| --------------------------------------- | --------------------------------------------------------------------------------------------------- |
| `ident::ident`                          | Đường dẫn namespace                                                                                 |
| `::path`                                | Đường dẫn liên kết tới crate gốc (tức là, một đường dẫn tuyệt đối)                                  |
| `self::path`                            | Đường dẫn liên kết tới module hiện tại (tức là, một đường dẫn tương đối).                           |
| `super::path`                           | Đường dẫn liên kết với module cha của module hiện tại                                               |
| `type::ident`, `<type as trait>::ident` | Các hằng số, hàm và kiểu liên kết                                                                   |
| `<type>::...`                           | Mục liên kết cho một kiểu không thể được đặt tên trực tiếp (ví dụ, `<&T>::...`, `<[T]>::...`, v.v.) |
| `trait::method(...)`                    | Làm rõ lời gọi phương thức bằng cách đặt tên trait định nghĩa nó                                    |
| `type::method(...)`                     | Làm rõ lời gọi phương thức bằng cách đặt tên kiểu mà nó được định nghĩa                             |
| `<type as trait>::method(...)`          | Làm rõ lời gọi phương thức bằng cách đặt tên trait và kiểu                                          |

Bảng B-4 hiển thị các ký hiệu xuất hiện trong ngữ cảnh của việc sử dụng các tham
số kiểu generic.

<span class="caption">Bảng B-4: Generics</span>

| Ký hiệu                        | Giải thích                                                                                                                                   |
| ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `path<...>`                    | Chỉ định tham số cho một kiểu generic trong một kiểu (ví dụ, `Vec<u8>`)                                                                      |
| `path::<...>`, `method::<...>` | Chỉ định tham số cho một kiểu generic, hàm hoặc phương thức trong một biểu thức; thường được gọi là turbofish (ví dụ, `"42".parse::<i32>()`) |
| `fn ident<...> ...`            | Định nghĩa hàm generic                                                                                                                       |
| `struct ident<...> ...`        | Định nghĩa struct generic                                                                                                                    |
| `enum ident<...> ...`          | Định nghĩa enum generic                                                                                                                      |
| `impl<...> ...`                | Định nghĩa triển khai generic                                                                                                                |
| `for<...> type`                | Ràng buộc lifetime bậc cao hơn                                                                                                               |
| `type<ident=type>`             | Một kiểu generic trong đó một hoặc nhiều kiểu liên kết có gán cụ thể (ví dụ, `Iterator<Item=T>`)                                             |

Bảng B-5 hiển thị các ký hiệu xuất hiện trong ngữ cảnh của việc ràng buộc các
tham số kiểu generic với các ràng buộc trait.

<span class="caption">Bảng B-5: Ràng buộc Trait</span>

| Ký hiệu                       | Giải thích                                                                                                                         |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| `T: U`                        | Tham số generic `T` bị ràng buộc vào các kiểu triển khai `U`                                                                       |
| `T: 'a`                       | Kiểu generic `T` phải tồn tại lâu hơn lifetime `'a` (nghĩa là kiểu không thể chứa bất kỳ tham chiếu nào có lifetime ngắn hơn `'a`) |
| `T: 'static`                  | Kiểu generic `T` không chứa các tham chiếu được mượn ngoài tham chiếu `'static`                                                    |
| `'b: 'a`                      | Lifetime generic `'b` phải tồn tại lâu hơn lifetime `'a`                                                                           |
| `T: ?Sized`                   | Cho phép tham số kiểu generic là một kiểu có kích thước động                                                                       |
| `'a + trait`, `trait + trait` | Ràng buộc kiểu phức hợp                                                                                                            |

Bảng B-6 hiển thị các ký hiệu xuất hiện trong ngữ cảnh của việc gọi hoặc định
nghĩa macro và chỉ định thuộc tính trên một mục.

<span class="caption">Bảng B-6: Macro và thuộc tính</span>

| Ký hiệu                                     | Giải thích       |
| ------------------------------------------- | ---------------- |
| `#[meta]`                                   | Thuộc tính ngoài |
| `#![meta]`                                  | Thuộc tính trong |
| `$ident`                                    | Thay thế macro   |
| `$ident:kind`                               | Biến siêu macro  |
| `$(...)...`                                 | Lặp lại macro    |
| `ident!(...)`, `ident!{...}`, `ident![...]` | Gọi macro        |

Bảng B-7 hiển thị các ký hiệu tạo bình luận.

<span class="caption">Bảng B-7: Bình luận</span>

| Ký hiệu    | Giải thích                        |
| ---------- | --------------------------------- |
| `//`       | Bình luận dòng                    |
| `//!`      | Bình luận tài liệu dòng bên trong |
| `///`      | Bình luận tài liệu dòng bên ngoài |
| `/*...*/`  | Bình luận khối                    |
| `/*!...*/` | Bình luận tài liệu khối bên trong |
| `/**...*/` | Bình luận tài liệu khối bên ngoài |

Bảng B-8 hiển thị các ngữ cảnh mà dấu ngoặc đơn được sử dụng.

<span class="caption">Bảng B-8: Dấu ngoặc đơn</span>

| Ký hiệu           | Giải thích                                                                               |
| ----------------- | ---------------------------------------------------------------------------------------- |
| `()`              | Tuple rỗng (còn gọi là unit), cả kiểu và giá trị                                         |
| `(expr)`          | Biểu thức trong ngoặc đơn                                                                |
| `(expr,)`         | Biểu thức tuple một phần tử                                                              |
| `(type,)`         | Kiểu tuple một phần tử                                                                   |
| `(expr, ...)`     | Biểu thức tuple                                                                          |
| `(type, ...)`     | Kiểu tuple                                                                               |
| `expr(expr, ...)` | Biểu thức gọi hàm; cũng được sử dụng để khởi tạo `struct` tuple và biến thể `enum` tuple |

Bảng B-9 hiển thị các ngữ cảnh mà dấu ngoặc nhọn được sử dụng.

<span class="caption">Bảng B-9: Dấu ngoặc nhọn</span>

| Ngữ cảnh     | Giải thích     |
| ------------ | -------------- |
| `{...}`      | Biểu thức khối |
| `Type {...}` | Giá trị struct |

Bảng B-10 hiển thị các ngữ cảnh mà dấu ngoặc vuông được sử dụng.

<span class="caption">Bảng B-10: Dấu ngoặc vuông</span>

| Ngữ cảnh                                           | Giải thích                                                                                                                       |
| -------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| `[...]`                                            | Giá trị mảng                                                                                                                     |
| `[expr; len]`                                      | Giá trị mảng chứa `len` bản sao của `expr`                                                                                       |
| `[type; len]`                                      | Kiểu mảng chứa `len` thể hiện của `type`                                                                                         |
| `expr[expr]`                                       | Truy cập phần tử của collection; có thể nạp chồng (`Index`, `IndexMut`)                                                          |
| `expr[..]`, `expr[a..]`, `expr[..b]`, `expr[a..b]` | Truy cập phần tử của collection thành lát cắt collection, sử dụng `Range`, `RangeFrom`, `RangeTo` hoặc `RangeFull` làm "chỉ mục" |
