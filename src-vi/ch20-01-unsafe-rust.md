## Unsafe Rust

Tất cả mã nguồn mà chúng ta đã thảo luận cho đến nay đều có các đảm bảo về an
toàn bộ nhớ của Rust được thực thi tại thời điểm biên dịch. Tuy nhiên, Rust có
một ngôn ngữ thứ hai ẩn bên trong nó không thực thi các đảm bảo an toàn bộ nhớ
này: nó được gọi là _unsafe Rust_ và hoạt động giống như Rust thông thường,
nhưng cung cấp cho chúng ta các siêu năng lực bổ sung.

Unsafe Rust tồn tại bởi vì, về bản chất, phân tích tĩnh là bảo thủ. Khi trình
biên dịch cố gắng xác định liệu mã có duy trì các đảm bảo hay không, tốt hơn là
nó từ chối một số chương trình hợp lệ còn hơn là chấp nhận một số chương trình
không hợp lệ. Mặc dù mã _có thể_ ổn, nhưng nếu trình biên dịch Rust không có đủ
thông tin để tự tin, nó sẽ từ chối mã. Trong những trường hợp này, bạn có thể sử
dụng mã unsafe để nói với trình biên dịch rằng, "Hãy tin tôi, tôi biết mình đang
làm gì." Tuy nhiên, hãy cảnh giác, rằng bạn sử dụng unsafe Rust với rủi ro của
riêng mình: nếu bạn sử dụng mã unsafe không chính xác, các vấn đề có thể xảy ra
do không an toàn bộ nhớ, chẳng hạn như giải tham chiếu con trỏ null.

Một lý do khác khiến Rust có một phiên bản khác là unsafe là vì phần cứng máy
tính cơ bản vốn không an toàn. Nếu Rust không cho phép bạn thực hiện các hoạt
động không an toàn, bạn không thể thực hiện một số tác vụ nhất định. Rust cần
cho phép bạn thực hiện lập trình hệ thống cấp thấp, chẳng hạn như tương tác trực
tiếp với hệ điều hành hoặc thậm chí viết hệ điều hành của riêng bạn. Làm việc
với lập trình hệ thống cấp thấp là một trong những mục tiêu của ngôn ngữ này.
Hãy khám phá những gì chúng ta có thể làm với unsafe Rust và cách thực hiện.

### Siêu Năng Lực Unsafe

Để chuyển sang unsafe Rust, sử dụng từ khóa `unsafe` và sau đó bắt đầu một khối
mới chứa mã unsafe. Bạn có thể thực hiện năm hành động trong unsafe Rust mà bạn
không thể thực hiện trong safe Rust, mà chúng ta gọi là _siêu năng lực unsafe_.
Những siêu năng lực đó bao gồm khả năng:

- 1. Giải tham chiếu một con trỏ thô
- 1. Gọi một hàm hoặc phương thức unsafe
- 1. Truy cập hoặc sửa đổi một biến tĩnh có thể thay đổi
- 1. Triển khai một trait unsafe
- 1. Truy cập các trường của một `union`

Điều quan trọng cần hiểu là `unsafe` không tắt trình kiểm tra mượn hoặc vô hiệu
hóa bất kỳ kiểm tra an toàn nào khác của Rust: nếu bạn sử dụng một tham chiếu
trong mã unsafe, nó vẫn sẽ được kiểm tra. Từ khóa `unsafe` chỉ cung cấp cho bạn
quyền truy cập vào năm tính năng này, sau đó không được trình biên dịch kiểm tra
về an toàn bộ nhớ. Bạn vẫn sẽ có một mức độ an toàn nhất định bên trong khối
unsafe.

Ngoài ra, `unsafe` không có nghĩa là mã bên trong khối nhất thiết phải nguy hiểm
hoặc chắc chắn sẽ có vấn đề về an toàn bộ nhớ: ý định là với tư cách là lập
trình viên, bạn sẽ đảm bảo rằng mã bên trong khối `unsafe` sẽ truy cập bộ nhớ
theo cách hợp lệ.

Con người có thể mắc lỗi và lỗi sẽ xảy ra, nhưng bằng cách yêu cầu năm hoạt động
unsafe này phải nằm trong các khối được chú thích với `unsafe`, bạn sẽ biết rằng
bất kỳ lỗi nào liên quan đến an toàn bộ nhớ phải nằm trong một khối `unsafe`.
Giữ các khối `unsafe` nhỏ; bạn sẽ biết ơn sau này khi điều tra các lỗi bộ nhớ.

