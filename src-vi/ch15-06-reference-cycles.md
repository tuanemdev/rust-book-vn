## Chu Kỳ Tham Chiếu Có Thể Rò Rỉ Bộ Nhớ

Các đảm bảo an toàn bộ nhớ của Rust khiến việc vô tình tạo ra bộ nhớ không bao
giờ được dọn dẹp (được gọi là _rò rỉ bộ nhớ_) trở nên khó khăn, nhưng không phải
là không thể. Việc ngăn chặn hoàn toàn rò rỉ bộ nhớ không phải là một trong
những đảm bảo của Rust, có nghĩa là rò rỉ bộ nhớ là an toàn về mặt bộ nhớ trong
Rust. Chúng ta có thể thấy rằng Rust cho phép rò rỉ bộ nhớ bằng cách sử dụng
`Rc<T>` và `RefCell<T>`: có thể tạo các tham chiếu trong đó các mục tham chiếu
lẫn nhau trong một chu kỳ. Điều này tạo ra rò rỉ bộ nhớ vì số lượng tham chiếu
của mỗi mục trong chu kỳ sẽ không bao giờ đạt đến 0, và các giá trị sẽ không bao
giờ bị hủy.

### Tạo Chu Kỳ Tham Chiếu

Hãy xem cách một chu kỳ tham chiếu có thể xảy ra và cách ngăn chặn nó, bắt đầu
với định nghĩa của enum `List` và một phương thức `tail` trong Listing 15-25.

<Listing number="15-25" file-name="src/main.rs" caption="Một định nghĩa danh sách cons nắm giữ một `RefCell<T>` để chúng ta có thể sửa đổi những gì một biến thể `Cons` đang tham chiếu đến">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-25/src/main.rs}}
```

</Listing>

Chúng ta đang sử dụng một biến thể khác của định nghĩa `List` từ Listing 15-5.
Phần tử thứ hai trong biến thể `Cons` bây giờ là `RefCell<Rc<List>>`, có nghĩa
là thay vì có khả năng sửa đổi giá trị `i32` như chúng ta đã làm trong Listing
15-24, chúng ta muốn sửa đổi giá trị `List` mà một biến thể `Cons` đang trỏ đến.
Chúng ta cũng thêm một phương thức `tail` để thuận tiện cho việc truy cập vào
mục thứ hai nếu chúng ta có một biến thể `Cons`.

Trong Listing 15-26, chúng ta đang thêm một hàm `main` sử dụng các định nghĩa
trong Listing 15-25. Mã này tạo một danh sách trong `a` và một danh sách trong
`b` trỏ đến danh sách trong `a`. Sau đó, nó sửa đổi danh sách trong `a` để trỏ
đến `b`, tạo thành một chu kỳ tham chiếu. Có các câu lệnh `println!` dọc theo
đường đi để hiển thị số lượng tham chiếu tại các điểm khác nhau trong quá trình
này.

<Listing number="15-26" file-name="src/main.rs" caption="Tạo một chu kỳ tham chiếu của hai giá trị `List` trỏ đến nhau">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-26/src/main.rs:here}}
```

</Listing>

Chúng ta tạo một thể hiện `Rc<List>` chứa một giá trị `List` trong biến `a` với
một danh sách ban đầu của `5, Nil`. Sau đó, chúng ta tạo một thể hiện `Rc<List>`
chứa một giá trị `List` khác trong biến `b` chứa giá trị `10` và trỏ đến danh
sách trong `a`.

Chúng ta sửa đổi `a` để nó trỏ đến `b` thay vì `Nil`, tạo thành một chu kỳ.
Chúng ta làm điều đó bằng cách sử dụng phương thức `tail` để lấy một tham chiếu
đến `RefCell<Rc<List>>` trong `a`, mà chúng ta đặt trong biến `link`. Sau đó,
chúng ta sử dụng phương thức `borrow_mut` trên `RefCell<Rc<List>>` để thay đổi
giá trị bên trong từ một `Rc<List>` chứa giá trị `Nil` thành `Rc<List>` trong
`b`.

