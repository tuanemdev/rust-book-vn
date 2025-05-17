## `RefCell<T>` và Mẫu Khả Biến Nội Bộ

_Khả biến nội bộ_ (Interior mutability) là một mẫu thiết kế trong Rust cho phép
bạn thay đổi dữ liệu ngay cả khi có các tham chiếu bất biến đến dữ liệu đó;
thông thường, hành động này bị cấm bởi các quy tắc mượn. Để thay đổi dữ liệu,
mẫu này sử dụng mã `unsafe` bên trong cấu trúc dữ liệu để uốn cong các quy tắc
thông thường của Rust về việc thay đổi và mượn. Mã không an toàn chỉ ra cho
trình biên dịch rằng chúng ta đang kiểm tra các quy tắc thủ công thay vì dựa vào
trình biên dịch để kiểm tra chúng cho chúng ta; chúng ta sẽ thảo luận thêm về mã
không an toàn trong Chương 20.

Chúng ta chỉ có thể sử dụng các kiểu sử dụng mẫu khả biến nội bộ khi chúng ta có
thể đảm bảo rằng các quy tắc mượn sẽ được tuân thủ trong thời gian chạy, mặc dù
trình biên dịch không thể đảm bảo điều đó. Mã `unsafe` liên quan sau đó được bọc
trong một API an toàn, và kiểu bên ngoài vẫn bất biến.

Hãy khám phá khái niệm này bằng cách xem xét kiểu `RefCell<T>` theo mẫu khả biến
nội bộ.

### Thực Thi Quy Tắc Mượn trong Thời Gian Chạy với `RefCell<T>`

Không giống như `Rc<T>`, kiểu `RefCell<T>` đại diện cho quyền sở hữu duy nhất
đối với dữ liệu mà nó chứa. Vậy điều gì làm cho `RefCell<T>` khác với một kiểu
như `Box<T>`? Hãy nhớ lại các quy tắc mượn mà bạn đã học trong Chương 4:

- Tại bất kỳ thời điểm nào, bạn có thể có _hoặc là_ một tham chiếu có thể thay
  đổi hoặc bất kỳ số lượng tham chiếu bất biến nào (nhưng không phải cả hai).
- Tham chiếu phải luôn hợp lệ.

Với tham chiếu và `Box<T>`, các bất biến của quy tắc mượn được thực thi tại thời
điểm biên dịch. Với `RefCell<T>`, các bất biến này được thực thi _trong thời
gian chạy_. Với tham chiếu, nếu bạn vi phạm các quy tắc này, bạn sẽ gặp lỗi
trình biên dịch. Với `RefCell<T>`, nếu bạn vi phạm các quy tắc này, chương trình
của bạn sẽ hoảng loạn và thoát.

Ưu điểm của việc kiểm tra các quy tắc mượn tại thời điểm biên dịch là các lỗi sẽ
được phát hiện sớm hơn trong quá trình phát triển, và không có tác động đến hiệu
suất thời gian chạy vì tất cả phân tích đều được hoàn thành trước đó. Vì những
lý do đó, kiểm tra các quy tắc mượn tại thời điểm biên dịch là lựa chọn tốt nhất
trong đa số trường hợp, đó là lý do tại sao đây là mặc định của Rust.

Ưu điểm của việc kiểm tra các quy tắc mượn trong thời gian chạy thay vì thời
gian biên dịch là một số kịch bản an toàn bộ nhớ nhất định được cho phép, trong
khi chúng sẽ bị từ chối bởi các kiểm tra thời gian biên dịch. Phân tích tĩnh,
như trình biên dịch Rust, về bản chất là bảo thủ. Một số thuộc tính của mã không
thể phát hiện được bằng cách phân tích mã: ví dụ nổi tiếng nhất là Bài toán Dừng
(Halting Problem), nằm ngoài phạm vi của cuốn sách này nhưng là một chủ đề thú
vị để nghiên cứu.