Để cô lập mã unsafe càng nhiều càng tốt, tốt nhất là bao bọc mã đó trong một
trừu tượng an toàn và cung cấp một API an toàn, điều mà chúng ta sẽ thảo luận
sau trong chương khi chúng ta xem xét các hàm và phương thức unsafe. Các phần
của thư viện tiêu chuẩn được triển khai như các trừu tượng an toàn trên mã
unsafe đã được kiểm tra. Bao bọc mã unsafe trong một trừu tượng an toàn ngăn
việc sử dụng `unsafe` rò rỉ ra tất cả các nơi mà bạn hoặc người dùng của bạn có
thể muốn sử dụng chức năng được triển khai với mã `unsafe`, bởi vì việc sử dụng
một trừu tượng an toàn là an toàn.

Hãy xem xét từng siêu năng lực unsafe một. Chúng ta cũng sẽ xem xét một số trừu
tượng cung cấp giao diện an toàn cho mã unsafe.

### Giải Tham Chiếu một Con Trỏ Thô

Trong Chương 4, trong ["Tham Chiếu Đang
Treo"][dangling-references]<!-- ignore -->, chúng ta đã đề cập rằng trình biên
dịch đảm bảo các tham chiếu luôn hợp lệ. Unsafe Rust có hai loại mới được gọi là
_con trỏ thô_ tương tự như tham chiếu. Giống như với các tham chiếu, con trỏ thô
có thể bất biến hoặc có thể thay đổi và được viết là `*const T` và `*mut T`,
tương ứng. Dấu hoa thị không phải là toán tử giải tham chiếu; nó là một phần của
tên kiểu. Trong bối cảnh của con trỏ thô, _bất biến_ có nghĩa là con trỏ không
thể được gán trực tiếp sau khi được giải tham chiếu.

Khác với tham chiếu và con trỏ thông minh, con trỏ thô:

- Được phép bỏ qua các quy tắc mượn bằng cách có cả con trỏ bất biến và có thể
  thay đổi hoặc nhiều con trỏ có thể thay đổi đến cùng một vị trí
- Không được đảm bảo trỏ đến bộ nhớ hợp lệ
- Được phép là null
- Không triển khai bất kỳ quá trình dọn dẹp tự động nào

Bằng cách chọn không để Rust thực thi các đảm bảo này, bạn có thể từ bỏ an toàn
được đảm bảo để đổi lấy hiệu suất cao hơn hoặc khả năng giao tiếp với ngôn ngữ
hoặc phần cứng khác nơi mà các đảm bảo của Rust không áp dụng.

Listing 20-1 cho thấy cách tạo một con trỏ bất biến và một con trỏ có thể thay
đổi.

<Listing number="20-1" caption="Tạo con trỏ thô với các toán tử mượn thô">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-01/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta không đưa từ khóa `unsafe` vào mã này. Chúng ta có thể tạo
con trỏ thô trong mã an toàn; chúng ta chỉ không thể giải tham chiếu con trỏ thô
bên ngoài một khối unsafe, như bạn sẽ thấy sau đây.

Chúng ta đã tạo con trỏ thô bằng cách sử dụng các toán tử mượn thô:
`&raw const num` tạo một con trỏ thô bất biến `*const i32`, và `&raw mut num`
tạo một con trỏ thô có thể thay đổi `*mut i32`. Bởi vì chúng ta tạo chúng trực
tiếp từ một biến cục bộ, chúng ta biết các con trỏ thô cụ thể này là hợp lệ,
nhưng chúng ta không thể đưa ra giả định đó về bất kỳ con trỏ thô nào.

Để minh họa điều này, tiếp theo chúng ta sẽ tạo một con trỏ thô mà tính hợp lệ
của nó chúng ta không thể chắc chắn, sử dụng từ khóa `as` để ép kiểu một giá trị
thay vì sử dụng toán tử mượn thô. Listing 20-2 cho thấy cách tạo một con trỏ thô
đến một vị trí bộ nhớ tùy ý. Việc cố gắng sử dụng bộ nhớ tùy ý là không xác
định: có thể có dữ liệu tại địa chỉ đó hoặc có thể không, trình biên dịch có thể
tối ưu hóa mã để không có truy cập bộ nhớ, hoặc chương trình có thể kết thúc với
lỗi phân đoạn. Thông thường, không có lý do tốt để viết mã như thế này, đặc biệt
là trong các trường hợp mà bạn có thể sử dụng toán tử mượn thô thay thế, nhưng
điều đó là có thể.

<Listing number="20-2" caption="Tạo một con trỏ thô đến một địa chỉ bộ nhớ tùy ý">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-02/src/main.rs:here}}
```

</Listing>

Nhớ rằng chúng ta có thể tạo con trỏ thô trong mã an toàn, nhưng chúng ta không
thể _giải tham chiếu_ con trỏ thô và đọc dữ liệu đang được trỏ đến. Trong
Listing 20-3, chúng ta sử dụng toán tử giải tham chiếu `*` trên một con trỏ thô
đòi hỏi một khối `unsafe`.

<Listing number="20-3" caption="Giải tham chiếu con trỏ thô trong một khối `unsafe`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-03/src/main.rs:here}}
```

