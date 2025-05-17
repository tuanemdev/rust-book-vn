## Lưu Trữ Danh Sách Giá Trị với Vector

Loại bộ sưu tập đầu tiên mà chúng ta sẽ xem xét là `Vec<T>`, còn được gọi là
_vector_. Vector cho phép bạn lưu trữ nhiều giá trị trong một cấu trúc dữ liệu
duy nhất, đặt tất cả các giá trị liền kề nhau trong bộ nhớ. Vector chỉ có thể
lưu trữ các giá trị cùng kiểu. Chúng rất hữu ích khi bạn có một danh sách các
mục, chẳng hạn như các dòng văn bản trong một tập tin hoặc giá của các mặt hàng
trong một giỏ hàng.

### Tạo một Vector Mới

Để tạo một vector rỗng mới, chúng ta gọi hàm `Vec::new`, như hiển thị trong
Listing 8-1.

<Listing number="8-1" caption="Tạo một vector rỗng mới để chứa các giá trị kiểu `i32`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-01/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta đã thêm một chú thích kiểu ở đây. Bởi vì chúng ta không chèn
bất kỳ giá trị nào vào vector này, Rust không biết chúng ta dự định lưu trữ
những phần tử kiểu gì. Đây là một điểm quan trọng. Vector được triển khai bằng
cách sử dụng generics; chúng ta sẽ đề cập đến cách sử dụng generics với các kiểu
riêng của bạn trong Chương 10. Hiện tại, hãy biết rằng kiểu `Vec<T>` được cung
cấp bởi thư viện chuẩn có thể chứa bất kỳ kiểu nào. Khi chúng ta tạo một vector
để chứa một kiểu cụ thể, chúng ta có thể chỉ định kiểu đó trong dấu ngoặc nhọn.
Trong Listing 8-1, chúng ta đã cho Rust biết rằng `Vec<T>` trong `v` sẽ chứa các
phần tử kiểu `i32`.

Thường thì bạn sẽ tạo một `Vec<T>` với các giá trị ban đầu và Rust sẽ suy luận
kiểu giá trị mà bạn muốn lưu trữ, vì vậy bạn hiếm khi cần phải chú thích kiểu.
Rust cung cấp một cách tiện lợi với macro `vec!`, sẽ tạo một vector mới chứa các
giá trị bạn cung cấp. Listing 8-2 tạo một `Vec<i32>` mới chứa các giá trị `1`,
`2`, và `3`. Kiểu số nguyên là `i32` vì đó là kiểu số nguyên mặc định, như chúng
ta đã thảo luận trong phần ["Kiểu Dữ Liệu"][data-types]<!-- ignore --> của
Chương 3.

<Listing number="8-2" caption="Tạo một vector mới chứa các giá trị">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-02/src/main.rs:here}}
```

</Listing>

Vì chúng ta đã cung cấp các giá trị `i32` ban đầu, Rust có thể suy luận rằng
kiểu của `v` là `Vec<i32>`, và không cần thiết phải chú thích kiểu. Tiếp theo,
chúng ta sẽ xem cách sửa đổi một vector.

### Cập Nhật một Vector

Để tạo một vector và sau đó thêm các phần tử vào nó, chúng ta có thể sử dụng
phương thức `push`, như hiển thị trong Listing 8-3.

<Listing number="8-3" caption="Sử dụng phương thức `push` để thêm giá trị vào vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-03/src/main.rs:here}}
```

</Listing>

Giống như với bất kỳ biến nào, nếu chúng ta muốn có thể thay đổi giá trị của nó,
chúng ta cần làm cho nó có thể thay đổi bằng cách sử dụng từ khóa `mut`, như đã
thảo luận trong Chương 3. Các số mà chúng ta đặt vào đều thuộc kiểu `i32`, và
Rust suy luận điều này từ dữ liệu, vì vậy chúng ta không cần chú thích
`Vec<i32>`.

