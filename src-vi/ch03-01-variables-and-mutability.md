## Biến và Tính Khả Biến

Như đã đề cập trong phần ["Lưu Trữ Giá Trị với
Biến"][storing-values-with-variables]<!-- ignore -->, theo mặc định, các biến là
không thể thay đổi (immutable). Đây là một trong nhiều sự hướng dẫn mà Rust cung
cấp cho bạn để viết code theo cách tận dụng lợi thế về tính an toàn và dễ dàng
trong xử lý đồng thời mà Rust mang lại. Tuy nhiên, bạn vẫn có tùy chọn để làm
cho các biến của mình có thể thay đổi (mutable). Hãy cùng khám phá cách thức và
lý do tại sao Rust khuyến khích bạn ưu tiên tính bất biến và lý do tại sao đôi
khi bạn có thể muốn lựa chọn khác.

Khi một biến là bất biến, sau khi một giá trị được gắn với một tên, bạn không
thể thay đổi giá trị đó. Để minh họa điều này, hãy tạo một dự án mới có tên là
_variables_ trong thư mục _projects_ của bạn bằng cách sử dụng
`cargo new variables`.

Sau đó, trong thư mục _variables_ mới của bạn, mở _src/main.rs_ và thay thế mã
của nó bằng đoạn mã sau, mã này sẽ chưa biên dịch được ngay:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/src/main.rs}}
```

Lưu và chạy chương trình bằng cách sử dụng `cargo run`. Bạn sẽ nhận được thông
báo lỗi liên quan đến lỗi bất biến, như được hiển thị trong kết quả này:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-01-variables-are-immutable/output.txt}}
```

Ví dụ này cho thấy cách trình biên dịch giúp bạn tìm ra lỗi trong các chương
trình của mình. Lỗi biên dịch có thể gây khó chịu, nhưng thực sự chúng chỉ có
nghĩa là chương trình của bạn chưa an toàn để thực hiện những gì bạn muốn; chúng
_không_ có nghĩa là bạn không phải là một lập trình viên giỏi! Ngay cả những
người dùng Rust có kinh nghiệm vẫn gặp lỗi biên dịch.

Bạn nhận được thông báo lỗi `` cannot assign twice to immutable variable `x` ``
bởi vì bạn đã cố gắn một giá trị thứ hai cho biến bất biến `x`.

Điều quan trọng là chúng ta gặp lỗi khi biên dịch khi chúng ta cố gắng thay đổi
một giá trị được chỉ định là bất biến bởi vì tình huống này có thể dẫn đến lỗi.
Nếu một phần trong code của chúng ta hoạt động dựa trên giả định rằng một giá
trị sẽ không bao giờ thay đổi và một phần khác của code thay đổi giá trị đó, thì
có thể phần đầu tiên của code sẽ không làm những gì nó được thiết kế để làm.
Nguyên nhân của loại lỗi này có thể khó theo dõi sau khi nó xảy ra, đặc biệt là
khi phần code thứ hai chỉ thay đổi giá trị _đôi khi_. Trình biên dịch của Rust
đảm bảo rằng khi bạn tuyên bố một giá trị sẽ không thay đổi, nó thực sự sẽ không
thay đổi, vì vậy bạn không phải tự theo dõi nó. Do đó, code của bạn dễ dàng suy
luận hơn.

Nhưng tính khả biến có thể rất hữu ích và có thể làm cho code trở nên thuận tiện
hơn khi viết. Mặc dù các biến là bất biến theo mặc định, bạn có thể làm cho
chúng trở nên khả biến bằng cách thêm `mut` trước tên biến như bạn đã làm trong
[Chương 2][storing-values-with-variables]<!-- ignore -->. Thêm `mut` cũng truyền
đạt ý định cho những người đọc code trong tương lai bằng cách chỉ ra rằng các
phần khác của code sẽ thay đổi giá trị của biến này.