</Listing>

Việc tạo một con trỏ không gây hại gì; chỉ khi chúng ta cố gắng truy cập giá trị
mà nó trỏ đến, chúng ta mới có thể phải đối mặt với một giá trị không hợp lệ.

Cũng lưu ý rằng trong Listings 20-1 và 20-3, chúng ta đã tạo các con trỏ thô
`*const i32` và `*mut i32` đều trỏ đến cùng một vị trí bộ nhớ, nơi `num` được
lưu trữ. Nếu thay vào đó, chúng ta cố gắng tạo một tham chiếu bất biến và một
tham chiếu có thể thay đổi đến `num`, mã sẽ không biên dịch được vì các quy tắc
sở hữu của Rust không cho phép một tham chiếu có thể thay đổi cùng lúc với bất
kỳ tham chiếu bất biến nào. Với con trỏ thô, chúng ta có thể tạo một con trỏ có
thể thay đổi và một con trỏ bất biến đến cùng một vị trí và thay đổi dữ liệu
thông qua con trỏ có thể thay đổi, có khả năng tạo ra cuộc đua dữ liệu. Hãy cẩn
thận!

Với tất cả những nguy hiểm này, tại sao bạn lại sử dụng con trỏ thô? Một trường
hợp sử dụng chính là khi giao tiếp với mã C, như bạn sẽ thấy trong phần tiếp
theo. Một trường hợp khác là khi xây dựng các trừu tượng an toàn mà trình kiểm
tra mượn không hiểu. Chúng ta sẽ giới thiệu các hàm unsafe và sau đó xem xét một
ví dụ về một trừu tượng an toàn sử dụng mã unsafe.

### Gọi Hàm hoặc Phương Thức Unsafe

Loại hoạt động thứ hai mà bạn có thể thực hiện trong một khối unsafe là gọi các
hàm unsafe. Các hàm và phương thức unsafe trông giống như các hàm và phương thức
thông thường, nhưng chúng có thêm `unsafe` trước phần còn lại của định nghĩa. Từ
khóa `unsafe` trong bối cảnh này chỉ ra rằng hàm có các yêu cầu mà chúng ta cần
duy trì khi gọi hàm này, bởi vì Rust không thể đảm bảo rằng chúng ta đã đáp ứng
các yêu cầu này. Bằng cách gọi một hàm unsafe trong một khối `unsafe`, chúng ta
đang nói rằng chúng ta đã đọc tài liệu của hàm này và chúng ta chịu trách nhiệm
duy trì các hợp đồng của hàm.

