## Đường dẫn để Tham chiếu đến một Item trong Cây Module

Để chỉ cho Rust vị trí một item trong cây module, chúng ta sử dụng một đường dẫn
tương tự như cách chúng ta sử dụng đường dẫn khi điều hướng trong hệ thống tệp.
Để gọi một hàm, chúng ta cần biết đường dẫn của nó.

Một đường dẫn có thể có hai hình thức:

- Một _đường dẫn tuyệt đối_ là đường dẫn đầy đủ bắt đầu từ gốc của crate; đối
  với mã từ một crate bên ngoài, đường dẫn tuyệt đối bắt đầu bằng tên crate, và
  đối với mã từ crate hiện tại, nó bắt đầu bằng từ khóa `crate`.
- Một _đường dẫn tương đối_ bắt đầu từ module hiện tại và sử dụng `self`,
  `super`, hoặc một định danh trong module hiện tại.

Cả đường dẫn tuyệt đối và tương đối đều được theo sau bởi một hoặc nhiều định
danh được phân tách bởi dấu hai chấm kép (`::`).

Quay lại Listing 7-1, giả sử chúng ta muốn gọi hàm `add_to_waitlist`. Điều này
cũng giống như việc hỏi: đâu là đường dẫn của hàm `add_to_waitlist`? Listing 7-3
chứa nội dung của Listing 7-1 nhưng đã loại bỏ một số modules và hàm.

Chúng ta sẽ thể hiện hai cách để gọi hàm `add_to_waitlist` từ một hàm mới,
`eat_at_restaurant`, được định nghĩa trong gốc của crate. Các đường dẫn này là
đúng, nhưng vẫn còn một vấn đề khác sẽ ngăn ví dụ này biên dịch như hiện tại.
Chúng ta sẽ giải thích lý do sau.

Hàm `eat_at_restaurant` là một phần của API công khai của thư viện crate của
chúng ta, vì vậy chúng ta đánh dấu nó bằng từ khóa `pub`. Trong phần ["Hiển thị
Đường dẫn với từ khóa `pub`"][pub]<!-- ignore -->, chúng ta sẽ đi sâu hơn về
`pub`.

<Listing number="7-3" file-name="src/lib.rs" caption="Gọi hàm `add_to_waitlist` sử dụng đường dẫn tuyệt đối và tương đối">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-03/src/lib.rs}}
```

</Listing>

Lần đầu tiên chúng ta gọi hàm `add_to_waitlist` trong `eat_at_restaurant`, chúng
ta sử dụng đường dẫn tuyệt đối. Hàm `add_to_waitlist` được định nghĩa trong cùng
crate với `eat_at_restaurant`, điều đó có nghĩa là chúng ta có thể sử dụng từ
khóa `crate` để bắt đầu một đường dẫn tuyệt đối. Sau đó, chúng ta bao gồm từng
module liên tiếp cho đến khi chúng ta đến được `add_to_waitlist`. Bạn có thể
tưởng tượng một hệ thống tệp với cùng cấu trúc: chúng ta sẽ chỉ định đường dẫn
`/front_of_house/hosting/add_to_waitlist` để chạy chương trình
`add_to_waitlist`; việc sử dụng tên `crate` để bắt đầu từ gốc của crate giống
như việc sử dụng `/` để bắt đầu từ gốc hệ thống tệp trong shell của bạn.

Lần thứ hai chúng ta gọi `add_to_waitlist` trong `eat_at_restaurant`, chúng ta
sử dụng một đường dẫn tương đối. Đường dẫn bắt đầu với `front_of_house`, tên của
module được định nghĩa ở cùng cấp độ của cây module với `eat_at_restaurant`. Ở
đây, tương đương với hệ thống tệp sẽ là sử dụng đường dẫn
`front_of_house/hosting/add_to_waitlist`. Việc bắt đầu bằng tên module có nghĩa
là đường dẫn là tương đối.

Việc chọn sử dụng đường dẫn tương đối hay tuyệt đối là một quyết định bạn sẽ đưa
ra dựa trên dự án của mình, và nó phụ thuộc vào việc bạn có nhiều khả năng di
chuyển mã định nghĩa item riêng biệt hay cùng với mã sử dụng item đó. Ví dụ, nếu
chúng ta di chuyển module `front_of_house` và hàm `eat_at_restaurant` vào một
module có tên `customer_experience`, chúng ta sẽ cần cập nhật đường dẫn tuyệt
đối đến `add_to_waitlist`, nhưng đường dẫn tương đối vẫn sẽ hợp lệ. Tuy nhiên,
nếu chúng ta di chuyển riêng hàm `eat_at_restaurant` vào một module có tên
`dining`, đường dẫn tuyệt đối đến lệnh gọi `add_to_waitlist` sẽ giữ nguyên,
nhưng đường dẫn tương đối sẽ cần được cập nhật. Ưu tiên chung của chúng ta là
chỉ định đường dẫn tuyệt đối vì có khả năng cao hơn là chúng ta sẽ muốn di
chuyển các định nghĩa mã và lệnh gọi các item độc lập với nhau.

Hãy thử biên dịch Listing 7-3 và tìm hiểu tại sao nó chưa thể biên dịch! Các lỗi
mà chúng ta nhận được được hiển thị trong Listing 7-4.

<Listing number="7-4" caption="Lỗi trình biên dịch từ việc xây dựng mã trong Listing 7-3">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-03/output.txt}}
```

