## Định nghĩa một Enum

Trong khi struct cung cấp cho bạn cách nhóm các trường và dữ liệu liên quan với
nhau, như một `Rectangle` với `width` và `height` của nó, enum cho bạn cách để
nói rằng một giá trị là một trong một tập hợp các giá trị có thể có. Ví dụ,
chúng ta có thể muốn nói rằng `Rectangle` là một trong một tập hợp các hình dạng
có thể có bao gồm cả `Circle` và `Triangle`. Để làm được điều này, Rust cho phép
chúng ta mã hóa các khả năng này như một enum.

Hãy xem xét một tình huống mà chúng ta có thể muốn biểu đạt trong mã và tìm hiểu
tại sao enum hữu ích và phù hợp hơn struct trong trường hợp này. Giả sử chúng ta
cần làm việc với địa chỉ IP. Hiện tại, hai chuẩn chính được sử dụng cho địa chỉ
IP: phiên bản bốn và phiên bản sáu. Vì đây là những khả năng duy nhất cho một
địa chỉ IP mà chương trình của chúng ta sẽ gặp phải, chúng ta có thể _liệt kê_
tất cả các biến thể có thể có, đó là nơi mà enumeration có được tên gọi của nó.

Bất kỳ địa chỉ IP nào cũng có thể là địa chỉ phiên bản bốn hoặc phiên bản sáu,
nhưng không thể là cả hai cùng một lúc. Đặc tính đó của địa chỉ IP làm cho cấu
trúc dữ liệu enum trở nên thích hợp bởi vì một giá trị enum chỉ có thể là một
trong các biến thể của nó. Cả hai địa chỉ phiên bản bốn và phiên bản sáu đều về
cơ bản vẫn là địa chỉ IP, vì vậy chúng nên được xử lý như cùng một kiểu khi mã
đang xử lý các tình huống áp dụng cho bất kỳ loại địa chỉ IP nào.

Chúng ta có thể biểu đạt khái niệm này trong mã bằng cách định nghĩa một enum
`IpAddrKind` và liệt kê các loại có thể có của địa chỉ IP, `V4` và `V6`. Đây là
các biến thể của enum:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:def}}
```

`IpAddrKind` bây giờ là một kiểu dữ liệu tùy chỉnh mà chúng ta có thể sử dụng ở
nơi khác trong mã của mình.

### Giá trị Enum

Chúng ta có thể tạo các instance của mỗi biến thể của `IpAddrKind` như thế này:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:instance}}
```

Lưu ý rằng các biến thể của enum được đặt tên trong không gian tên dưới định
danh của nó, và chúng ta sử dụng dấu hai chấm kép để tách chúng. Điều này hữu
ích bởi vì bây giờ cả hai giá trị `IpAddrKind::V4` và `IpAddrKind::V6` đều thuộc
cùng một kiểu: `IpAddrKind`. Chúng ta sau đó có thể, ví dụ, định nghĩa một hàm
nhận bất kỳ `IpAddrKind` nào:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn}}
```

Và chúng ta có thể gọi hàm này với cả hai biến thể:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-01-defining-enums/src/main.rs:fn_call}}
```

Sử dụng enum thậm chí còn có nhiều lợi thế hơn. Khi suy nghĩ nhiều hơn về kiểu
địa chỉ IP của chúng ta, hiện tại chúng ta không có cách nào để lưu trữ dữ liệu
thực tế của địa chỉ IP; chúng ta chỉ biết nó thuộc loại nào. Vì bạn vừa mới học
về struct trong Chương 5, bạn có thể bị cám dỗ để giải quyết vấn đề này với
struct như được thể hiện trong Listing 6-1.

<Listing number="6-1" caption="Lưu trữ dữ liệu và biến thể `IpAddrKind` của một địa chỉ IP bằng một `struct`">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-01/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta đã định nghĩa một struct `IpAddr` có hai trường: một trường
`kind` là kiểu `IpAddrKind` (enum mà chúng ta đã định nghĩa trước đó) và một
trường `address` kiểu `String`. Chúng ta có hai instance của struct này.
Instance đầu tiên là `home`, và nó có giá trị `IpAddrKind::V4` làm `kind` với dữ
liệu địa chỉ liên quan là `127.0.0.1`. Instance thứ hai là `loopback`. Nó có
biến thể khác của `IpAddrKind` làm giá trị `kind` của nó, `V6`, và có địa chỉ
`::1` liên kết với nó. Chúng ta đã sử dụng một struct để nhóm các giá trị `kind`
và `address` lại với nhau, vì vậy bây giờ biến thể được liên kết với giá trị.

