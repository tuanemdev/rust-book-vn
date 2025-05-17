## Trait Nâng Cao

Chúng ta đã đề cập đến trait trong ["Trait: Định Nghĩa Hành Vi
Chung"][traits-defining-shared-behavior]<!-- ignore --> ở Chương 10, nhưng chúng
ta chưa thảo luận về các chi tiết nâng cao hơn. Giờ đây khi bạn đã biết nhiều
hơn về Rust, chúng ta có thể đi sâu vào chi tiết.

<!-- Old link, do not remove -->

<a id="specifying-placeholder-types-in-trait-definitions-with-associated-types"></a>

### Kiểu Liên Kết (Associated Types)

_Kiểu liên kết_ kết nối một placeholder kiểu với một trait để các định nghĩa
phương thức của trait có thể sử dụng các placeholder kiểu này trong chữ ký của
chúng. Người triển khai trait sẽ chỉ định kiểu cụ thể được sử dụng thay vì kiểu
placeholder cho việc triển khai cụ thể. Bằng cách đó, chúng ta có thể định nghĩa
một trait sử dụng một số kiểu mà không cần biết chính xác những kiểu đó là gì
cho đến khi trait được triển khai.

Chúng ta đã mô tả hầu hết các tính năng nâng cao trong chương này là ít khi cần
đến. Kiểu liên kết nằm đâu đó ở giữa: chúng được sử dụng ít thường xuyên hơn các
tính năng được giải thích trong phần còn lại của cuốn sách nhưng phổ biến hơn
nhiều tính năng khác được thảo luận trong chương này.

Một ví dụ về trait với kiểu liên kết là trait `Iterator` mà thư viện chuẩn cung
cấp. Kiểu liên kết được đặt tên là `Item` và đại diện cho kiểu của các giá trị
mà kiểu thực hiện trait `Iterator` đang lặp qua. Định nghĩa của trait `Iterator`
được hiển thị trong Listing 20-13.

<Listing number="20-13" caption="Định nghĩa của trait `Iterator` có kiểu liên kết `Item`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-13/src/lib.rs}}
```

</Listing>

Kiểu `Item` là một placeholder, và định nghĩa của phương thức `next` cho thấy nó
sẽ trả về các giá trị kiểu `Option<Self::Item>`. Người triển khai trait
`Iterator` sẽ chỉ định kiểu cụ thể cho `Item`, và phương thức `next` sẽ trả về
một `Option` chứa giá trị của kiểu cụ thể đó.

Kiểu liên kết có thể giống với khái niệm generics, vì generics cũng cho phép
chúng ta định nghĩa một hàm mà không cần chỉ định kiểu nào nó có thể xử lý. Để
xem xét sự khác biệt giữa hai khái niệm này, chúng ta sẽ xem xét việc triển khai
trait `Iterator` trên một kiểu có tên là `Counter` chỉ định kiểu `Item` là
`u32`:

<Listing file-name="src/lib.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-22-iterator-on-counter/src/lib.rs:ch19}}
```

</Listing>

Cú pháp này có vẻ tương đương với generics. Vậy tại sao không chỉ định nghĩa
trait `Iterator` với generics, như được hiển thị trong Listing 20-14?

<Listing number="20-14" caption="Một định nghĩa giả thuyết của trait `Iterator` sử dụng generics">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-14/src/lib.rs}}
```

</Listing>

Sự khác biệt là khi sử dụng generics, như trong Listing 20-14, chúng ta phải chú
thích các kiểu trong mỗi triển khai; bởi vì chúng ta cũng có thể triển khai
`Iterator<String> for Counter` hoặc bất kỳ kiểu nào khác, chúng ta có thể có
nhiều triển khai của `Iterator` cho `Counter`. Nói cách khác, khi một trait có
tham số generic, nó có thể được triển khai cho một kiểu nhiều lần, thay đổi kiểu
cụ thể của các tham số kiểu generic mỗi lần. Khi chúng ta sử dụng phương thức
`next` trên `Counter`, chúng ta sẽ phải cung cấp chú thích kiểu để chỉ ra triển
khai nào của `Iterator` chúng ta muốn sử dụng.

Với kiểu liên kết, chúng ta không cần phải chú thích kiểu vì chúng ta không thể
triển khai một trait trên một kiểu nhiều lần. Trong Listing 20-13 với định nghĩa
sử dụng kiểu liên kết, chúng ta chỉ có thể chọn kiểu của `Item` sẽ là gì một lần
vì chỉ có thể có một `impl Iterator for Counter`. Chúng ta không phải chỉ định
rằng chúng ta muốn một iterator của các giá trị `u32` ở mọi nơi chúng ta gọi
`next` trên `Counter`.

Kiểu liên kết cũng trở thành một phần của hợp đồng trait: người triển khai trait
phải cung cấp một kiểu để thay thế cho placeholder kiểu liên kết. Kiểu liên kết
thường có tên mô tả cách kiểu sẽ được sử dụng, và việc ghi tài liệu cho kiểu
liên kết trong tài liệu API là một thực hành tốt.

### Tham Số Kiểu Generic Mặc Định và Nạp Chồng Toán Tử

Khi chúng ta sử dụng các tham số kiểu generic, chúng ta có thể chỉ định một kiểu
cụ thể mặc định cho kiểu generic. Điều này loại bỏ nhu cầu cho người triển khai
trait phải chỉ định một kiểu cụ thể nếu kiểu mặc định hoạt động. Bạn chỉ định
một kiểu mặc định khi khai báo một kiểu generic với cú pháp
`<PlaceholderType=ConcreteType>`.

Một ví dụ tuyệt vời về tình huống mà kỹ thuật này hữu ích là với _nạp chồng toán
tử_, trong đó bạn tùy chỉnh hành vi của một toán tử (như `+`) trong các tình
huống cụ thể.

Rust không cho phép bạn tạo các toán tử của riêng mình hoặc nạp chồng các toán
tử tùy ý. Nhưng bạn có thể nạp chồng các hoạt động và các trait tương ứng được
liệt kê trong `std::ops` bằng cách triển khai các trait liên quan đến toán tử.
Ví dụ, trong Listing 20-15, chúng ta nạp chồng toán tử `+` để cộng hai thể hiện
`Point` với nhau. Chúng ta làm điều này bằng cách triển khai trait `Add` trên
struct `Point`.

<Listing number="20-15" file-name="src/main.rs" caption="Triển khai trait `Add` để nạp chồng toán tử `+` cho các thể hiện `Point`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-15/src/main.rs}}
```

