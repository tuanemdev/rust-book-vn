## Đặc điểm của Ngôn ngữ Hướng đối tượng

Không có sự đồng thuận trong cộng đồng lập trình về việc một ngôn ngữ cần có
những tính năng gì để được coi là hướng đối tượng. Rust chịu ảnh hưởng từ nhiều
mô hình lập trình, bao gồm cả OOP; ví dụ, chúng ta đã khám phá các tính năng từ
lập trình hàm trong Chương 13. Có thể nói, các ngôn ngữ OOP chia sẻ một số đặc
điểm chung nhất định, cụ thể là đối tượng, tính đóng gói và tính kế thừa. Hãy
xem xét ý nghĩa của từng đặc điểm đó và liệu Rust có hỗ trợ chúng hay không.

### Đối tượng Chứa Dữ liệu và Hành vi

Cuốn sách _Design Patterns: Elements of Reusable Object-Oriented Software_ của
Erich Gamma, Richard Helm, Ralph Johnson, và John Vlissides (Addison-Wesley,
1994), thường được gọi là cuốn sách _Gang of Four_ (Bộ tứ), là một danh mục các
mẫu thiết kế hướng đối tượng. Nó định nghĩa OOP theo cách này:

> Các chương trình hướng đối tượng được tạo thành từ các đối tượng. Một **đối
> tượng** đóng gói cả dữ liệu và các thủ tục hoạt động trên dữ liệu đó. Các thủ
> tục này thường được gọi là **phương thức** hoặc **hoạt động**.

Theo định nghĩa này, Rust là hướng đối tượng: các struct và enum có dữ liệu, và
các khối `impl` cung cấp phương thức cho struct và enum. Mặc dù struct và enum
với các phương thức không _được gọi_ là đối tượng, chúng cung cấp cùng một chức
năng, theo định nghĩa về đối tượng của Gang of Four.

### Tính Đóng gói Ẩn Chi tiết Triển khai

Một khía cạnh khác thường được liên kết với OOP là ý tưởng về _tính đóng gói_,
có nghĩa là chi tiết triển khai của một đối tượng không thể truy cập bởi mã sử
dụng đối tượng đó. Do đó, cách duy nhất để tương tác với một đối tượng là thông
qua API công khai của nó; mã sử dụng đối tượng không nên có khả năng truy cập
vào bên trong đối tượng và thay đổi dữ liệu hoặc hành vi trực tiếp. Điều này cho
phép lập trình viên thay đổi và tái cấu trúc bên trong của một đối tượng mà
không cần phải thay đổi mã sử dụng đối tượng đó.

Chúng ta đã thảo luận về cách kiểm soát tính đóng gói trong Chương 7: chúng ta
có thể sử dụng từ khóa `pub` để quyết định module, kiểu, hàm và phương thức nào
trong mã của chúng ta nên được công khai, và theo mặc định mọi thứ khác đều là
riêng tư. Ví dụ, chúng ta có thể định nghĩa một struct `AveragedCollection` có
một trường chứa một vector các giá trị `i32`. Struct này cũng có thể có một
trường chứa giá trị trung bình của các giá trị trong vector, nghĩa là giá trị
trung bình không cần phải được tính toán theo yêu cầu mỗi khi ai đó cần nó. Nói
cách khác, `AveragedCollection` sẽ lưu trữ giá trị trung bình đã tính toán cho
chúng ta. Listing 18-1 có định nghĩa của struct `AveragedCollection`.

<Listing number="18-1" file-name="src/lib.rs" caption="Một struct `AveragedCollection` duy trì danh sách các số nguyên và giá trị trung bình của các phần tử trong bộ sưu tập">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-01/src/lib.rs}}
```

</Listing>

Struct được đánh dấu `pub` để mã khác có thể sử dụng nó, nhưng các trường trong
struct vẫn là riêng tư. Điều này quan trọng trong trường hợp này vì chúng ta
muốn đảm bảo rằng bất cứ khi nào một giá trị được thêm vào hoặc xóa khỏi danh
sách, giá trị trung bình cũng được cập nhật. Chúng ta thực hiện điều này bằng
cách triển khai các phương thức `add`, `remove` và `average` trên struct, như
được hiển thị trong Listing 18-2.

<Listing number="18-2" file-name="src/lib.rs" caption="Triển khai các phương thức công khai `add`, `remove` và `average` trên `AveragedCollection`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-02/src/lib.rs:here}}
```

</Listing>

Các phương thức công khai `add`, `remove` và `average` là những cách duy nhất để
truy cập hoặc sửa đổi dữ liệu trong một thể hiện của `AveragedCollection`. Khi
một mục được thêm vào `list` bằng phương thức `add` hoặc bị xóa bằng phương thức
`remove`, các triển khai của mỗi phương thức gọi phương thức riêng tư
`update_average` xử lý việc cập nhật trường `average`.

Chúng ta để các trường `list` và `average` là riêng tư để không có cách nào cho
mã bên ngoài thêm hoặc xóa các mục vào hoặc khỏi trường `list` trực tiếp; nếu
không, trường `average` có thể không đồng bộ khi `list` thay đổi. Phương thức
`average` trả về giá trị trong trường `average`, cho phép mã bên ngoài đọc
`average` nhưng không thể sửa đổi nó.