Đây là một hàm unsafe có tên `dangerous` không làm gì trong thân hàm:

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/no-listing-01-unsafe-fn/src/main.rs:here}}
```

Chúng ta phải gọi hàm `dangerous` trong một khối `unsafe` riêng biệt. Nếu chúng
ta cố gắng gọi `dangerous` mà không có khối `unsafe`, chúng ta sẽ gặp lỗi:

```console
{{#include ../listings/ch20-advanced-features/output-only-01-missing-unsafe/output.txt}}
```

Với khối `unsafe`, chúng ta đang khẳng định với Rust rằng chúng ta đã đọc tài
liệu của hàm, chúng ta hiểu cách sử dụng nó đúng cách và chúng ta đã xác minh
rằng chúng ta đang thực hiện hợp đồng của hàm.

Để thực hiện các hoạt động unsafe trong thân của một hàm `unsafe`, bạn vẫn cần
sử dụng một khối `unsafe`, giống như trong một hàm thông thường, và trình biên
dịch sẽ cảnh báo bạn nếu bạn quên. Điều này giúp chúng ta giữ cho các khối
`unsafe` càng nhỏ càng tốt, vì các hoạt động unsafe có thể không cần thiết trên
toàn bộ thân hàm.

#### Tạo một Trừu Tượng An Toàn trên Mã Unsafe

Chỉ vì một hàm chứa mã unsafe không có nghĩa là chúng ta cần đánh dấu toàn bộ
hàm là unsafe. Trên thực tế, việc bao bọc mã unsafe trong một hàm an toàn là một
trừu tượng phổ biến. Ví dụ, hãy nghiên cứu hàm `split_at_mut` từ thư viện tiêu
chuẩn, hàm này yêu cầu một số mã unsafe. Chúng ta sẽ khám phá cách chúng ta có
thể triển khai nó. Phương thức an toàn này được định nghĩa trên các slice có thể
thay đổi: nó lấy một slice và tạo ra hai slice bằng cách chia slice tại chỉ số
được đưa vào dưới dạng đối số. Listing 20-4 cho thấy cách sử dụng
`split_at_mut`.

<Listing number="20-4" caption="Sử dụng hàm `split_at_mut` an toàn">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-04/src/main.rs:here}}
```

</Listing>

Chúng ta không thể triển khai hàm này chỉ bằng Rust an toàn. Một nỗ lực có thể
trông giống như Listing 20-5, nhưng sẽ không biên dịch được. Để đơn giản, chúng
ta sẽ triển khai `split_at_mut` dưới dạng một hàm thay vì một phương thức và chỉ
cho các slice của giá trị `i32` thay vì cho một kiểu chung `T`.

<Listing number="20-5" caption="Một nỗ lực triển khai `split_at_mut` chỉ sử dụng Rust an toàn">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-05/src/main.rs:here}}
```

</Listing>

Hàm này đầu tiên lấy tổng độ dài của slice. Sau đó, nó khẳng định rằng chỉ số
được đưa vào dưới dạng tham số nằm trong slice bằng cách kiểm tra xem nó có nhỏ
hơn hoặc bằng độ dài hay không. Sự khẳng định có nghĩa là nếu chúng ta truyền
một chỉ số lớn hơn độ dài để chia slice, hàm sẽ panic trước khi nó cố gắng sử
dụng chỉ số đó.

Sau đó, chúng ta trả về hai slice có thể thay đổi trong một tuple: một từ đầu
slice ban đầu đến chỉ số `mid` và một từ `mid` đến cuối slice.

Khi chúng ta cố gắng biên dịch mã trong Listing 20-5, chúng ta sẽ gặp lỗi:

```console
{{#include ../listings/ch20-advanced-features/listing-20-05/output.txt}}
```

Trình kiểm tra mượn của Rust không thể hiểu rằng chúng ta đang mượn các phần
khác nhau của slice; nó chỉ biết rằng chúng ta đang mượn từ cùng một slice hai
lần. Việc mượn các phần khác nhau của một slice về cơ bản là ổn vì hai slice
không chồng chéo nhau, nhưng Rust không đủ thông minh để biết điều này. Khi
chúng ta biết mã là ổn, nhưng Rust không biết, đó là lúc chúng ta cần sử dụng mã
unsafe.

Listing 20-6 cho thấy cách sử dụng khối `unsafe`, con trỏ thô và một số lệnh gọi
đến các hàm unsafe để làm cho việc triển khai `split_at_mut` hoạt động.

<Listing number="20-6" caption="Sử dụng mã unsafe trong việc triển khai hàm `split_at_mut`">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-06/src/main.rs:here}}
```

</Listing>

Nhớ lại từ ["Kiểu Slice"][the-slice-type]<!-- ignore --> trong Chương 4 rằng một
slice là một con trỏ đến một số dữ liệu và độ dài của slice. Chúng ta sử dụng
phương thức `len` để lấy độ dài của một slice và phương thức `as_mut_ptr` để
truy cập con trỏ thô của một slice. Trong trường hợp này, vì chúng ta có một
slice có thể thay đổi cho các giá trị `i32`, `as_mut_ptr` trả về một con trỏ thô
với kiểu `*mut i32`, mà chúng ta đã lưu trữ trong biến `ptr`.

Chúng ta giữ khẳng định rằng chỉ số `mid` nằm trong slice. Sau đó, chúng ta đi
đến mã unsafe: hàm `slice::from_raw_parts_mut` nhận một con trỏ thô và một độ
dài, và nó tạo ra một slice. Chúng ta sử dụng hàm này để tạo một slice bắt đầu
từ `ptr` và dài `mid` phần tử. Sau đó, chúng ta gọi phương thức `add` trên `ptr`
với `mid` là đối số để có được một con trỏ thô bắt đầu tại `mid`, và chúng ta
tạo một slice sử dụng con trỏ đó và số phần tử còn lại sau `mid` làm độ dài.

Hàm `slice::from_raw_parts_mut` là unsafe vì nó nhận một con trỏ thô và phải tin
rằng con trỏ này là hợp lệ. Phương thức `add` trên các con trỏ thô cũng là
unsafe vì nó phải tin rằng vị trí offset cũng là một con trỏ hợp lệ. Do đó,
chúng ta phải đặt một khối `unsafe` xung quanh các lệnh gọi của chúng ta đến
`slice::from_raw_parts_mut` và `add` để chúng ta có thể gọi chúng. Bằng cách xem
xét mã và thêm khẳng định rằng `mid` phải nhỏ hơn hoặc bằng `len`, chúng ta có
thể nói rằng tất cả các con trỏ thô được sử dụng trong khối `unsafe` sẽ là các
con trỏ hợp lệ đến dữ liệu trong slice. Đây là một cách sử dụng `unsafe` có thể
chấp nhận và thích hợp.

Lưu ý rằng chúng ta không cần phải đánh dấu hàm `split_at_mut` kết quả là
`unsafe`, và chúng ta có thể gọi hàm này từ Rust an toàn. Chúng ta đã tạo một
trừu tượng an toàn cho mã unsafe với một triển khai của hàm sử dụng mã `unsafe`
theo cách an toàn, bởi vì nó chỉ tạo các con trỏ hợp lệ từ dữ liệu mà hàm này có
quyền truy cập.

Ngược lại, việc sử dụng `slice::from_raw_parts_mut` trong Listing 20-7 có thể sẽ
gây crash khi slice được sử dụng. Mã này lấy một vị trí bộ nhớ tùy ý và tạo một
slice dài 10.000 phần tử.

<Listing number="20-7" caption="Tạo một slice từ một vị trí bộ nhớ tùy ý">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-07/src/main.rs:here}}
```

</Listing>

Chúng ta không sở hữu bộ nhớ tại vị trí tùy ý này, và không có đảm bảo rằng
slice mà mã này tạo ra chứa các giá trị `i32` hợp lệ. Cố gắng sử dụng `values`
như thể nó là một slice hợp lệ dẫn đến hành vi không xác định.

#### Sử dụng Hàm `extern` để Gọi Mã Bên Ngoài

Đôi khi mã Rust của bạn có thể cần tương tác với mã được viết bằng một ngôn ngữ
khác. Đối với điều này, Rust có từ khóa `extern` tạo điều kiện cho việc tạo và
sử dụng một _Giao Diện Hàm Ngoại (FFI)_, là một cách để một ngôn ngữ lập trình
định nghĩa các hàm và cho phép một ngôn ngữ lập trình khác (ngoại) gọi các hàm
đó.

Listing 20-8 minh họa cách thiết lập tích hợp với hàm `abs` từ thư viện tiêu
chuẩn C. Các hàm được khai báo trong các khối `extern` thường không an toàn khi
gọi từ mã Rust, vì vậy các khối `extern` cũng phải được đánh dấu là `unsafe`. Lý
do là vì các ngôn ngữ khác không thực thi các quy tắc và đảm bảo của Rust, và
Rust không thể kiểm tra chúng, vì vậy trách nhiệm thuộc về lập trình viên để đảm
bảo an toàn.

<Listing number="20-8" file-name="src/main.rs" caption="Khai báo và gọi một hàm `extern` được định nghĩa trong một ngôn ngữ khác">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-08/src/main.rs}}
```