</Listing>

Phương thức `add` cộng các giá trị `x` của hai thể hiện `Point` và các giá trị
`y` của hai thể hiện `Point` để tạo ra một `Point` mới. Trait `Add` có một kiểu
liên kết có tên là `Output` xác định kiểu trả về từ phương thức `add`.

Kiểu generic mặc định trong mã này nằm trong trait `Add`. Đây là định nghĩa của
nó:

```rust
trait Add<Rhs=Self> {
    type Output;

    fn add(self, rhs: Rhs) -> Self::Output;
}
```

Mã này có vẻ quen thuộc: một trait với một phương thức và một kiểu liên kết.
Phần mới là `Rhs=Self`: cú pháp này được gọi là _tham số kiểu mặc định_. Tham số
kiểu generic `Rhs` (viết tắt của "right-hand side") định nghĩa kiểu của tham số
`rhs` trong phương thức `add`. Nếu chúng ta không chỉ định một kiểu cụ thể cho
`Rhs` khi triển khai trait `Add`, kiểu của `Rhs` sẽ mặc định là `Self`, là kiểu
mà chúng ta đang triển khai `Add` trên đó.

Khi chúng ta triển khai `Add` cho `Point`, chúng ta đã sử dụng giá trị mặc định
cho `Rhs` bởi vì chúng ta muốn cộng hai thể hiện `Point`. Hãy xem xét một ví dụ
về việc triển khai trait `Add` trong đó chúng ta muốn tùy chỉnh kiểu `Rhs` thay
vì sử dụng giá trị mặc định.