Bởi vì một số phân tích là không thể, nếu trình biên dịch Rust không thể chắc
chắn rằng mã tuân thủ các quy tắc sở hữu, nó có thể từ chối một chương trình
đúng; theo cách này, nó là bảo thủ. Nếu Rust chấp nhận một chương trình không
chính xác, người dùng sẽ không thể tin tưởng vào các đảm bảo mà Rust đưa ra. Tuy
nhiên, nếu Rust từ chối một chương trình đúng, lập trình viên sẽ bị bất tiện,
nhưng không có gì thảm khốc có thể xảy ra. Kiểu `RefCell<T>` hữu ích khi bạn
chắc chắn rằng mã của bạn tuân theo các quy tắc mượn nhưng trình biên dịch không
thể hiểu và đảm bảo điều đó.

Tương tự như `Rc<T>`, `RefCell<T>` chỉ để sử dụng trong các tình huống đơn luồng
và sẽ đưa ra lỗi thời gian biên dịch nếu bạn cố gắng sử dụng nó trong bối cảnh
đa luồng. Chúng ta sẽ nói về cách lấy chức năng của `RefCell<T>` trong chương
trình đa luồng trong Chương 16.

Đây là một tóm tắt về lý do để chọn `Box<T>`, `Rc<T>`, hoặc `RefCell<T>`:

- `Rc<T>` cho phép nhiều chủ sở hữu của cùng một dữ liệu; `Box<T>` và
  `RefCell<T>` có chủ sở hữu duy nhất.
- `Box<T>` cho phép mượn bất biến hoặc có thể thay đổi được kiểm tra tại thời
  điểm biên dịch; `Rc<T>` chỉ cho phép mượn bất biến được kiểm tra tại thời điểm
  biên dịch; `RefCell<T>` cho phép mượn bất biến hoặc có thể thay đổi được kiểm
  tra trong thời gian chạy.
- Bởi vì `RefCell<T>` cho phép mượn có thể thay đổi được kiểm tra trong thời
  gian chạy, bạn có thể thay đổi giá trị bên trong `RefCell<T>` ngay cả khi
  `RefCell<T>` là bất biến.

Thay đổi giá trị bên trong một giá trị bất biến là mẫu _khả biến nội bộ_. Hãy
xem xét một tình huống mà khả biến nội bộ hữu ích và xem xét cách nó có thể thực
hiện được.

### Khả Biến Nội Bộ: Mượn Có Thể Thay Đổi cho một Giá Trị Bất Biến