</Listing>

Trong khối `unsafe extern "C"`, chúng ta liệt kê tên và chữ ký của các hàm bên
ngoài từ một ngôn ngữ khác mà chúng ta muốn gọi. Phần `"C"` định nghĩa _giao
diện nhị phân ứng dụng (ABI)_ mà hàm bên ngoài sử dụng: ABI định nghĩa cách gọi
hàm ở cấp độ assembly. ABI `"C"` là phổ biến nhất và tuân theo ABI của ngôn ngữ
lập trình C. Thông tin về tất cả các ABI mà Rust hỗ trợ có sẵn trong [Tài liệu
tham khảo Rust][ABI].

Mỗi mục được khai báo trong một khối `unsafe extern` đều được ngầm hiểu là
`unsafe`. Tuy nhiên, một số hàm FFI _thực sự_ an toàn để gọi. Ví dụ, hàm `abs`
từ thư viện tiêu chuẩn của C không có bất kỳ cân nhắc an toàn bộ nhớ nào và
chúng ta biết nó có thể được gọi với bất kỳ giá trị `i32` nào. Trong những
trường hợp như thế này, chúng ta có thể sử dụng từ khóa `safe` để nói rằng hàm
cụ thể này an toàn để gọi ngay cả khi nó nằm trong một khối `unsafe extern`. Sau
khi chúng ta thực hiện thay đổi đó, việc gọi nó không còn yêu cầu một khối
`unsafe`, như thể hiện trong Listing 20-9.

