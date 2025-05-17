## Tham Chiếu và Mượn

Vấn đề với đoạn mã tuple trong Listing 4-5 là chúng ta phải trả lại `String` cho
hàm gọi để chúng ta vẫn có thể sử dụng `String` sau cuộc gọi đến
`calculate_length`, bởi vì `String` đã được chuyển vào `calculate_length`. Thay
vào đó, chúng ta có thể cung cấp một tham chiếu đến giá trị `String`. _Tham
chiếu (reference)_ giống như một con trỏ ở chỗ nó là một địa chỉ mà chúng ta có
thể theo dõi để truy cập dữ liệu được lưu trữ tại địa chỉ đó; dữ liệu đó thuộc
sở hữu của một biến khác. Không giống như con trỏ, một tham chiếu được đảm bảo
trỏ đến một giá trị hợp lệ của một kiểu dữ liệu cụ thể trong suốt thời gian tồn
tại của tham chiếu đó.

Đây là cách bạn sẽ định nghĩa và sử dụng một hàm `calculate_length` có một tham
chiếu đến một đối tượng làm tham số thay vì lấy quyền sở hữu của giá trị:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:all}}
```

</Listing>

Đầu tiên, lưu ý rằng tất cả các mã tuple trong khai báo biến và giá trị trả về
của hàm đã biến mất. Thứ hai, lưu ý rằng chúng ta truyền `&s1` vào
`calculate_length` và trong định nghĩa của nó, chúng ta lấy `&String` thay vì
`String`. Các dấu và (&) này đại diện cho _tham chiếu_, và chúng cho phép bạn
tham chiếu đến một số giá trị mà không lấy quyền sở hữu của nó. Hình 4-6 minh
họa khái niệm này.

<img alt="Ba bảng: bảng cho s chỉ chứa một con trỏ đến bảng
cho s1. Bảng cho s1 chứa dữ liệu stack cho s1 và trỏ đến
dữ liệu chuỗi trên heap." src="img/trpl04-06.svg" class="center" />

<span class="caption">Hình 4-6: Một sơ đồ của `&String s` trỏ đến
`String s1`</span>

> Lưu ý: Ngược lại với việc tham chiếu bằng cách sử dụng `&` là _giải tham chiếu
> (dereferencing)_, được thực hiện với toán tử giải tham chiếu, `*`. Chúng ta sẽ
> thấy một số cách sử dụng toán tử giải tham chiếu trong Chương 8 và thảo luận
> chi tiết về giải tham chiếu trong Chương 15.

Hãy xem xét kỹ hơn cuộc gọi hàm ở đây:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-07-reference/src/main.rs:here}}
```

Cú pháp `&s1` cho phép chúng ta tạo một tham chiếu _trỏ đến_ giá trị của `s1`
nhưng không sở hữu nó. Vì tham chiếu không sở hữu nó, giá trị mà nó trỏ đến sẽ
không bị hủy khi tham chiếu không còn được sử dụng nữa.

Tương tự, chữ ký của hàm sử dụng `&` để chỉ ra rằng kiểu của tham số `s` là một
tham chiếu. Hãy thêm một số chú thích giải thích:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-08-reference-with-annotations/src/main.rs:here}}
```

Phạm vi mà biến `s` có giá trị giống như phạm vi của bất kỳ tham số hàm nào,
nhưng giá trị được trỏ đến bởi tham chiếu không bị hủy khi `s` không còn được sử
dụng nữa, vì `s` không có quyền sở hữu. Khi các hàm có tham chiếu làm tham số
thay vì các giá trị thực tế, chúng ta sẽ không cần trả lại các giá trị để trả
lại quyền sở hữu, vì chúng ta chưa bao giờ có quyền sở hữu.

Chúng ta gọi hành động tạo một tham chiếu là _mượn (borrowing)_. Như trong cuộc
sống thực, nếu một người sở hữu một thứ gì đó, bạn có thể mượn nó từ họ. Khi bạn
xong việc, bạn phải trả lại nó. Bạn không sở hữu nó.

Vậy, điều gì sẽ xảy ra nếu chúng ta cố gắng sửa đổi thứ gì đó mà chúng ta đang
mượn? Hãy thử mã trong Listing 4-6. Cảnh báo trước: nó không hoạt động!

<Listing number="4-6" file-name="src/main.rs" caption="Cố gắng sửa đổi một giá trị đã mượn">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-06/src/main.rs}}
```