</Listing>

Các thông báo lỗi nói rằng module `hosting` là riêng tư. Nói cách khác, chúng ta
có các đường dẫn đúng cho module `hosting` và hàm `add_to_waitlist`, nhưng Rust
không cho phép chúng ta sử dụng chúng vì nó không có quyền truy cập vào các phần
riêng tư. Trong Rust, tất cả các item (hàm, phương thức, cấu trúc, enums,
modules, và hằng số) mặc định là riêng tư đối với các module cha. Nếu bạn muốn
làm cho một item như một hàm hoặc cấu trúc riêng tư, bạn đặt nó trong một
module.

Các item trong một module cha không thể sử dụng các item riêng tư bên trong các
module con, nhưng các item trong các module con có thể sử dụng các item trong
module tổ tiên của chúng. Điều này là vì các module con bọc và ẩn chi tiết triển
khai của chúng, nhưng các module con có thể thấy bối cảnh mà chúng được định
nghĩa. Để tiếp tục với ẩn dụ của chúng ta, hãy nghĩ về các quy tắc quyền riêng
tư giống như văn phòng phía sau của một nhà hàng: những gì diễn ra ở đó là riêng
tư đối với khách hàng nhà hàng, nhưng các quản lý văn phòng có thể thấy và làm
mọi thứ trong nhà hàng mà họ điều hành.

Rust đã chọn để hệ thống module hoạt động theo cách này để việc ẩn các chi tiết
triển khai bên trong là mặc định. Bằng cách đó, bạn biết những phần nào của mã
bên trong mà bạn có thể thay đổi mà không làm hỏng mã bên ngoài. Tuy nhiên, Rust
có cung cấp cho bạn tùy chọn để hiển thị các phần bên trong của mã module con
cho các module tổ tiên bên ngoài bằng cách sử dụng từ khóa `pub` để làm một item
công khai.

### Hiển thị Đường dẫn với từ khóa `pub`

Hãy quay lại lỗi trong Listing 7-4 đã nói với chúng ta rằng module `hosting` là
riêng tư. Chúng ta muốn hàm `eat_at_restaurant` trong module cha có quyền truy
cập vào hàm `add_to_waitlist` trong module con, vì vậy chúng ta đánh dấu module
`hosting` bằng từ khóa `pub`, như được hiển thị trong Listing 7-5.

