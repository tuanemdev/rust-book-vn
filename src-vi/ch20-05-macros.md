## Macro

Chúng ta đã sử dụng các macro như `println!` trong suốt cuốn sách này, nhưng
chúng ta chưa khám phá đầy đủ về macro là gì và cách nó hoạt động. Thuật ngữ
_macro_ đề cập đến một họ các tính năng trong Rust: macro _khai báo_ với
`macro_rules!` và ba loại macro _thủ tục_:

- Macro `#[derive]` tùy chỉnh chỉ định mã được thêm vào với thuộc tính `derive`
  được sử dụng trên các struct và enum
- Macro giống thuộc tính định nghĩa các thuộc tính tùy chỉnh có thể sử dụng trên
  bất kỳ mục nào
- Macro giống hàm trông giống như lời gọi hàm nhưng hoạt động trên các token
  được chỉ định làm đối số của chúng

Chúng ta sẽ nói về từng loại này lần lượt, nhưng trước tiên, hãy xem tại sao
chúng ta thậm chí cần macro khi chúng ta đã có hàm.

### Sự Khác Biệt Giữa Macro và Hàm

Về cơ bản, macro là một cách để viết mã tạo ra mã khác, điều này được gọi là
_lập trình meta_. Trong Phụ lục C, chúng ta thảo luận về thuộc tính `derive`,
tạo ra một triển khai của các trait khác nhau cho bạn. Chúng ta cũng đã sử dụng
các macro `println!` và `vec!` trong suốt cuốn sách. Tất cả các macro này _mở
rộng_ để tạo ra nhiều mã hơn so với mã bạn đã viết thủ công.

Lập trình meta hữu ích để giảm lượng mã bạn phải viết và bảo trì, đây cũng là
một trong những vai trò của hàm. Tuy nhiên, macro có một số sức mạnh bổ sung mà
hàm không có.

Chữ ký hàm phải khai báo số lượng và kiểu của các tham số mà hàm có. Ngược lại,
macro có thể nhận một số lượng tham số thay đổi: chúng ta có thể gọi
`println!("hello")` với một đối số hoặc `println!("hello {}", name)` với hai đối
số. Ngoài ra, macro được mở rộng trước khi trình biên dịch diễn giải ý nghĩa của
mã, vì vậy một macro có thể, ví dụ, triển khai một trait trên một kiểu nhất
định. Một hàm không thể làm điều này, vì nó được gọi trong thời gian chạy và một
trait cần được triển khai tại thời điểm biên dịch.

Nhược điểm của việc triển khai một macro thay vì một hàm là định nghĩa macro
phức tạp hơn định nghĩa hàm vì bạn đang viết mã Rust để viết mã Rust. Do sự gián
tiếp này, định nghĩa macro thường khó đọc, khó hiểu và khó bảo trì hơn định
nghĩa hàm.

Một sự khác biệt quan trọng khác giữa macro và hàm là bạn phải định nghĩa macro
hoặc đưa chúng vào phạm vi _trước_ khi bạn gọi chúng trong một tệp, trái ngược
với các hàm mà bạn có thể định nghĩa ở bất kỳ đâu và gọi ở bất kỳ đâu.

### Macro Khai Báo với `macro_rules!` cho Lập Trình Meta Tổng Quát

Hình thức macro được sử dụng rộng rãi nhất trong Rust là _macro khai báo_. Đôi
khi chúng còn được gọi là "macro bằng ví dụ," "macro `macro_rules!`," hoặc đơn
giản là "macro." Về cơ bản, macro khai báo cho phép bạn viết một cái gì đó tương
tự như một biểu thức `match` của Rust. Như đã thảo luận trong Chương 6, biểu
thức `match` là cấu trúc điều khiển nhận một biểu thức, so sánh giá trị kết quả
của biểu thức với các mẫu, và sau đó chạy mã liên kết với mẫu phù hợp. Macro
cũng so sánh một giá trị với các mẫu được liên kết với mã cụ thể: trong tình
huống này, giá trị là mã nguồn Rust theo nghĩa đen được truyền cho macro; các
mẫu được so sánh với cấu trúc của mã nguồn đó; và mã liên kết với mỗi mẫu, khi
khớp, thay thế mã được truyền cho macro. Tất cả điều này diễn ra trong quá trình
biên dịch.

