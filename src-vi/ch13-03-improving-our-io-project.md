## Cải thiện Dự án I/O của chúng ta

Với kiến thức mới về iterators, chúng ta có thể cải thiện dự án I/O trong Chương
12 bằng cách sử dụng iterators để làm cho những phần trong mã nguồn trở nên rõ
ràng và ngắn gọn hơn. Hãy xem cách iterators có thể cải thiện cách triển khai
hàm `Config::build` và hàm `search` của chúng ta.

### Loại bỏ `clone` bằng cách sử dụng Iterator

Trong Listing 12-6, chúng ta đã thêm mã lấy một slice của các giá trị `String`
và tạo một thể hiện của struct `Config` bằng cách lập chỉ mục vào slice và sao
chép các giá trị, cho phép struct `Config` sở hữu những giá trị đó. Trong
Listing 13-17, chúng ta đã tái hiện lại cách triển khai hàm `Config::build` như
trong Listing 12-23.

<Listing number="13-17" file-name="src/main.rs" caption="Reproduction of the `Config::build` function from Listing 12-23">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-23-reproduced/src/main.rs:ch13}}
```

</Listing>

Vào thời điểm đó, chúng ta đã nói đừng lo lắng về việc gọi `clone` không hiệu
quả vì chúng ta sẽ loại bỏ chúng trong tương lai. Vâng, thời điểm đó là bây giờ!

Chúng ta cần `clone` ở đây vì chúng ta có một slice với các phần tử `String`
trong tham số `args`, nhưng hàm `build` không sở hữu `args`. Để trả về quyền sở
hữu của một thể hiện `Config`, chúng ta đã phải sao chép giá trị từ trường
`query` và `file_path` của `Config` để thể hiện `Config` có thể sở hữu các giá
trị của nó.

Với kiến thức mới của chúng ta về iterators, chúng ta có thể thay đổi hàm
`build` để nhận quyền sở hữu một iterator làm đối số thay vì mượn một slice.
Chúng ta sẽ sử dụng chức năng của iterator thay vì mã kiểm tra độ dài của slice
và lập chỉ mục vào các vị trí cụ thể. Điều này sẽ làm rõ những gì hàm
`Config::build` đang làm vì iterator sẽ truy cập các giá trị.

Một khi `Config::build` nắm quyền sở hữu iterator và ngừng sử dụng các thao tác
lập chỉ mục đang mượn, chúng ta có thể di chuyển các giá trị `String` từ
iterator vào `Config` thay vì gọi `clone` và tạo một phân bổ mới.

#### Sử dụng Iterator trả về trực tiếp

Mở file _src/main.rs_ của dự án I/O, nó sẽ trông như thế này:

<span class="filename">Filename: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-12-24-reproduced/src/main.rs:ch13}}
```

Đầu tiên chúng ta sẽ thay đổi phần đầu của hàm `main` mà chúng ta đã có trong
Listing 12-24 thành mã trong Listing 13-18, lần này sử dụng một iterator. Điều
này sẽ không biên dịch cho đến khi chúng ta cũng cập nhật `Config::build`.

<Listing number="13-18" file-name="src/main.rs" caption="Passing the return value of `env::args` to `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-18/src/main.rs:here}}
```

</Listing>

Hàm `env::args` trả về một iterator! Thay vì thu thập các giá trị iterator vào
một vector và sau đó truyền một slice cho `Config::build`, bây giờ chúng ta đang
truyền quyền sở hữu của iterator được trả về từ `env::args` trực tiếp cho
`Config::build`.

Tiếp theo, chúng ta cần cập nhật định nghĩa của `Config::build`. Hãy thay đổi
chữ ký của `Config::build` để trông giống như Listing 13-19. Điều này vẫn chưa
biên dịch được, vì chúng ta cần cập nhật phần thân hàm.

<Listing number="13-19" file-name="src/main.rs" caption="Updating the signature of `Config::build` to expect an iterator">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-19/src/main.rs:here}}
```

</Listing>

Tài liệu thư viện tiêu chuẩn cho hàm `env::args` cho thấy rằng kiểu của iterator
nó trả về là `std::env::Args`, và kiểu đó triển khai trait `Iterator` và trả về
các giá trị `String`.

Chúng ta đã cập nhật chữ ký của hàm `Config::build` để tham số `args` có một
kiểu tổng quát với ràng buộc trait `impl Iterator<Item = String>` thay vì
`&[String]`. Cách sử dụng cú pháp `impl Trait` mà chúng ta đã thảo luận trong
phần ["Traits as Parameters"][impl-trait]<!-- ignore --> của Chương 10 có nghĩa
là `args` có thể là bất kỳ kiểu nào triển khai trait `Iterator` và trả về các
mục `String`.

Bởi vì chúng ta đang lấy quyền sở hữu của `args` và chúng ta sẽ thay đổi `args`
bằng cách lặp qua nó, chúng ta có thể thêm từ khóa `mut` vào chỉ định của tham
số `args` để làm cho nó có thể thay đổi.

#### Sử dụng các phương thức Trait `Iterator` thay vì lập chỉ mục

Tiếp theo, chúng ta sẽ sửa phần thân của `Config::build`. Bởi vì `args` triển
khai trait `Iterator`, chúng ta biết rằng chúng ta có thể gọi phương thức `next`
trên nó! Listing 13-20 cập nhật mã từ Listing 12-23 để sử dụng phương thức
`next`.