### Đọc Các Phần Tử của Vector

Có hai cách để tham chiếu đến một giá trị được lưu trữ trong vector: thông qua
chỉ mục hoặc bằng cách sử dụng phương thức `get`. Trong các ví dụ sau, chúng ta
đã chú thích các kiểu của giá trị được trả về từ các hàm này để có thêm độ rõ
ràng.

Listing 8-4 hiển thị cả hai phương pháp truy cập giá trị trong vector, với cú
pháp lập chỉ mục và phương thức `get`.

<Listing number="8-4" caption="Sử dụng cú pháp lập chỉ mục và phương thức `get` để truy cập vào một phần tử trong vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-04/src/main.rs:here}}
```

</Listing>

Lưu ý một vài chi tiết ở đây. Chúng ta sử dụng giá trị chỉ mục `2` để lấy phần
tử thứ ba vì vector được đánh chỉ mục bằng số, bắt đầu từ không. Sử dụng `&` và
`[]` cho chúng ta một tham chiếu đến phần tử tại giá trị chỉ mục. Khi chúng ta
sử dụng phương thức `get` với chỉ mục được truyền vào dưới dạng đối số, chúng ta
nhận được một `Option<&T>` mà chúng ta có thể sử dụng với `match`.

Rust cung cấp hai cách để tham chiếu đến một phần tử để bạn có thể chọn cách
chương trình hoạt động khi bạn cố gắng sử dụng một giá trị chỉ mục nằm ngoài
phạm vi của các phần tử hiện có. Ví dụ, hãy xem điều gì xảy ra khi chúng ta có
một vector gồm năm phần tử và sau đó chúng ta cố gắng truy cập một phần tử tại
chỉ mục 100 với mỗi kỹ thuật, như hiển thị trong Listing 8-5.

<Listing number="8-5" caption="Cố gắng truy cập phần tử tại chỉ mục 100 trong một vector chứa năm phần tử">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-05/src/main.rs:here}}
```

</Listing>

Khi chúng ta chạy mã này, phương thức `[]` đầu tiên sẽ làm cho chương trình
hoảng loạn vì nó tham chiếu đến một phần tử không tồn tại. Phương pháp này được
sử dụng tốt nhất khi bạn muốn chương trình của mình gặp sự cố nếu có một nỗ lực
truy cập một phần tử ngoài phạm vi của vector.

Khi phương thức `get` được truyền một chỉ mục ngoài vector, nó trả về `None` mà
không hoảng loạn. Bạn sẽ sử dụng phương pháp này nếu việc truy cập một phần tử
ngoài phạm vi của vector có thể xảy ra thỉnh thoảng trong các trường hợp bình
thường. Mã của bạn sau đó sẽ có logic để xử lý việc có cả `Some(&element)` hoặc
`None`, như đã thảo luận trong Chương 6. Ví dụ, chỉ mục có thể đến từ một người
nhập một số. Nếu họ vô tình nhập một số quá lớn và chương trình nhận được giá
trị `None`, bạn có thể cho người dùng biết có bao nhiêu mục trong vector hiện
tại và cho họ một cơ hội khác để nhập một giá trị hợp lệ. Điều đó sẽ thân thiện
với người dùng hơn là làm cho chương trình gặp sự cố do một lỗi đánh máy!

Khi chương trình có một tham chiếu hợp lệ, bộ kiểm tra mượn sẽ thực thi các quy
tắc quyền sở hữu và mượn (được đề cập trong Chương 4) để đảm bảo tham chiếu này
và bất kỳ tham chiếu nào khác tới nội dung của vector vẫn có giá trị. Nhớ lại
quy tắc cho rằng bạn không thể có tham chiếu có thể thay đổi và không thể thay
đổi trong cùng phạm vi. Quy tắc đó áp dụng trong Listing 8-6, nơi chúng ta giữ
một tham chiếu bất biến đến phần tử đầu tiên trong một vector và cố gắng thêm
một phần tử vào cuối. Chương trình này sẽ không hoạt động nếu chúng ta cũng cố
gắng tham chiếu đến phần tử đó sau đó trong hàm.