<Listing number="7-5" file-name="src/lib.rs" caption="Khai báo module `hosting` là `pub` để sử dụng nó từ `eat_at_restaurant`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-05/src/lib.rs:here}}
```

</Listing>

Thật không may, mã trong Listing 7-5 vẫn dẫn đến lỗi trình biên dịch, như được
hiển thị trong Listing 7-6.

<Listing number="7-6" caption="Lỗi trình biên dịch từ việc xây dựng mã trong Listing 7-5">

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-05/output.txt}}
```

</Listing>

Chuyện gì đã xảy ra? Thêm từ khóa `pub` trước `mod hosting` làm cho module công
khai. Với thay đổi này, nếu chúng ta có thể truy cập `front_of_house`, chúng ta
có thể truy cập `hosting`. Nhưng _nội dung_ của `hosting` vẫn là riêng tư; việc
làm cho module công khai không làm cho nội dung của nó công khai. Từ khóa `pub`
trên một module chỉ cho phép mã trong các module tổ tiên của nó tham chiếu đến
nó, không phải truy cập vào mã bên trong của nó. Bởi vì các module là vùng chứa,
không có nhiều điều chúng ta có thể làm chỉ bằng cách làm cho module công khai;
chúng ta cần đi xa hơn và chọn làm cho một hoặc nhiều item trong module công
khai.

Các lỗi trong Listing 7-6 nói rằng hàm `add_to_waitlist` là riêng tư. Các quy
tắc quyền riêng tư áp dụng cho các cấu trúc, enums, hàm, và phương thức cũng như
các module.

Hãy cũng làm cho hàm `add_to_waitlist` công khai bằng cách thêm từ khóa `pub`
trước định nghĩa của nó, như trong Listing 7-7.

