<!-- Old heading. Do not remove or links may break. -->

<a id="closures-anonymous-functions-that-can-capture-their-environment"></a>

## Closures: Các Hàm Ẩn Danh Có Thể Capture Môi Trường Của Chúng

Closures trong Rust là các hàm ẩn danh mà bạn có thể lưu trong một biến hoặc
truyền như đối số cho các hàm khác. Bạn có thể tạo closure tại một nơi và sau đó
gọi closure ở nơi khác để đánh giá nó trong một ngữ cảnh khác. Không giống như
các hàm thông thường, closures có thể capture (nắm bắt) các giá trị từ phạm vi
mà chúng được định nghĩa. Chúng ta sẽ chứng minh cách các tính năng của closure
cho phép tái sử dụng mã và tùy chỉnh hành vi.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-an-abstraction-of-behavior-with-closures"></a>
<a id="refactoring-using-functions"></a>
<a id="refactoring-with-closures-to-store-code"></a>

### Capture Môi Trường với Closures

Đầu tiên, chúng ta sẽ xem xét cách chúng ta có thể sử dụng closures để capture
các giá trị từ môi trường mà chúng được định nghĩa để sử dụng sau này. Đây là
kịch bản: thỉnh thoảng, công ty áo thun của chúng ta tặng một chiếc áo độc
quyền, phiên bản giới hạn cho ai đó trong danh sách gửi thư của chúng ta như một
khuyến mãi. Những người trong danh sách gửi thư có thể tùy chọn thêm màu yêu
thích của họ vào hồ sơ của họ. Nếu người được chọn để nhận áo miễn phí đã đặt
màu yêu thích của họ, họ sẽ nhận được áo có màu đó. Nếu người đó chưa chỉ định
màu yêu thích, họ sẽ nhận được bất kỳ màu nào mà công ty hiện có nhiều nhất.

Có nhiều cách để thực hiện điều này. Đối với ví dụ này, chúng ta sẽ sử dụng một
enum gọi là `ShirtColor` có các biến thể `Red` và `Blue` (giới hạn số lượng màu
có sẵn để đơn giản hóa). Chúng ta biểu diễn kho hàng của công ty bằng một struct
`Inventory` có một trường tên là `shirts` chứa một `Vec<ShirtColor>` đại diện
cho các màu áo hiện có trong kho. Phương thức `giveaway` được định nghĩa trên
`Inventory` lấy tùy chọn màu áo ưa thích của người thắng áo miễn phí và trả về
màu áo mà người đó sẽ nhận được. Thiết lập này được hiển thị trong Listing 13-1.