</Listing>

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/listing-04-06/output.txt}}
```

Giống như các biến là không thay đổi theo mặc định, các tham chiếu cũng vậy.
Chúng ta không được phép sửa đổi một thứ mà chúng ta có tham chiếu đến.

### Tham Chiếu Có Thể Thay Đổi

Chúng ta có thể sửa mã từ Listing 4-6 để cho phép chúng ta sửa đổi một giá trị
đã mượn chỉ với một vài điều chỉnh nhỏ sử dụng, thay vào đó, một _tham chiếu có
thể thay đổi (mutable reference)_:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-09-fixes-listing-04-06/src/main.rs}}
```

</Listing>

Đầu tiên, chúng ta thay đổi `s` thành `mut`. Sau đó, chúng ta tạo một tham chiếu
có thể thay đổi với `&mut s` khi chúng ta gọi hàm `change`, và cập nhật chữ ký
hàm để chấp nhận một tham chiếu có thể thay đổi với `some_string: &mut String`.
Điều này làm cho nó rất rõ ràng rằng hàm `change` sẽ thay đổi giá trị mà nó
mượn.

Tham chiếu có thể thay đổi có một hạn chế lớn: nếu bạn có một tham chiếu có thể
thay đổi đến một giá trị, bạn không thể có tham chiếu nào khác đến giá trị đó.
Mã này cố gắng tạo hai tham chiếu có thể thay đổi đến `s` sẽ thất bại:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/src/main.rs:here}}
```

</Listing>

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-10-multiple-mut-not-allowed/output.txt}}
```

Lỗi này nói rằng mã này không hợp lệ vì chúng ta không thể mượn `s` như một biến
có thể thay đổi nhiều hơn một lần tại một thời điểm. Lần mượn có thể thay đổi
đầu tiên là trong `r1` và phải kéo dài cho đến khi nó được sử dụng trong
`println!`, nhưng giữa việc tạo tham chiếu có thể thay đổi đó và việc sử dụng
nó, chúng ta đã cố gắng tạo một tham chiếu có thể thay đổi khác trong `r2` mà
mượn cùng dữ liệu như `r1`.

Hạn chế ngăn nhiều tham chiếu có thể thay đổi đến cùng một dữ liệu tại cùng một
thời điểm cho phép thay đổi nhưng theo một cách rất có kiểm soát. Đây là điều mà
các lập trình viên Rust mới gặp khó khăn vì hầu hết các ngôn ngữ cho phép bạn
thay đổi bất cứ khi nào bạn muốn. Lợi ích của việc có hạn chế này là Rust có thể
ngăn chặn đua dữ liệu tại thời điểm biên dịch. _Đua dữ liệu (data race)_ tương
tự như một điều kiện đua và xảy ra khi có ba hành vi sau:

- Hai hoặc nhiều con trỏ truy cập cùng một dữ liệu tại cùng một thời điểm.
- Ít nhất một trong các con trỏ đang được sử dụng để ghi vào dữ liệu.
- Không có cơ chế nào đang được sử dụng để đồng bộ hóa việc truy cập vào dữ
  liệu.

Đua dữ liệu gây ra hành vi không xác định và có thể khó chẩn đoán và sửa chữa
khi bạn đang cố gắng theo dõi chúng trong thời gian chạy; Rust ngăn chặn vấn đề
này bằng cách từ chối biên dịch mã có đua dữ liệu!

Như mọi khi, chúng ta có thể sử dụng dấu ngoặc nhọn để tạo một phạm vi mới, cho
phép nhiều tham chiếu có thể thay đổi, miễn là chúng không _đồng thời_ xuất
hiện:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-11-muts-in-separate-scopes/src/main.rs:here}}
```

Rust áp dụng một quy tắc tương tự cho việc kết hợp tham chiếu có thể thay đổi và
không thể thay đổi. Mã này dẫn đến lỗi:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/src/main.rs:here}}
```

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-12-immutable-and-mutable-not-allowed/output.txt}}
```

Ôi! Chúng ta _cũng_ không thể có một tham chiếu có thể thay đổi trong khi chúng
ta có một tham chiếu không thể thay đổi đến cùng một giá trị.

Người dùng của một tham chiếu không thể thay đổi không mong đợi giá trị đột
nhiên thay đổi dưới chân họ! Tuy nhiên, nhiều tham chiếu không thể thay đổi được
cho phép vì không ai chỉ đọc dữ liệu có khả năng ảnh hưởng đến việc đọc dữ liệu
của người khác.

Lưu ý rằng phạm vi của một tham chiếu bắt đầu từ nơi nó được giới thiệu và tiếp
tục thông qua lần cuối cùng tham chiếu đó được sử dụng. Ví dụ, mã này sẽ biên
dịch vì lần sử dụng cuối cùng của các tham chiếu không thể thay đổi là trong
`println!`, trước khi tham chiếu có thể thay đổi được giới thiệu:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-13-reference-scope-ends/src/main.rs:here}}
```