<Listing number="8-6" caption="Cố gắng thêm một phần tử vào một vector trong khi đang giữ một tham chiếu đến một mục">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-06/src/main.rs:here}}
```

</Listing>

Biên dịch mã này sẽ dẫn đến lỗi này:

```console
{{#include ../listings/ch08-common-collections/listing-08-06/output.txt}}
```

Mã trong Listing 8-6 có vẻ như nên hoạt động: tại sao một tham chiếu đến phần tử
đầu tiên lại quan tâm đến những thay đổi ở cuối vector? Lỗi này là do cách
vector hoạt động: vì vector đặt các giá trị liền kề nhau trong bộ nhớ, việc thêm
một phần tử mới vào cuối vector có thể yêu cầu cấp phát bộ nhớ mới và sao chép
các phần tử cũ sang không gian mới, nếu không có đủ chỗ để đặt tất cả các phần
tử liền kề nhau tại nơi vector hiện đang được lưu trữ. Trong trường hợp đó, tham
chiếu đến phần tử đầu tiên sẽ trỏ đến bộ nhớ đã giải phóng. Các quy tắc mượn
ngăn chặn các chương trình rơi vào tình huống đó.

> Lưu ý: Để biết thêm về chi tiết triển khai của kiểu `Vec<T>`, hãy xem ["The
> Rustonomicon"][nomicon].

### Lặp Qua Các Giá Trị trong một Vector

Để truy cập từng phần tử trong một vector theo lượt, chúng ta sẽ lặp qua tất cả
các phần tử thay vì sử dụng chỉ mục để truy cập từng phần tử một lần. Listing
8-7 hiển thị cách sử dụng vòng lặp `for` để lấy các tham chiếu bất biến đến từng
phần tử trong một vector các giá trị `i32` và in chúng ra.

<Listing number="8-7" caption="In từng phần tử trong một vector bằng cách lặp qua các phần tử sử dụng vòng lặp `for`">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-07/src/main.rs:here}}
```

</Listing>

Chúng ta cũng có thể lặp qua các tham chiếu có thể thay đổi đến từng phần tử
trong một vector có thể thay đổi để thực hiện các thay đổi cho tất cả các phần
tử. Vòng lặp `for` trong Listing 8-8 sẽ thêm `50` vào mỗi phần tử.

<Listing number="8-8" caption="Lặp qua các tham chiếu có thể thay đổi đến các phần tử trong vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-08/src/main.rs:here}}
```

</Listing>

Để thay đổi giá trị mà tham chiếu có thể thay đổi đề cập đến, chúng ta phải sử
dụng toán tử giải tham chiếu `*` để đến được giá trị trong `i` trước khi có thể
sử dụng toán tử `+=`. Chúng ta sẽ nói thêm về toán tử giải tham chiếu trong phần
["Tham chiếu đến giá trị"][deref]<!-- ignore --> của Chương 15.

Việc lặp qua một vector, dù là bất biến hay có thể thay đổi, đều an toàn nhờ vào
quy tắc của bộ kiểm tra mượn. Nếu chúng ta cố gắng chèn hoặc xóa các mục trong
thân vòng lặp `for` trong Listing 8-7 và Listing 8-8, chúng ta sẽ gặp lỗi biên
dịch tương tự như lỗi chúng ta gặp với mã trong Listing 8-6. Tham chiếu đến
vector mà vòng lặp `for` giữ ngăn chặn việc sửa đổi đồng thời toàn bộ vector.

### Sử dụng Enum để Lưu Trữ Nhiều Kiểu

Vector chỉ có thể lưu trữ các giá trị có cùng kiểu. Điều này có thể gây bất
tiện; chắc chắn có những trường hợp sử dụng cần lưu trữ một danh sách các mục
thuộc các kiểu khác nhau. May mắn thay, các biến thể của một enum được định
nghĩa dưới cùng một kiểu enum, vì vậy khi chúng ta cần một kiểu để đại diện cho
các phần tử của các kiểu khác nhau, chúng ta có thể định nghĩa và sử dụng một
enum!

Ví dụ, giả sử chúng ta muốn lấy các giá trị từ một hàng trong một bảng tính
trong đó một số cột trong hàng chứa số nguyên, một số chứa số dấu phẩy động, và
một số chứa chuỗi. Chúng ta có thể định nghĩa một enum có các biến thể sẽ chứa
các kiểu giá trị khác nhau, và tất cả các biến thể enum sẽ được coi là cùng một
kiểu: kiểu của enum đó. Sau đó chúng ta có thể tạo một vector để chứa enum đó và
do đó, cuối cùng, chứa các kiểu khác nhau. Chúng ta đã minh họa điều này trong
Listing 8-9.

<Listing number="8-9" caption="Định nghĩa một `enum` để lưu trữ các giá trị thuộc các kiểu khác nhau trong một vector">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-09/src/main.rs:here}}
```

</Listing>

Rust cần biết những kiểu nào sẽ có trong vector tại thời điểm biên dịch để biết
chính xác bao nhiêu bộ nhớ trên heap sẽ cần để lưu trữ mỗi phần tử. Chúng ta
cũng phải rõ ràng về những kiểu nào được phép trong vector này. Nếu Rust cho
phép một vector chứa bất kỳ kiểu nào, sẽ có khả năng một hoặc nhiều kiểu sẽ gây
ra lỗi với các hoạt động được thực hiện trên các phần tử của vector. Sử dụng
enum cùng với biểu thức `match` nghĩa là Rust sẽ đảm bảo tại thời điểm biên dịch
rằng mọi trường hợp khả dĩ đều được xử lý, như đã thảo luận trong Chương 6.

Nếu bạn không biết tập hợp đầy đủ các kiểu mà một chương trình sẽ nhận được tại
thời điểm chạy để lưu trữ trong vector, kỹ thuật enum sẽ không hoạt động. Thay
vào đó, bạn có thể sử dụng trait object, mà chúng ta sẽ đề cập trong Chương 18.

Giờ chúng ta đã thảo luận về một số cách sử dụng vector phổ biến nhất, hãy đảm
bảo xem xét [tài liệu API][vec-api]<!-- ignore --> để biết tất cả các phương
thức hữu ích được định nghĩa cho `Vec<T>` bởi thư viện chuẩn. Ví dụ, ngoài
`push`, phương thức `pop` loại bỏ và trả về phần tử cuối cùng.

### Giải phóng một Vector sẽ Giải phóng Các Phần tử của Nó

Giống như bất kỳ `struct` nào khác, một vector được giải phóng khi nó ra khỏi
phạm vi, như được chú thích trong Listing 8-10.

<Listing number="8-10" caption="Hiển thị nơi vector và các phần tử của nó được giải phóng">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-10/src/main.rs:here}}
```

</Listing>

Khi vector bị giải phóng, tất cả nội dung của nó cũng bị giải phóng, nghĩa là
các số nguyên mà nó chứa sẽ được dọn sạch. Bộ kiểm tra mượn đảm bảo rằng bất kỳ
tham chiếu nào đến nội dung của vector chỉ được sử dụng khi bản thân vector còn
có giá trị.

Hãy chuyển sang loại bộ sưu tập tiếp theo: `String`!

[data-types]: ch03-02-data-types.html#data-types
[nomicon]: ../nomicon/vec/vec.html
[vec-api]: ../std/vec/struct.Vec.html
[deref]:
  ch15-02-deref.html#following-the-pointer-to-the-value-with-the-dereference-operator
