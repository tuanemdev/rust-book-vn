## Đưa Đường dẫn vào Phạm vi với từ khóa `use`

Việc phải viết ra các đường dẫn để gọi hàm có thể cảm thấy bất tiện và lặp đi
lặp lại. Trong Listing 7-7, cho dù chúng ta chọn đường dẫn tuyệt đối hoặc tương
đối đến hàm `add_to_waitlist`, mỗi khi chúng ta muốn gọi `add_to_waitlist` chúng
ta phải chỉ định cả `front_of_house` và `hosting`. May mắn thay, có một cách để
đơn giản hóa quá trình này: chúng ta có thể tạo một lối tắt đến đường dẫn với từ
khóa `use` một lần, và sau đó sử dụng tên ngắn hơn ở mọi nơi khác trong phạm vi.

Trong Listing 7-11, chúng ta đưa module `crate::front_of_house::hosting` vào
phạm vi của hàm `eat_at_restaurant` để chúng ta chỉ cần chỉ định
`hosting::add_to_waitlist` để gọi hàm `add_to_waitlist` trong
`eat_at_restaurant`.

<Listing number="7-11" file-name="src/lib.rs" caption="Đưa một module vào phạm vi với `use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-11/src/lib.rs}}
```

</Listing>

Việc thêm `use` và một đường dẫn trong một phạm vi tương tự như việc tạo một
liên kết tượng trưng trong hệ thống tệp. Bằng cách thêm
`use crate::front_of_house::hosting` trong gốc crate, `hosting` bây giờ là một
tên hợp lệ trong phạm vi đó, giống như thể module `hosting` đã được định nghĩa
trong gốc crate. Đường dẫn được đưa vào phạm vi bằng `use` cũng kiểm tra quyền
riêng tư, giống như bất kỳ đường dẫn nào khác.

Lưu ý rằng `use` chỉ tạo lối tắt cho phạm vi cụ thể mà `use` xảy ra. Listing
7-12 di chuyển hàm `eat_at_restaurant` vào một module con mới có tên là
`customer`, sau đó là một phạm vi khác với câu lệnh `use`, vì vậy phần thân hàm
sẽ không biên dịch được.

<Listing number="7-12" file-name="src/lib.rs" caption="Câu lệnh `use` chỉ áp dụng trong phạm vi mà nó ở trong.">

```rust,noplayground,test_harness,does_not_compile,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-12/src/lib.rs}}
```

</Listing>

Lỗi trình biên dịch cho thấy lối tắt không còn áp dụng trong module `customer`:

```console
{{#include ../listings/ch07-managing-growing-projects/listing-07-12/output.txt}}
```

Lưu ý cũng có một cảnh báo rằng `use` không còn được sử dụng trong phạm vi của
nó! Để khắc phục vấn đề này, hãy di chuyển `use` vào module `customer`, hoặc
tham chiếu lối tắt trong module cha với `super::hosting` trong module con
`customer`.

### Tạo Đường dẫn `use` Thuần phong

Trong Listing 7-11, bạn có thể đã tự hỏi tại sao chúng ta chỉ định
`use crate::front_of_house::hosting` và sau đó gọi `hosting::add_to_waitlist`
trong `eat_at_restaurant`, thay vì chỉ định đường dẫn `use` hoàn toàn đến hàm
`add_to_waitlist` để đạt được kết quả tương tự, như trong Listing 7-13.

<Listing number="7-13" file-name="src/lib.rs" caption="Đưa hàm `add_to_waitlist` vào phạm vi với `use`, cách này không được khuyến khích">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-13/src/lib.rs}}
```

</Listing>

Mặc dù cả Listing 7-11 và Listing 7-13 đều thực hiện cùng một nhiệm vụ, Listing
7-11 là cách thuần phong để đưa một hàm vào phạm vi với `use`. Việc đưa module
cha của hàm vào phạm vi với `use` có nghĩa là chúng ta phải chỉ định module cha
khi gọi hàm. Chỉ định module cha khi gọi hàm làm rõ rằng hàm không được định
nghĩa cục bộ trong khi vẫn giảm thiểu việc lặp lại đường dẫn đầy đủ. Đoạn mã
trong Listing 7-13 không rõ ràng về nơi `add_to_waitlist` được định nghĩa.

Mặt khác, khi đưa vào structs, enums, và các item khác với `use`, việc chỉ định
đường dẫn đầy đủ là thuần phong. Listing 7-14 thể hiện cách thuần phong để đưa
struct `HashMap` của thư viện chuẩn vào phạm vi của một binary crate.

<Listing number="7-14" file-name="src/main.rs" caption="Đưa `HashMap` vào phạm vi theo cách thuần phong">

```rust
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-14/src/main.rs}}
```

</Listing>

Không có lý do mạnh mẽ đằng sau cách viết này: đó chỉ là quy ước đã nổi lên, và
mọi người đã quen với việc đọc và viết mã Rust theo cách này.

Ngoại lệ cho cách viết này là nếu chúng ta đưa hai item có cùng tên vào phạm vi
với các câu lệnh `use`, vì Rust không cho phép điều đó. Listing 7-15 thể hiện
cách đưa hai kiểu `Result` vào phạm vi có cùng tên nhưng có module cha khác
nhau, và cách tham chiếu đến chúng.

<Listing number="7-15" file-name="src/lib.rs" caption="Việc đưa hai kiểu có cùng tên vào cùng một phạm vi yêu cầu sử dụng các module cha của chúng.">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-15/src/lib.rs:here}}
```

</Listing>

Như bạn có thể thấy, việc sử dụng các module cha phân biệt hai kiểu `Result`.
Nếu thay vào đó chúng ta chỉ định `use std::fmt::Result` và
`use std::io::Result`, chúng ta sẽ có hai kiểu `Result` trong cùng một phạm vi,
và Rust sẽ không biết chúng ta muốn nói đến kiểu nào khi chúng ta sử dụng
`Result`.

### Cung cấp Tên Mới với từ khóa `as`

Có một giải pháp khác cho vấn đề đưa hai kiểu có cùng tên vào cùng một phạm vi
với `use`: sau đường dẫn, chúng ta có thể chỉ định `as` và một tên cục bộ mới,
hoặc _alias_ (biệt danh), cho kiểu đó. Listing 7-16 thể hiện một cách khác để
viết mã trong Listing 7-15 bằng cách đổi tên một trong hai kiểu `Result` sử dụng
`as`.

<Listing number="7-16" file-name="src/lib.rs" caption="Đổi tên một kiểu khi nó được đưa vào phạm vi với từ khóa `as`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-16/src/lib.rs:here}}
```