Ví dụ, hãy thay đổi _src/main.rs_ thành như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/src/main.rs}}
```

Khi chúng ta chạy chương trình bây giờ, chúng ta sẽ nhận được:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-02-adding-mut/output.txt}}
```

Chúng ta được phép thay đổi giá trị gắn với `x` từ `5` thành `6` khi sử dụng
`mut`. Cuối cùng, quyết định sử dụng tính khả biến hay không tùy thuộc vào bạn
và phụ thuộc vào những gì bạn nghĩ là rõ ràng nhất trong tình huống cụ thể đó.

<!-- Old headings. Do not remove or links may break. -->
<a id="constants"></a>

### Khai báo Hằng số

Giống như các biến bất biến, _hằng số_ là các giá trị được gắn với một tên và
không được phép thay đổi, nhưng có một vài sự khác biệt giữa hằng số và biến.

Đầu tiên, bạn không được phép sử dụng `mut` với hằng số. Hằng số không chỉ bất
biến theo mặc định—chúng luôn bất biến. Bạn khai báo hằng số bằng từ khóa
`const` thay vì từ khóa `let`, và kiểu của giá trị _phải_ được chú thích. Chúng
ta sẽ đề cập đến các kiểu và chú thích kiểu trong phần tiếp theo, ["Kiểu Dữ
Liệu"][data-types]<!-- ignore -->, vì vậy đừng lo lắng về các chi tiết ngay bây
giờ. Chỉ cần biết rằng bạn phải luôn chú thích kiểu.

Hằng số có thể được khai báo trong bất kỳ phạm vi nào, bao gồm cả phạm vi toàn
cục, làm cho chúng hữu ích cho các giá trị mà nhiều phần của code cần biết.

Sự khác biệt cuối cùng là hằng số chỉ có thể được đặt thành một biểu thức hằng,
không phải là kết quả của một giá trị chỉ có thể được tính toán trong thời gian
chạy.

Đây là một ví dụ về khai báo hằng số:

```rust
const THREE_HOURS_IN_SECONDS: u32 = 60 * 60 * 3;
```

Tên của hằng số là `THREE_HOURS_IN_SECONDS` và giá trị của nó được đặt thành kết
quả của việc nhân 60 (số giây trong một phút) với 60 (số phút trong một giờ) với
3 (số giờ mà chúng ta muốn đếm trong chương trình này). Quy ước đặt tên của Rust
cho hằng số là sử dụng tất cả các chữ in hoa với dấu gạch dưới giữa các từ.
Trình biên dịch có thể đánh giá một tập hợp giới hạn các hoạt động tại thời điểm
biên dịch, cho phép chúng ta chọn cách viết giá trị này theo cách dễ hiểu và xác
minh hơn, thay vì đặt hằng số này thành giá trị 10.800. Xem [phần đánh giá hằng
số trong Rust Reference][const-eval] để biết thêm thông tin về các hoạt động có
thể được sử dụng khi khai báo hằng số.

Hằng số có hiệu lực trong suốt thời gian chương trình chạy, trong phạm vi mà
chúng được khai báo. Thuộc tính này làm cho hằng số hữu ích cho các giá trị
trong lĩnh vực ứng dụng của bạn mà nhiều phần của chương trình có thể cần biết,
chẳng hạn như số điểm tối đa mà bất kỳ người chơi nào của trò chơi được phép
kiếm được, hoặc tốc độ ánh sáng.

Việc đặt tên cho các giá trị cứng được sử dụng trong suốt chương trình của bạn
dưới dạng hằng số là hữu ích trong việc truyền đạt ý nghĩa của giá trị đó cho
những người duy trì code trong tương lai. Nó cũng giúp bạn chỉ cần có một nơi
trong code mà bạn cần thay đổi nếu giá trị cứng cần được cập nhật trong tương
lai.

### Che Khuất (Shadowing)

Như bạn đã thấy trong hướng dẫn trò chơi đoán số trong [Chương
2][comparing-the-guess-to-the-secret-number]<!-- ignore -->, bạn có thể khai báo
một biến mới với cùng tên với một biến trước đó. Người dùng Rust nói rằng biến
đầu tiên bị _che khuất_ bởi biến thứ hai, có nghĩa là biến thứ hai là những gì
trình biên dịch sẽ thấy khi bạn sử dụng tên của biến. Trên thực tế, biến thứ hai
che khuất biến đầu tiên, lấy bất kỳ việc sử dụng nào của tên biến cho chính nó
cho đến khi nó tự bị che khuất hoặc phạm vi kết thúc. Chúng ta có thể che khuất
một biến bằng cách sử dụng cùng tên biến và lặp lại việc sử dụng từ khóa `let`
như sau:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/src/main.rs}}
```

Chương trình này đầu tiên gắn `x` với giá trị `5`. Sau đó, nó tạo một biến mới
`x` bằng cách lặp lại `let x =`, lấy giá trị ban đầu và cộng thêm `1` để giá trị
của `x` sau đó là `6`. Sau đó, trong phạm vi bên trong được tạo bằng dấu ngoặc
nhọn, câu lệnh `let` thứ ba cũng che khuất `x` và tạo một biến mới, nhân giá trị
trước đó với `2` để cho `x` giá trị là `12`. Khi phạm vi đó kết thúc, việc che
khuất bên trong kết thúc và `x` trở về giá trị `6`. Khi chúng ta chạy chương
trình này, nó sẽ xuất ra như sau:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-03-shadowing/output.txt}}
```