Để định nghĩa một macro, bạn sử dụng cấu trúc `macro_rules!`. Hãy khám phá cách
sử dụng `macro_rules!` bằng cách xem cách định nghĩa macro `vec!`. Chương 8 đã
đề cập đến cách chúng ta có thể sử dụng macro `vec!` để tạo một vector mới với
các giá trị cụ thể. Ví dụ, macro sau tạo một vector mới chứa ba số nguyên:

```rust
let v: Vec<u32> = vec![1, 2, 3];
```

Chúng ta cũng có thể sử dụng macro `vec!` để tạo một vector của hai số nguyên
hoặc một vector của năm slice chuỗi. Chúng ta sẽ không thể sử dụng một hàm để
làm điều tương tự vì chúng ta sẽ không biết số lượng hoặc kiểu giá trị trước.

Listing 20-35 hiển thị một định nghĩa hơi đơn giản hóa của macro `vec!`.

<Listing number="20-35" file-name="src/lib.rs" caption="Một phiên bản đơn giản hóa của định nghĩa macro `vec!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-35/src/lib.rs}}
```

</Listing>

> Lưu ý: Định nghĩa thực tế của macro `vec!` trong thư viện chuẩn bao gồm mã để
> cấp phát trước đúng lượng bộ nhớ. Mã đó là một tối ưu hóa mà chúng ta không
> bao gồm ở đây, để làm cho ví dụ đơn giản hơn.

Chú thích `#[macro_export]` chỉ ra rằng macro này nên được làm cho có sẵn bất cứ
khi nào crate mà macro được định nghĩa trong đó được đưa vào phạm vi. Nếu không
có chú thích này, macro không thể được đưa vào phạm vi.

Sau đó, chúng ta bắt đầu định nghĩa macro với `macro_rules!` và tên của macro
chúng ta đang định nghĩa _không có_ dấu chấm than. Tên, trong trường hợp này là
`vec`, được theo sau bởi dấu ngoặc nhọn biểu thị thân của định nghĩa macro.

Cấu trúc trong thân `vec!` tương tự như cấu trúc của một biểu thức `match`. Ở
đây, chúng ta có một nhánh với mẫu `( $( $x:expr ),* )`, theo sau là `=>` và
khối mã liên kết với mẫu này. Nếu mẫu khớp, khối mã liên kết sẽ được phát ra. Vì
đây là mẫu duy nhất trong macro này, chỉ có một cách hợp lệ để khớp; bất kỳ mẫu
nào khác sẽ dẫn đến lỗi. Các macro phức tạp hơn sẽ có nhiều hơn một nhánh.

Cú pháp mẫu hợp lệ trong định nghĩa macro khác với cú pháp mẫu được đề cập trong
Chương 19 vì các mẫu macro được khớp với cấu trúc mã Rust chứ không phải giá
trị. Hãy đi qua ý nghĩa của các phần mẫu trong Listing 20-29; để biết cú pháp
mẫu macro đầy đủ, hãy xem [Tài liệu tham khảo Rust][ref].

Đầu tiên, chúng ta sử dụng một bộ dấu ngoặc đơn để bao quanh toàn bộ mẫu. Chúng
ta sử dụng dấu đô la (`$`) để khai báo một biến trong hệ thống macro sẽ chứa mã
Rust khớp với mẫu. Dấu đô la làm rõ đây là một biến macro chứ không phải một
biến Rust thông thường. Tiếp theo là một bộ dấu ngoặc đơn bắt các giá trị khớp
với mẫu trong dấu ngoặc đơn để sử dụng trong mã thay thế. Trong `$()` là
`$x:expr`, khớp với bất kỳ biểu thức Rust nào và gán cho biểu thức tên `$x`.