Một hệ quả của các quy tắc mượn là khi bạn có một giá trị bất biến, bạn không
thể mượn nó một cách có thể thay đổi. Ví dụ, đoạn mã này sẽ không biên dịch:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/src/main.rs}}
```

Nếu bạn cố gắng biên dịch mã này, bạn sẽ gặp lỗi sau:

```console
{{#include ../listings/ch15-smart-pointers/no-listing-01-cant-borrow-immutable-as-mutable/output.txt}}
```

Tuy nhiên, có những tình huống mà sẽ rất hữu ích nếu một giá trị có thể tự thay
đổi bản thân trong các phương thức của nó nhưng xuất hiện bất biến đối với mã
khác. Mã bên ngoài các phương thức của giá trị sẽ không thể thay đổi giá trị. Sử
dụng `RefCell<T>` là một cách để có khả năng khả biến nội bộ, nhưng `RefCell<T>`
không hoàn toàn vượt qua các quy tắc mượn: trình kiểm tra mượn trong trình biên
dịch cho phép khả biến nội bộ này, và các quy tắc mượn được kiểm tra trong thời
gian chạy thay vì. Nếu bạn vi phạm các quy tắc, bạn sẽ nhận được một `panic!`
thay vì lỗi trình biên dịch.

Hãy làm việc thông qua một ví dụ thực tế mà chúng ta có thể sử dụng `RefCell<T>`
để thay đổi một giá trị bất biến và xem tại sao điều đó lại hữu ích.

#### Một Trường Hợp Sử Dụng cho Khả Biến Nội Bộ: Đối Tượng Giả Lập

Đôi khi trong quá trình kiểm thử, một lập trình viên sẽ sử dụng một kiểu thay
thế cho một kiểu khác, để quan sát hành vi cụ thể và khẳng định rằng nó được
triển khai chính xác. Kiểu thay thế này được gọi là _test double_. Hãy nghĩ về
nó theo nghĩa của một diễn viên đóng thế trong làm phim, nơi một người bước vào
và thay thế cho một diễn viên để thực hiện một cảnh đặc biệt khó khăn. Test
double đứng thay cho các kiểu khác khi chúng ta đang chạy kiểm thử. _Đối tượng
giả lập_ (Mock objects) là các loại test double cụ thể ghi lại những gì xảy ra
trong quá trình kiểm thử để bạn có thể khẳng định rằng các hành động đúng đã
diễn ra.

Rust không có đối tượng theo cùng nghĩa như các ngôn ngữ khác có đối tượng, và
Rust không có chức năng đối tượng giả lập được tích hợp vào thư viện chuẩn như
một số ngôn ngữ khác. Tuy nhiên, bạn chắc chắn có thể tạo một struct sẽ phục vụ
cùng mục đích như một đối tượng giả lập.

Đây là kịch bản chúng ta sẽ kiểm thử: chúng ta sẽ tạo một thư viện theo dõi một
giá trị so với một giá trị tối đa và gửi tin nhắn dựa trên mức độ gần với giá
trị tối đa của giá trị hiện tại. Thư viện này có thể được sử dụng để theo dõi
hạn ngạch của người dùng đối với số lượng cuộc gọi API mà họ được phép thực
hiện, ví dụ.

Thư viện của chúng ta sẽ chỉ cung cấp chức năng theo dõi mức độ gần với tối đa
của một giá trị và các tin nhắn nên là gì ở thời điểm nào. Các ứng dụng sử dụng
thư viện của chúng ta sẽ được kỳ vọng cung cấp cơ chế để gửi tin nhắn: ứng dụng
có thể đặt một tin nhắn trong ứng dụng, gửi email, gửi tin nhắn văn bản, hoặc
làm việc khác. Thư viện không cần biết chi tiết đó. Tất cả những gì nó cần là
một thứ triển khai một trait mà chúng ta sẽ cung cấp gọi là `Messenger`. Listing
15-20 hiển thị mã thư viện.

<Listing number="15-20" file-name="src/lib.rs" caption="Một thư viện để theo dõi mức độ gần của một giá trị với một giá trị tối đa và cảnh báo khi giá trị ở mức nhất định">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-20/src/lib.rs}}
```

</Listing>

Một phần quan trọng của mã này là trait `Messenger` có một phương thức gọi là
`send` nhận một tham chiếu bất biến đến `self` và văn bản của tin nhắn. Trait
này là giao diện mà đối tượng giả lập của chúng ta cần triển khai để có thể được
sử dụng theo cùng cách như một đối tượng thực. Phần quan trọng khác là chúng ta
muốn kiểm thử hành vi của phương thức `set_value` trên `LimitTracker`. Chúng ta
có thể thay đổi những gì chúng ta truyền vào cho tham số `value`, nhưng
`set_value` không trả về bất cứ thứ gì để chúng ta đưa ra khẳng định. Chúng ta
muốn có thể nói rằng nếu chúng ta tạo một `LimitTracker` với thứ gì đó triển
khai trait `Messenger` và một giá trị cụ thể cho `max`, khi chúng ta truyền các
số khác nhau cho `value`, messenger được yêu cầu gửi các tin nhắn thích hợp.

Chúng ta cần một đối tượng giả lập mà, thay vì gửi email hoặc tin nhắn văn bản
khi chúng ta gọi `send`, sẽ chỉ theo dõi các tin nhắn mà nó được yêu cầu gửi.
Chúng ta có thể tạo một thể hiện mới của đối tượng giả lập, tạo một
`LimitTracker` sử dụng đối tượng giả lập, gọi phương thức `set_value` trên
`LimitTracker`, và sau đó kiểm tra xem đối tượng giả lập có các tin nhắn mà
chúng ta mong đợi không. Listing 15-21 hiển thị một nỗ lực triển khai một đối
tượng giả lập để làm điều đó, nhưng trình kiểm tra mượn sẽ không cho phép.

<Listing number="15-21" file-name="src/lib.rs" caption="Một nỗ lực triển khai một `MockMessenger` không được phép bởi trình kiểm tra mượn">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-21/src/lib.rs:here}}
```

</Listing>

Mã kiểm thử này định nghĩa một struct `MockMessenger` có trường `sent_messages`
với một `Vec` của các giá trị `String` để theo dõi các tin nhắn mà nó được yêu
cầu gửi. Chúng ta cũng định nghĩa một hàm liên kết `new` để thuận tiện cho việc
tạo các giá trị `MockMessenger` mới bắt đầu với một danh sách tin nhắn trống.
Sau đó, chúng ta triển khai trait `Messenger` cho `MockMessenger` để chúng ta có
thể cung cấp một `MockMessenger` cho một `LimitTracker`. Trong định nghĩa của
phương thức `send`, chúng ta lấy tin nhắn được truyền vào dưới dạng một tham số
và lưu trữ nó trong danh sách `sent_messages` của `MockMessenger`.

Trong kiểm thử, chúng ta đang kiểm tra những gì xảy ra khi `LimitTracker` được
yêu cầu đặt `value` thành một cái gì đó lớn hơn 75 phần trăm của giá trị `max`.
Đầu tiên, chúng ta tạo một `MockMessenger` mới, sẽ bắt đầu với một danh sách tin
nhắn trống. Sau đó, chúng ta tạo một `LimitTracker` mới và cung cấp cho nó một
tham chiếu đến `MockMessenger` mới và một giá trị `max` là `100`. Chúng ta gọi
phương thức `set_value` trên `LimitTracker` với một giá trị `80`, lớn hơn 75
phần trăm của 100. Sau đó, chúng ta khẳng định rằng danh sách tin nhắn mà
`MockMessenger` đang theo dõi bây giờ nên có một tin nhắn trong đó.

Tuy nhiên, có một vấn đề với kiểm thử này, như được hiển thị ở đây:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-21/output.txt}}
```

