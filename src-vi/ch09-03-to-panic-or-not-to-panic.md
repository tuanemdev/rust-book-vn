## `panic!` Hay Không `panic!`

Vậy làm thế nào để quyết định khi nào nên gọi `panic!` và khi nào nên trả về
`Result`? Khi code panic, không có cách nào để phục hồi. Bạn có thể gọi `panic!`
cho bất kỳ tình huống lỗi nào, dù có cách phục hồi hay không, nhưng như vậy bạn
đang đưa ra quyết định rằng một tình huống là không thể khôi phục thay mặt cho
mã gọi. Khi bạn chọn trả về giá trị `Result`, bạn đang cung cấp cho mã gọi các
lựa chọn. Mã gọi có thể chọn thử phục hồi theo cách phù hợp với tình huống của
nó, hoặc có thể quyết định rằng giá trị `Err` trong trường hợp này là không thể
khôi phục, vì vậy nó có thể gọi `panic!` và biến lỗi có thể khôi phục thành
không thể khôi phục. Do đó, việc trả về `Result` là lựa chọn mặc định tốt khi
bạn định nghĩa một hàm có thể thất bại.

Trong các tình huống như ví dụ, mã nguyên mẫu và kiểm thử, thì việc viết mã
panic thay vì trả về `Result` là phù hợp hơn. Hãy tìm hiểu lý do tại sao, sau đó
thảo luận về các tình huống mà trình biên dịch không thể biết thất bại là không
thể, nhưng bạn với tư cách là con người thì có thể biết. Chương này sẽ kết thúc
với một số hướng dẫn chung về cách quyết định liệu có nên panic trong mã thư
viện hay không.

### Ví dụ, Mã Nguyên mẫu và Kiểm thử

Khi bạn đang viết một ví dụ để minh họa một khái niệm nào đó, việc thêm vào mã
xử lý lỗi mạnh mẽ có thể làm cho ví dụ kém rõ ràng hơn. Trong các ví dụ, mọi
người hiểu rằng việc gọi một phương thức như `unwrap` có thể panic là nhằm làm
giữ chỗ cho cách bạn muốn ứng dụng của mình xử lý lỗi, điều này có thể khác nhau
dựa trên những gì phần còn lại của mã bạn đang làm.

Tương tự, các phương thức `unwrap` và `expect` rất tiện lợi khi tạo nguyên mẫu,
trước khi bạn sẵn sàng quyết định cách xử lý lỗi. Chúng để lại các dấu hiệu rõ
ràng trong mã của bạn cho khi bạn sẵn sàng làm cho chương trình của mình mạnh mẽ
hơn.

Nếu một lệnh gọi phương thức thất bại trong một bài kiểm thử, bạn muốn toàn bộ
bài kiểm thử thất bại, ngay cả khi phương thức đó không phải là chức năng đang
được kiểm thử. Bởi vì `panic!` là cách mà một bài kiểm thử được đánh dấu là thất
bại, việc gọi `unwrap` hoặc `expect` chính xác là điều nên xảy ra.

### Các trường hợp mà bạn có nhiều thông tin hơn trình biên dịch

Việc gọi `expect` cũng thích hợp khi bạn có một số logic khác đảm bảo rằng
`Result` sẽ có giá trị `Ok`, nhưng logic đó không phải là điều mà trình biên
dịch hiểu được. Bạn vẫn sẽ có một giá trị `Result` mà bạn cần xử lý: bất kỳ hoạt
động nào bạn đang gọi vẫn có khả năng thất bại nói chung, mặc dù về mặt logic
điều đó là không thể trong tình huống cụ thể của bạn. Nếu bạn có thể đảm bảo
bằng cách kiểm tra mã theo cách thủ công rằng bạn sẽ không bao giờ có biến thể
`Err`, thì hoàn toàn có thể chấp nhận được việc gọi `expect`, và thậm chí tốt
hơn là ghi lại lý do tại sao bạn nghĩ rằng bạn sẽ không bao giờ có biến thể
`Err` trong văn bản đối số. Đây là một ví dụ:

```rust
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-08-unwrap-that-cant-fail/src/main.rs:here}}
```