Khi chúng ta chạy mã này, giữ cho `println!` cuối cùng được comment lại trong
thời điểm này, chúng ta sẽ nhận được đầu ra này:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-26/output.txt}}
```

Số lượng tham chiếu của các thể hiện `Rc<List>` trong cả `a` và `b` là 2 sau khi
chúng ta thay đổi danh sách trong `a` để trỏ đến `b`. Vào cuối `main`, Rust loại
bỏ biến `b`, giảm số lượng tham chiếu của thể hiện `Rc<List>` trong `b` từ 2
xuống 1. Bộ nhớ mà `Rc<List>` có trên heap sẽ không bị hủy tại thời điểm này vì
số lượng tham chiếu của nó là 1, không phải 0. Sau đó, Rust loại bỏ `a`, giảm số
lượng tham chiếu của thể hiện `Rc<List>` trong `a` từ 2 xuống 1. Bộ nhớ của thể
hiện này cũng không thể bị hủy, vì thể hiện `Rc<List>` khác vẫn tham chiếu đến
nó. Bộ nhớ được cấp phát cho danh sách sẽ vẫn không được thu gom mãi mãi. Để
hình dung chu kỳ tham chiếu này, chúng ta đã tạo sơ đồ trong Hình 15-4.

<img alt="Một hình chữ nhật có nhãn 'a' trỏ đến một hình chữ nhật chứa số nguyên 5. Một hình chữ nhật có nhãn 'b' trỏ đến một hình chữ nhật chứa số nguyên 10. Hình chữ nhật chứa 5 trỏ đến hình chữ nhật chứa 10, và hình chữ nhật chứa 10 trỏ ngược lại hình chữ nhật chứa 5, tạo thành một chu kỳ" src="img/trpl15-04.svg" class="center" />

<span class="caption">Hình 15-4: Một chu kỳ tham chiếu của danh sách `a` và `b`
trỏ đến nhau</span>

Nếu bạn bỏ comment dòng `println!` cuối cùng và chạy chương trình, Rust sẽ cố
gắng in chu kỳ này với `a` trỏ đến `b` trỏ đến `a` và cứ thế cho đến khi nó tràn
ngăn xếp.

So với một chương trình thực tế, hậu quả của việc tạo chu kỳ tham chiếu trong ví
dụ này không quá nghiêm trọng: ngay sau khi chúng ta tạo chu kỳ tham chiếu,
chương trình kết thúc. Tuy nhiên, nếu một chương trình phức tạp hơn cấp phát
nhiều bộ nhớ trong một chu kỳ và giữ nó trong thời gian dài, chương trình sẽ sử
dụng nhiều bộ nhớ hơn mức cần thiết và có thể làm quá tải hệ thống, khiến nó hết
bộ nhớ khả dụng.

Việc tạo chu kỳ tham chiếu không dễ dàng thực hiện, nhưng cũng không phải là
không thể. Nếu bạn có các giá trị `RefCell<T>` chứa các giá trị `Rc<T>` hoặc các
kết hợp lồng nhau tương tự của các kiểu với khả biến nội bộ và đếm tham chiếu,
bạn phải đảm bảo rằng bạn không tạo chu kỳ; bạn không thể dựa vào Rust để phát
hiện chúng. Việc tạo một chu kỳ tham chiếu sẽ là một lỗi logic trong chương
trình của bạn mà bạn nên sử dụng các bài kiểm tra tự động, đánh giá mã và các
thực hành phát triển phần mềm khác để giảm thiểu.

Một giải pháp khác để tránh chu kỳ tham chiếu là tổ chức lại cấu trúc dữ liệu
của bạn để một số tham chiếu thể hiện quyền sở hữu và một số tham chiếu không.
Kết quả là, bạn có thể có các chu kỳ được tạo thành từ một số mối quan hệ sở hữu
và một số mối quan hệ không sở hữu, và chỉ có các mối quan hệ sở hữu ảnh hưởng
đến việc liệu một giá trị có thể bị hủy hay không. Trong Listing 15-25, chúng ta
luôn muốn các biến thể `Cons` sở hữu danh sách của chúng, vì vậy việc tổ chức
lại cấu trúc dữ liệu là không thể. Hãy xem một ví dụ sử dụng đồ thị được tạo
thành từ các nút cha và nút con để thấy khi nào các mối quan hệ không sở hữu là
một cách thích hợp để ngăn chặn chu kỳ tham chiếu.

<!-- Old link, do not remove -->

<a id="preventing-reference-cycles-turning-an-rct-into-a-weakt"></a>

### Ngăn Chặn Chu Kỳ Tham Chiếu Sử Dụng `Weak<T>`

Cho đến nay, chúng ta đã chứng minh rằng gọi `Rc::clone` tăng `strong_count` của
một thể hiện `Rc<T>`, và một thể hiện `Rc<T>` chỉ được dọn dẹp nếu
`strong_count` của nó là 0. Bạn cũng có thể tạo tham chiếu yếu đến giá trị trong
một thể hiện `Rc<T>` bằng cách gọi `Rc::downgrade` và truyền một tham chiếu đến
`Rc<T>`. _Tham chiếu mạnh_ là cách bạn có thể chia sẻ quyền sở hữu của một thể
hiện `Rc<T>`. _Tham chiếu yếu_ không thể hiện một mối quan hệ sở hữu, và số
lượng của chúng không ảnh hưởng đến thời điểm một thể hiện `Rc<T>` được dọn dẹp.
Chúng sẽ không gây ra chu kỳ tham chiếu vì bất kỳ chu kỳ nào liên quan đến một
số tham chiếu yếu sẽ bị phá vỡ khi số lượng tham chiếu mạnh của các giá trị liên
quan là 0.

Khi bạn gọi `Rc::downgrade`, bạn nhận được một con trỏ thông minh kiểu
`Weak<T>`. Thay vì tăng `strong_count` trong thể hiện `Rc<T>` lên 1, gọi
`Rc::downgrade` tăng `weak_count` lên 1. Kiểu `Rc<T>` sử dụng `weak_count` để
theo dõi có bao nhiêu tham chiếu `Weak<T>` tồn tại, tương tự như `strong_count`.
Sự khác biệt là `weak_count` không cần phải là 0 để thể hiện `Rc<T>` được dọn
dẹp.

Bởi vì giá trị mà `Weak<T>` tham chiếu đến có thể đã bị hủy, để làm bất cứ điều
gì với giá trị mà một `Weak<T>` đang trỏ đến, bạn phải đảm bảo rằng giá trị vẫn
tồn tại. Làm điều này bằng cách gọi phương thức `upgrade` trên một thể hiện
`Weak<T>`, sẽ trả về một `Option<Rc<T>>`. Bạn sẽ nhận được kết quả là `Some` nếu
giá trị `Rc<T>` chưa bị hủy và kết quả là `None` nếu giá trị `Rc<T>` đã bị hủy.
Bởi vì `upgrade` trả về một `Option<Rc<T>>`, Rust sẽ đảm bảo rằng trường hợp
`Some` và trường hợp `None` được xử lý, và sẽ không có con trỏ không hợp lệ.

Ví dụ, thay vì sử dụng một danh sách mà các mục chỉ biết về mục tiếp theo, chúng
ta sẽ tạo một cây mà các mục biết về các mục con _và_ các mục cha của chúng.

#### Tạo Cấu Trúc Dữ Liệu Cây: Một `Node` với Các Nút Con

Để bắt đầu, chúng ta sẽ xây dựng một cây với các nút biết về các nút con của
chúng. Chúng ta sẽ tạo một struct có tên `Node` chứa giá trị `i32` của riêng nó
cũng như tham chiếu đến các giá trị `Node` con của nó:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:here}}
```

