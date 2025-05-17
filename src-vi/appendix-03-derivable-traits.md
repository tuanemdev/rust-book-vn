## Phụ lục C: Các Trait có thể dẫn xuất

Trong nhiều phần của cuốn sách này, chúng ta đã thảo luận về thuộc tính
`derive`, có thể áp dụng cho định nghĩa struct hoặc enum. Thuộc tính `derive`
tạo ra mã sẽ triển khai một trait với triển khai mặc định của nó trên kiểu mà
bạn đã chú thích với cú pháp `derive` (dẫn xuất).

Trong phụ lục này, chúng tôi cung cấp một tài liệu tham khảo về tất cả các trait
trong thư viện chuẩn mà bạn có thể sử dụng với `derive`. Mỗi phần bao gồm:

- Những toán tử và phương thức nào mà trait này sẽ kích hoạt
- Triển khai của trait được cung cấp bởi `derive` làm gì
- Việc triển khai trait báo hiệu điều gì về kiểu đó
- Những điều kiện mà bạn được phép hoặc không được phép triển khai trait
- Các ví dụ về các hoạt động yêu cầu trait này

Nếu bạn muốn có hành vi khác với hành vi được cung cấp bởi thuộc tính `derive`,
hãy tham khảo
[tài liệu thư viện chuẩn](https://doc.rust-lang.org/std/index.html)<!-- ignore -->
cho mỗi trait để biết chi tiết về cách triển khai chúng thủ công.

Các trait được liệt kê ở đây là những trait được định nghĩa bởi thư viện chuẩn
mà có thể được triển khai cho các kiểu của bạn bằng cách sử dụng `derive`. Các
trait khác được định nghĩa trong thư viện chuẩn không có hành vi mặc định hợp
lý, vì vậy việc triển khai chúng theo cách phù hợp với mục đích của bạn là tùy
thuộc vào bạn.

Một ví dụ về trait không thể được derive là `Display`, xử lý việc định dạng cho
người dùng cuối. Bạn nên luôn cân nhắc cách thích hợp để hiển thị một kiểu cho
người dùng cuối. Những phần nào của kiểu mà người dùng cuối nên được phép xem?
Những phần nào họ sẽ thấy liên quan? Định dạng dữ liệu nào sẽ liên quan nhất đối
với họ? Trình biên dịch Rust không có cái nhìn sâu sắc này, vì vậy nó không thể
cung cấp hành vi mặc định phù hợp cho bạn.

Danh sách các trait có thể derive được cung cấp trong phụ lục này không toàn
diện: Các thư viện có thể triển khai `derive` cho các trait của riêng họ, làm
cho danh sách các trait mà bạn có thể sử dụng `derive` thực sự mở rộng. Việc
triển khai `derive` liên quan đến việc sử dụng một macro thủ tục, được đề cập
trong phần ["Macro `derive` tuỳ chỉnh"][custom-derive-macros]<!-- ignore -->
trong Chương 20.

### `Debug` cho đầu ra dành cho lập trình viên

Trait `Debug` cho phép định dạng gỡ lỗi trong chuỗi định dạng, bạn chỉ định bằng
cách thêm `:?` trong các placeholder `{}`.

Trait `Debug` cho phép bạn in các thể hiện của một kiểu cho mục đích gỡ lỗi, để
bạn và các lập trình viên khác sử dụng kiểu của bạn có thể kiểm tra một thể hiện
tại một điểm cụ thể trong quá trình thực thi của chương trình.

Trait `Debug` được yêu cầu, ví dụ, khi sử dụng macro `assert_eq!`. Macro này in
ra giá trị của các thể hiện được đưa ra làm đối số nếu phép khẳng định bằng thất
bại để các lập trình viên có thể thấy tại sao hai thể hiện không bằng nhau.

### `PartialEq` và `Eq` cho so sánh bằng

Trait `PartialEq` cho phép bạn so sánh các thể hiện của một kiểu để kiểm tra sự
bằng nhau và cho phép sử dụng các toán tử `==` và `!=`.

Việc derive `PartialEq` triển khai phương thức `eq`. Khi `PartialEq` được derive
trên các struct, hai thể hiện chỉ bình đẳng khi _tất cả_ các trường bằng nhau,
và các thể hiện không bằng nhau nếu _bất kỳ_ trường nào không bằng nhau. Khi
được derive trên enum, mỗi biến thể bằng nhau với chính nó và không bằng nhau
với các biến thể khác.

Trait `PartialEq` là cần thiết, ví dụ, khi sử dụng macro `assert_eq!`, cần có
khả năng so sánh hai thể hiện của một kiểu về sự bằng nhau.

Trait `Eq` không có phương thức nào. Mục đích của nó là báo hiệu rằng với mọi
giá trị của kiểu được chú thích, giá trị đó bằng chính nó. Trait `Eq` chỉ có thể
được áp dụng cho các kiểu cũng triển khai `PartialEq`, mặc dù không phải tất cả
các kiểu triển khai `PartialEq` đều có thể triển khai `Eq`. Một ví dụ về điều
này là các kiểu số thực dấu phẩy động: Việc triển khai số thực dấu phẩy động nêu
rõ rằng hai thể hiện của giá trị không phải một số (`NaN`) không bằng nhau.

Một ví dụ về khi `Eq` được yêu cầu là đối với các khóa trong `HashMap<K, V>` để
`HashMap<K, V>` có thể biết liệu hai khóa có giống nhau hay không.

### `PartialOrd` và `Ord` cho so sánh thứ tự

Trait `PartialOrd` cho phép bạn so sánh các thể hiện của một kiểu để sắp xếp.
Một kiểu triển khai `PartialOrd` có thể được sử dụng với các toán tử `<`, `>`,
`<=` và `>=`. Bạn chỉ có thể áp dụng trait `PartialOrd` cho các kiểu cũng triển
khai `PartialEq`.

Việc derive `PartialOrd` triển khai phương thức `partial_cmp`, trả về một
`Option<Ordering>` sẽ là `None` khi các giá trị được đưa ra không tạo ra một thứ
tự. Một ví dụ về giá trị không tạo ra thứ tự, mặc dù hầu hết các giá trị của
kiểu đó có thể được so sánh, là giá trị số dấu phẩy động `NaN`. Gọi
`partial_cmp` với bất kỳ số dấu phẩy động nào và giá trị dấu phẩy động `NaN` sẽ
trả về `None`.

Khi được derive trên struct, `PartialOrd` so sánh hai thể hiện bằng cách so sánh
giá trị trong mỗi trường theo thứ tự mà các trường xuất hiện trong định nghĩa
struct. Khi được derive trên enum, các biến thể của enum được khai báo sớm hơn
trong định nghĩa enum được coi là nhỏ hơn các biến thể được liệt kê sau đó.

Trait `PartialOrd` được yêu cầu, ví dụ, cho phương thức `gen_range` từ crate
`rand` tạo ra một giá trị ngẫu nhiên trong phạm vi được chỉ định bởi một biểu
thức phạm vi.

Trait `Ord` cho phép bạn biết rằng đối với bất kỳ hai giá trị nào của kiểu được
chú thích, một thứ tự hợp lệ sẽ tồn tại. Trait `Ord` triển khai phương thức
`cmp`, trả về một `Ordering` thay vì một `Option<Ordering>` vì một thứ tự hợp lệ
sẽ luôn có thể xảy ra. Bạn chỉ có thể áp dụng trait `Ord` cho các kiểu cũng
triển khai `PartialOrd` và `Eq` (và `Eq` yêu cầu `PartialEq`). Khi được derive
trên struct và enum, `cmp` hoạt động giống như triển khai được derive cho
`partial_cmp` với `PartialOrd`.

Một ví dụ về khi `Ord` được yêu cầu là khi lưu trữ giá trị trong `BTreeSet<T>`,
một cấu trúc dữ liệu lưu trữ dữ liệu dựa trên thứ tự sắp xếp của các giá trị.

### `Clone` và `Copy` để Sao chép Giá trị

Trait `Clone` cho phép bạn tạo một bản sao sâu của một giá trị một cách rõ ràng,
và quá trình sao chép có thể liên quan đến việc chạy mã tùy ý và sao chép dữ
liệu trên heap. Xem phần [Biến và Dữ liệu Tương tác với
Clone][variables-and-data-interacting-with-clone]<!-- ignore --> trong Chương 4
để biết thêm thông tin về `Clone`.

Việc derive `Clone` triển khai phương thức `clone`, khi được triển khai cho toàn
bộ kiểu, gọi `clone` trên từng phần của kiểu. Điều này có nghĩa là tất cả các
trường hoặc giá trị trong kiểu cũng phải triển khai `Clone` để derive `Clone`.

Một ví dụ về khi `Clone` được yêu cầu là khi gọi phương thức `to_vec` trên một
slice. Slice không sở hữu các thể hiện kiểu mà nó chứa, nhưng vector trả về từ
`to_vec` sẽ cần sở hữu các thể hiện của nó, vì vậy `to_vec` gọi `clone` trên mỗi
mục. Do đó, kiểu được lưu trữ trong slice phải triển khai `Clone`.

Trait `Copy` cho phép bạn sao chép một giá trị bằng cách chỉ sao chép các bit
được lưu trữ trên ngăn xếp; không cần mã tùy ý nào. Xem phần ["Dữ liệu
Chỉ-Ngăn-Xếp: Copy"][stack-only-data-copy]<!-- ignore --> trong Chương 4 để biết
thêm thông tin về `Copy`.

Trait `Copy` không định nghĩa bất kỳ phương thức nào để ngăn chặn các lập trình
viên ghi đè các phương thức đó và vi phạm giả định rằng không có mã tùy ý nào
đang được chạy. Bằng cách đó, tất cả các lập trình viên có thể giả định rằng
việc sao chép một giá trị sẽ rất nhanh.

Bạn có thể derive `Copy` trên bất kỳ kiểu nào mà tất cả các phần của nó đều
triển khai `Copy`. Một kiểu triển khai `Copy` cũng phải triển khai `Clone` bởi
vì một kiểu triển khai `Copy` có một triển khai tầm thường của `Clone` thực hiện
cùng một nhiệm vụ như `Copy`.

Trait `Copy` hiếm khi được yêu cầu; các kiểu triển khai `Copy` có các tối ưu hóa
khả dụng, có nghĩa là bạn không phải gọi `clone`, điều này làm cho mã ngắn gọn
hơn.

Mọi thứ có thể làm được với `Copy` bạn cũng có thể thực hiện với `Clone`, nhưng
mã có thể chậm hơn hoặc phải sử dụng `clone` ở nhiều nơi.

### `Hash` để Ánh xạ một Giá trị thành một Giá trị có Kích thước Cố định

Trait `Hash` cho phép bạn lấy một thể hiện của một kiểu có kích thước tùy ý và
ánh xạ thể hiện đó thành một giá trị có kích thước cố định bằng cách sử dụng một
hàm băm. Việc derive `Hash` triển khai phương thức `hash`. Triển khai được
derive của phương thức `hash` kết hợp kết quả của việc gọi `hash` trên từng phần
của kiểu, có nghĩa là tất cả các trường hoặc giá trị cũng phải triển khai `Hash`
để derive `Hash`.

Một ví dụ về khi `Hash` được yêu cầu là trong việc lưu trữ khóa trong
`HashMap<K, V>` để lưu trữ dữ liệu một cách hiệu quả.

### `Default` cho Giá trị Mặc định

Trait `Default` cho phép bạn tạo một giá trị mặc định cho một kiểu. Việc derive
`Default` triển khai hàm `default`. Triển khai được derive của hàm `default` gọi
hàm `default` trên từng phần của kiểu, có nghĩa là tất cả các trường hoặc giá
trị trong kiểu cũng phải triển khai `Default` để derive `Default`.

Hàm `Default::default` thường được sử dụng kết hợp với cú pháp cập nhật struct
được thảo luận trong phần ["Tạo Thể hiện từ Các Thể hiện Khác với Cú pháp Cập
nhật
Struct"][creating-instances-from-other-instances-with-struct-update-syntax]<!-- ignore -->
trong Chương 5. Bạn có thể tùy chỉnh một vài trường của một struct và sau đó
thiết lập và sử dụng giá trị mặc định cho phần còn lại của các trường bằng cách
sử dụng `..Default::default()`.

Trait `Default` được yêu cầu khi bạn sử dụng phương thức `unwrap_or_default`
trên các thể hiện `Option<T>`, ví dụ. Nếu `Option<T>` là `None`, phương thức
`unwrap_or_default` sẽ trả về kết quả của `Default::default` cho kiểu `T` được
lưu trữ trong `Option<T>`.

[creating-instances-from-other-instances-with-struct-update-syntax]:
  ch05-01-defining-structs.html#creating-instances-from-other-instances-with-struct-update-syntax
[stack-only-data-copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
[variables-and-data-interacting-with-clone]:
  ch04-01-what-is-ownership.html#variables-and-data-interacting-with-clone
[custom-derive-macros]: ch20-05-macros.html#custom-derive-macros