Chúng ta đang tạo một thể hiện `IpAddr` bằng cách phân tích một chuỗi được mã
hóa cứng. Chúng ta có thể thấy rằng `127.0.0.1` là một địa chỉ IP hợp lệ, vì vậy
việc sử dụng `expect` là chấp nhận được ở đây. Tuy nhiên, việc có một chuỗi hợp
lệ được mã hóa cứng không thay đổi kiểu trả về của phương thức `parse`: chúng ta
vẫn nhận được một giá trị `Result`, và trình biên dịch sẽ vẫn yêu cầu chúng ta
xử lý `Result` như thể biến thể `Err` là một khả năng bởi vì trình biên dịch
không đủ thông minh để thấy rằng chuỗi này luôn là một địa chỉ IP hợp lệ. Nếu
chuỗi địa chỉ IP đến từ người dùng thay vì được mã hóa cứng vào chương trình và
do đó _có_ khả năng thất bại, chúng ta chắc chắn sẽ muốn xử lý `Result` bằng một
cách mạnh mẽ hơn. Việc đề cập đến giả định rằng địa chỉ IP này được mã hóa cứng
sẽ nhắc nhở chúng ta thay đổi `expect` thành mã xử lý lỗi tốt hơn nếu trong
tương lai, chúng ta cần lấy địa chỉ IP từ một nguồn khác thay vì mã hóa cứng.

### Hướng dẫn cho việc xử lý lỗi

Nên cho phép code của bạn panic khi có khả năng code của bạn có thể kết thúc
trong một trạng thái xấu. Trong ngữ cảnh này, một _trạng thái xấu_ là khi một số
giả định, đảm bảo, hợp đồng, hoặc bất biến đã bị phá vỡ, chẳng hạn như khi các
giá trị không hợp lệ, giá trị mâu thuẫn nhau, hoặc giá trị bị thiếu được truyền
vào code của bạn—cộng với một hoặc nhiều điều kiện sau:

- Trạng thái xấu là điều không mong đợi, trái ngược với điều gì đó có thể xảy ra
  thỉnh thoảng, như người dùng nhập dữ liệu sai định dạng.
- Code của bạn sau điểm này cần dựa vào việc không ở trong trạng thái xấu này,
  thay vì kiểm tra vấn đề ở từng bước.