<Listing number="20-9" file-name="src/main.rs" caption="Đánh dấu một hàm là `safe` một cách rõ ràng trong một khối `unsafe extern` và gọi nó một cách an toàn">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-09/src/main.rs}}
```

</Listing>

Việc đánh dấu một hàm là `safe` không tự nhiên làm cho nó an toàn! Thay vào đó,
nó giống như một lời hứa mà bạn đang tạo với Rust rằng nó _là_ an toàn. Vẫn là
trách nhiệm của bạn để đảm bảo lời hứa đó được giữ!

> #### Gọi Hàm Rust từ Các Ngôn Ngữ Khác
>
> Chúng ta cũng có thể sử dụng `extern` để tạo một giao diện cho phép các ngôn
> ngữ khác gọi các hàm Rust. Thay vì tạo một khối `extern` hoàn chỉnh, chúng ta
> thêm từ khóa `extern` và chỉ định ABI để sử dụng ngay trước từ khóa `fn` cho
> hàm liên quan. Chúng ta cũng cần thêm chú thích `#[unsafe(no_mangle)]` để nói
> với trình biên dịch Rust không làm rối tên của hàm này. _Làm rối_ là khi một
> trình biên dịch thay đổi tên mà chúng ta đã đặt cho một hàm thành một tên khác
> chứa thêm thông tin cho các phần khác của quá trình biên dịch sử dụng nhưng ít
> dễ đọc hơn đối với con người. Mỗi trình biên dịch ngôn ngữ lập trình làm rối
> tên hơi khác nhau, vì vậy để một hàm Rust có thể được đặt tên bởi các ngôn ngữ
> khác, chúng ta phải vô hiệu hóa việc làm rối tên của trình biên dịch Rust.
> Điều này là không an toàn vì có thể có xung đột tên giữa các thư viện mà không
> có cơ chế làm rối tên tích hợp, vì vậy đó là trách nhiệm của chúng ta để đảm
> bảo tên mà chúng ta chọn là an toàn để xuất mà không làm rối.
>
> Trong ví dụ sau, chúng ta làm cho hàm `call_from_c` có thể truy cập từ mã C,
> sau khi nó được biên dịch thành một thư viện chia sẻ và được liên kết từ C:
>
> ```rust
> #[unsafe(no_mangle)]
> pub extern "C" fn call_from_c() {
>     println!("Just called a Rust function from C!");
> }
> ```
>
> Cách sử dụng `extern` này chỉ yêu cầu `unsafe` trong thuộc tính, không phải
> trên khối `extern`.

### Truy Cập hoặc Sửa Đổi một Biến Tĩnh Có Thể Thay Đổi

Trong sách này, chúng ta vẫn chưa nói về các biến toàn cục, mà Rust có hỗ trợ
nhưng có thể gây ra vấn đề với các quy tắc sở hữu của Rust. Nếu hai luồng đang
truy cập cùng một biến toàn cục có thể thay đổi, nó có thể gây ra đua dữ liệu.

Trong Rust, các biến toàn cục được gọi là _biến tĩnh_. Listing 20-10 cho thấy
một ví dụ về khai báo và sử dụng một biến tĩnh với một slice chuỗi làm giá trị.

<Listing number="20-10" file-name="src/main.rs" caption="Định nghĩa và sử dụng một biến tĩnh bất biến">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-10/src/main.rs}}
```

</Listing>

Các biến tĩnh tương tự như các hằng số, mà chúng ta đã thảo luận trong ["Hằng
Số"][differences-between-variables-and-constants]<!-- ignore --> trong Chương 3.
Tên của các biến tĩnh là `SCREAMING_SNAKE_CASE` theo quy ước. Các biến tĩnh chỉ
có thể lưu trữ các tham chiếu với thời gian sống `'static`, có nghĩa là trình
biên dịch Rust có thể tính toán thời gian sống và chúng ta không cần phải chú
thích nó một cách rõ ràng. Truy cập một biến tĩnh bất biến là an toàn.

Một sự khác biệt tinh tế giữa các hằng số và các biến tĩnh bất biến là các giá
trị trong một biến tĩnh có một địa chỉ cố định trong bộ nhớ. Việc sử dụng giá
trị sẽ luôn truy cập cùng một dữ liệu. Các hằng số, mặt khác, được phép nhân bản
dữ liệu của chúng bất cứ khi nào chúng được sử dụng. Một sự khác biệt khác là
các biến tĩnh có thể có thể thay đổi. Truy cập và sửa đổi các biến tĩnh có thể
thay đổi là _không an toàn_. Listing 20-11 cho thấy cách khai báo, truy cập và
sửa đổi một biến tĩnh có thể thay đổi có tên là `COUNTER`.