Chúng ta có hai struct, `Millimeters` và `Meters`, chứa các giá trị trong các
đơn vị khác nhau. Việc bọc mỏng của một kiểu hiện có trong một struct khác được
gọi là _mẫu newtype_, mà chúng ta mô tả chi tiết hơn trong phần ["Sử Dụng Mẫu
Newtype để Triển Khai Trait Bên Ngoài trên Kiểu Bên
Ngoài"][newtype]<!-- ignore -->. Chúng ta muốn cộng các giá trị tính bằng
milimet với các giá trị tính bằng mét và có triển khai của `Add` thực hiện
chuyển đổi một cách chính xác. Chúng ta có thể triển khai `Add` cho
`Millimeters` với `Meters` là `Rhs`, như hiển thị trong Listing 20-16.

<Listing number="20-16" file-name="src/lib.rs" caption="Triển khai trait `Add` trên `Millimeters` để cộng `Millimeters` và `Meters`">

```rust,noplayground
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-16/src/lib.rs}}
```

</Listing>

Để cộng `Millimeters` và `Meters`, chúng ta chỉ định `impl Add<Meters>` để đặt
giá trị của tham số kiểu `Rhs` thay vì sử dụng giá trị mặc định là `Self`.

Bạn sẽ sử dụng các tham số kiểu mặc định theo hai cách chính:

1. Để mở rộng một kiểu mà không phá vỡ mã hiện có
2. Để cho phép tùy chỉnh trong các trường hợp cụ thể mà hầu hết người dùng không
   cần

Trait `Add` của thư viện chuẩn là một ví dụ cho mục đích thứ hai: thông thường,
bạn sẽ cộng hai kiểu giống nhau, nhưng trait `Add` cung cấp khả năng tùy chỉnh
vượt ra ngoài điều đó. Sử dụng một tham số kiểu mặc định trong định nghĩa trait
`Add` có nghĩa là bạn không phải chỉ định tham số bổ sung trong hầu hết các
trường hợp. Nói cách khác, một chút mã soạn sẵn triển khai không cần thiết, làm
cho việc sử dụng trait dễ dàng hơn.

Mục đích đầu tiên tương tự như mục đích thứ hai nhưng ngược lại: nếu bạn muốn
thêm một tham số kiểu vào một trait hiện có, bạn có thể cho nó một giá trị mặc
định để cho phép mở rộng chức năng của trait mà không phá vỡ mã triển khai hiện
có.

<!-- Old link, do not remove -->

<a id="fully-qualified-syntax-for-disambiguation-calling-methods-with-the-same-name"></a>

### Phân Biệt Giữa Các Phương Thức Có Cùng Tên

Không có gì trong Rust ngăn cản một trait có một phương thức có cùng tên với
phương thức của trait khác, Rust cũng không ngăn bạn triển khai cả hai trait
trên một kiểu. Cũng có thể triển khai một phương thức trực tiếp trên kiểu với
cùng tên như các phương thức từ các trait.

Khi gọi các phương thức có cùng tên, bạn cần phải nói cho Rust biết bạn muốn sử
dụng cái nào. Xem xét mã trong Listing 20-17 trong đó chúng ta đã định nghĩa hai
trait, `Pilot` và `Wizard`, cả hai đều có một phương thức được gọi là `fly`. Sau
đó, chúng ta triển khai cả hai trait trên một kiểu `Human` đã có một phương thức
tên là `fly` được triển khai trên nó. Mỗi phương thức `fly` làm một điều gì đó
khác nhau.

<Listing number="20-17" file-name="src/main.rs" caption="Hai trait được định nghĩa để có một phương thức `fly` và được triển khai trên kiểu `Human`, và một phương thức `fly` được triển khai trực tiếp trên `Human`.">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-17/src/main.rs:here}}
```

</Listing>

Khi chúng ta gọi `fly` trên một thể hiện của `Human`, trình biên dịch mặc định
gọi phương thức được triển khai trực tiếp trên kiểu, như hiển thị trong Listing
20-18.

<Listing number="20-18" file-name="src/main.rs" caption="Gọi `fly` trên một thể hiện của `Human`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-18/src/main.rs:here}}
```

</Listing>

Chạy mã này sẽ in ra `*waving arms furiously*`, cho thấy Rust đã gọi phương thức
`fly` được triển khai trực tiếp trên `Human`.

Để gọi các phương thức `fly` từ trait `Pilot` hoặc trait `Wizard`, chúng ta cần
sử dụng cú pháp rõ ràng hơn để chỉ định phương thức `fly` nào chúng ta muốn.
Listing 20-19 minh họa cú pháp này.