- Không có cách tốt để mã hóa thông tin này trong các kiểu dữ liệu bạn sử dụng.
  Chúng ta sẽ nghiên cứu một ví dụ về ý nghĩa của điều này trong ["Mã hóa Trạng
  thái và Hành vi như Kiểu dữ liệu"][encoding]<!-- ignore --> trong Chương 18.

Nếu ai đó gọi code của bạn và truyền vào các giá trị không hợp lý, tốt nhất là
trả về một lỗi nếu bạn có thể để người dùng thư viện có thể quyết định họ muốn
làm gì trong trường hợp đó. Tuy nhiên, trong các trường hợp mà việc tiếp tục có
thể không an toàn hoặc có hại, lựa chọn tốt nhất có thể là gọi `panic!` và cảnh
báo người sử dụng thư viện của bạn về lỗi trong code của họ để họ có thể sửa nó
trong quá trình phát triển. Tương tự, `panic!` thường là lựa chọn phù hợp nếu
bạn đang gọi code bên ngoài mà bạn không kiểm soát được và nó trả về một trạng
thái không hợp lệ mà bạn không có cách nào để sửa chữa.

Tuy nhiên, khi thất bại là điều được mong đợi, thì việc trả về `Result` sẽ phù
hợp hơn là gọi `panic!`. Ví dụ bao gồm một trình phân tích cú pháp được cung cấp
dữ liệu không đúng định dạng hoặc một yêu cầu HTTP trả về trạng thái chỉ ra rằng
bạn đã đạt đến giới hạn tỷ lệ. Trong những trường hợp này, việc trả về `Result`
cho biết rằng thất bại là một khả năng được mong đợi mà mã gọi phải quyết định
cách xử lý.

Khi code của bạn thực hiện một hoạt động có thể gây rủi ro cho người dùng nếu nó
được gọi bằng các giá trị không hợp lệ, code của bạn nên xác minh các giá trị
hợp lệ trước và panic nếu các giá trị không hợp lệ. Điều này chủ yếu là vì lý do
an toàn: cố gắng hoạt động trên dữ liệu không hợp lệ có thể làm lộ code của bạn
trước các lỗ hổng. Đây là lý do chính khiến thư viện tiêu chuẩn sẽ gọi `panic!`
nếu bạn cố gắng truy cập bộ nhớ ngoài giới hạn: việc cố gắng truy cập bộ nhớ
không thuộc về cấu trúc dữ liệu hiện tại là một vấn đề bảo mật phổ biến. Các hàm
thường có _hợp đồng_: hành vi của chúng chỉ được đảm bảo nếu đầu vào đáp ứng các
yêu cầu cụ thể. Việc panic khi hợp đồng bị vi phạm là hợp lý vì vi phạm hợp đồng
luôn chỉ ra lỗi ở phía gọi, và không phải là loại lỗi mà bạn muốn mã gọi phải xử
lý một cách rõ ràng. Trên thực tế, không có cách hợp lý nào để mã gọi có thể
khôi phục; _lập trình viên_ gọi cần sửa mã. Các hợp đồng cho một hàm, đặc biệt
là khi vi phạm sẽ gây ra panic, nên được giải thích trong tài liệu API cho hàm
đó.

Tuy nhiên, việc có nhiều kiểm tra lỗi trong tất cả các hàm của bạn sẽ dài dòng
và phiền toái. May mắn thay, bạn có thể sử dụng hệ thống kiểu của Rust (và do đó
việc kiểm tra kiểu được thực hiện bởi trình biên dịch) để thực hiện nhiều kiểm
tra cho bạn. Nếu hàm của bạn có một kiểu cụ thể làm tham số, bạn có thể tiếp tục
với logic mã của bạn, biết rằng trình biên dịch đã đảm bảo bạn có một giá trị
hợp lệ. Ví dụ, nếu bạn có một kiểu thay vì một `Option`, chương trình của bạn
mong đợi có _cái gì đó_ chứ không phải _không có gì_. Mã của bạn sau đó không
phải xử lý hai trường hợp cho các biến thể `Some` và `None`: nó sẽ chỉ có một
trường hợp cho việc chắc chắn có một giá trị. Mã cố gắng truyền không vào hàm
của bạn sẽ thậm chí không biên dịch được, vì vậy hàm của bạn không phải kiểm tra
trường hợp đó trong thời gian chạy. Một ví dụ khác là sử dụng kiểu số nguyên
không dấu như `u32`, điều này đảm bảo tham số không bao giờ âm.

### Tạo kiểu tùy chỉnh để xác thực

Hãy phát triển ý tưởng sử dụng hệ thống kiểu của Rust để đảm bảo chúng ta có một
giá trị hợp lệ thêm một bước nữa và xem xét việc tạo một kiểu tùy chỉnh để xác
thực. Hãy nhớ lại trò chơi đoán số ở Chương 2 trong đó mã của chúng ta yêu cầu
người dùng đoán một số từ 1 đến 100. Chúng ta chưa bao giờ xác thực rằng số đoán
của người dùng nằm giữa các số đó trước khi kiểm tra nó với số bí mật của chúng
ta; chúng ta chỉ xác thực rằng số đoán là dương. Trong trường hợp này, hậu quả
không quá nghiêm trọng: kết quả "Quá cao" hoặc "Quá thấp" của chúng ta vẫn sẽ
chính xác. Nhưng sẽ là một cải tiến hữu ích để hướng dẫn người dùng đưa ra các
đoán hợp lệ và có các hành vi khác nhau khi người dùng đoán một số ngoài phạm vi
so với khi người dùng nhập, ví dụ, các chữ cái thay vì số.

Một cách để làm điều này là phân tích số đoán dưới dạng `i32` thay vì chỉ là
`u32` để cho phép các số âm tiềm năng, và sau đó thêm một kiểm tra xem số có nằm
trong phạm vi hay không, như sau:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-09-guess-out-of-range/src/main.rs:here}}
```

</Listing>

Biểu thức `if` kiểm tra xem giá trị của chúng ta có ngoài phạm vi không, thông
báo cho người dùng về vấn đề, và gọi `continue` để bắt đầu vòng lặp tiếp theo và
yêu cầu một đoán khác. Sau biểu thức `if`, chúng ta có thể tiếp tục với các so
sánh giữa `guess` và số bí mật, biết rằng `guess` là từ 1 đến 100.

Tuy nhiên, đây không phải là một giải pháp lý tưởng: nếu điều quan trọng tuyệt
đối là chương trình chỉ hoạt động trên các giá trị từ 1 đến 100, và nó có nhiều
hàm với yêu cầu này, việc có một kiểm tra như thế này trong mọi hàm sẽ là tẻ
nhạt (và có thể ảnh hưởng đến hiệu suất).

Thay vào đó, chúng ta có thể tạo một kiểu mới trong một module chuyên dụng và
đặt các xác thực vào một hàm để tạo một thể hiện của kiểu thay vì lặp lại các
xác thực ở khắp mọi nơi. Bằng cách này, sẽ an toàn cho các hàm sử dụng kiểu mới
trong chữ ký của chúng và tự tin sử dụng các giá trị chúng nhận được. Listing
9-13 cho thấy một cách để định nghĩa một kiểu `Guess` sẽ chỉ tạo một thể hiện
của `Guess` nếu hàm `new` nhận một giá trị từ 1 đến 100.

<Listing number="9-13" caption="Một kiểu `Guess` chỉ tiếp tục với các giá trị từ 1 đến 100" file-name="src/guessing_game.rs">

```rust
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-13/src/guessing_game.rs}}
```

</Listing>

Lưu ý rằng mã này trong *src/guessing_game.rs* phụ thuộc vào việc thêm khai báo
module `mod guessing_game;` trong *src/lib.rs* mà chúng ta chưa hiển thị ở đây.
Trong file của module mới này, chúng ta định nghĩa một struct trong module đó có
tên `Guess` có một trường có tên `value` chứa một `i32`. Đây là nơi số sẽ được
lưu trữ.

Sau đó chúng ta thực hiện một hàm liên kết có tên `new` trên `Guess` tạo các thể
hiện của giá trị `Guess`. Hàm `new` được định nghĩa có một tham số có tên
`value` kiểu `i32` và trả về một `Guess`. Mã trong thân hàm `new` kiểm tra
`value` để đảm bảo nó nằm trong khoảng từ 1 đến 100. Nếu `value` không vượt qua
bài kiểm tra này, chúng ta thực hiện lệnh gọi `panic!`, điều này sẽ cảnh báo lập
trình viên đang viết mã gọi rằng họ có lỗi cần sửa, vì việc tạo một `Guess` với
`value` nằm ngoài phạm vi này sẽ vi phạm hợp đồng mà `Guess::new` đang dựa vào.
Các điều kiện trong đó `Guess::new` có thể panic nên được thảo luận trong tài
liệu API hướng đến công chúng của nó; chúng ta sẽ đề cập đến các quy ước tài
liệu chỉ ra khả năng của một `panic!` trong tài liệu API mà bạn tạo trong
Chương 14. Nếu `value` vượt qua bài kiểm tra, chúng ta tạo một `Guess` mới với
trường `value` được thiết lập thành tham số `value` và trả về `Guess`.

Tiếp theo, chúng ta thực hiện một phương thức có tên `value` mượn `self`, không
có bất kỳ tham số nào khác, và trả về một `i32`. Loại phương thức này đôi khi
được gọi là _getter_ vì mục đích của nó là lấy một số dữ liệu từ các trường của
nó và trả về nó. Phương thức công khai này là cần thiết vì trường `value` của
cấu trúc `Guess` là private. Điều quan trọng là trường `value` phải private để
mã sử dụng cấu trúc `Guess` không được phép đặt `value` trực tiếp: mã bên ngoài
module `guessing_game` _phải_ sử dụng hàm `Guess::new` để tạo một thể hiện của
`Guess`, từ đó đảm bảo không có cách nào để `Guess` có một `value` chưa được
kiểm tra bởi các điều kiện trong hàm `Guess::new`.

Một hàm có tham số hoặc chỉ trả về số từ 1 đến 100 có thể khai báo trong chữ ký
của nó rằng nó nhận hoặc trả về một `Guess` thay vì một `i32` và sẽ không cần
thực hiện bất kỳ kiểm tra bổ sung nào trong nội dung của nó.

## Tóm tắt

Các tính năng xử lý lỗi của Rust được thiết kế để giúp bạn viết mã mạnh mẽ hơn.
Macro `panic!` báo hiệu rằng chương trình của bạn đang ở trạng thái mà nó không
thể xử lý và cho phép bạn yêu cầu quá trình dừng thay vì cố gắng tiếp tục với
các giá trị không hợp lệ hoặc không chính xác. Enum `Result` sử dụng hệ thống
kiểu của Rust để chỉ ra rằng các hoạt động có thể thất bại theo cách mà mã của
bạn có thể khôi phục từ đó. Bạn có thể sử dụng `Result` để nói với mã gọi mã của
bạn rằng nó cần xử lý khả năng thành công hoặc thất bại. Sử dụng `panic!` và
`Result` trong các tình huống thích hợp sẽ làm cho mã của bạn đáng tin cậy hơn
đối mặt với các vấn đề không thể tránh khỏi.

Bây giờ bạn đã thấy những cách hữu ích mà thư viện tiêu chuẩn sử dụng generics
với các enum `Option` và `Result`, chúng ta sẽ nói về cách thức hoạt động của
generics và cách bạn có thể sử dụng chúng trong mã của mình.

[encoding]:
  ch18-03-oo-design-patterns.html#encoding-states-and-behavior-as-types