Chúng ta không thể sửa đổi `MockMessenger` để theo dõi các tin nhắn, vì phương
thức `send` nhận một tham chiếu bất biến đến `self`. Chúng ta cũng không thể làm
theo gợi ý từ văn bản lỗi để sử dụng `&mut self` trong cả phương thức `impl` và
định nghĩa `trait`. Chúng ta không muốn thay đổi trait `Messenger` chỉ vì mục
đích kiểm thử. Thay vào đó, chúng ta cần tìm một cách để làm cho mã kiểm thử của
chúng ta hoạt động chính xác với thiết kế hiện tại của chúng ta.

Đây là một tình huống mà khả biến nội bộ có thể giúp đỡ! Chúng ta sẽ lưu trữ
`sent_messages` trong một `RefCell<T>`, và sau đó phương thức `send` sẽ có thể
sửa đổi `sent_messages` để lưu trữ các tin nhắn chúng ta đã thấy. Listing 15-22
hiển thị nó trông như thế nào.

<Listing number="15-22" file-name="src/lib.rs" caption="Sử dụng `RefCell<T>` để thay đổi một giá trị bên trong trong khi giá trị bên ngoài được coi là bất biến">

```rust,noplayground
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-22/src/lib.rs:here}}
```

</Listing>

Trường `sent_messages` bây giờ có kiểu `RefCell<Vec<String>>` thay vì
`Vec<String>`. Trong hàm `new`, chúng ta tạo một thể hiện `RefCell<Vec<String>>`
mới xung quanh vectơ trống.

Đối với việc triển khai phương thức `send`, tham số đầu tiên vẫn là một mượn bất
biến của `self`, phù hợp với định nghĩa trait. Chúng ta gọi `borrow_mut` trên
`RefCell<Vec<String>>` trong `self.sent_messages` để có được một tham chiếu có
thể thay đổi đến giá trị bên trong `RefCell<Vec<String>>`, đó là vectơ. Sau đó,
chúng ta có thể gọi `push` trên tham chiếu có thể thay đổi đến vectơ để theo dõi
các tin nhắn được gửi trong quá trình kiểm thử.

Thay đổi cuối cùng chúng ta phải thực hiện là trong khẳng định: để xem có bao
nhiêu mục trong vectơ bên trong, chúng ta gọi `borrow` trên
`RefCell<Vec<String>>` để có được một tham chiếu bất biến đến vectơ.

Bây giờ bạn đã thấy cách sử dụng `RefCell<T>`, hãy đi sâu vào cách nó hoạt động!

#### Theo dõi các Mượn tại Thời gian Chạy với `RefCell<T>`