Dấu phẩy sau `$()` chỉ ra rằng một ký tự phân cách dấu phẩy theo nghĩa đen phải
xuất hiện giữa mỗi thể hiện của mã khớp với mã trong `$()`. Dấu `*` chỉ định
rằng mẫu khớp với zero hoặc nhiều của bất cứ thứ gì đứng trước dấu `*`.

Khi chúng ta gọi macro này với `vec![1, 2, 3];`, mẫu `$x` khớp ba lần với ba
biểu thức `1`, `2`, và `3`.

Bây giờ, hãy nhìn vào mẫu trong phần thân của mã liên kết với nhánh này:
`temp_vec.push()` trong `$()*` được tạo ra cho mỗi phần khớp với `$()` trong mẫu
zero hoặc nhiều lần tùy thuộc vào số lần mẫu khớp. Biến `$x` được thay thế bằng
mỗi biểu thức khớp. Khi chúng ta gọi macro này với `vec![1, 2, 3];`, mã được tạo
ra thay thế lời gọi macro này sẽ là như sau:

```rust,ignore
{
    let mut temp_vec = Vec::new();
    temp_vec.push(1);
    temp_vec.push(2);
    temp_vec.push(3);
    temp_vec
}
```

Chúng ta đã định nghĩa một macro có thể nhận bất kỳ số lượng đối số nào với bất
kỳ kiểu nào và có thể tạo mã để tạo một vector chứa các phần tử được chỉ định.

Để tìm hiểu thêm về cách viết macro, hãy tham khảo tài liệu trực tuyến hoặc các
tài nguyên khác, chẳng hạn như ["The Little Book of Rust Macros"][tlborm] do
Daniel Keep bắt đầu và Lukas Wirth tiếp tục.

### Macro Thủ Tục cho Tạo Mã từ Thuộc Tính

Hình thức thứ hai của macro là macro thủ tục, hoạt động giống như một hàm (và là
một loại thủ tục). _Macro thủ tục_ chấp nhận một số mã làm đầu vào, hoạt động
trên mã đó, và tạo ra một số mã làm đầu ra thay vì khớp với các mẫu và thay thế
mã bằng mã khác như macro khai báo. Ba loại macro thủ tục là `derive` tùy chỉnh,
giống thuộc tính, và giống hàm, và tất cả đều hoạt động theo cách tương tự.

Khi tạo macro thủ tục, các định nghĩa phải nằm trong crate riêng của chúng với
một loại crate đặc biệt. Điều này là do lý do kỹ thuật phức tạp mà chúng tôi hy
vọng sẽ loại bỏ trong tương lai. Trong Listing 20-36, chúng tôi cho thấy cách
định nghĩa một macro thủ tục, trong đó `some_attribute` là một placeholder cho
việc sử dụng một biến thể macro cụ thể.

<Listing number="20-36" file-name="src/lib.rs" caption="Một ví dụ định nghĩa một macro thủ tục">

```rust,ignore
use proc_macro;

#[some_attribute]
pub fn some_name(input: TokenStream) -> TokenStream {
}
```

</Listing>

Hàm định nghĩa một macro thủ tục nhận một `TokenStream` làm đầu vào và tạo ra
một `TokenStream` làm đầu ra. Kiểu `TokenStream` được định nghĩa bởi crate
`proc_macro` được bao gồm với Rust và đại diện cho một chuỗi các token. Đây là
cốt lõi của macro: mã nguồn mà macro đang hoạt động trên tạo thành `TokenStream`
đầu vào, và mã mà macro tạo ra là `TokenStream` đầu ra. Hàm cũng có một thuộc
tính được gắn vào nó chỉ định loại macro thủ tục nào chúng ta đang tạo. Chúng ta
có thể có nhiều loại macro thủ tục trong cùng một crate.

Hãy xem xét các loại macro thủ tục khác nhau. Chúng ta sẽ bắt đầu với một macro
`derive` tùy chỉnh và sau đó giải thích sự khác biệt nhỏ làm cho các hình thức
khác khác nhau.