Chúng ta muốn một `Node` sở hữu các con của nó, và chúng ta muốn chia sẻ quyền
sở hữu đó với các biến để chúng ta có thể truy cập trực tiếp vào mỗi `Node`
trong cây. Để làm điều này, chúng ta định nghĩa các mục `Vec<T>` là các giá trị
kiểu `Rc<Node>`. Chúng ta cũng muốn sửa đổi nút nào là con của nút khác, vì vậy
chúng ta có một `RefCell<T>` trong `children` bao quanh `Vec<Rc<Node>>`.

Tiếp theo, chúng ta sẽ sử dụng định nghĩa struct của mình và tạo một thể hiện
`Node` có tên `leaf` với giá trị `3` và không có con, và một thể hiện khác có
tên `branch` với giá trị `5` và `leaf` là một trong những đứa con của nó, như
trong Listing 15-27.

<Listing number="15-27" file-name="src/main.rs" caption="Tạo một nút `leaf` không có con và một nút `branch` với `leaf` là một trong những đứa con của nó">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-27/src/main.rs:there}}
```

</Listing>

Chúng ta sao chép `Rc<Node>` trong `leaf` và lưu trữ nó trong `branch`, có nghĩa
là `Node` trong `leaf` bây giờ có hai chủ sở hữu: `leaf` và `branch`. Chúng ta
có thể đi từ `branch` đến `leaf` thông qua `branch.children`, nhưng không có
cách nào để đi từ `leaf` đến `branch`. Lý do là `leaf` không có tham chiếu đến
`branch` và không biết chúng có liên quan. Chúng ta muốn `leaf` biết rằng
`branch` là cha của nó. Chúng ta sẽ làm điều đó tiếp theo.

#### Thêm Tham Chiếu từ Nút Con đến Nút Cha của Nó

Để làm cho nút con nhận thức được nút cha của nó, chúng ta cần thêm một trường
`parent` vào định nghĩa struct `Node` của chúng ta. Vấn đề là quyết định kiểu
của `parent` nên là gì. Chúng ta biết rằng nó không thể chứa một `Rc<T>`, vì
điều đó sẽ tạo ra một chu kỳ tham chiếu với `leaf.parent` trỏ đến `branch` và
`branch.children` trỏ đến `leaf`, điều này sẽ khiến giá trị `strong_count` của
chúng không bao giờ là 0.

Nghĩ về mối quan hệ theo một cách khác, một nút cha nên sở hữu các con của nó:
nếu một nút cha bị hủy, các nút con của nó cũng nên bị hủy. Tuy nhiên, một nút
con không nên sở hữu nút cha của nó: nếu chúng ta hủy một nút con, nút cha vẫn
nên tồn tại. Đây là một trường hợp cho tham chiếu yếu!

Vì vậy thay vì `Rc<T>`, chúng ta sẽ làm cho kiểu của `parent` sử dụng `Weak<T>`,
cụ thể là một `RefCell<Weak<Node>>`. Bây giờ định nghĩa struct `Node` của chúng
ta trông như thế này:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:here}}
```