<Listing number="20-19" file-name="src/main.rs" caption="Chỉ định phương thức `fly` của trait nào chúng ta muốn gọi">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-19/src/main.rs:here}}
```

</Listing>

Chỉ định tên trait trước tên phương thức làm rõ cho Rust biết chúng ta muốn gọi
triển khai `fly` nào. Chúng ta cũng có thể viết `Human::fly(&person)`, tương
đương với `person.fly()` mà chúng ta đã sử dụng trong Listing 20-19, nhưng điều
này dài hơn một chút nếu chúng ta không cần phân biệt.

Chạy mã này in ra như sau:

```console
{{#include ../listings/ch20-advanced-features/listing-20-19/output.txt}}
```

Bởi vì phương thức `fly` nhận một tham số `self`, nếu chúng ta có hai _kiểu_ đều
triển khai một _trait_, Rust có thể xác định triển khai nào của trait sẽ được sử
dụng dựa trên kiểu của `self`.

Tuy nhiên, các hàm liên kết không phải là phương thức không có tham số `self`.
Khi có nhiều kiểu hoặc trait định nghĩa các hàm không phải phương thức với cùng
tên hàm, Rust không phải lúc nào cũng biết bạn muốn ý nghĩa kiểu nào trừ khi bạn
sử dụng _cú pháp đầy đủ_. Ví dụ, trong Listing 20-20, chúng ta tạo một trait cho
một nơi trú ẩn động vật muốn đặt tên cho tất cả chó con là Spot. Chúng ta tạo
một trait `Animal` với một hàm liên kết không phải phương thức `baby_name`.
Trait `Animal` được triển khai cho struct `Dog`, trên đó chúng ta cũng cung cấp
một hàm liên kết không phải phương thức `baby_name` trực tiếp.

<Listing number="20-20" file-name="src/main.rs" caption="Một trait với một hàm liên kết và một kiểu với một hàm liên kết cùng tên cũng triển khai trait đó">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-20/src/main.rs}}
```

</Listing>

Chúng ta triển khai mã để đặt tên cho tất cả chó con là Spot trong hàm liên kết
`baby_name` được định nghĩa trên `Dog`. Kiểu `Dog` cũng triển khai trait
`Animal`, mô tả các đặc điểm mà tất cả động vật có. Chó con được gọi là puppies,
và điều đó được thể hiện trong việc triển khai trait `Animal` trên `Dog` trong
hàm `baby_name` liên kết với trait `Animal`.

Trong `main`, chúng ta gọi hàm `Dog::baby_name`, hàm này gọi hàm liên kết được
định nghĩa trực tiếp trên `Dog`. Mã này in ra:

```console
{{#include ../listings/ch20-advanced-features/listing-20-20/output.txt}}
```

Đầu ra này không phải là những gì chúng ta muốn. Chúng ta muốn gọi hàm
`baby_name` là một phần của trait `Animal` mà chúng ta đã triển khai trên `Dog`
để mã in ra `A baby dog is called a puppy`. Kỹ thuật chỉ định tên trait mà chúng
ta đã sử dụng trong Listing 20-19 không giúp ích ở đây; nếu chúng ta thay đổi
`main` thành mã trong Listing 20-21, chúng ta sẽ nhận được lỗi biên dịch.

<Listing number="20-21" file-name="src/main.rs" caption="Cố gắng gọi hàm `baby_name` từ trait `Animal`, nhưng Rust không biết triển khai nào sẽ dùng">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-21/src/main.rs:here}}
```

</Listing>

Bởi vì `Animal::baby_name` không có tham số `self`, và có thể có các kiểu khác
triển khai trait `Animal`, Rust không thể xác định triển khai
`Animal::baby_name` nào chúng ta muốn. Chúng ta sẽ nhận được lỗi trình biên dịch
này:

```console
{{#include ../listings/ch20-advanced-features/listing-20-21/output.txt}}
```

Để loại bỏ sự không rõ ràng và nói cho Rust biết rằng chúng ta muốn sử dụng
triển khai của `Animal` cho `Dog` thay vì triển khai của `Animal` cho một số
kiểu khác, chúng ta cần sử dụng cú pháp đầy đủ. Listing 20-22 minh họa cách sử
dụng cú pháp đầy đủ.

<Listing number="20-22" file-name="src/main.rs" caption="Sử dụng cú pháp đầy đủ để chỉ định rằng chúng ta muốn gọi hàm `baby_name` từ trait `Animal` như được triển khai trên `Dog`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-22/src/main.rs:here}}
```

</Listing>

Chúng ta đang cung cấp cho Rust một chú thích kiểu trong dấu ngoặc nhọn, cho
biết chúng ta muốn gọi phương thức `baby_name` từ trait `Animal` như được triển
khai trên `Dog` bằng cách nói rằng chúng ta muốn coi kiểu `Dog` như một `Animal`
cho lệnh gọi hàm này. Mã này giờ đây sẽ in ra những gì chúng ta muốn:

```console
{{#include ../listings/ch20-advanced-features/listing-20-22/output.txt}}
```

Nhìn chung, cú pháp đầy đủ được định nghĩa như sau:

```rust,ignore
<Type as Trait>::function(receiver_if_method, next_arg, ...);
```

Đối với các hàm liên kết không phải phương thức, sẽ không có `receiver`: sẽ chỉ
có danh sách các đối số khác. Bạn có thể sử dụng cú pháp đầy đủ ở mọi nơi mà bạn
gọi hàm hoặc phương thức. Tuy nhiên, bạn được phép bỏ qua bất kỳ phần nào của cú
pháp này mà Rust có thể tìm ra từ thông tin khác trong chương trình. Bạn chỉ cần
sử dụng cú pháp dài dòng hơn này trong các trường hợp có nhiều triển khai sử
dụng cùng một tên và Rust cần trợ giúp để xác định triển khai nào bạn muốn gọi.

<!-- Old link, do not remove -->

<a id="using-supertraits-to-require-one-traits-functionality-within-another-trait"></a>

### Sử Dụng Supertraits

Đôi khi bạn có thể viết một định nghĩa trait phụ thuộc vào một trait khác: để
một kiểu triển khai trait đầu tiên, bạn muốn yêu cầu kiểu đó cũng phải triển
khai trait thứ hai. Bạn sẽ làm điều này để định nghĩa trait của bạn có thể sử
dụng các phần tử liên kết của trait thứ hai. Trait mà định nghĩa trait của bạn
dựa vào được gọi là _supertrait_ của trait của bạn.

Ví dụ, giả sử chúng ta muốn tạo một trait `OutlinePrint` với một phương thức
`outline_print` sẽ in một giá trị đã cho được định dạng để nó được đóng khung
bằng dấu hoa thị. Nghĩa là, với một struct `Point` triển khai trait `Display`
của thư viện chuẩn để có kết quả là `(x, y)`, khi chúng ta gọi `outline_print`
trên một thể hiện `Point` có `1` cho `x` và `3` cho `y`, nó sẽ in ra:

```text
**********
*        *
* (1, 3) *
*        *
**********
```

Trong việc triển khai phương thức `outline_print`, chúng ta muốn sử dụng chức
năng của trait `Display`. Do đó, chúng ta cần chỉ định rằng trait `OutlinePrint`
sẽ chỉ hoạt động cho các kiểu cũng triển khai `Display` và cung cấp chức năng mà
`OutlinePrint` cần. Chúng ta có thể làm điều đó trong định nghĩa trait bằng cách
chỉ định `OutlinePrint: Display`. Kỹ thuật này tương tự như việc thêm một ràng
buộc trait cho trait. Listing 20-23 cho thấy một triển khai của trait
`OutlinePrint`.

<Listing number="20-23" file-name="src/main.rs" caption="Triển khai trait `OutlinePrint` đòi hỏi chức năng từ `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-23/src/main.rs:here}}
```

</Listing>

Bởi vì chúng ta đã chỉ định rằng `OutlinePrint` yêu cầu trait `Display`, chúng
ta có thể sử dụng hàm `to_string` được tự động triển khai cho bất kỳ kiểu nào
triển khai `Display`. Nếu chúng ta cố gắng sử dụng `to_string` mà không thêm dấu
hai chấm và chỉ định trait `Display` sau tên trait, chúng ta sẽ gặp lỗi nói rằng
không tìm thấy phương thức nào có tên là `to_string` cho kiểu `&Self` trong phạm
vi hiện tại.

Hãy xem điều gì xảy ra khi chúng ta cố gắng triển khai `OutlinePrint` trên một
kiểu không triển khai `Display`, chẳng hạn như struct `Point`:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/src/main.rs:here}}
```

</Listing>

Chúng ta nhận được lỗi nói rằng `Display` là cần thiết nhưng không được triển
khai:

```console
{{#include ../listings/ch20-advanced-features/no-listing-02-impl-outlineprint-for-point/output.txt}}
```

Để sửa lỗi này, chúng ta triển khai `Display` trên `Point` và thỏa mãn ràng buộc
mà `OutlinePrint` yêu cầu, như sau:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-03-impl-display-for-point/src/main.rs:here}}
```

</Listing>

Sau đó, việc triển khai trait `OutlinePrint` trên `Point` sẽ biên dịch thành
công, và chúng ta có thể gọi `outline_print` trên một thể hiện `Point` để hiển
thị nó trong một đường viền dấu hoa thị.

### Sử Dụng Mẫu Newtype để Triển Khai Trait Bên Ngoài trên Kiểu Bên Ngoài

Trong ["Triển Khai một Trait trên một
Kiểu"][implementing-a-trait-on-a-type]<!-- ignore --> ở Chương 10, chúng ta đã
đề cập đến quy tắc orphan nói rằng chúng ta chỉ được phép triển khai một trait
trên một kiểu nếu một trong hai trait hoặc kiểu, hoặc cả hai, là cục bộ đối với
crate của chúng ta. Có thể vượt qua hạn chế này bằng cách sử dụng _mẫu newtype_,
liên quan đến việc tạo một kiểu mới trong một tuple struct. (Chúng ta đã đề cập
đến tuple struct trong ["Sử Dụng Tuple Structs Không Có Trường Đặt Tên để Tạo
Các Kiểu Khác Nhau"][tuple-structs]<!-- ignore --> ở Chương 5.) Tuple struct sẽ
có một trường và là một bản bọc mỏng xung quanh kiểu mà chúng ta muốn triển khai
trait. Sau đó, kiểu bọc sẽ là cục bộ đối với crate của chúng ta, và chúng ta có
thể triển khai trait trên bản bọc. _Newtype_ là một thuật ngữ có nguồn gốc từ
ngôn ngữ lập trình Haskell. Không có mất mát hiệu suất thời gian chạy khi sử
dụng mẫu này, và kiểu bọc được loại bỏ tại thời điểm biên dịch.

Ví dụ, giả sử chúng ta muốn triển khai `Display` trên `Vec<T>`, mà quy tắc
orphan ngăn chúng ta làm trực tiếp vì trait `Display` và kiểu `Vec<T>` được định
nghĩa bên ngoài crate của chúng ta. Chúng ta có thể tạo một struct `Wrapper`
chứa một thể hiện của `Vec<T>`; sau đó chúng ta có thể triển khai `Display` trên
`Wrapper` và sử dụng giá trị `Vec<T>`, như hiển thị trong Listing 20-24.

<Listing number="20-24" file-name="src/main.rs" caption="Tạo một kiểu `Wrapper` xung quanh `Vec<String>` để triển khai `Display`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-24/src/main.rs}}
```