Khi tạo tham chiếu bất biến và có thể thay đổi, chúng ta sử dụng cú pháp `&` và
`&mut`, tương ứng. Với `RefCell<T>`, chúng ta sử dụng các phương thức `borrow`
và `borrow_mut`, là một phần của API an toàn thuộc về `RefCell<T>`. Phương thức
`borrow` trả về kiểu con trỏ thông minh `Ref<T>`, và `borrow_mut` trả về kiểu
con trỏ thông minh `RefMut<T>`. Cả hai kiểu đều triển khai `Deref`, vì vậy chúng
ta có thể coi chúng như các tham chiếu thông thường.

`RefCell<T>` theo dõi có bao nhiêu con trỏ thông minh `Ref<T>` và `RefMut<T>`
đang hoạt động. Mỗi lần chúng ta gọi `borrow`, `RefCell<T>` tăng số lượng mượn
bất biến đang hoạt động. Khi giá trị `Ref<T>` ra khỏi phạm vi, số lượng mượn bất
biến giảm xuống 1. Giống như các quy tắc mượn thời gian biên dịch, `RefCell<T>`
cho phép chúng ta có nhiều mượn bất biến hoặc một mượn có thể thay đổi tại bất
kỳ thời điểm nào.

Nếu chúng ta cố gắng vi phạm các quy tắc này, thay vì nhận được lỗi trình biên
dịch như chúng ta sẽ làm với tham chiếu, việc triển khai `RefCell<T>` sẽ hoảng
loạn trong thời gian chạy. Listing 15-23 hiển thị một sửa đổi của việc triển
khai `send` trong Listing 15-22. Chúng ta cố tình cố gắng tạo hai mượn có thể
thay đổi hoạt động cho cùng một phạm vi để minh họa rằng `RefCell<T>` ngăn chúng
ta làm điều này trong thời gian chạy.

<Listing number="15-23" file-name="src/lib.rs" caption="Tạo hai tham chiếu có thể thay đổi trong cùng phạm vi để thấy rằng `RefCell<T>` sẽ hoảng loạn">

```rust,ignore,panics
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-23/src/lib.rs:here}}
```

</Listing>

Chúng ta tạo một biến `one_borrow` cho con trỏ thông minh `RefMut<T>` được trả
về từ `borrow_mut`. Sau đó, chúng ta tạo một mượn có thể thay đổi khác theo cùng
cách trong biến `two_borrow`. Điều này tạo ra hai tham chiếu có thể thay đổi
trong cùng một phạm vi, không được phép. Khi chúng ta chạy các kiểm thử cho thư
viện của mình, mã trong Listing 15-23 sẽ biên dịch mà không có bất kỳ lỗi nào,
nhưng kiểm thử sẽ thất bại:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-23/output.txt}}
```

Lưu ý rằng mã hoảng loạn với thông báo `already borrowed: BorrowMutError`. Đây
là cách `RefCell<T>` xử lý vi phạm các quy tắc mượn trong thời gian chạy.

Việc chọn để bắt các lỗi mượn trong thời gian chạy thay vì thời gian biên dịch,
như chúng ta đã làm ở đây, có nghĩa là bạn có khả năng tìm thấy lỗi trong mã của
mình sau này trong quá trình phát triển: có thể không phải cho đến khi mã của
bạn được triển khai lên môi trường sản xuất. Ngoài ra, mã của bạn sẽ phải chịu
một hình phạt hiệu suất thời gian chạy nhỏ do phải theo dõi các mượn trong thời
gian chạy thay vì thời gian biên dịch. Tuy nhiên, việc sử dụng `RefCell<T>` làm
cho việc viết một đối tượng giả lập có thể tự sửa đổi để theo dõi các tin nhắn
mà nó đã thấy khi bạn đang sử dụng nó trong một bối cảnh chỉ cho phép các giá
trị bất biến trở nên khả thi. Bạn có thể sử dụng `RefCell<T>` bất chấp sự đánh
đổi của nó để có được nhiều chức năng hơn so với các tham chiếu thông thường
cung cấp.

<!-- Old link, do not remove -->

<a id="having-multiple-owners-of-mutable-data-by-combining-rc-t-and-ref-cell-t"></a>

### Cho Phép Nhiều Chủ Sở Hữu của Dữ Liệu Có Thể Thay Đổi với `Rc<T>` và `RefCell<T>`

Một cách phổ biến để sử dụng `RefCell<T>` là kết hợp với `Rc<T>`. Hãy nhớ rằng
`Rc<T>` cho phép bạn có nhiều chủ sở hữu của một số dữ liệu, nhưng nó chỉ cung
cấp quyền truy cập bất biến vào dữ liệu đó. Nếu bạn có một `Rc<T>` chứa một
`RefCell<T>`, bạn có thể có được một giá trị có thể có nhiều chủ sở hữu _và_ mà
bạn có thể thay đổi!

Ví dụ, hãy nhớ lại ví dụ danh sách cons trong Listing 15-18 nơi chúng ta sử dụng
`Rc<T>` để cho phép nhiều danh sách chia sẻ quyền sở hữu của một danh sách khác.
Bởi vì `Rc<T>` chỉ chứa các giá trị bất biến, chúng ta không thể thay đổi bất kỳ
giá trị nào trong danh sách một khi chúng ta đã tạo chúng. Hãy thêm vào
`RefCell<T>` để có khả năng thay đổi các giá trị trong danh sách. Listing 15-24
hiển thị rằng bằng cách sử dụng `RefCell<T>` trong định nghĩa `Cons`, chúng ta
có thể sửa đổi giá trị được lưu trữ trong tất cả các danh sách.

<Listing number="15-24" file-name="src/main.rs" caption="Sử dụng `Rc<RefCell<i32>>` để tạo một `List` mà chúng ta có thể thay đổi">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-24/src/main.rs}}
```