<Listing number="20-11" file-name="src/main.rs" caption="Đọc từ hoặc viết vào một biến tĩnh có thể thay đổi là không an toàn">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-11/src/main.rs}}
```

</Listing>

Giống như với các biến thông thường, chúng ta chỉ định khả năng thay đổi bằng từ
khóa `mut`. Bất kỳ mã nào đọc hoặc ghi từ `COUNTER` phải nằm trong một khối
`unsafe`. Mã này biên dịch và in `COUNTER: 3` như chúng ta mong đợi vì nó là đơn
luồng. Việc có nhiều luồng truy cập `COUNTER` có thể dẫn đến đua dữ liệu, vì vậy
nó là hành vi không xác định. Do đó, chúng ta cần đánh dấu toàn bộ hàm là
`unsafe`, và ghi chú giới hạn an toàn, để bất kỳ ai gọi hàm đều biết những gì họ
được và không được phép làm một cách an toàn.

Bất cứ khi nào chúng ta viết một hàm unsafe, theo quy ước, chúng ta nên viết một
bình luận bắt đầu bằng `SAFETY` và giải thích những gì người gọi cần làm để gọi
hàm một cách an toàn. Tương tự, bất cứ khi nào chúng ta thực hiện một hoạt động
unsafe, theo quy ước, chúng ta nên viết một bình luận bắt đầu bằng `SAFETY` để
giải thích cách các quy tắc an toàn được duy trì.

Ngoài ra, trình biên dịch sẽ không cho phép bạn tạo các tham chiếu đến một biến
tĩnh có thể thay đổi. Bạn chỉ có thể truy cập nó thông qua một con trỏ thô, được
tạo bằng một trong các toán tử mượn thô. Điều đó bao gồm cả trong các trường hợp
khi tham chiếu được tạo một cách vô hình, như khi nó được sử dụng trong
`println!` trong mã này. Yêu cầu rằng các tham chiếu đến các biến tĩnh có thể
thay đổi chỉ có thể được tạo thông qua các con trỏ thô giúp làm cho các yêu cầu
an toàn để sử dụng chúng trở nên rõ ràng hơn.

Với dữ liệu có thể thay đổi mà có thể truy cập toàn cục, khó để đảm bảo rằng
không có đua dữ liệu, đó là lý do tại sao Rust coi các biến tĩnh có thể thay đổi
là không an toàn. Khi có thể, tốt hơn là sử dụng các kỹ thuật đồng thời và các
con trỏ thông minh an toàn với luồng mà chúng ta đã thảo luận trong Chương 16 để
trình biên dịch kiểm tra rằng việc truy cập dữ liệu từ các luồng khác nhau được
thực hiện một cách an toàn.

### Triển Khai một Trait Unsafe

Chúng ta có thể sử dụng `unsafe` để triển khai một trait unsafe. Một trait là
unsafe khi ít nhất một trong các phương thức của nó có một bất biến mà trình
biên dịch không thể xác minh. Chúng ta khai báo rằng một trait là `unsafe` bằng
cách thêm từ khóa `unsafe` trước `trait` và đánh dấu việc triển khai trait cũng
là `unsafe`, như thể hiện trong Listing 20-12.

<Listing number="20-12" caption="Định nghĩa và triển khai một trait unsafe">

```rust
{{#rustdoc_include ../listings/ch20-advanced-features/listing-20-12/src/main.rs:here}}
```

</Listing>

Bằng cách sử dụng `unsafe impl`, chúng ta đang hứa rằng chúng ta sẽ duy trì các
bất biến mà trình biên dịch không thể xác minh.

Ví dụ, nhớ lại các trait đánh dấu `Sync` và `Send` mà chúng ta đã thảo luận
trong ["Khả năng mở rộng đồng thời với các Trait `Sync` và
`Send`"][extensible-concurrency-with-the-sync-and-send-traits]<!-- ignore -->
trong Chương 16: trình biên dịch triển khai các trait này tự động nếu các kiểu
của chúng ta được tạo thành hoàn toàn từ các kiểu khác triển khai `Send` và
`Sync`. Nếu chúng ta triển khai một kiểu chứa một kiểu không triển khai `Send`
hoặc `Sync`, chẳng hạn như con trỏ thô, và chúng ta muốn đánh dấu kiểu đó là
`Send` hoặc `Sync`, chúng ta phải sử dụng `unsafe`. Rust không thể xác minh rằng
kiểu của chúng ta duy trì các đảm bảo rằng nó có thể được gửi an toàn qua các
luồng hoặc truy cập từ nhiều luồng; do đó, chúng ta cần thực hiện các kiểm tra
đó thủ công và chỉ ra như vậy với `unsafe`.

### Truy Cập các Trường của một Union

Hành động cuối cùng chỉ hoạt động với `unsafe` là truy cập các trường của một
union. Một `union` tương tự như một `struct`, nhưng chỉ một trường được khai báo
được sử dụng trong một phiên bản cụ thể tại một thời điểm. Unions chủ yếu được
sử dụng để giao tiếp với các unions trong mã C. Truy cập các trường của union là
unsafe vì Rust không thể đảm bảo kiểu của dữ liệu hiện đang được lưu trữ trong
phiên bản union. Bạn có thể tìm hiểu thêm về unions trong [Tài liệu tham khảo
Rust][unions].

### Sử Dụng Miri để Kiểm Tra Mã Unsafe

Khi viết mã unsafe, bạn có thể muốn kiểm tra rằng những gì bạn đã viết thực sự
là an toàn và chính xác. Một trong những cách tốt nhất để làm điều đó là sử dụng
Miri, một công cụ Rust chính thức để phát hiện hành vi không xác định. Trong khi
trình kiểm tra mượn là một công cụ _tĩnh_ hoạt động tại thời điểm biên dịch,
Miri là một công cụ _động_ hoạt động tại thời điểm chạy. Nó kiểm tra mã của bạn
bằng cách chạy chương trình của bạn, hoặc bộ kiểm tra của nó, và phát hiện khi
bạn vi phạm các quy tắc mà nó hiểu về cách Rust nên hoạt động.

Sử dụng Miri yêu cầu một bản dựng nightly của Rust (mà chúng ta nói thêm trong
[Phụ lục G: Rust được tạo ra như thế nào và "Rust Nightly"][nightly]). Bạn có
thể cài đặt cả phiên bản nightly của Rust và công cụ Miri bằng cách nhập
`rustup +nightly component add miri`. Điều này không thay đổi phiên bản Rust mà
dự án của bạn sử dụng; nó chỉ thêm công cụ vào hệ thống của bạn để bạn có thể sử
dụng nó khi muốn. Bạn có thể chạy Miri trên một dự án bằng cách nhập
`cargo +nightly miri run` hoặc `cargo +nightly miri test`.

Ví dụ về việc điều này có thể hữu ích như thế nào, hãy xem điều gì xảy ra khi
chúng ta chạy nó với Listing 20-7.

```console
{{#include ../listings/ch20-advanced-features/listing-20-07/output.txt}}
```

Miri chính xác cảnh báo chúng ta rằng chúng ta đang ép kiểu một số nguyên thành
một con trỏ, điều này có thể là một vấn đề nhưng Miri không thể phát hiện nếu có
vì nó không biết con trỏ bắt nguồn từ đâu. Sau đó, Miri trả về một lỗi trong đó
Listing 20-7 có hành vi không xác định vì chúng ta có một con trỏ lơ lửng. Nhờ
Miri, chúng ta bây giờ biết có rủi ro về hành vi không xác định, và chúng ta có
thể suy nghĩ về cách làm cho mã an toàn. Trong một số trường hợp, Miri thậm chí
có thể đưa ra các khuyến nghị về cách khắc phục lỗi.

Miri không bắt tất cả mọi thứ mà bạn có thể làm sai khi viết mã unsafe. Miri là
một công cụ phân tích động, vì vậy nó chỉ bắt các vấn đề với mã thực sự được
chạy. Điều đó có nghĩa là bạn sẽ cần sử dụng nó kết hợp với các kỹ thuật kiểm
tra tốt để tăng sự tự tin về mã unsafe mà bạn đã viết. Miri cũng không bao gồm
mọi cách có thể mà mã của bạn có thể không đúng.

Nói cách khác: Nếu Miri _phát hiện_ một vấn đề, bạn biết có một lỗi, nhưng chỉ
vì Miri _không_ bắt được lỗi không có nghĩa là không có vấn đề. Tuy nhiên, nó có
thể bắt được nhiều lỗi. Hãy thử chạy nó trên các ví dụ khác về mã unsafe trong
chương này và xem nó nói gì!

Bạn có thể tìm hiểu thêm về Miri tại [kho lưu trữ GitHub của nó][miri].

### Khi Nào Nên Sử Dụng Mã Unsafe

Việc sử dụng `unsafe` để sử dụng một trong năm siêu năng lực vừa thảo luận không
sai hoặc thậm chí không bị coi là xấu, nhưng nó phức tạp hơn để có được mã
`unsafe` chính xác vì trình biên dịch không thể giúp duy trì an toàn bộ nhớ. Khi
bạn có lý do để sử dụng mã `unsafe`, bạn có thể làm như vậy, và việc có chú
thích `unsafe` rõ ràng giúp dễ dàng theo dõi nguồn gốc của các vấn đề khi chúng
xảy ra. Bất cứ khi nào bạn viết mã unsafe, bạn có thể sử dụng Miri để giúp bạn
tự tin hơn rằng mã bạn đã viết tuân theo các quy tắc của Rust.

Để khám phá sâu hơn về cách làm việc hiệu quả với unsafe Rust, hãy đọc hướng dẫn
chính thức của Rust về chủ đề này, [Rustonomicon][nomicon].

[dangling-references]: ch04-02-references-and-borrowing.html#dangling-references
[ABI]: ../reference/items/external-blocks.html#abi
[differences-between-variables-and-constants]:
  ch03-01-variables-and-mutability.html#constants
[extensible-concurrency-with-the-sync-and-send-traits]:
  ch16-04-extensible-concurrency-sync-and-send.html#extensible-concurrency-with-the-sync-and-send-traits
[the-slice-type]: ch04-03-slices.html#the-slice-type
[unions]: ../reference/items/unions.html
[miri]: https://github.com/rust-lang/miri
[editions]: appendix-05-editions.html
[nightly]: appendix-07-nightly-rust.html
[nomicon]: https://doc.rust-lang.org/nomicon/