</Listing>

Trong câu lệnh `use` thứ hai, chúng ta đã chọn tên mới `IoResult` cho kiểu
`std::io::Result`, điều này sẽ không xung đột với `Result` từ `std::fmt` mà
chúng ta cũng đã đưa vào phạm vi. Listing 7-15 và Listing 7-16 đều được coi là
thuần phong, vì vậy sự lựa chọn là ở bạn!

### Tái xuất Tên với `pub use`

Khi chúng ta đưa một tên vào phạm vi với từ khóa `use`, tên đó là riêng tư đối
với phạm vi mà chúng ta đã nhập nó vào. Để cho phép mã bên ngoài phạm vi đó tham
chiếu đến tên đó như thể nó đã được định nghĩa trong phạm vi đó, chúng ta có thể
kết hợp `pub` và `use`. Kỹ thuật này được gọi là _re-exporting_ (tái xuất) vì
chúng ta đưa một item vào phạm vi nhưng cũng làm cho item đó có sẵn cho người
khác đưa vào phạm vi của họ.

Listing 7-17 thể hiện mã trong Listing 7-11 với `use` trong module gốc được thay
đổi thành `pub use`.

<Listing number="7-17" file-name="src/lib.rs" caption="Làm cho một tên sẵn có cho mọi mã để sử dụng từ một phạm vi mới với `pub use`">

```rust,noplayground,test_harness
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-17/src/lib.rs}}
```

</Listing>

Trước thay đổi này, mã bên ngoài sẽ phải gọi hàm `add_to_waitlist` bằng cách sử
dụng đường dẫn `restaurant::front_of_house::hosting::add_to_waitlist()`, điều
này cũng đòi hỏi module `front_of_house` phải được đánh dấu là `pub`. Bây giờ
`pub use` này đã tái xuất module `hosting` từ module gốc, mã bên ngoài có thể sử
dụng đường dẫn `restaurant::hosting::add_to_waitlist()` thay thế.