</Listing>

Chúng ta tạo một giá trị là một thể hiện của `Rc<RefCell<i32>>` và lưu trữ nó
trong một biến có tên `value` để chúng ta có thể truy cập nó trực tiếp sau này.
Sau đó, chúng ta tạo một `List` trong `a` với một biến thể `Cons` chứa `value`.
Chúng ta cần sao chép `value` để cả `a` và `value` đều có quyền sở hữu của giá
trị bên trong `5` thay vì chuyển quyền sở hữu từ `value` sang `a` hoặc có `a`
mượn từ `value`.

Chúng ta bọc danh sách `a` trong một `Rc<T>` để khi chúng ta tạo danh sách `b`
và `c`, cả hai đều có thể tham chiếu đến `a`, đó là những gì chúng ta đã làm
trong Listing 15-18.

Sau khi chúng ta đã tạo danh sách trong `a`, `b`, và `c`, chúng ta muốn thêm 10
vào giá trị trong `value`. Chúng ta làm điều này bằng cách gọi `borrow_mut` trên
`value`, sử dụng tính năng tự động tham chiếu mà chúng ta đã thảo luận trong
["Where's the `->` Operator?"][wheres-the---operator]<!-- ignore -->) trong
Chương 5 để tham chiếu đến `Rc<T>` đến giá trị `RefCell<T>` bên trong. Phương
thức `borrow_mut` trả về một con trỏ thông minh `RefMut<T>`, và chúng ta sử dụng
toán tử tham chiếu trên nó và thay đổi giá trị bên trong.

Khi chúng ta in `a`, `b`, và `c`, chúng ta có thể thấy rằng tất cả chúng đều có
giá trị sửa đổi là `15` thay vì `5`:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-24/output.txt}}
```

Kỹ thuật này khá gọn gàng! Bằng cách sử dụng `RefCell<T>`, chúng ta có một giá
trị `List` bất biến bên ngoài. Nhưng chúng ta có thể sử dụng các phương thức
trên `RefCell<T>` cung cấp quyền truy cập vào khả biến nội bộ của nó để chúng ta
có thể sửa đổi dữ liệu của mình khi chúng ta cần. Các kiểm tra thời gian chạy
của các quy tắc mượn bảo vệ chúng ta khỏi các cuộc đua dữ liệu, và đôi khi đáng
để đánh đổi một chút tốc độ để có được sự linh hoạt này trong cấu trúc dữ liệu
của chúng ta. Lưu ý rằng `RefCell<T>` không hoạt động cho mã đa luồng!
`Mutex<T>` là phiên bản an toàn cho luồng của `RefCell<T>`, và chúng ta sẽ thảo
luận về `Mutex<T>` trong Chương 16.

[wheres-the---operator]: ch05-03-method-syntax.html#wheres-the---operator