Tuy nhiên, biểu diễn cùng một khái niệm chỉ bằng enum thì ngắn gọn hơn: thay vì
một enum bên trong một struct, chúng ta có thể đưa dữ liệu trực tiếp vào mỗi
biến thể enum. Định nghĩa mới này của enum `IpAddr` nói rằng cả hai biến thể
`V4` và `V6` sẽ có các giá trị `String` liên quan:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-02-enum-with-data/src/main.rs:here}}
```

Chúng ta gắn dữ liệu trực tiếp vào mỗi biến thể của enum, vì vậy không cần một
struct bổ sung. Ở đây, cũng dễ dàng hơn để thấy một chi tiết khác về cách enum
hoạt động: tên của mỗi biến thể enum mà chúng ta định nghĩa cũng trở thành một
hàm xây dựng một instance của enum đó. Nghĩa là, `IpAddr::V4()` là một lệnh gọi
hàm nhận một đối số `String` và trả về một instance của kiểu `IpAddr`. Chúng ta
tự động nhận được hàm constructor này được định nghĩa là kết quả của việc định
nghĩa enum.

Có một lợi thế khác khi sử dụng enum thay vì struct: mỗi biến thể có thể có các
loại và số lượng dữ liệu liên quan khác nhau. Địa chỉ IP phiên bản bốn sẽ luôn
có bốn thành phần số mà sẽ có giá trị từ 0 đến 255. Nếu chúng ta muốn lưu trữ
địa chỉ `V4` dưới dạng bốn giá trị `u8` nhưng vẫn biểu đạt địa chỉ `V6` dưới
dạng một giá trị `String`, chúng ta sẽ không thể làm được với một struct. Enum
xử lý trường hợp này một cách dễ dàng:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-03-variants-with-different-data/src/main.rs:here}}
```

Chúng ta đã thể hiện một số cách khác nhau để định nghĩa cấu trúc dữ liệu để lưu
trữ phiên bản bốn và phiên bản sáu địa chỉ IP. Tuy nhiên, hóa ra, muốn lưu trữ
địa chỉ IP và mã hóa loại nào của chúng là rất phổ biến đến nỗi [thư viện chuẩn
đã có một định nghĩa mà chúng ta có thể sử dụng!][IpAddr]<!-- ignore --> Hãy xem
cách thư viện chuẩn định nghĩa `IpAddr`: nó có chính xác enum và các biến thể mà
chúng ta đã định nghĩa và sử dụng, nhưng nó nhúng dữ liệu địa chỉ bên trong các
biến thể dưới dạng hai struct khác nhau, được định nghĩa khác nhau cho mỗi biến
thể:

```rust
struct Ipv4Addr {
    // --snip--
}

struct Ipv6Addr {
    // --snip--
}

enum IpAddr {
    V4(Ipv4Addr),
    V6(Ipv6Addr),
}
```

Đoạn mã này minh họa rằng bạn có thể đặt bất kỳ loại dữ liệu nào bên trong một
biến thể enum: chuỗi, kiểu số, hoặc struct, ví dụ vậy. Bạn thậm chí có thể bao
gồm một enum khác! Ngoài ra, các kiểu thư viện chuẩn thường không phức tạp hơn
nhiều so với những gì bạn có thể nghĩ ra.

Lưu ý rằng mặc dù thư viện chuẩn chứa một định nghĩa cho `IpAddr`, chúng ta vẫn
có thể tạo và sử dụng định nghĩa của riêng mình mà không gây xung đột vì chúng
ta chưa đưa định nghĩa của thư viện chuẩn vào phạm vi của mình. Chúng ta sẽ nói
thêm về việc đưa các kiểu vào phạm vi trong Chương 7.