Phạm vi của các tham chiếu không thể thay đổi `r1` và `r2` kết thúc sau
`println!` nơi chúng được sử dụng lần cuối, đó là trước khi tham chiếu có thể
thay đổi `r3` được tạo. Các phạm vi này không chồng lấn, vì vậy mã này được cho
phép: trình biên dịch có thể biết rằng tham chiếu không còn được sử dụng tại một
điểm trước khi kết thúc phạm vi.

Mặc dù các lỗi mượn có thể gây bực bội đôi khi, hãy nhớ rằng đó là trình biên
dịch Rust chỉ ra một lỗi tiềm ẩn sớm (tại thời điểm biên dịch thay vì tại thời
điểm chạy) và chỉ cho bạn chính xác nơi vấn đề nằm ở. Sau đó, bạn không phải
theo dõi lý do tại sao dữ liệu của bạn không phải là những gì bạn nghĩ nó là.

### Tham Chiếu Treo

Trong các ngôn ngữ có con trỏ, rất dễ vô tình tạo ra một _con trỏ treo (dangling
pointer)_—một con trỏ tham chiếu đến một vị trí trong bộ nhớ có thể đã được cấp
cho ai đó khác—bằng cách giải phóng một số bộ nhớ trong khi vẫn giữ một con trỏ
đến bộ nhớ đó. Ngược lại, trong Rust, trình biên dịch đảm bảo rằng các tham
chiếu sẽ không bao giờ là tham chiếu treo: nếu bạn có một tham chiếu đến một số
dữ liệu, trình biên dịch sẽ đảm bảo rằng dữ liệu sẽ không ra khỏi phạm vi trước
tham chiếu đến dữ liệu đó.

Hãy thử tạo một tham chiếu treo để xem cách Rust ngăn chúng với một lỗi thời
điểm biên dịch:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/src/main.rs}}
```

</Listing>

Đây là lỗi:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-14-dangling-reference/output.txt}}
```

Thông báo lỗi này đề cập đến một tính năng mà chúng ta chưa đề cập: thời gian
tồn tại (lifetimes). Chúng ta sẽ thảo luận về thời gian tồn tại chi tiết trong
Chương 10. Nhưng, nếu bạn bỏ qua các phần về thời gian tồn tại, thông báo có
chứa chìa khóa cho lý do tại sao mã này là một vấn đề:

```text
kiểu trả về của hàm này chứa một giá trị đã mượn, nhưng không có giá trị
nào để mượn từ đó
```

Hãy xem xét kỹ hơn chính xác những gì đang xảy ra ở mỗi giai đoạn của mã
`dangle` của chúng ta:

<Listing file-name="src/main.rs">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-15-dangling-reference-annotated/src/main.rs:here}}
```

</Listing>

Vì `s` được tạo bên trong `dangle`, khi mã của `dangle` hoàn thành, `s` sẽ bị
giải phóng. Nhưng chúng ta đã cố gắng trả về một tham chiếu đến nó. Điều đó có
nghĩa là tham chiếu này sẽ trỏ đến một `String` không hợp lệ. Điều đó không tốt!
Rust sẽ không cho phép chúng ta làm điều này.

Giải pháp ở đây là trả về `String` trực tiếp:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-16-no-dangle/src/main.rs:here}}
```

Điều này hoạt động mà không có vấn đề gì. Quyền sở hữu được chuyển ra ngoài và
không có gì bị giải phóng.

### Các Quy Tắc của Tham Chiếu

Hãy tổng kết những gì chúng ta đã thảo luận về tham chiếu:

- Tại bất kỳ thời điểm nào, bạn có thể có _hoặc là_ một tham chiếu có thể thay
  đổi _hoặc là_ bất kỳ số lượng tham chiếu không thể thay đổi nào.
- Tham chiếu phải luôn hợp lệ.

Tiếp theo, chúng ta sẽ xem xét một loại tham chiếu khác: slice.