Tái xuất rất hữu ích khi cấu trúc nội bộ của mã của bạn khác với cách các lập
trình viên gọi mã của bạn sẽ nghĩ về miền. Ví dụ, trong phép ẩn dụ nhà hàng này,
những người điều hành nhà hàng nghĩ về "front of house" và "back of house."
Nhưng khách hàng ghé thăm một nhà hàng có lẽ sẽ không nghĩ về các phần của nhà
hàng theo những thuật ngữ đó. Với `pub use`, chúng ta có thể viết mã của mình
với một cấu trúc nhưng hiển thị một cấu trúc khác. Làm như vậy giúp thư viện của
chúng ta được tổ chức tốt cho các lập trình viên làm việc trên thư viện và các
lập trình viên gọi thư viện. Chúng ta sẽ xem một ví dụ khác về `pub use` và cách
nó ảnh hưởng đến tài liệu crate của bạn trong ["Xuất một Public API Tiện lợi với
`pub use`"][ch14-pub-use]<!-- ignore --> trong Chương 14.

### Sử dụng Các Gói Bên ngoài

Trong Chương 2, chúng ta đã lập trình một dự án trò chơi đoán số sử dụng một gói
bên ngoài có tên là `rand` để có được số ngẫu nhiên. Để sử dụng `rand` trong dự
án của chúng ta, chúng ta đã thêm dòng này vào _Cargo.toml_:

<!-- Khi cập nhật phiên bản của `rand` được sử dụng, cũng cập nhật phiên bản của
`rand` được sử dụng trong các tệp này để tất cả đều khớp:
* ch02-00-guessing-game-tutorial.md
* ch14-03-cargo-workspaces.md
-->

<Listing file-name="Cargo.toml">

```toml
{{#include ../listings/ch02-guessing-game-tutorial/listing-02-02/Cargo.toml:9:}}
```

</Listing>

Thêm `rand` như một phụ thuộc trong _Cargo.toml_ cho Cargo biết phải tải xuống
gói `rand` và bất kỳ phụ thuộc nào từ [crates.io](https://crates.io/) và làm cho
`rand` có sẵn cho dự án của chúng ta.

Sau đó, để đưa các định nghĩa `rand` vào phạm vi của gói của chúng ta, chúng ta
đã thêm một dòng `use` bắt đầu bằng tên của crate, `rand`, và liệt kê các item
chúng ta muốn đưa vào phạm vi. Hãy nhớ lại rằng trong ["Tạo một Số Ngẫu
nhiên"][rand]<!-- ignore --> trong Chương 2, chúng ta đã đưa trait `Rng` vào
phạm vi và gọi hàm `rand::thread_rng`:

```rust,ignore
{{#rustdoc_include ../listings/ch02-guessing-game-tutorial/listing-02-03/src/main.rs:ch07-04}}
```

Các thành viên của cộng đồng Rust đã làm cho nhiều gói có sẵn tại
[crates.io](https://crates.io/), và việc kéo bất kỳ gói nào vào gói của bạn đều
liên quan đến các bước tương tự: liệt kê chúng trong tệp _Cargo.toml_ của gói
của bạn và sử dụng `use` để đưa các item từ các crate đó vào phạm vi.

Lưu ý rằng thư viện chuẩn `std` cũng là một crate bên ngoài đối với gói của
chúng ta. Bởi vì thư viện chuẩn được đi kèm với ngôn ngữ Rust, chúng ta không
cần phải thay đổi _Cargo.toml_ để bao gồm `std`. Nhưng chúng ta cần tham chiếu
đến nó với `use` để đưa các item từ đó vào phạm vi của gói của chúng ta. Ví dụ,
với `HashMap` chúng ta sẽ sử dụng dòng này:

```rust
use std::collections::HashMap;
```

Đây là một đường dẫn tuyệt đối bắt đầu bằng `std`, tên của crate thư viện chuẩn.

### Sử dụng Đường dẫn Lồng nhau để Làm sạch Danh sách `use` Lớn

Nếu chúng ta sử dụng nhiều item được định nghĩa trong cùng một crate hoặc cùng
một module, việc liệt kê mỗi item trên một dòng riêng có thể chiếm nhiều không
gian dọc trong các tệp của chúng ta. Ví dụ, hai câu lệnh `use` này chúng ta đã
có trong trò chơi đoán số trong Listing 2-4 đưa các item từ `std` vào phạm vi:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/no-listing-01-use-std-unnested/src/main.rs:here}}
```

</Listing>

Thay vào đó, chúng ta có thể sử dụng đường dẫn lồng nhau để đưa các item tương
tự vào phạm vi trong một dòng. Chúng ta làm điều này bằng cách chỉ định phần
chung của đường dẫn, theo sau là hai dấu hai chấm, và sau đó là dấu ngoặc nhọn
quanh danh sách các phần của đường dẫn khác nhau, như được hiển thị trong
Listing 7-18.

<Listing number="7-18" file-name="src/main.rs" caption="Chỉ định một đường dẫn lồng nhau để đưa nhiều item có cùng tiền tố vào phạm vi">

```rust,ignore
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-18/src/main.rs:here}}
```

</Listing>

Trong các chương trình lớn hơn, việc đưa nhiều item vào phạm vi từ cùng một
crate hoặc module sử dụng đường dẫn lồng nhau có thể giảm đáng kể số lượng câu
lệnh `use` riêng biệt cần thiết!

Chúng ta có thể sử dụng đường dẫn lồng nhau ở bất kỳ cấp độ nào trong một đường
dẫn, điều này rất hữu ích khi kết hợp hai câu lệnh `use` có chung một đường dẫn
con. Ví dụ, Listing 7-19 thể hiện hai câu lệnh `use`: một câu lệnh đưa `std::io`
vào phạm vi và một câu lệnh đưa `std::io::Write` vào phạm vi.

<Listing number="7-19" file-name="src/lib.rs" caption="Hai câu lệnh `use` mà một câu lệnh là đường dẫn con của câu lệnh kia">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-19/src/lib.rs}}
```