Che khuất khác với việc đánh dấu một biến là `mut` vì chúng ta sẽ gặp lỗi biên
dịch nếu chúng ta vô tình cố gắng gán lại cho biến này mà không sử dụng từ khóa
`let`. Bằng cách sử dụng `let`, chúng ta có thể thực hiện một vài biến đổi trên
một giá trị nhưng làm cho biến đó bất biến sau khi các biến đổi đó đã hoàn
thành.

Sự khác biệt khác giữa `mut` và che khuất là vì chúng ta đang tạo một biến mới
khi chúng ta sử dụng từ khóa `let` lần nữa, chúng ta có thể thay đổi kiểu của
giá trị nhưng vẫn tái sử dụng cùng một tên. Ví dụ, giả sử chương trình của chúng
ta yêu cầu người dùng hiển thị số lượng khoảng trắng mà họ muốn giữa một số văn
bản bằng cách nhập các ký tự khoảng trắng, và sau đó chúng ta muốn lưu trữ đầu
vào đó dưới dạng số:

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-04-shadowing-can-change-types/src/main.rs:here}}
```

Biến `spaces` đầu tiên là kiểu chuỗi và biến `spaces` thứ hai là kiểu số. Việc
che khuất do đó giúp chúng ta không phải nghĩ ra các tên khác nhau, như
`spaces_str` và `spaces_num`; thay vào đó, chúng ta có thể tái sử dụng cái tên
đơn giản hơn là `spaces`. Tuy nhiên, nếu chúng ta cố gắng sử dụng `mut` cho việc
này, như được hiển thị ở đây, chúng ta sẽ gặp lỗi biên dịch:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/src/main.rs:here}}
```

Lỗi nói rằng chúng ta không được phép thay đổi kiểu của một biến:

```console
{{#include ../listings/ch03-common-programming-concepts/no-listing-05-mut-cant-change-types/output.txt}}
```

Bây giờ chúng ta đã khám phá cách hoạt động của các biến, hãy xem xét nhiều loại
dữ liệu khác mà biến có thể có.

[comparing-the-guess-to-the-secret-number]:
  ch02-00-guessing-game-tutorial.html#comparing-the-guess-to-the-secret-number
[data-types]: ch03-02-data-types.html#data-types
[storing-values-with-variables]:
  ch02-00-guessing-game-tutorial.html#storing-values-with-variables
[const-eval]: ../reference/const_eval.html