</Listing>

Triển khai của `Display` sử dụng `self.0` để truy cập `Vec<T>` bên trong, vì
`Wrapper` là một tuple struct và `Vec<T>` là phần tử ở chỉ số 0 trong tuple. Sau
đó, chúng ta có thể sử dụng chức năng của trait `Display` trên `Wrapper`.

Nhược điểm của việc sử dụng kỹ thuật này là `Wrapper` là một kiểu mới, vì vậy nó
không có các phương thức của giá trị mà nó đang giữ. Chúng ta sẽ phải triển khai
trực tiếp tất cả các phương thức của `Vec<T>` trên `Wrapper` sao cho các phương
thức ủy quyền cho `self.0`, điều này sẽ cho phép chúng ta đối xử với `Wrapper`
chính xác như một `Vec<T>`. Nếu chúng ta muốn kiểu mới có mọi phương thức mà
kiểu bên trong có, việc triển khai trait `Deref` trên `Wrapper` để trả về kiểu
bên trong sẽ là một giải pháp (chúng ta đã thảo luận về việc triển khai trait
`Deref` trong ["Đối Xử với Smart Pointers Như Tham Chiếu Thông Thường với Trait
`Deref`"][smart-pointer-deref]<!-- ignore --> ở Chương 15). Nếu chúng ta không
muốn kiểu `Wrapper` có tất cả các phương thức của kiểu bên trong — ví dụ, để hạn
chế hành vi của kiểu `Wrapper` — chúng ta sẽ phải tự triển khai các phương thức
mà chúng ta muốn.

Mẫu newtype này cũng hữu ích ngay cả khi các trait không liên quan. Hãy chuyển
sang xem xét một số cách tiên tiến để tương tác với hệ thống kiểu của Rust.

[newtype]:
  ch20-02-advanced-traits.html#using-the-newtype-pattern-to-implement-external-traits-on-external-types
[implementing-a-trait-on-a-type]:
  ch10-02-traits.html#implementing-a-trait-on-a-type
[traits-defining-shared-behavior]:
  ch10-02-traits.html#traits-defining-shared-behavior
[smart-pointer-deref]:
  ch15-02-deref.html#treating-smart-pointers-like-regular-references-with-the-deref-trait
[tuple-structs]:
  ch05-01-defining-structs.html#using-tuple-structs-without-named-fields-to-create-different-types