</Listing>

Phần chung của hai đường dẫn này là `std::io`, và đó là toàn bộ đường dẫn đầu
tiên. Để hợp nhất hai đường dẫn này thành một câu lệnh `use`, chúng ta có thể sử
dụng `self` trong đường dẫn lồng nhau, như được hiển thị trong Listing 7-20.

<Listing number="7-20" file-name="src/lib.rs" caption="Kết hợp các đường dẫn trong Listing 7-19 thành một câu lệnh `use`">

```rust,noplayground
{{#rustdoc_include ../listings/ch07-managing-growing-projects/listing-07-20/src/lib.rs}}
```

</Listing>

Dòng này đưa `std::io` và `std::io::Write` vào phạm vi.

### Toán tử Glob

Nếu chúng ta muốn đưa _tất cả_ các item công khai được định nghĩa trong một
đường dẫn vào phạm vi, chúng ta có thể chỉ định đường dẫn đó theo sau bởi toán
tử glob `*`:

```rust
use std::collections::*;
```

Câu lệnh `use` này đưa tất cả các item công khai được định nghĩa trong
`std::collections` vào phạm vi hiện tại. Hãy cẩn thận khi sử dụng toán tử glob!
Glob có thể làm cho nó khó hơn để biết những tên nào đang trong phạm vi và tên
được sử dụng trong chương trình của bạn được định nghĩa ở đâu. Ngoài ra, nếu phụ
thuộc thay đổi các định nghĩa của nó, những gì bạn đã import cũng thay đổi theo,
điều này có thể dẫn đến lỗi trình biên dịch khi bạn nâng cấp phụ thuộc nếu phụ
thuộc đó thêm một định nghĩa có cùng tên với một định nghĩa của bạn trong cùng
phạm vi, chẳng hạn.

Toán tử glob thường được sử dụng khi kiểm thử để đưa mọi thứ đang được kiểm thử
vào module `tests`; chúng ta sẽ nói về điều đó trong ["Cách Viết Kiểm
thử"][writing-tests]<!-- ignore --> trong Chương 11. Toán tử glob cũng đôi khi
được sử dụng như một phần của mẫu prelude: xem
[tài liệu thư viện chuẩn](../std/prelude/index.html#other-preludes)<!-- ignore -->
để biết thêm thông tin về mẫu đó.

[ch14-pub-use]:
  ch14-02-publishing-to-crates-io.html#exporting-a-convenient-public-api-with-pub-use
[rand]: ch02-00-guessing-game-tutorial.html#generating-a-random-number
[writing-tests]: ch11-01-writing-tests.html#how-to-write-tests