Hãy xem một ví dụ khác về enum trong Listing 6-2: ví dụ này có sự đa dạng về các
kiểu nhúng trong các biến thể của nó.

<Listing number="6-2" caption="Một enum `Message` mà các biến thể của nó lưu trữ số lượng và loại giá trị khác nhau">

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/listing-06-02/src/main.rs:here}}
```

</Listing>

Enum này có bốn biến thể với các kiểu khác nhau:

- `Quit` Không có dữ liệu nào liên kết với nó.
- `Move` Có các trường có tên, giống như một struct.
- `Write` Bao gồm một `String` duy nhất.
- `ChangeColor` Bao gồm ba giá trị `i32`.

Định nghĩa một enum với các biến thể như trong Listing 6-2 tương tự như việc
định nghĩa các loại định nghĩa struct khác nhau, ngoại trừ việc enum không sử
dụng từ khóa `struct` và tất cả các biến thể được nhóm lại với nhau dưới kiểu
`Message`. Các struct sau đây có thể lưu trữ cùng dữ liệu mà các biến thể enum
trước đó lưu trữ:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-04-structs-similar-to-message-enum/src/main.rs:here}}
```

Nhưng nếu chúng ta sử dụng các struct khác nhau, mỗi struct có kiểu riêng, chúng
ta không thể dễ dàng định nghĩa một hàm để nhận bất kỳ loại thông điệp nào trong
số này như chúng ta có thể làm với enum `Message` được định nghĩa trong Listing
6-2, vốn là một kiểu duy nhất.