Một nút sẽ có thể tham chiếu đến nút cha của nó nhưng không sở hữu nút cha của
nó. Trong Listing 15-28, chúng ta cập nhật `main` để sử dụng định nghĩa mới này
để nút `leaf` sẽ có cách tham chiếu đến nút cha của nó, `branch`.

<Listing number="15-28" file-name="src/main.rs" caption="Một nút `leaf` với tham chiếu yếu đến nút cha của nó `branch`">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-28/src/main.rs:there}}
```

</Listing>

Việc tạo nút `leaf` trông giống với Listing 15-27 ngoại trừ trường `parent`:
`leaf` bắt đầu mà không có cha, vì vậy chúng ta tạo một thể hiện tham chiếu
`Weak<Node>` mới và trống.

Tại thời điểm này, khi chúng ta cố gắng lấy tham chiếu đến cha của `leaf` bằng
cách sử dụng phương thức `upgrade`, chúng ta nhận được giá trị `None`. Chúng ta
thấy điều này trong đầu ra từ câu lệnh `println!` đầu tiên:

```text
leaf parent = None
```

Khi chúng ta tạo nút `branch`, nó cũng sẽ có một tham chiếu `Weak<Node>` mới
trong trường `parent` vì `branch` không có nút cha. Chúng ta vẫn có `leaf` là
một trong những đứa con của `branch`. Một khi chúng ta có thể hiện `Node` trong
`branch`, chúng ta có thể sửa đổi `leaf` để cung cấp cho nó một tham chiếu
`Weak<Node>` đến nút cha của nó. Chúng ta sử dụng phương thức `borrow_mut` trên
`RefCell<Weak<Node>>` trong trường `parent` của `leaf`, và sau đó chúng ta sử
dụng hàm `Rc::downgrade` để tạo một tham chiếu `Weak<Node>` đến `branch` từ
`Rc<Node>` trong `branch`.

Khi chúng ta in cha của `leaf` lần nữa, lần này chúng ta sẽ nhận được một biến
thể `Some` chứa `branch`: bây giờ `leaf` có thể truy cập nút cha của nó! Khi
chúng ta in `leaf`, chúng ta cũng tránh được chu kỳ cuối cùng dẫn đến tràn ngăn
xếp như chúng ta đã có trong Listing 15-26; các tham chiếu `Weak<Node>` được in
dưới dạng `(Weak)`:

```text
leaf parent = Some(Node { value: 5, parent: RefCell { value: (Weak) },
children: RefCell { value: [Node { value: 3, parent: RefCell { value: (Weak) },
children: RefCell { value: [] } }] } })
```

Sự thiếu đầu ra vô hạn cho thấy mã này không tạo ra một chu kỳ tham chiếu. Chúng
ta cũng có thể nhận thấy điều này bằng cách nhìn vào các giá trị mà chúng ta
nhận được từ việc gọi `Rc::strong_count` và `Rc::weak_count`.

#### Trực Quan Hóa Thay Đổi đối với `strong_count` và `weak_count`

Hãy xem giá trị `strong_count` và `weak_count` của các thể hiện `Rc<Node>` thay
đổi như thế nào bằng cách tạo một phạm vi bên trong mới và di chuyển việc tạo
`branch` vào phạm vi đó. Bằng cách làm như vậy, chúng ta có thể thấy điều gì xảy
ra khi `branch` được tạo và sau đó bị hủy khi nó ra khỏi phạm vi. Các sửa đổi
được hiển thị trong Listing 15-29.

<Listing number="15-29" file-name="src/main.rs" caption="Tạo `branch` trong một phạm vi bên trong và kiểm tra số lượng tham chiếu mạnh và yếu">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-29/src/main.rs:here}}
```