<Listing number="13-20" file-name="src/main.rs" caption="Changing the body of `Config::build` to use iterator methods">

```rust,ignore,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-20/src/main.rs:here}}
```

</Listing>

Hãy nhớ rằng giá trị đầu tiên trong giá trị trả về của `env::args` là tên của
chương trình. Chúng ta muốn bỏ qua nó và chuyển đến giá trị tiếp theo, vì vậy
đầu tiên chúng ta gọi `next` và không làm gì với giá trị trả về. Tiếp theo chúng
ta gọi `next` để lấy giá trị mà chúng ta muốn đặt vào trường `query` của
`Config`. Nếu `next` trả về `Some`, chúng ta sử dụng `match` để trích xuất giá
trị. Nếu nó trả về `None`, điều đó có nghĩa là không đủ đối số được đưa ra và
chúng ta trả về sớm với một giá trị `Err`. Chúng ta cũng làm tương tự cho giá
trị `file_path`.

### Làm cho mã rõ ràng hơn với bộ điều hợp Iterator

Chúng ta cũng có thể tận dụng iterators trong hàm `search` trong dự án I/O của
chúng ta, được tái hiện ở đây trong Listing 13-21 như trong Listing 12-19.

<Listing number="13-21" file-name="src/lib.rs" caption="The implementation of the `search` function from Listing 12-19">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-19/src/lib.rs:ch13}}
```

</Listing>

Chúng ta có thể viết mã này một cách ngắn gọn hơn bằng cách sử dụng phương thức
bộ điều hợp iterator. Làm như vậy cũng cho phép chúng ta tránh phải có một
vector `results` trung gian có thể thay đổi. Phong cách lập trình hàm ưu tiên
việc giảm thiểu lượng trạng thái có thể thay đổi để làm cho mã rõ ràng hơn. Loại
bỏ trạng thái có thể thay đổi có thể cho phép một cải tiến trong tương lai để
làm cho việc tìm kiếm diễn ra song song vì chúng ta sẽ không phải quản lý truy
cập đồng thời vào vector `results`. Listing 13-22 cho thấy sự thay đổi này.

<Listing number="13-22" file-name="src/lib.rs" caption="Using iterator adapter methods in the implementation of the `search` function">

```rust,ignore
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-22/src/lib.rs:here}}
```

</Listing>

Nhớ lại rằng mục đích của hàm `search` là trả về tất cả các dòng trong
`contents` có chứa `query`. Tương tự như ví dụ `filter` trong Listing 13-16, mã
này sử dụng bộ điều hợp `filter` để chỉ giữ lại các dòng mà
`line.contains(query)` trả về `true`. Sau đó, chúng ta thu thập các dòng phù hợp
vào một vector khác với `collect`. Đơn giản hơn nhiều! Hãy tự nhiên thực hiện
thay đổi tương tự để sử dụng phương thức iterator trong hàm
`search_case_insensitive`.

Để cải tiến thêm, hãy trả về một iterator từ hàm `search` bằng cách loại bỏ lệnh
gọi `collect` và thay đổi kiểu trả về thành `impl Iterator<Item = &'a str>` để
hàm trở thành một bộ điều hợp iterator. Lưu ý rằng bạn cũng sẽ cần cập nhật các
test! Tìm kiếm qua một file lớn bằng công cụ `minigrep` của bạn trước và sau khi
thực hiện thay đổi này để quan sát sự khác biệt trong hành vi. Trước thay đổi
này, chương trình sẽ không in bất kỳ kết quả nào cho đến khi nó đã thu thập tất
cả kết quả, nhưng sau thay đổi, kết quả sẽ được in khi mỗi dòng phù hợp được tìm
thấy vì vòng lặp `for` trong hàm `run` có thể tận dụng tính lười biếng của
iterator.

<!-- Old heading. Do not remove or links may break. -->

<a id="choosing-between-loops-or-iterators"></a>

### Lựa chọn giữa Vòng lặp và Iterators

Câu hỏi hợp lý tiếp theo là phong cách nào bạn nên chọn trong mã của riêng bạn
và tại sao: triển khai ban đầu trong Listing 13-21 hoặc phiên bản sử dụng
iterators trong Listing 13-22 (giả sử chúng ta đang thu thập tất cả kết quả
trước khi trả về chúng thay vì trả về iterator). Hầu hết các lập trình viên Rust
đều thích sử dụng phong cách iterator. Nó hơi khó nắm bắt lúc đầu, nhưng một khi
bạn đã cảm nhận được các bộ điều hợp iterator khác nhau và những gì chúng làm,
iterators có thể dễ hiểu hơn. Thay vì phải vật lộn với các chi tiết khác nhau
của vòng lặp và xây dựng vector mới, mã tập trung vào mục tiêu cấp cao của vòng
lặp. Điều này trừu tượng hóa một số mã thông thường để dễ dàng hơn khi thấy các
khái niệm độc đáo cho mã này, chẳng hạn như điều kiện lọc mỗi phần tử trong
iterator phải vượt qua.

Nhưng hai cách triển khai có thực sự tương đương không? Giả định trực giác có
thể là vòng lặp cấp thấp hơn sẽ nhanh hơn. Hãy nói về hiệu suất.

[impl-trait]: ch10-02-traits.html#traits-as-parameters