Có một điểm tương đồng nữa giữa enum và struct: giống như chúng ta có thể định
nghĩa các phương thức trên struct bằng cách sử dụng `impl`, chúng ta cũng có thể
định nghĩa các phương thức trên enum. Đây là một phương thức có tên `call` mà
chúng ta có thể định nghĩa trên enum `Message` của chúng ta:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-05-methods-on-enums/src/main.rs:here}}
```

Nội dung của phương thức sẽ sử dụng `self` để lấy giá trị mà chúng ta đã gọi
phương thức trên đó. Trong ví dụ này, chúng ta đã tạo một biến `m` có giá trị
`Message::Write(String::from("hello"))`, và đó là giá trị mà `self` sẽ là trong
nội dung của phương thức `call` khi `m.call()` chạy.

Hãy xem một enum khác trong thư viện chuẩn rất phổ biến và hữu ích: `Option`.

### Enum `Option` và Lợi thế của Nó So với Giá trị Null

Phần này khám phá một trường hợp nghiên cứu của `Option`, một enum khác được
định nghĩa bởi thư viện chuẩn. Kiểu `Option` mã hóa kịch bản rất phổ biến trong
đó một giá trị có thể là một thứ gì đó hoặc có thể là không có gì.

Ví dụ, nếu bạn yêu cầu phần tử đầu tiên trong danh sách không rỗng, bạn sẽ nhận
được một giá trị. Nếu bạn yêu cầu phần tử đầu tiên trong danh sách rỗng, bạn sẽ
không nhận được gì. Biểu đạt khái niệm này trong hệ thống kiểu có nghĩa là trình
biên dịch có thể kiểm tra liệu bạn đã xử lý tất cả các trường hợp mà bạn nên xử
lý hay chưa; điều này có thể ngăn chặn các lỗi cực kỳ phổ biến trong các ngôn
ngữ lập trình khác.

Thiết kế ngôn ngữ lập trình thường được nghĩ đến theo nghĩa của các tính năng
bạn bao gồm, nhưng các tính năng bạn loại trừ cũng quan trọng. Rust không có
tính năng null mà nhiều ngôn ngữ khác có. _Null_ là một giá trị có nghĩa là
không có giá trị nào ở đó. Trong các ngôn ngữ có null, biến luôn có thể ở một
trong hai trạng thái: null hoặc không-null.

Trong bài thuyết trình năm 2009 của mình "Null References: The Billion Dollar
Mistake," Tony Hoare, người phát minh ra null, đã nói như sau:

> Tôi gọi đó là sai lầm hàng tỷ đô la của mình. Vào thời điểm đó, tôi đang thiết
> kế hệ thống kiểu đầu tiên toàn diện cho các tham chiếu trong một ngôn ngữ
> hướng đối tượng. Mục tiêu của tôi là đảm bảo rằng tất cả việc sử dụng tham
> chiếu phải hoàn toàn an toàn, với việc kiểm tra được thực hiện tự động bởi
> trình biên dịch. Nhưng tôi đã không thể cưỡng lại cám dỗ đặt vào một tham
> chiếu null, đơn giản vì nó rất dễ thực hiện. Điều này đã dẫn đến vô số lỗi, lỗ
> hổng và hệ thống bị sập, có lẽ đã gây ra tổn thất và thiệt hại hàng tỷ đô la
> trong bốn mươi năm qua.

Vấn đề với giá trị null là nếu bạn cố gắng sử dụng một giá trị null như một giá
trị không-null, bạn sẽ gặp một loại lỗi nào đó. Bởi vì thuộc tính null hoặc
không-null này là phổ biến, rất dễ mắc phải loại lỗi này.

Tuy nhiên, khái niệm mà null đang cố gắng biểu đạt vẫn là một khái niệm hữu ích:
một null là một giá trị hiện không hợp lệ hoặc vắng mặt vì một lý do nào đó.

Vấn đề thực sự không phải là với khái niệm mà là với cách triển khai cụ thể. Vì
vậy, Rust không có null, nhưng nó có một enum có thể mã hóa khái niệm về một giá
trị đang hiện diện hoặc vắng mặt. Enum này là `Option<T>`, và nó được [định
nghĩa bởi thư viện chuẩn][option]<!-- ignore --> như sau:

```rust
enum Option<T> {
    None,
    Some(T),
}
```

Enum `Option<T>` rất hữu ích đến nỗi nó thậm chí được đưa vào prelude; bạn không
cần phải đưa nó vào phạm vi một cách rõ ràng. Các biến thể của nó cũng được bao
gồm trong prelude: bạn có thể sử dụng `Some` và `None` trực tiếp mà không cần
tiền tố `Option::`. Enum `Option<T>` vẫn chỉ là một enum thông thường, và
`Some(T)` và `None` vẫn là các biến thể của kiểu `Option<T>`.

Cú pháp `<T>` là một tính năng của Rust mà chúng ta chưa nói đến. Đó là một tham
số kiểu generic, và chúng ta sẽ đề cập đến generic chi tiết hơn trong Chương 10.
Hiện tại, tất cả những gì bạn cần biết là `<T>` có nghĩa là biến thể `Some` của
enum `Option` có thể chứa một phần dữ liệu của bất kỳ kiểu nào, và mỗi kiểu cụ
thể được sử dụng thay thế cho `T` làm cho toàn bộ kiểu `Option<T>` trở thành một
kiểu khác. Dưới đây là một số ví dụ về việc sử dụng các giá trị `Option` để lưu
trữ các kiểu số và kiểu ký tự:

```rust
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-06-option-examples/src/main.rs:here}}
```

Kiểu của `some_number` là `Option<i32>`. Kiểu của `some_char` là `Option<char>`,
là một kiểu khác. Rust có thể suy ra các kiểu này bởi vì chúng ta đã chỉ định
một giá trị bên trong biến thể `Some`. Đối với `absent_number`, Rust yêu cầu
chúng ta chú thích toàn bộ kiểu `Option`: trình biên dịch không thể suy ra kiểu
mà biến thể `Some` tương ứng sẽ giữ bằng cách chỉ nhìn vào một giá trị `None`. Ở
đây, chúng ta nói với Rust rằng chúng ta muốn `absent_number` có kiểu
`Option<i32>`.

Khi chúng ta có một giá trị `Some`, chúng ta biết rằng một giá trị đang hiện
diện và giá trị đó được giữ bên trong `Some`. Khi chúng ta có một giá trị
`None`, theo một nghĩa nào đó, nó có nghĩa giống như null: chúng ta không có một
giá trị hợp lệ. Vậy tại sao việc có `Option<T>` lại tốt hơn việc có null?

Nói ngắn gọn, bởi vì `Option<T>` và `T` (trong đó `T` có thể là bất kỳ kiểu nào)
là các kiểu khác nhau, trình biên dịch sẽ không cho phép chúng ta sử dụng giá
trị `Option<T>` như thể nó là một giá trị hợp lệ chắc chắn. Ví dụ, đoạn mã này
sẽ không biên dịch, bởi vì nó đang cố gắng cộng một `i8` với một `Option<i8>`:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/src/main.rs:here}}
```