</Listing>

Sau khi `leaf` được tạo, `Rc<Node>` của nó có số lượng mạnh là 1 và số lượng yếu
là 0. Trong phạm vi bên trong, chúng ta tạo `branch` và liên kết nó với `leaf`,
tại thời điểm đó khi chúng ta in số lượng, `Rc<Node>` trong `branch` sẽ có số
lượng mạnh là 1 và số lượng yếu là 1 (vì `leaf.parent` trỏ đến `branch` với một
`Weak<Node>`). Khi chúng ta in số lượng trong `leaf`, chúng ta sẽ thấy nó sẽ có
số lượng mạnh là 2 vì `branch` bây giờ có một bản sao của `Rc<Node>` của `leaf`
được lưu trữ trong `branch.children`, nhưng vẫn có số lượng yếu là 0.

Khi phạm vi bên trong kết thúc, `branch` ra khỏi phạm vi và số lượng mạnh của
`Rc<Node>` giảm xuống 0, vì vậy `Node` của nó bị hủy. Số lượng yếu là 1 từ
`leaf.parent` không ảnh hưởng đến việc `Node` có bị hủy hay không, vì vậy chúng
ta không bị rò rỉ bộ nhớ!

Nếu chúng ta cố gắng truy cập vào cha của `leaf` sau khi kết thúc phạm vi, chúng
ta sẽ lại nhận được `None`. Vào cuối chương trình, `Rc<Node>` trong `leaf` có số
lượng mạnh là 1 và số lượng yếu là 0 vì biến `leaf` bây giờ lại là tham chiếu
duy nhất đến `Rc<Node>`.

Tất cả logic quản lý số lượng và hủy giá trị được tích hợp vào `Rc<T>` và
`Weak<T>` và triển khai của chúng về trait `Drop`. Bằng cách chỉ định rằng mối
quan hệ từ một nút con đến nút cha của nó nên là một tham chiếu `Weak<T>` trong
định nghĩa của `Node`, bạn có thể có các nút cha trỏ đến các nút con và ngược
lại mà không tạo ra một chu kỳ tham chiếu và rò rỉ bộ nhớ.

## Tóm Tắt

Chương này đã đề cập đến cách sử dụng con trỏ thông minh để đưa ra các đảm bảo
và đánh đổi khác nhau so với những gì Rust mặc định cung cấp với các tham chiếu
thông thường. Kiểu `Box<T>` có kích thước đã biết và trỏ đến dữ liệu được cấp
phát trên heap. Kiểu `Rc<T>` theo dõi số lượng tham chiếu đến dữ liệu trên heap
để dữ liệu có thể có nhiều chủ sở hữu. Kiểu `RefCell<T>` với khả biến nội bộ của
nó cung cấp cho chúng ta một kiểu mà chúng ta có thể sử dụng khi cần một kiểu
bất biến nhưng cần thay đổi giá trị bên trong của kiểu đó; nó cũng thực thi các
quy tắc mượn trong thời gian chạy thay vì tại thời điểm biên dịch.

Chúng ta cũng đã thảo luận về các trait `Deref` và `Drop`, cho phép nhiều chức
năng của con trỏ thông minh. Chúng ta đã khám phá các chu kỳ tham chiếu có thể
gây ra rò rỉ bộ nhớ và cách ngăn chặn chúng bằng cách sử dụng `Weak<T>`.

Nếu chương này đã kích thích sự quan tâm của bạn và bạn muốn triển khai con trỏ
thông minh của riêng mình, hãy xem ["The Rustonomicon"][nomicon] để biết thêm
thông tin hữu ích.

Tiếp theo, chúng ta sẽ nói về đồng thời trong Rust. Bạn thậm chí sẽ tìm hiểu về
một vài con trỏ thông minh mới.

[nomicon]: ../nomicon/index.html