### Cách Viết Một Macro `derive` Tùy Chỉnh

Hãy tạo một crate có tên `hello_macro` định nghĩa một trait có tên `HelloMacro`
với một hàm liên kết có tên `hello_macro`. Thay vì yêu cầu người dùng của chúng
ta triển khai trait `HelloMacro` cho từng kiểu của họ, chúng ta sẽ cung cấp một
macro thủ tục để người dùng có thể chú thích kiểu của họ với
`#[derive(HelloMacro)]` để có được một triển khai mặc định của hàm
`hello_macro`. Triển khai mặc định sẽ in `Hello, Macro! My name is TypeName!`
trong đó `TypeName` là tên của kiểu mà trait này đã được định nghĩa trên. Nói
cách khác, chúng ta sẽ viết một crate cho phép một lập trình viên khác viết mã
như Listing 20-37 sử dụng crate của chúng ta.

<Listing number="20-37" file-name="src/main.rs" caption="Mã mà người dùng crate của chúng ta sẽ có thể viết khi sử dụng macro thủ tục của chúng ta">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-37/src/main.rs}}
```

</Listing>

Mã này sẽ in `Hello, Macro! My name is Pancakes!` khi chúng ta hoàn thành. Bước
đầu tiên là tạo một crate thư viện mới, như sau:

```console
$ cargo new hello_macro --lib
```

Tiếp theo, trong Listing 20-38, chúng ta sẽ định nghĩa trait `HelloMacro` và hàm
liên kết của nó.

<Listing file-name="src/lib.rs" number="20-38" caption="Một trait đơn giản mà chúng ta sẽ sử dụng với macro `derive`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-38/hello_macro/src/lib.rs}}
```

</Listing>

Chúng ta có một trait và hàm của nó. Tại thời điểm này, người dùng crate của
chúng ta có thể triển khai trait để đạt được chức năng mong muốn, như trong
Listing 20-39.