Nếu chúng ta chạy mã này, chúng ta sẽ nhận được một thông báo lỗi như thế này:

```console
{{#include ../listings/ch06-enums-and-pattern-matching/no-listing-07-cant-use-option-directly/output.txt}}
```

Mạnh mẽ! Về thực chất, thông báo lỗi này có nghĩa là Rust không hiểu cách cộng
một `i8` và một `Option<i8>`, bởi vì chúng là các kiểu khác nhau. Khi chúng ta
có một giá trị của một kiểu như `i8` trong Rust, trình biên dịch sẽ đảm bảo rằng
chúng ta luôn có một giá trị hợp lệ. Chúng ta có thể tiến hành một cách tự tin
mà không cần kiểm tra null trước khi sử dụng giá trị đó. Chỉ khi chúng ta có một
`Option<i8>` (hoặc bất kỳ kiểu giá trị nào mà chúng ta đang làm việc) thì chúng
ta mới phải lo lắng về việc có thể không có một giá trị, và trình biên dịch sẽ
đảm bảo chúng ta xử lý trường hợp đó trước khi sử dụng giá trị.

Nói cách khác, bạn phải chuyển đổi một `Option<T>` thành một `T` trước khi bạn
có thể thực hiện các hoạt động `T` với nó. Nói chung, điều này giúp bắt được một
trong những vấn đề phổ biến nhất với null: giả định rằng một thứ không phải null
khi nó thực sự là null.

Việc loại bỏ nguy cơ giả định không chính xác một giá trị không-null giúp bạn tự
tin hơn về mã của mình. Để có một giá trị có thể là null, bạn phải chọn tham gia
một cách rõ ràng bằng cách làm cho kiểu của giá trị đó là `Option<T>`. Sau đó,
khi bạn sử dụng giá trị đó, bạn bắt buộc phải xử lý một cách rõ ràng trường hợp
khi giá trị là null. Mọi nơi mà một giá trị có kiểu không phải là `Option<T>`,
bạn _có thể_ an toàn giả định rằng giá trị không phải là null. Đây là một quyết
định thiết kế có chủ ý của Rust để hạn chế sự phổ biến của null và tăng tính an
toàn của mã Rust.

Vậy làm thế nào để bạn lấy giá trị `T` ra khỏi một biến thể `Some` khi bạn có
một giá trị kiểu `Option<T>` để bạn có thể sử dụng giá trị đó? Enum `Option<T>`
có một số lượng lớn các phương thức hữu ích trong nhiều tình huống khác nhau;
bạn có thể kiểm tra chúng trong [tài liệu của nó][docs]<!-- ignore -->. Làm quen
với các phương thức trên `Option<T>` sẽ cực kỳ hữu ích trong hành trình của bạn
với Rust.

Nói chung, để sử dụng một giá trị `Option<T>`, bạn muốn có mã sẽ xử lý mỗi biến
thể. Bạn muốn một số mã chỉ chạy khi bạn có giá trị `Some(T)`, và mã này được
phép sử dụng `T` bên trong. Bạn muốn một số mã khác chỉ chạy nếu bạn có một giá
trị `None`, và mã đó không có giá trị `T` nào để sử dụng. Biểu thức `match` là
một cấu trúc luồng điều khiển làm chính xác điều này khi được sử dụng với enum:
nó sẽ chạy mã khác nhau tùy thuộc vào biến thể nào của enum mà nó có, và mã đó
có thể sử dụng dữ liệu bên trong giá trị phù hợp.

[IpAddr]: ../std/net/enum.IpAddr.html
[option]: ../std/option/enum.Option.html
[docs]: ../std/option/enum.Option.html