<Listing number="7-7" file-name="src/lib.rs" caption="Thêm từ khóa `pub` vào `mod hosting` và `fn add_to_waitlist` cho phép chúng ta gọi hàm từ `eat_at_restaurant`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-07/src/lib.rs:here}}
```

</Listing>

Bây giờ mã sẽ biên dịch! Để hiểu tại sao việc thêm từ khóa `pub` cho phép chúng
ta sử dụng các đường dẫn này trong `eat_at_restaurant` liên quan đến các quy tắc
quyền riêng tư, hãy xem đường dẫn tuyệt đối và tương đối.

Trong đường dẫn tuyệt đối, chúng ta bắt đầu với `crate`, gốc của cây module của
crate. Module `front_of_house` được định nghĩa trong gốc của crate. Mặc dù
`front_of_house` không công khai, vì hàm `eat_at_restaurant` được định nghĩa
trong cùng module với `front_of_house` (nghĩa là, `eat_at_restaurant` và
`front_of_house` là anh chị em), chúng ta có thể tham chiếu đến `front_of_house`
từ `eat_at_restaurant`. Tiếp theo là module `hosting` được đánh dấu là `pub`.
Chúng ta có thể truy cập module cha của `hosting`, nên chúng ta có thể truy cập
`hosting`. Cuối cùng, hàm `add_to_waitlist` được đánh dấu là `pub` và chúng ta
có thể truy cập module cha của nó, vì vậy lệnh gọi hàm này hoạt động!

Trong đường dẫn tương đối, logic giống với đường dẫn tuyệt đối ngoại trừ bước
đầu tiên: thay vì bắt đầu từ gốc của crate, đường dẫn bắt đầu từ
`front_of_house`. Module `front_of_house` được định nghĩa trong cùng module với
`eat_at_restaurant`, nên đường dẫn tương đối bắt đầu từ module mà
`eat_at_restaurant` được định nghĩa hoạt động. Sau đó, vì `hosting` và
`add_to_waitlist` được đánh dấu là `pub`, phần còn lại của đường dẫn hoạt động,
và lệnh gọi hàm này hợp lệ!

Nếu bạn có kế hoạch chia sẻ crate thư viện của mình để các dự án khác có thể sử
dụng mã của bạn, API công khai của bạn là hợp đồng của bạn với người dùng của
crate xác định cách họ có thể tương tác với mã của bạn. Có nhiều cân nhắc xung
quanh việc quản lý thay đổi đối với API công khai của bạn để giúp mọi người dễ
dàng hơn khi phụ thuộc vào crate của bạn. Những cân nhắc này nằm ngoài phạm vi
của cuốn sách này; nếu bạn quan tâm đến chủ đề này, hãy xem [Các Hướng dẫn API
của Rust][api-guidelines].

> #### Các Phương pháp Tốt nhất cho Packages với Binary và Library
>
> Chúng ta đã đề cập rằng một package có thể chứa cả gốc của binary crate
> _src/main.rs_ cũng như gốc của library crate _src/lib.rs_, và cả hai crates sẽ
> có tên package theo mặc định. Thông thường, các package với mô hình này chứa
> cả library và binary crate sẽ có đủ lượng mã trong binary crate để khởi động
> một chương trình thực thi gọi mã được khai báo trong library crate. Điều này
> cho phép các dự án khác hưởng lợi từ hầu hết chức năng mà package cung cấp vì
> mã của library crate có thể được chia sẻ.
>
> Cây module nên được định nghĩa trong _src/lib.rs_. Sau đó, bất kỳ item công
> khai nào có thể được sử dụng trong binary crate bằng cách bắt đầu đường dẫn
> với tên của package. Binary crate trở thành người dùng của library crate giống
> như một crate bên ngoài hoàn toàn sẽ sử dụng library crate: nó chỉ có thể sử
> dụng API công khai. Điều này giúp bạn thiết kế một API tốt; không chỉ là bạn
> là tác giả, bạn cũng là khách hàng!
>
> Trong [Chương 12][ch12]<!-- ignore -->, chúng ta sẽ minh họa phương pháp tổ
> chức này với một chương trình dòng lệnh sẽ chứa cả binary crate và library
> crate.

### Bắt đầu Đường dẫn Tương đối với `super`

Chúng ta có thể xây dựng đường dẫn tương đối bắt đầu ở module cha, thay vì
module hiện tại hoặc gốc của crate, bằng cách sử dụng `super` ở đầu đường dẫn.
Điều này giống như việc bắt đầu một đường dẫn hệ thống tệp với cú pháp `..` có
nghĩa là đi đến thư mục cha. Sử dụng `super` cho phép chúng ta tham chiếu đến
một item mà chúng ta biết là trong module cha, điều này có thể làm cho việc sắp
xếp lại cây module dễ dàng hơn khi module có liên quan chặt chẽ với module cha
nhưng module cha có thể được di chuyển đến nơi khác trong cây module vào một
ngày nào đó.

Hãy xem xét mã trong Listing 7-8 mô hình hóa tình huống mà một đầu bếp sửa một
đơn hàng không chính xác và tự mang ra cho khách hàng. Hàm `fix_incorrect_order`
được định nghĩa trong module `back_of_house` gọi hàm `deliver_order` được định
nghĩa trong module cha bằng cách chỉ định đường dẫn đến `deliver_order`, bắt đầu
với `super`.

<Listing number="7-8" file-name="src/lib.rs" caption="Gọi một hàm sử dụng đường dẫn tương đối bắt đầu bằng `super`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-08/src/lib.rs}}
```

</Listing>

Hàm `fix_incorrect_order` nằm trong module `back_of_house`, nên chúng ta có thể
sử dụng `super` để đi đến module cha của `back_of_house`, trong trường hợp này
là `crate`, gốc. Từ đó, chúng ta tìm kiếm `deliver_order` và tìm thấy nó. Thành
công! Chúng ta nghĩ rằng module `back_of_house` và hàm `deliver_order` có khả
năng sẽ giữ nguyên mối quan hệ với nhau và được di chuyển cùng nhau nếu chúng ta
quyết định tổ chức lại cây module của crate. Do đó, chúng ta đã sử dụng `super`
để chúng ta sẽ có ít nơi hơn để cập nhật mã trong tương lai nếu mã này được di
chuyển đến một module khác.

### Làm cho Structs và Enums Công khai