<Listing number="20-39" file-name="src/main.rs" caption="Nó sẽ trông như thế nào nếu người dùng viết một triển khai thủ công của trait `HelloMacro`">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-39/pancakes/src/main.rs}}
```

</Listing>

Tuy nhiên, họ sẽ cần phải viết khối triển khai cho mỗi kiểu họ muốn sử dụng với
`hello_macro`; chúng ta muốn giúp họ không phải làm công việc này.

Ngoài ra, chúng ta chưa thể cung cấp hàm `hello_macro` với triển khai mặc định
sẽ in tên của kiểu mà trait được triển khai: Rust không có khả năng phản ánh, vì
vậy nó không thể tra cứu tên kiểu tại thời điểm chạy. Chúng ta cần một macro để
tạo mã tại thời điểm biên dịch.

Bước tiếp theo là định nghĩa macro thủ tục. Tại thời điểm viết bài này, macro
thủ tục cần phải nằm trong crate riêng của chúng. Cuối cùng, hạn chế này có thể
được dỡ bỏ. Quy ước cho việc cấu trúc các crate và crate macro là như sau: đối
với một crate có tên `foo`, một crate macro thủ tục `derive` tùy chỉnh được gọi
là `foo_derive`. Hãy bắt đầu một crate mới có tên `hello_macro_derive` bên trong
dự án `hello_macro` của chúng ta:

```console
$ cargo new hello_macro_derive --lib
```

Hai crate của chúng ta có liên quan chặt chẽ với nhau, vì vậy chúng ta tạo crate
macro thủ tục trong thư mục của crate `hello_macro`. Nếu chúng ta thay đổi định
nghĩa trait trong `hello_macro`, chúng ta cũng sẽ phải thay đổi triển khai của
macro thủ tục trong `hello_macro_derive`. Hai crate sẽ cần được xuất bản riêng
biệt, và các lập trình viên sử dụng các crate này sẽ cần thêm cả hai làm phụ
thuộc và đưa cả hai vào phạm vi. Chúng ta có thể thay vào đó cho crate
`hello_macro` sử dụng `hello_macro_derive` làm phụ thuộc và tái xuất mã macro
thủ tục. Tuy nhiên, cách chúng ta đã cấu trúc dự án làm cho các lập trình viên
có thể sử dụng `hello_macro` ngay cả khi họ không muốn chức năng `derive`.

Chúng ta cần khai báo crate `hello_macro_derive` là một crate macro thủ tục.
Chúng ta cũng sẽ cần chức năng từ các crate `syn` và `quote`, như bạn sẽ thấy
sau đây, vì vậy chúng ta cần thêm chúng làm phụ thuộc. Thêm phần sau vào tệp
_Cargo.toml_ cho `hello_macro_derive`:

<Listing file-name="hello_macro_derive/Cargo.toml">

```toml
{{#include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/Cargo.toml:6:12}}
```

</Listing>

Để bắt đầu định nghĩa macro thủ tục, đặt mã trong Listing 20-40 vào tệp
_src/lib.rs_ của bạn cho crate `hello_macro_derive`. Lưu ý rằng mã này sẽ không
biên dịch cho đến khi chúng ta thêm một định nghĩa cho hàm `impl_hello_macro`.

<Listing number="20-40" file-name="hello_macro_derive/src/lib.rs" caption="Mã mà hầu hết các crate macro thủ tục sẽ yêu cầu để xử lý mã Rust">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-40/hello_macro/hello_macro_derive/src/lib.rs}}
```

</Listing>

Lưu ý rằng chúng ta đã chia mã thành hàm `hello_macro_derive`, chịu trách nhiệm
phân tích `TokenStream`, và hàm `impl_hello_macro`, chịu trách nhiệm chuyển đổi
cây cú pháp: điều này làm cho việc viết một macro thủ tục thuận tiện hơn. Mã
trong hàm ngoài (trong trường hợp này là `hello_macro_derive`) sẽ giống nhau cho
hầu hết mọi crate macro thủ tục mà bạn thấy hoặc tạo. Mã mà bạn chỉ định trong
thân hàm bên trong (trong trường hợp này là `impl_hello_macro`) sẽ khác nhau tùy
thuộc vào mục đích của macro thủ tục của bạn.

Chúng ta đã giới thiệu ba crate mới: `proc_macro`, [`syn`][syn]<!--ignore -->,
và [`quote`][quote]<!-- ignore -->. Crate `proc_macro` đi kèm với Rust, vì vậy
chúng ta không cần thêm nó vào phụ thuộc trong _Cargo.toml_. Crate `proc_macro`
là API của trình biên dịch cho phép chúng ta đọc và thao tác mã Rust từ mã của
chúng ta.

Crate `syn` phân tích mã Rust từ một chuỗi thành một cấu trúc dữ liệu mà chúng
ta có thể thực hiện các thao tác. Crate `quote` chuyển đổi các cấu trúc dữ liệu
`syn` trở lại mã Rust. Các crate này làm cho việc phân tích bất kỳ loại mã Rust
nào mà chúng ta có thể muốn xử lý trở nên đơn giản hơn nhiều: viết một trình
phân tích cú pháp đầy đủ cho mã Rust không phải là một nhiệm vụ đơn giản.

Hàm `hello_macro_derive` sẽ được gọi khi người dùng của thư viện của chúng ta
chỉ định `#[derive(HelloMacro)]` trên một kiểu. Điều này có thể vì chúng ta đã
chú thích hàm `hello_macro_derive` ở đây với `proc_macro_derive` và chỉ định tên
`HelloMacro`, khớp với tên trait của chúng ta; đây là quy ước mà hầu hết các
macro thủ tục tuân theo.

Hàm `hello_macro_derive` đầu tiên chuyển đổi `input` từ một `TokenStream` thành
một cấu trúc dữ liệu mà sau đó chúng ta có thể giải thích và thực hiện các thao
tác. Đây là nơi `syn` phát huy tác dụng. Hàm `parse` trong `syn` nhận một
`TokenStream` và trả về một cấu trúc `DeriveInput` đại diện cho mã Rust đã được
phân tích. Listing 20-41 hiển thị các phần liên quan của cấu trúc `DeriveInput`
mà chúng ta nhận được khi phân tích chuỗi `struct Pancakes;`.

<Listing number="20-41" caption="Thể hiện `DeriveInput` mà chúng ta nhận được khi phân tích mã có thuộc tính macro trong Listing 20-37">

```rust,ignore
DeriveInput {
    // --snip--

    ident: Ident {
        ident: "Pancakes",
        span: #0 bytes(95..103)
    },
    data: Struct(
        DataStruct {
            struct_token: Struct,
            fields: Unit,
            semi_token: Some(
                Semi
            )
        }
    )
}
```

</Listing>

Các trường của cấu trúc này cho thấy rằng mã Rust mà chúng ta đã phân tích là
một struct đơn vị với `ident` (_định danh_, nghĩa là tên) là `Pancakes`. Có
nhiều trường hơn trên cấu trúc này để mô tả tất cả các loại mã Rust; kiểm tra
[tài liệu `syn` cho `DeriveInput`][syn-docs] để biết thêm thông tin.

Chẳng bao lâu nữa chúng ta sẽ định nghĩa hàm `impl_hello_macro`, nơi mà chúng ta
sẽ xây dựng mã Rust mới mà chúng ta muốn bao gồm. Nhưng trước khi chúng ta làm
điều đó, hãy lưu ý rằng đầu ra cho macro `derive` của chúng ta cũng là một
`TokenStream`. `TokenStream` trả về được thêm vào mã mà người dùng crate của
chúng ta viết, vì vậy khi họ biên dịch crate của họ, họ sẽ nhận được chức năng
bổ sung mà chúng ta cung cấp trong `TokenStream` đã được sửa đổi.

Bạn có thể đã nhận thấy rằng chúng ta gọi `unwrap` để khiến hàm
`hello_macro_derive` panic nếu lời gọi hàm `syn::parse` không thành công ở đây.
Cần thiết cho macro thủ tục của chúng ta panic về lỗi vì các hàm
`proc_macro_derive` phải trả về `TokenStream` chứ không phải `Result` để phù hợp
với API macro thủ tục. Chúng ta đã đơn giản hóa ví dụ này bằng cách sử dụng
`unwrap`; trong mã sản xuất, bạn nên cung cấp thông báo lỗi cụ thể hơn về những
gì đã xảy ra sai bằng cách sử dụng `panic!` hoặc `expect`.

Bây giờ chúng ta đã có mã để chuyển đổi mã Rust được chú thích từ một
`TokenStream` thành một thể hiện `DeriveInput`, hãy tạo mã triển khai trait
`HelloMacro` trên kiểu được chú thích, như hiển thị trong Listing 20-42.

<Listing number="20-42" file-name="hello_macro_derive/src/lib.rs" caption="Triển khai trait `HelloMacro` sử dụng mã Rust đã phân tích">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-42/hello_macro/hello_macro_derive/src/lib.rs:here}}
```

</Listing>

Chúng ta nhận được một thể hiện của cấu trúc `Ident` chứa tên (định danh) của
kiểu được chú thích bằng cách sử dụng `ast.ident`. Cấu trúc trong Listing 20-41
cho thấy rằng khi chúng ta chạy hàm `impl_hello_macro` trên mã trong Listing
20-37, `ident` mà chúng ta nhận được sẽ có trường `ident` với giá trị là
`"Pancakes"`. Do đó, biến `name` trong Listing 20-42 sẽ chứa một thể hiện của
cấu trúc `Ident` mà, khi được in ra, sẽ là chuỗi `"Pancakes"`, tên của struct
trong Listing 20-37.

Macro `quote!` cho phép chúng ta định nghĩa mã Rust mà chúng ta muốn trả về.
Trình biên dịch mong đợi một thứ khác với kết quả trực tiếp của thực thi macro
`quote!`, vì vậy chúng ta cần chuyển đổi nó thành một `TokenStream`. Chúng ta
làm điều này bằng cách gọi phương thức `into`, tiêu thụ biểu diễn trung gian này
và trả về một giá trị của kiểu `TokenStream` cần thiết.

Macro `quote!` cũng cung cấp một số cơ chế mẫu rất tuyệt: chúng ta có thể nhập
`#name`, và `quote!` sẽ thay thế nó bằng giá trị trong biến `name`. Bạn thậm chí
có thể làm một số lặp lại tương tự như cách hoạt động của các macro thông
thường. Kiểm tra [tài liệu của crate `quote`][quote-docs] để có một giới thiệu
kỹ lưỡng.