Bởi vì chúng ta đã đóng gói các chi tiết triển khai của struct
`AveragedCollection`, chúng ta có thể dễ dàng thay đổi các khía cạnh, chẳng hạn
như cấu trúc dữ liệu, trong tương lai. Ví dụ, chúng ta có thể sử dụng
`HashSet<i32>` thay vì `Vec<i32>` cho trường `list`. Miễn là chữ ký của các
phương thức công khai `add`, `remove` và `average` không thay đổi, mã sử dụng
`AveragedCollection` sẽ không cần phải thay đổi. Nếu chúng ta làm cho `list`
công khai, điều này không nhất thiết là trường hợp: `HashSet<i32>` và `Vec<i32>`
có các phương thức khác nhau để thêm và xóa các mục, vì vậy mã bên ngoài có thể
sẽ phải thay đổi nếu nó đang sửa đổi `list` trực tiếp.

Nếu tính đóng gói là một khía cạnh bắt buộc để một ngôn ngữ được coi là hướng
đối tượng, thì Rust đáp ứng yêu cầu đó. Tùy chọn sử dụng `pub` hoặc không cho
các phần khác nhau của mã cho phép đóng gói các chi tiết triển khai.

### Tính kế thừa như một Hệ thống Kiểu và như Chia sẻ Mã

_Tính kế thừa_ là một cơ chế mà một đối tượng có thể kế thừa các phần tử từ định
nghĩa của một đối tượng khác, do đó có được dữ liệu và hành vi của đối tượng cha
mà không cần phải định nghĩa lại chúng.

Nếu một ngôn ngữ phải có tính kế thừa để được coi là hướng đối tượng, thì Rust
không phải là một ngôn ngữ như vậy. Không có cách nào để định nghĩa một struct
kế thừa các trường của struct cha và triển khai phương thức mà không sử dụng
macro.

Tuy nhiên, nếu bạn quen với việc có tính kế thừa trong bộ công cụ lập trình của
mình, bạn có thể sử dụng các giải pháp khác trong Rust, tùy thuộc vào lý do bạn
chọn tính kế thừa từ đầu.

Bạn sẽ chọn tính kế thừa vì hai lý do chính. Một là để tái sử dụng mã: bạn có
thể triển khai một hành vi cụ thể cho một kiểu, và tính kế thừa cho phép bạn tái
sử dụng triển khai đó cho một kiểu khác. Bạn có thể làm điều này một cách hạn
chế trong mã Rust bằng cách sử dụng triển khai phương thức trait mặc định, mà
bạn đã thấy trong Listing 10-14 khi chúng ta thêm một triển khai mặc định của
phương thức `summarize` trên trait `Summary`. Bất kỳ kiểu nào triển khai trait
`Summary` sẽ có sẵn phương thức `summarize` mà không cần thêm mã nào nữa. Điều
này tương tự như một lớp cha có một triển khai của một phương thức và một lớp
con kế thừa cũng có triển khai của phương thức đó. Chúng ta cũng có thể ghi đè
triển khai mặc định của phương thức `summarize` khi chúng ta triển khai trait
`Summary`, điều này tương tự như một lớp con ghi đè triển khai của một phương
thức kế thừa từ lớp cha.

Lý do khác để sử dụng tính kế thừa liên quan đến hệ thống kiểu: để cho phép một
kiểu con được sử dụng ở những nơi mà kiểu cha được sử dụng. Điều này còn được
gọi là _tính đa hình_, có nghĩa là bạn có thể thay thế nhiều đối tượng cho nhau
trong thời gian chạy nếu chúng chia sẻ một số đặc điểm nhất định.

> ### Tính đa hình
>
> Đối với nhiều người, tính đa hình đồng nghĩa với tính kế thừa. Nhưng thực tế
> nó là một khái niệm tổng quát hơn, đề cập đến mã có thể làm việc với dữ liệu
> của nhiều kiểu. Đối với tính kế thừa, những kiểu đó thường là các lớp con.
>
> Thay vào đó, Rust sử dụng generics để trừu tượng hóa các kiểu khác nhau có thể
> có và các ràng buộc trait để áp đặt các ràng buộc về những gì các kiểu đó phải
> cung cấp. Điều này đôi khi được gọi là _tính đa hình tham số có giới hạn_
> (bounded parametric polymorphism).

Rust đã chọn một tập hợp các đánh đổi khác nhau bằng cách không cung cấp tính kế
thừa. Tính kế thừa thường có nguy cơ chia sẻ nhiều mã hơn mức cần thiết. Các lớp
con không phải lúc nào cũng nên chia sẻ tất cả các đặc điểm của lớp cha của
chúng nhưng sẽ làm như vậy với tính kế thừa. Điều này có thể làm cho thiết kế
của một chương trình kém linh hoạt hơn. Nó cũng tạo ra khả năng gọi các phương
thức trên các lớp con mà không hợp lý hoặc gây ra lỗi vì các phương thức không
áp dụng cho lớp con. Ngoài ra, một số ngôn ngữ chỉ cho phép _đơn kế thừa_ (có
nghĩa là một lớp con chỉ có thể kế thừa từ một lớp), hạn chế hơn nữa tính linh
hoạt của thiết kế chương trình.

Vì những lý do này, Rust sử dụng cách tiếp cận khác bằng cách sử dụng các đối
tượng trait thay vì tính kế thừa để cho phép tính đa hình. Hãy xem cách các đối
tượng trait hoạt động.