<Listing number="13-1" file-name="src/main.rs" caption="Tình huống tặng áo của công ty áo">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-01/src/main.rs}}
```

</Listing>

`store` được định nghĩa trong `main` có hai áo màu xanh và một áo màu đỏ còn lại
để phân phối cho khuyến mãi phiên bản giới hạn này. Chúng ta gọi phương thức
`giveaway` cho một người dùng có sở thích cho áo màu đỏ và một người dùng không
có bất kỳ sở thích nào.

Một lần nữa, mã này có thể được triển khai theo nhiều cách, và ở đây, để tập
trung vào closures, chúng ta đã gắn bó với các khái niệm bạn đã học, ngoại trừ
phần thân của phương thức `giveaway` sử dụng một closure. Trong phương thức
`giveaway`, chúng ta nhận sở thích của người dùng như một tham số kiểu
`Option<ShirtColor>` và gọi phương thức `unwrap_or_else` trên `user_preference`.
Phương thức [`unwrap_or_else` trên `Option<T>`][unwrap-or-else]<!-- ignore -->
được định nghĩa bởi thư viện tiêu chuẩn. Nó lấy một đối số: một closure không có
đối số nào trả về một giá trị `T` (cùng kiểu được lưu trữ trong biến thể `Some`
của `Option<T>`, trong trường hợp này là `ShirtColor`). Nếu `Option<T>` là biến
thể `Some`, `unwrap_or_else` trả về giá trị từ bên trong `Some`. Nếu `Option<T>`
là biến thể `None`, `unwrap_or_else` gọi closure và trả về giá trị được trả về
bởi closure.

Chúng ta chỉ định biểu thức closure `|| self.most_stocked()` làm đối số cho
`unwrap_or_else`. Đây là một closure không có tham số (nếu closure có tham số,
chúng sẽ xuất hiện giữa hai dấu gạch đứng). Phần thân của closure gọi
`self.most_stocked()`. Chúng ta đang định nghĩa closure ở đây, và việc triển
khai của `unwrap_or_else` sẽ đánh giá closure sau này nếu kết quả là cần thiết.

Chạy mã này in ra:

```console
{{#include ../listings/ch13-functional-features/listing-13-01/output.txt}}
```

Một khía cạnh thú vị ở đây là chúng ta đã truyền một closure gọi
`self.most_stocked()` trên phiên bản `Inventory` hiện tại. Thư viện tiêu chuẩn
không cần biết bất cứ điều gì về các kiểu `Inventory` hoặc `ShirtColor` mà chúng
ta đã định nghĩa, hoặc logic mà chúng ta muốn sử dụng trong kịch bản này.
Closure capture một tham chiếu bất biến đến phiên bản `self` `Inventory` và
truyền nó cùng với mã mà chúng ta chỉ định cho phương thức `unwrap_or_else`.
Ngược lại, các hàm không thể capture môi trường của chúng theo cách này.

### Suy Luận Kiểu và Chú Thích Closure

Có nhiều sự khác biệt giữa các hàm và closures. Closures thường không yêu cầu
bạn phải chú thích kiểu của các tham số hoặc giá trị trả về như các hàm `fn`
làm. Chú thích kiểu được yêu cầu trên các hàm vì các kiểu là một phần của giao
diện rõ ràng được hiển thị cho người dùng của bạn. Định nghĩa giao diện này một
cách nghiêm ngặt là quan trọng để đảm bảo rằng mọi người đồng ý về các kiểu giá
trị mà một hàm sử dụng và trả về. Ngược lại, closures không được sử dụng trong
một giao diện hiển thị như thế này: chúng được lưu trữ trong các biến và được sử
dụng mà không cần đặt tên chúng và hiển thị chúng cho người dùng của thư viện
chúng ta.

Closures thường ngắn gọn và chỉ liên quan trong một ngữ cảnh hẹp thay vì trong
bất kỳ tình huống tùy ý nào. Trong các ngữ cảnh hạn chế này, trình biên dịch có
thể suy ra các kiểu của tham số và kiểu trả về, tương tự như cách nó có thể suy
ra kiểu của hầu hết các biến (có những trường hợp hiếm khi trình biên dịch cần
chú thích kiểu closure).

Giống như với các biến, chúng ta có thể thêm chú thích kiểu nếu chúng ta muốn
tăng sự rõ ràng và rõ ràng với chi phí là dài dòng hơn mức cần thiết. Chú thích
các kiểu cho một closure sẽ trông giống như định nghĩa được hiển thị trong
Listing 13-2. Trong ví dụ này, chúng ta đang định nghĩa một closure và lưu trữ
nó trong một biến thay vì định nghĩa closure ở vị trí chúng ta truyền nó làm một
đối số, như chúng ta đã làm trong Listing 13-1.

<Listing number="13-2" file-name="src/main.rs" caption="Thêm chú thích kiểu tùy chọn cho tham số và kiểu giá trị trả về trong closure">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-02/src/main.rs:here}}
```

</Listing>

Với chú thích kiểu được thêm vào, cú pháp của closures trông giống hơn với cú
pháp của các hàm. Ở đây, chúng ta định nghĩa một hàm thêm 1 vào tham số của nó
và một closure có cùng hành vi, để so sánh. Chúng ta đã thêm một số khoảng trắng
để căn chỉnh các phần liên quan. Điều này minh họa cách cú pháp closure tương tự
với cú pháp hàm ngoại trừ việc sử dụng dấu gạch đứng và lượng cú pháp là tùy
chọn:

```rust,ignore
fn  add_one_v1   (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1  ;
```

Dòng đầu tiên hiển thị định nghĩa hàm và dòng thứ hai hiển thị một định nghĩa
closure được chú thích đầy đủ. Trong dòng thứ ba, chúng ta loại bỏ các chú thích
kiểu từ định nghĩa closure. Trong dòng thứ tư, chúng ta loại bỏ các dấu ngoặc
nhọn, là tùy chọn vì thân closure chỉ có một biểu thức. Tất cả đều là các định
nghĩa hợp lệ sẽ tạo ra cùng một hành vi khi chúng được gọi. Các dòng
`add_one_v3` và `add_one_v4` yêu cầu các closure phải được đánh giá để có thể
biên dịch vì các kiểu sẽ được suy ra từ việc sử dụng chúng. Điều này tương tự
như `let v = Vec::new();` cần chú thích kiểu hoặc giá trị của một số kiểu được
chèn vào `Vec` để Rust có thể suy ra kiểu.

Đối với các định nghĩa closure, trình biên dịch sẽ suy ra một kiểu cụ thể cho
mỗi tham số và cho giá trị trả về của chúng. Ví dụ, Listing 13-3 hiển thị định
nghĩa của một closure ngắn chỉ trả về giá trị mà nó nhận được như một tham số.
Closure này không hữu ích lắm ngoại trừ mục đích của ví dụ này. Lưu ý rằng chúng
ta chưa thêm bất kỳ chú thích kiểu nào cho định nghĩa. Vì không có chú thích
kiểu, chúng ta có thể gọi closure với bất kỳ kiểu nào, điều mà chúng ta đã làm ở
đây với `String` lần đầu tiên. Nếu sau đó chúng ta thử gọi `example_closure` với
một số nguyên, chúng ta sẽ gặp lỗi.

<Listing number="13-3" file-name="src/main.rs" caption="Cố gắng gọi closure có kiểu được suy ra với hai kiểu khác nhau">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-03/src/main.rs:here}}
```

</Listing>

Trình biên dịch đưa ra lỗi này:

```console
{{#include ../listings/ch13-functional-features/listing-13-03/output.txt}}
```

Lần đầu tiên chúng ta gọi `example_closure` với giá trị `String`, trình biên
dịch suy ra kiểu của `x` và kiểu trả về của closure là `String`. Những kiểu đó
sau đó được khóa vào closure trong `example_closure`, và chúng ta nhận được một
lỗi kiểu khi chúng ta tiếp theo cố gắng sử dụng một kiểu khác với cùng một
closure.

### Capture Tham Chiếu hoặc Di Chuyển Quyền Sở Hữu

Closures có thể capture các giá trị từ môi trường của chúng theo ba cách, điều
này trực tiếp ánh xạ đến ba cách mà một hàm có thể lấy một tham số: mượn bất
biến, mượn khả biến và lấy quyền sở hữu. Closure sẽ quyết định cái nào trong số
này để sử dụng dựa trên những gì phần thân của hàm làm với các giá trị được
capture.

Trong Listing 13-4, chúng ta định nghĩa một closure capture một tham chiếu bất
biến đến vector có tên `list` vì nó chỉ cần một tham chiếu bất biến để in giá
trị.

<Listing number="13-4" file-name="src/main.rs" caption="Định nghĩa và gọi một closure capture một tham chiếu bất biến">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-04/src/main.rs}}
```

</Listing>

Ví dụ này cũng minh họa rằng một biến có thể liên kết với một định nghĩa
closure, và sau đó chúng ta có thể gọi closure bằng cách sử dụng tên biến và dấu
ngoặc đơn như thể tên biến là tên hàm.

Bởi vì chúng ta có thể có nhiều tham chiếu bất biến đến `list` cùng một lúc,
`list` vẫn có thể truy cập được từ mã trước khi định nghĩa closure, sau khi định
nghĩa closure nhưng trước khi closure được gọi, và sau khi closure được gọi. Mã
này biên dịch, chạy và in:

```console
{{#include ../listings/ch13-functional-features/listing-13-04/output.txt}}
```

Tiếp theo, trong Listing 13-5, chúng ta thay đổi phần thân closure để nó thêm
một phần tử vào vector `list`. Closure bây giờ capture một tham chiếu khả biến.

<Listing number="13-5" file-name="src/main.rs" caption="Định nghĩa và gọi một closure capture một tham chiếu khả biến">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-05/src/main.rs}}
```

</Listing>

Mã này biên dịch, chạy và in:

```console
{{#include ../listings/ch13-functional-features/listing-13-05/output.txt}}
```

Lưu ý rằng không còn `println!` giữa định nghĩa và lệnh gọi của closure
`borrows_mutably`: khi `borrows_mutably` được định nghĩa, nó capture một tham
chiếu khả biến đến `list`. Chúng ta không sử dụng closure nữa sau khi closure
được gọi, vì vậy việc mượn khả biến kết thúc. Giữa định nghĩa closure và lệnh
gọi closure, một tham chiếu bất biến để in không được phép vì không có tham
chiếu khác nào được phép khi có một tham chiếu khả biến. Thử thêm một `println!`
ở đó để xem bạn nhận được thông báo lỗi gì!

Nếu bạn muốn buộc closure lấy quyền sở hữu của các giá trị mà nó sử dụng trong
môi trường mặc dù phần thân của closure không cần thiết phải có quyền sở hữu,
bạn có thể sử dụng từ khóa `move` trước danh sách tham số.

Kỹ thuật này chủ yếu hữu ích khi truyền một closure cho một luồng mới để di
chuyển dữ liệu để nó được sở hữu bởi luồng mới. Chúng ta sẽ thảo luận về luồng
và tại sao bạn muốn sử dụng chúng chi tiết trong Chương 16 khi chúng ta nói về
đồng thời, nhưng bây giờ, hãy khám phá nhanh việc tạo một luồng mới bằng cách sử
dụng một closure cần từ khóa `move`. Listing 13-6 hiển thị Listing 13-4 đã được
sửa đổi để in vector trong một luồng mới thay vì trong luồng chính.

<Listing number="13-6" file-name="src/main.rs" caption="Sử dụng `move` để buộc closure cho luồng lấy quyền sở hữu của `list`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-06/src/main.rs}}
```

</Listing>

Chúng ta tạo ra một luồng mới, cung cấp cho luồng một closure để chạy như một
đối số. Phần thân closure in ra danh sách. Trong Listing 13-4, closure chỉ
capture `list` bằng cách sử dụng một tham chiếu bất biến vì đó là lượng truy cập
tối thiểu vào `list` cần thiết để in nó. Trong ví dụ này, mặc dù phần thân
closure vẫn chỉ cần một tham chiếu bất biến, chúng ta cần chỉ định rằng `list`
nên được di chuyển vào closure bằng cách đặt từ khóa `move` ở đầu định nghĩa
closure. Nếu luồng chính thực hiện nhiều hoạt động khác trước khi gọi `join`
trên luồng mới, luồng mới có thể kết thúc trước phần còn lại của luồng chính kết
thúc, hoặc luồng chính có thể kết thúc trước. Nếu luồng chính duy trì quyền sở
hữu của `list` nhưng kết thúc trước khi luồng mới và loại bỏ `list`, tham chiếu
bất biến trong luồng sẽ không hợp lệ. Do đó, trình biên dịch yêu cầu rằng `list`
phải được di chuyển vào closure được cung cấp cho luồng mới để tham chiếu sẽ hợp
lệ. Hãy thử loại bỏ từ khóa `move` hoặc sử dụng `list` trong luồng chính sau khi
closure được định nghĩa để xem bạn nhận được lỗi biên dịch gì!

<!-- Old headings. Do not remove or links may break. -->

<a id="storing-closures-using-generic-parameters-and-the-fn-traits"></a>
<a id="limitations-of-the-cacher-implementation"></a>
<a id="moving-captured-values-out-of-the-closure-and-the-fn-traits"></a>

### Di Chuyển Các Giá Trị Đã Capture Ra Khỏi Closures và Các Trait `Fn`

Một khi closure đã capture một tham chiếu hoặc capture quyền sở hữu của một giá
trị từ môi trường nơi closure được định nghĩa (do đó ảnh hưởng đến việc gì, nếu
có, được di chuyển _vào_ closure), mã trong phần thân của closure định nghĩa
những gì xảy ra với các tham chiếu hoặc giá trị khi closure được đánh giá sau đó
(do đó ảnh hưởng đến việc gì, nếu có, được di chuyển _ra khỏi_ closure).

Phần thân closure có thể thực hiện bất kỳ điều nào sau đây: di chuyển một giá
trị đã capture ra khỏi closure, thay đổi giá trị đã capture, không di chuyển
cũng không thay đổi giá trị, hoặc không capture gì từ môi trường từ ban đầu.

Cách mà một closure capture và xử lý các giá trị từ môi trường ảnh hưởng đến các
trait mà closure thực hiện, và trait là cách mà các hàm và struct có thể chỉ
định loại closures nào chúng có thể sử dụng. Closures sẽ tự động thực hiện một,
hai hoặc cả ba trait `Fn` này, theo cách cộng dồn, tùy thuộc vào cách mà phần
thân closure xử lý các giá trị:

- `FnOnce` áp dụng cho các closure có thể được gọi một lần. Tất cả các closure
  thực hiện ít nhất trait này vì tất cả các closure có thể được gọi. Một closure
  di chuyển các giá trị đã capture ra khỏi phần thân của nó sẽ chỉ thực hiện
  `FnOnce` và không phải các trait `Fn` khác, vì nó chỉ có thể được gọi một lần.
- `FnMut` áp dụng cho các closure không di chuyển các giá trị đã capture ra khỏi
  phần thân của chúng, nhưng có thể thay đổi các giá trị đã capture. Các closure
  này có thể được gọi nhiều lần.
- `Fn` áp dụng cho các closure không di chuyển các giá trị đã capture ra khỏi
  phần thân của chúng và không thay đổi các giá trị đã capture, cũng như các
  closure không capture gì từ môi trường của chúng. Các closure này có thể được
  gọi nhiều lần mà không thay đổi môi trường của chúng, điều này quan trọng
  trong các trường hợp như gọi một closure nhiều lần đồng thời.

Hãy xem xét định nghĩa của phương thức `unwrap_or_else` trên `Option<T>` mà
chúng ta đã sử dụng trong Listing 13-1:

```rust,ignore
impl<T> Option<T> {
    pub fn unwrap_or_else<F>(self, f: F) -> T
    where
        F: FnOnce() -> T
    {
        match self {
            Some(x) => x,
            None => f(),
        }
    }
}
```

Nhớ lại rằng `T` là kiểu generic đại diện cho kiểu của giá trị trong biến thể
`Some` của một `Option`. Kiểu `T` đó cũng là kiểu trả về của hàm
`unwrap_or_else`: mã gọi `unwrap_or_else` trên một `Option<String>`, ví dụ, sẽ
nhận được một `String`.

Tiếp theo, hãy lưu ý rằng hàm `unwrap_or_else` có tham số kiểu generic bổ sung
`F`. Kiểu `F` là kiểu của tham số có tên `f`, đó là closure mà chúng ta cung cấp
khi gọi `unwrap_or_else`.

Ràng buộc trait được chỉ định trên kiểu generic `F` là `FnOnce() -> T`, có nghĩa
là `F` phải có khả năng được gọi một lần, không nhận tham số, và trả về một `T`.
Sử dụng `FnOnce` trong ràng buộc trait thể hiện ràng buộc rằng `unwrap_or_else`
chỉ sẽ gọi `f` nhiều nhất một lần. Trong phần thân `unwrap_or_else`, chúng ta có
thể thấy rằng nếu `Option` là `Some`, `f` sẽ không được gọi. Nếu `Option` là
`None`, `f` sẽ được gọi một lần. Bởi vì tất cả các closure đều thực hiện
`FnOnce`, `unwrap_or_else` chấp nhận cả ba loại closure và linh hoạt nhất có
thể.

> Lưu ý: Nếu những gì chúng ta muốn làm không yêu cầu capture một giá trị từ môi
> trường, chúng ta có thể sử dụng tên của một hàm thay vì một closure ở nơi
> chúng ta cần thứ gì đó thực hiện một trong các trait `Fn`. Ví dụ, trên một giá
> trị `Option<Vec<T>>`, chúng ta có thể gọi `unwrap_or_else(Vec::new)` để nhận
> một vector mới, trống nếu giá trị là `None`. Trình biên dịch tự động thực hiện
> bất cứ trait `Fn` nào áp dụng được cho một định nghĩa hàm.

Bây giờ hãy xem xét phương thức thư viện tiêu chuẩn `sort_by_key`, được định
nghĩa trên slices, để xem nó khác với `unwrap_or_else` như thế nào và tại sao
`sort_by_key` sử dụng `FnMut` thay vì `FnOnce` cho ràng buộc trait. Closure nhận
một đối số dưới dạng một tham chiếu đến mục hiện tại trong slice đang được xem
xét, và trả về một giá trị kiểu `K` có thể được sắp xếp. Hàm này hữu ích khi bạn
muốn sắp xếp một slice theo một thuộc tính cụ thể của mỗi mục. Trong Listing
13-7, chúng ta có một danh sách các phiên bản `Rectangle` và chúng ta sử dụng
`sort_by_key` để sắp xếp chúng theo thuộc tính `width` của chúng từ thấp đến
cao.

<Listing number="13-7" file-name="src/main.rs" caption="Sử dụng `sort_by_key` để sắp xếp các hình chữ nhật theo chiều rộng">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-07/src/main.rs}}
```

</Listing>

Mã này in:

```console
{{#include ../listings/ch13-functional-features/listing-13-07/output.txt}}
```

Lý do `sort_by_key` được định nghĩa để lấy một closure `FnMut` là vì nó gọi
closure nhiều lần: một lần cho mỗi mục trong slice. Closure `|r| r.width` không
capture, thay đổi, hoặc di chuyển bất cứ thứ gì ra từ môi trường của nó, vì vậy
nó đáp ứng các yêu cầu ràng buộc trait.

Ngược lại, Listing 13-8 hiển thị một ví dụ về một closure chỉ thực hiện trait
`FnOnce`, vì nó di chuyển một giá trị ra khỏi môi trường. Trình biên dịch sẽ
không cho phép chúng ta sử dụng closure này với `sort_by_key`.

<Listing number="13-8" file-name="src/main.rs" caption="Cố gắng sử dụng một closure `FnOnce` với `sort_by_key`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-08/src/main.rs}}
```

</Listing>

Đây là một cách phức tạp và rắc rối (không hoạt động) để cố gắng đếm số lần
`sort_by_key` gọi closure khi sắp xếp `list`. Mã này cố gắng làm điều này bằng
cách đẩy `value`—một `String` từ môi trường của closure—vào vector
`sort_operations`. Closure capture `value` và sau đó di chuyển `value` ra khỏi
closure bằng cách chuyển quyền sở hữu của `value` cho vector `sort_operations`.
Closure này có thể được gọi một lần; cố gắng gọi nó lần thứ hai sẽ không hoạt
động vì `value` sẽ không còn trong môi trường để được đẩy vào `sort_operations`
nữa! Do đó, closure này chỉ thực hiện `FnOnce`. Khi chúng ta cố gắng biên dịch
mã này, chúng ta nhận được lỗi này rằng `value` không thể được di chuyển ra khỏi
closure vì closure phải thực hiện `FnMut`:

```console
{{#include ../listings/ch13-functional-features/listing-13-08/output.txt}}
```

Lỗi chỉ ra dòng trong phần thân closure di chuyển `value` ra khỏi môi trường. Để
sửa lỗi này, chúng ta cần thay đổi phần thân closure để nó không di chuyển các
giá trị ra khỏi môi trường. Giữ một bộ đếm trong môi trường và tăng giá trị của
nó trong phần thân closure là một cách trực tiếp hơn để đếm số lần closure được
gọi. Closure trong Listing 13-9 làm việc với `sort_by_key` vì nó chỉ capture một
tham chiếu khả biến đến bộ đếm `num_sort_operations` và do đó có thể được gọi
nhiều hơn một lần:

<Listing number="13-9" file-name="src/main.rs" caption="Sử dụng một closure `FnMut` với `sort_by_key` là được phép">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-09/src/main.rs}}
```

</Listing>

Các trait `Fn` là quan trọng khi định nghĩa hoặc sử dụng các hàm hoặc kiểu mà sử
dụng closures. Trong phần tiếp theo, chúng ta sẽ thảo luận về iterators. Nhiều
phương thức iterator lấy các đối số closure, vì vậy hãy ghi nhớ các chi tiết về
closure này khi chúng ta tiếp tục!

[unwrap-or-else]: ../std/option/enum.Option.html#method.unwrap_or_else