Chúng ta muốn macro thủ tục của chúng ta tạo ra một triển khai của trait
`HelloMacro` cho kiểu mà người dùng đã chú thích, đó là những gì chúng ta có thể
nhận được bằng cách sử dụng `#name`. Việc triển khai trait có một hàm
`hello_macro`, thân của nó chứa chức năng mà chúng ta muốn cung cấp: in
`Hello, Macro! My name is` và sau đó là tên của kiểu được chú thích.

Macro `stringify!` được sử dụng ở đây là được tích hợp sẵn trong Rust. Nó lấy
một biểu thức Rust, chẳng hạn như `1 + 2`, và vào thời điểm biên dịch chuyển đổi
biểu thức thành một chuỗi theo nghĩa đen, chẳng hạn như `"1 + 2"`. Điều này khác
với `format!` hoặc `println!`, các macro đánh giá biểu thức và sau đó chuyển đổi
kết quả thành một `String`. Có khả năng là đầu vào `#name` có thể là một biểu
thức để in ra theo nghĩa đen, vì vậy chúng ta sử dụng `stringify!`. Sử dụng
`stringify!` cũng tiết kiệm một phân bổ bằng cách chuyển đổi `#name` thành một
chuỗi theo nghĩa đen vào thời điểm biên dịch.