Chúng ta cũng có thể sử dụng `pub` để chỉ định structs và enums là công khai,
nhưng có một vài chi tiết bổ sung đối với việc sử dụng `pub` với structs và
enums. Nếu chúng ta sử dụng `pub` trước định nghĩa struct, chúng ta làm cho
struct công khai, nhưng các trường của struct vẫn sẽ là riêng tư. Chúng ta có
thể làm cho từng trường công khai hoặc không tùy theo từng trường hợp. Trong
Listing 7-9, chúng ta đã định nghĩa một struct `back_of_house::Breakfast` công
khai với một trường `toast` công khai nhưng một trường `seasonal_fruit` riêng
tư. Điều này mô hình hóa trường hợp trong một nhà hàng mà khách hàng có thể chọn
loại bánh mì đi kèm với bữa ăn, nhưng đầu bếp quyết định loại trái cây đi kèm
với bữa ăn dựa trên mùa nào và có sẵn trong kho. Loại trái cây có sẵn thay đổi
nhanh chóng, vì vậy khách hàng không thể chọn trái cây hoặc thậm chí không thể
thấy họ sẽ nhận được loại trái cây nào.

<Listing number="7-9" file-name="src/lib.rs" caption="Một struct với một số trường công khai và một số trường riêng tư">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-09/src/lib.rs}}
```

</Listing>

Bởi vì trường `toast` trong struct `back_of_house::Breakfast` là công khai,
trong `eat_at_restaurant` chúng ta có thể viết và đọc trường `toast` bằng cách
sử dụng ký hiệu dấu chấm. Lưu ý rằng chúng ta không thể sử dụng trường
`seasonal_fruit` trong `eat_at_restaurant`, bởi vì `seasonal_fruit` là riêng tư.
Hãy thử bỏ comment dòng sửa đổi giá trị trường `seasonal_fruit` để xem lỗi bạn
nhận được!

Cũng lưu ý rằng bởi vì `back_of_house::Breakfast` có một trường riêng tư, struct
cần cung cấp một hàm liên kết công khai để xây dựng một thể hiện của `Breakfast`
(chúng ta đã đặt tên nó là `summer` ở đây). Nếu `Breakfast` không có một hàm như
vậy, chúng ta không thể tạo một thể hiện của `Breakfast` trong
`eat_at_restaurant` bởi vì chúng ta không thể đặt giá trị của trường riêng tư
`seasonal_fruit` trong `eat_at_restaurant`.

Ngược lại, nếu chúng ta làm cho một enum công khai, tất cả các biến thể của nó
đều công khai. Chúng ta chỉ cần từ khóa `pub` trước từ khóa `enum`, như được
hiển thị trong Listing 7-10.

<Listing number="7-10" file-name="src/lib.rs" caption="Chỉ định một enum là công khai làm cho tất cả các biến thể của nó công khai.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-10/src/lib.rs}}
```

</Listing>

Vì chúng ta đã làm cho enum `Appetizer` công khai, chúng ta có thể sử dụng các
biến thể `Soup` và `Salad` trong `eat_at_restaurant`.

Enums không hữu ích lắm trừ khi các biến thể của chúng là công khai; sẽ rất
phiền toái khi phải chú thích tất cả các biến thể enum với `pub` trong mọi
trường hợp, vì vậy mặc định cho các biến thể enum là công khai. Structs thường
hữu ích mà không cần các trường của chúng là công khai, vì vậy các trường struct
tuân theo quy tắc chung là mọi thứ mặc định là riêng tư trừ khi được chú thích
với `pub`.

Có một tình huống khác liên quan đến `pub` mà chúng ta chưa đề cập, và đó là
tính năng hệ thống module cuối cùng của chúng ta: từ khóa `use`. Chúng ta sẽ đề
cập đến `use` riêng lẻ trước, và sau đó chúng ta sẽ thể hiện cách kết hợp `pub`
và `use`.

[pub]:
  ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html#exposing-paths-with-the-pub-keyword
[api-guidelines]: https://rust-lang.github.io/api-guidelines/
[ch12]: ch12-00-an-io-project.html