Tại thời điểm này, `cargo build` nên hoàn thành thành công trong cả
`hello_macro` và `hello_macro_derive`. Hãy kết nối các crate này với mã trong
Listing 20-31 để xem macro thủ tục hoạt động! Tạo một dự án nhị phân mới trong
thư mục _projects_ của bạn bằng cách sử dụng `cargo new pancakes`. Chúng ta cần
thêm `hello_macro` và `hello_macro_derive` làm phụ thuộc trong crate `pancakes`
của _Cargo.toml_. Nếu bạn đang xuất bản phiên bản của `hello_macro` và
`hello_macro_derive` lên [crates.io](https://crates.io/), chúng sẽ là phụ thuộc
thông thường; nếu không, bạn có thể chỉ định chúng như phụ thuộc `path` như sau:

```toml
{{#include ../listings/ch20-advanced-features/no-listing-21-pancakes/pancakes/Cargo.toml:6:8}}
```

Đặt mã trong Listing 20-37 vào _src/main.rs_, và chạy `cargo run`: nó sẽ in
`Hello, Macro! My name is Pancakes!` Việc triển khai trait `HelloMacro` từ macro
thủ tục đã được bao gồm mà không cần crate `pancakes` phải triển khai nó;
`#[derive(HelloMacro)]` đã thêm việc triển khai trait.

Tiếp theo, hãy khám phá cách các loại macro thủ tục khác khác với macro `derive`
tùy chỉnh.

### Macro Giống Thuộc Tính

Macro giống thuộc tính tương tự như macro `derive` tùy chỉnh, nhưng thay vì tạo
mã cho thuộc tính `derive`, chúng cho phép bạn tạo thuộc tính mới. Chúng cũng
linh hoạt hơn: `derive` chỉ hoạt động cho struct và enum; thuộc tính có thể được
áp dụng cho các mục khác, chẳng hạn như hàm. Đây là một ví dụ về việc sử dụng
một macro giống thuộc tính. Giả sử bạn có một thuộc tính tên là `route` chú
thích các hàm khi sử dụng một framework ứng dụng web:

```rust,ignore
#[route(GET, "/")]
fn index() {
```

Thuộc tính `#[route]` này sẽ được định nghĩa bởi framework như một macro thủ
tục. Chữ ký của hàm định nghĩa macro sẽ trông như thế này:

```rust,ignore
#[proc_macro_attribute]
pub fn route(attr: TokenStream, item: TokenStream) -> TokenStream {
```

Tại đây, chúng ta có hai tham số kiểu `TokenStream`. Tham số đầu tiên là cho nội
dung của thuộc tính: phần `GET, "/"`. Tham số thứ hai là cho phần thân của mục
mà thuộc tính được gắn vào: trong trường hợp này, `fn index() {}` và phần còn
lại của thân hàm.

Ngoài ra, macro giống thuộc tính hoạt động theo cách tương tự như macro `derive`
tùy chỉnh: bạn tạo một crate với loại crate `proc-macro` và triển khai một hàm
tạo ra mã bạn muốn!

### Macro Giống Hàm

Macro giống hàm định nghĩa các macro trông giống như lời gọi hàm. Tương tự như
macro `macro_rules!`, chúng linh hoạt hơn các hàm; ví dụ, chúng có thể nhận một
số lượng đối số không xác định. Tuy nhiên, macro `macro_rules!` chỉ có thể được
định nghĩa bằng cách sử dụng cú pháp giống match mà chúng ta đã thảo luận trong
["Macro Khai Báo với `macro_rules!` cho Lập Trình Meta Tổng
Quát"][decl]<!-- ignore --> trước đó. Macro giống hàm nhận một tham số
`TokenStream`, và định nghĩa của chúng thao tác trên `TokenStream` đó bằng mã
Rust giống như hai loại macro thủ tục khác. Một ví dụ về một macro giống hàm là
macro `sql!` có thể được gọi như sau:

```rust,ignore
let sql = sql!(SELECT * FROM posts WHERE id=1);
```

Macro này sẽ phân tích câu lệnh SQL bên trong nó và kiểm tra xem nó có đúng cú
pháp hay không, đây là quá trình xử lý phức tạp hơn nhiều so với những gì một
macro `macro_rules!` có thể làm. Macro `sql!` sẽ được định nghĩa như sau:

```rust,ignore
#[proc_macro]
pub fn sql(input: TokenStream) -> TokenStream {
```

Định nghĩa này tương tự như chữ ký của macro `derive` tùy chỉnh: chúng ta nhận
các token nằm trong dấu ngoặc đơn và trả về mã mà chúng ta muốn tạo ra.

## Tóm Tắt

Whew! Bây giờ bạn đã có một số tính năng Rust trong hộp công cụ của mình mà bạn
có thể sẽ không sử dụng thường xuyên, nhưng bạn sẽ biết chúng có sẵn trong những
tình huống rất cụ thể. Chúng ta đã giới thiệu một số chủ đề phức tạp để khi bạn
gặp chúng trong các gợi ý thông báo lỗi hoặc trong mã của người khác, bạn sẽ có
thể nhận ra các khái niệm và cú pháp này. Sử dụng chương này như một tài liệu
tham khảo để hướng dẫn bạn đến các giải pháp.

Tiếp theo, chúng ta sẽ đưa tất cả những gì chúng ta đã thảo luận trong suốt cuốn
sách vào thực hành và thực hiện một dự án nữa!

[ref]: ../reference/macros-by-example.html
[tlborm]: https://veykril.github.io/tlborm/
[syn]: https://crates.io/crates/syn
[quote]: https://crates.io/crates/quote
[syn-docs]: https://docs.rs/syn/2.0/syn/struct.DeriveInput.html
[quote-docs]: https://docs.rs/quote
[decl]: #declarative-macros-with-macro_rules-for-general-metaprogramming
