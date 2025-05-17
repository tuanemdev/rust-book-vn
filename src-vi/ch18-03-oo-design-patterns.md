## Triển khai một Mẫu Thiết kế Hướng đối tượng

_Mẫu trạng thái_ (state pattern) là một mẫu thiết kế hướng đối tượng. Trọng tâm
của mẫu này là chúng ta định nghĩa một tập hợp các trạng thái mà một giá trị có
thể có bên trong. Các trạng thái được biểu diễn bởi một tập hợp các _đối tượng
trạng thái_, và hành vi của giá trị thay đổi dựa trên trạng thái của nó. Chúng
ta sẽ tiến hành thực hiện một ví dụ về một struct bài đăng blog có một trường để
lưu trữ trạng thái của nó, trường này sẽ là một đối tượng trạng thái từ tập hợp
"nháp", "đang xét duyệt", hoặc "đã xuất bản".

Các đối tượng trạng thái chia sẻ chức năng: trong Rust, tất nhiên, chúng ta sử
dụng struct và trait thay vì đối tượng và tính kế thừa. Mỗi đối tượng trạng thái
chịu trách nhiệm cho hành vi của chính nó và cho việc quản lý khi nào nó nên
chuyển sang trạng thái khác. Giá trị chứa đối tượng trạng thái không biết gì về
các hành vi khác nhau của các trạng thái hoặc khi nào chuyển đổi giữa các trạng
thái.

Lợi thế của việc sử dụng mẫu trạng thái là khi các yêu cầu nghiệp vụ của chương
trình thay đổi, chúng ta sẽ không cần phải thay đổi mã của giá trị đang giữ
trạng thái hoặc mã sử dụng giá trị đó. Chúng ta sẽ chỉ cần cập nhật mã bên trong
một trong các đối tượng trạng thái để thay đổi quy tắc của nó hoặc có thể thêm
nhiều đối tượng trạng thái hơn.

Đầu tiên, chúng ta sẽ triển khai mẫu trạng thái theo cách hướng đối tượng truyền
thống hơn, sau đó chúng ta sẽ sử dụng một cách tiếp cận tự nhiên hơn một chút
trong Rust. Hãy đi sâu vào việc triển khai dần dần một quy trình làm việc của
bài đăng blog bằng cách sử dụng mẫu trạng thái.

Chức năng cuối cùng sẽ trông như thế này:

1. Một bài đăng blog bắt đầu như một bản nháp trống.
1. Khi bản nháp hoàn thành, một đánh giá của bài đăng được yêu cầu.
1. Khi bài đăng được phê duyệt, nó được xuất bản.
1. Chỉ có các bài đăng blog đã xuất bản mới trả về nội dung để in, vì vậy các
   bài đăng chưa được phê duyệt không thể vô tình được xuất bản.

Bất kỳ thay đổi nào khác được thử trên một bài đăng sẽ không có hiệu lực. Ví dụ,
nếu chúng ta cố gắng phê duyệt một bài đăng blog nháp trước khi chúng ta yêu cầu
đánh giá, bài đăng vẫn sẽ là một bản nháp chưa xuất bản.

### Một Cách Tiếp cận Hướng đối tượng Truyền thống

Có vô số cách để cấu trúc mã để giải quyết cùng một vấn đề, mỗi cách có những
đánh đổi khác nhau. Việc triển khai của phần này thiên về phong cách hướng đối
tượng truyền thống hơn, điều này có thể viết được trong Rust, nhưng không tận
dụng một số điểm mạnh của Rust. Sau đó, chúng ta sẽ trình bày một giải pháp khác
vẫn sử dụng mẫu thiết kế hướng đối tượng nhưng được cấu trúc theo cách có thể
trông ít quen thuộc hơn đối với các lập trình viên có kinh nghiệm hướng đối
tượng. Chúng ta sẽ so sánh hai giải pháp để trải nghiệm những đánh đổi khi thiết
kế mã Rust khác với mã trong các ngôn ngữ khác.

Danh sách 18-11 hiển thị quy trình làm việc này dưới dạng mã: đây là một ví dụ
về việc sử dụng API mà chúng ta sẽ triển khai trong một thư viện crate có tên là
`blog`. Điều này sẽ không biên dịch được vì chúng ta chưa triển khai crate
`blog`.

<Listing number="18-11" file-name="src/main.rs" caption="Mã minh họa hành vi mong muốn mà chúng ta muốn crate `blog` của chúng ta có">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:all}}
```

</Listing>

Chúng ta muốn cho phép người dùng tạo một bài đăng blog nháp mới với
`Post::new`. Chúng ta muốn cho phép thêm văn bản vào bài đăng blog. Nếu chúng ta
cố gắng lấy nội dung của bài đăng ngay lập tức, trước khi phê duyệt, chúng ta sẽ
không nhận được bất kỳ văn bản nào vì bài đăng vẫn đang ở trạng thái nháp. Chúng
ta đã thêm `assert_eq!` trong mã để minh họa. Một bài kiểm tra đơn vị xuất sắc
cho trường hợp này sẽ khẳng định rằng một bài đăng blog nháp trả về một chuỗi
rỗng từ phương thức `content`, nhưng chúng ta sẽ không viết kiểm tra cho ví dụ
này.

Tiếp theo, chúng ta muốn cho phép yêu cầu đánh giá bài đăng, và chúng ta muốn
`content` trả về một chuỗi rỗng trong khi chờ đánh giá. Khi bài đăng nhận được
sự chấp thuận, nó sẽ được xuất bản, nghĩa là văn bản của bài đăng sẽ được trả về
khi gọi `content`.

Lưu ý rằng kiểu duy nhất mà chúng ta tương tác từ crate là kiểu `Post`. Kiểu này
sẽ sử dụng mẫu trạng thái và sẽ chứa một giá trị sẽ là một trong ba đối tượng
trạng thái đại diện cho các trạng thái khác nhau mà một bài đăng có thể có—nháp,
đang xét duyệt, hoặc đã xuất bản. Việc thay đổi từ trạng thái này sang trạng
thái khác sẽ được quản lý nội bộ trong kiểu `Post`. Các trạng thái thay đổi để
đáp ứng với các phương thức được gọi bởi người dùng thư viện của chúng ta trên
thực thể `Post`, nhưng họ không phải quản lý các thay đổi trạng thái trực tiếp.
Ngoài ra, người dùng không thể mắc lỗi với các trạng thái, chẳng hạn như xuất
bản một bài đăng trước khi nó được xem xét.

#### Định nghĩa `Post` và Tạo một Thực thể Mới ở Trạng thái Nháp

Hãy bắt đầu với việc triển khai thư viện! Chúng ta biết rằng chúng ta cần một
struct `Post` công khai chứa một số nội dung, vì vậy chúng ta sẽ bắt đầu với
định nghĩa của struct và một hàm `new` công khai liên quan để tạo một thực thể
của `Post`, như trong Danh sách 18-12. Chúng ta cũng sẽ tạo một trait `State`
riêng tư sẽ định nghĩa hành vi mà tất cả các đối tượng trạng thái cho một `Post`
phải có.

Sau đó, `Post` sẽ chứa một đối tượng trait của `Box<dyn State>` bên trong một
`Option<T>` trong một trường riêng tư có tên là `state` để lưu trữ đối tượng
trạng thái. Bạn sẽ thấy tại sao `Option<T>` là cần thiết sau một chút.

<Listing number="18-12" file-name="src/lib.rs" caption="Định nghĩa của một struct `Post` và một hàm `new` tạo ra một thực thể `Post` mới, một trait `State`, và một struct `Draft`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-12/src/lib.rs}}
```

</Listing>

Trait `State` định nghĩa hành vi được chia sẻ bởi các trạng thái bài đăng khác
nhau. Các đối tượng trạng thái là `Draft`, `PendingReview`, và `Published`, và
tất cả chúng sẽ triển khai trait `State`. Hiện tại, trait không có bất kỳ phương
thức nào, và chúng ta sẽ bắt đầu bằng cách định nghĩa chỉ trạng thái `Draft` vì
đó là trạng thái mà chúng ta muốn một bài đăng bắt đầu.

Khi chúng ta tạo một `Post` mới, chúng ta đặt trường `state` của nó thành một
giá trị `Some` chứa một `Box`. Box này trỏ đến một thực thể mới của struct
`Draft`. Điều này đảm bảo rằng bất cứ khi nào chúng ta tạo một thực thể mới của
`Post`, nó sẽ bắt đầu như một bản nháp. Vì trường `state` của `Post` là riêng
tư, không có cách nào để tạo một `Post` ở bất kỳ trạng thái nào khác! Trong hàm
`Post::new`, chúng ta đặt trường `content` thành một `String` rỗng, mới.

#### Lưu trữ Văn bản của Nội dung Bài đăng

Chúng ta đã thấy trong Danh sách 18-11 rằng chúng ta muốn có thể gọi một phương
thức có tên là `add_text` và truyền cho nó một `&str` để sau đó được thêm vào
như là nội dung văn bản của bài đăng blog. Chúng ta triển khai điều này như một
phương thức, thay vì để lộ trường `content` dưới dạng `pub`, để sau này chúng ta
có thể triển khai một phương thức sẽ kiểm soát cách dữ liệu của trường `content`
được đọc. Phương thức `add_text` khá đơn giản, vì vậy hãy thêm triển khai trong
Danh sách 18-13 vào khối `impl Post`.

<Listing number="18-13" file-name="src/lib.rs" caption="Triển khai phương thức `add_text` để thêm văn bản vào `content` của một bài đăng">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-13/src/lib.rs:here}}
```

</Listing>

Phương thức `add_text` lấy một tham chiếu có thể thay đổi đến `self` vì chúng ta
đang thay đổi thực thể `Post` mà chúng ta đang gọi `add_text` trên đó. Sau đó,
chúng ta gọi `push_str` trên `String` trong `content` và truyền đối số `text` để
thêm vào `content` đã lưu. Hành vi này không phụ thuộc vào trạng thái mà bài
đăng đang ở, vì vậy nó không phải là một phần của mẫu trạng thái. Phương thức
`add_text` không tương tác với trường `state` chút nào, nhưng nó là một phần của
hành vi mà chúng ta muốn hỗ trợ.

#### Đảm bảo Nội dung của một Bài đăng Nháp là Trống

Ngay cả sau khi chúng ta đã gọi `add_text` và thêm một số nội dung vào bài đăng
của chúng ta, chúng ta vẫn muốn phương thức `content` trả về một lát cắt chuỗi
rỗng vì bài đăng vẫn đang ở trạng thái nháp, như được hiển thị trên dòng 7 của
Danh sách 18-11. Hiện tại, hãy triển khai phương thức `content` với thứ đơn giản
nhất sẽ hoàn thành yêu cầu này: luôn trả về một lát cắt chuỗi rỗng. Chúng ta sẽ
thay đổi điều này sau khi chúng ta triển khai khả năng thay đổi trạng thái của
một bài đăng để nó có thể được xuất bản. Cho đến nay, các bài đăng chỉ có thể ở
trạng thái nháp, vì vậy nội dung bài đăng sẽ luôn trống. Danh sách 18-14 hiển
thị triển khai giữ chỗ này.

<Listing number="18-14" file-name="src/lib.rs" caption="Thêm một triển khai giữ chỗ cho phương thức `content` trên `Post` luôn trả về một lát cắt chuỗi rỗng">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-14/src/lib.rs:here}}
```

</Listing>

Với phương thức `content` đã thêm này, mọi thứ trong Danh sách 18-11 cho đến
dòng 7 hoạt động như dự định.

<!-- Old headings. Do not remove or links may break. -->

<a id="requesting-a-review-of-the-post-changes-its-state"></a>

#### Yêu cầu Đánh giá Thay đổi Trạng thái của Bài đăng

Tiếp theo, chúng ta cần thêm chức năng để yêu cầu đánh giá một bài đăng, điều
này sẽ thay đổi trạng thái của nó từ `Draft` sang `PendingReview`. Danh sách
18-15 hiển thị mã này.

<Listing number="18-15" file-name="src/lib.rs" caption="Triển khai các phương thức `request_review` trên `Post` và trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-15/src/lib.rs:here}}
```

</Listing>

Chúng ta cung cấp cho `Post` một phương thức công khai có tên là
`request_review` sẽ lấy một tham chiếu có thể thay đổi đến `self`. Sau đó, chúng
ta gọi một phương thức `request_review` nội bộ trên trạng thái hiện tại của
`Post`, và phương thức `request_review` thứ hai này tiêu thụ trạng thái hiện tại
và trả về một trạng thái mới.

Chúng ta thêm phương thức `request_review` vào trait `State`; tất cả các kiểu
triển khai trait sẽ cần phải triển khai phương thức `request_review`. Lưu ý rằng
thay vì có `self`, `&self`, hoặc `&mut self` làm tham số đầu tiên của phương
thức, chúng ta có `self: Box<Self>`. Cú pháp này có nghĩa là phương thức chỉ hợp
lệ khi được gọi trên một `Box` chứa kiểu. Cú pháp này lấy quyền sở hữu của
`Box<Self>`, làm mất hiệu lực trạng thái cũ để giá trị trạng thái của `Post` có
thể chuyển đổi thành một trạng thái mới.

Để tiêu thụ trạng thái cũ, phương thức `request_review` cần lấy quyền sở hữu của
giá trị trạng thái. Đây là nơi mà `Option` trong trường `state` của `Post` xuất
hiện: chúng ta gọi phương thức `take` để lấy giá trị `Some` ra khỏi trường
`state` và để lại một `None` trong vị trí của nó vì Rust không cho phép chúng ta
có các trường không được điền trong struct. Điều này cho phép chúng ta di chuyển
giá trị `state` ra khỏi `Post` thay vì mượn nó. Sau đó, chúng ta sẽ đặt giá trị
`state` của bài đăng thành kết quả của thao tác này.

Chúng ta cần đặt `state` thành `None` tạm thời thay vì đặt nó trực tiếp với mã
như `self.state = self.state.request_review();` để lấy quyền sở hữu của giá trị
`state`. Điều này đảm bảo `Post` không thể sử dụng giá trị `state` cũ sau khi
chúng ta đã chuyển đổi nó thành một trạng thái mới.

Phương thức `request_review` trên `Draft` trả về một thực thể mới, được đóng hộp
của một struct `PendingReview` mới, đại diện cho trạng thái khi một bài đăng
đang chờ đánh giá. Struct `PendingReview` cũng triển khai phương thức
`request_review` nhưng không thực hiện bất kỳ chuyển đổi nào. Thay vào đó, nó
trả về chính nó vì khi chúng ta yêu cầu đánh giá một bài đăng đã ở trạng thái
`PendingReview`, nó nên tiếp tục ở trạng thái `PendingReview`.

Bây giờ chúng ta có thể bắt đầu thấy những lợi thế của mẫu trạng thái: phương
thức `request_review` trên `Post` giống nhau bất kể giá trị `state` của nó là
gì. Mỗi trạng thái chịu trách nhiệm cho các quy tắc của riêng nó.

Chúng ta sẽ để phương thức `content` trên `Post` như hiện tại, trả về một lát
cắt chuỗi rỗng. Bây giờ chúng ta có thể có một `Post` ở trạng thái
`PendingReview` cũng như ở trạng thái `Draft`, nhưng chúng ta muốn có hành vi
giống nhau ở trạng thái `PendingReview`. Danh sách 18-11 hiện hoạt động cho đến
dòng 10!

<!-- Old headings. Do not remove or links may break. -->

<a id="adding-the-approve-method-that-changes-the-behavior-of-content"></a>

#### Thêm `approve` để Thay đổi Hành vi của `content`

Phương thức `approve` sẽ tương tự như phương thức `request_review`: nó sẽ đặt
`state` thành giá trị mà trạng thái hiện tại nói rằng nó nên có khi trạng thái
đó được phê duyệt, như được hiển thị trong Danh sách 18-16.

<Listing number="18-16" file-name="src/lib.rs" caption="Triển khai phương thức `approve` trên `Post` và trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-16/src/lib.rs:here}}
```

</Listing>

Chúng ta thêm phương thức `approve` vào trait `State` và thêm một struct mới
triển khai `State`, trạng thái `Published`.

Tương tự như cách `request_review` trên `PendingReview` hoạt động, nếu chúng ta
gọi phương thức `approve` trên một `Draft`, nó sẽ không có hiệu lực gì vì
`approve` sẽ trả về `self`. Khi chúng ta gọi `approve` trên `PendingReview`, nó
trả về một thực thể mới, được đóng hộp của struct `Published`. Struct
`Published` triển khai trait `State`, và đối với cả phương thức `request_review`
và phương thức `approve`, nó trả về chính nó vì bài đăng nên ở trong trạng thái
`Published` trong các trường hợp đó.

Bây giờ chúng ta cần cập nhật phương thức `content` trên `Post`. Chúng ta muốn
giá trị trả về từ `content` phụ thuộc vào trạng thái hiện tại của `Post`, vì vậy
chúng ta sẽ để `Post` ủy quyền cho một phương thức `content` được định nghĩa
trên `state` của nó, như được hiển thị trong Danh sách 18-17.

<Listing number="18-17" file-name="src/lib.rs" caption="Cập nhật phương thức `content` trên `Post` để ủy quyền cho một phương thức `content` trên `State`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch18-oop/listing-18-17/src/lib.rs:here}}
```

</Listing>

Vì mục tiêu là giữ tất cả các quy tắc này bên trong các struct triển khai
`State`, chúng ta gọi một phương thức `content` trên giá trị trong `state` và
truyền thực thể bài đăng (tức là `self`) như một đối số. Sau đó, chúng ta trả về
giá trị được trả về từ việc sử dụng phương thức `content` trên giá trị `state`.

Chúng ta gọi phương thức `as_ref` trên `Option` vì chúng ta muốn một tham chiếu
đến giá trị bên trong `Option` chứ không phải quyền sở hữu của giá trị. Vì
`state` là một `Option<Box<dyn State>>`, khi chúng ta gọi `as_ref`, một
`Option<&Box<dyn State>>` được trả về. Nếu chúng ta không gọi `as_ref`, chúng ta
sẽ gặp lỗi vì chúng ta không thể di chuyển `state` ra khỏi `&self` được mượn của
tham số hàm.

Sau đó, chúng ta gọi phương thức `unwrap`, mà chúng ta biết sẽ không bao giờ gây
panic vì chúng ta biết các phương thức trên `Post` đảm bảo rằng `state` sẽ luôn
chứa một giá trị `Some` khi các phương thức đó hoàn thành. Đây là một trong
những trường hợp chúng ta đã nói đến trong ["Các trường hợp Trong Đó Bạn Có Thêm
Thông tin Hơn Trình biên dịch"][more-info-than-rustc]<!-- ignore --> trong
Chương 9 khi chúng ta biết rằng một giá trị `None` không bao giờ có thể xảy ra,
mặc dù trình biên dịch không thể hiểu điều đó.

Ở thời điểm này, khi chúng ta gọi `content` trên `&Box<dyn State>`, ép buộc giải
tham chiếu sẽ có hiệu lực trên `&` và `Box` để phương thức `content` cuối cùng
sẽ được gọi trên kiểu triển khai trait `State`. Điều đó có nghĩa là chúng ta cần
thêm `content` vào định nghĩa trait `State`, và đó là nơi chúng ta sẽ đặt logic
cho nội dung nào sẽ trả về tùy thuộc vào trạng thái nào chúng ta có, như được
hiển thị trong Danh sách 18-18.

<Listing number="18-18" file-name="src/lib.rs" caption="Thêm phương thức `content` vào trait `State`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-18/src/lib.rs:here}}
```

</Listing>

Chúng ta thêm một triển khai mặc định cho phương thức `content` trả về một lát
cắt chuỗi rỗng. Điều đó có nghĩa là chúng ta không cần triển khai `content` trên
các struct `Draft` và `PendingReview`. Struct `Published` sẽ ghi đè phương thức
`content` và trả về giá trị trong `post.content`. Mặc dù thuận tiện, việc có
phương thức `content` trên `State` quyết định `content` của `Post` đang làm mờ
ranh giới giữa trách nhiệm của `State` và trách nhiệm của `Post`.

Lưu ý rằng chúng ta cần chú thích thời gian sống cho phương thức này, như chúng
ta đã thảo luận trong Chương 10. Chúng ta đang lấy một tham chiếu đến một `post`
làm đối số và trả về một tham chiếu đến một phần của `post` đó, vì vậy thời gian
sống của tham chiếu trả về có liên quan đến thời gian sống của đối số `post`.

Và chúng ta đã hoàn thành—tất cả Danh sách 18-11 hiện hoạt động! Chúng ta đã
triển khai mẫu trạng thái với các quy tắc của quy trình làm việc bài đăng blog.
Logic liên quan đến các quy tắc nằm trong các đối tượng trạng thái thay vì bị
phân tán khắp `Post`.

> ### Tại sao Không Phải Một Enum?
>
> Bạn có thể đã tự hỏi tại sao chúng ta không sử dụng một `enum` với các biến
> thể trạng thái bài đăng khác nhau. Đó chắc chắn là một giải pháp có thể; hãy
> thử nó và so sánh kết quả cuối cùng để xem bạn thích cái nào hơn! Một nhược
> điểm của việc sử dụng enum là mỗi nơi kiểm tra giá trị của enum sẽ cần một
> biểu thức `match` hoặc tương tự để xử lý mọi biến thể có thể. Điều này có thể
> trở nên lặp lại hơn so với giải pháp đối tượng trait này.

#### Nhược điểm của Mẫu Trạng thái

Chúng ta đã chỉ ra rằng Rust có khả năng triển khai mẫu trạng thái hướng đối
tượng để đóng gói các loại hành vi khác nhau mà một bài đăng nên có trong mỗi
trạng thái. Các phương thức trên `Post` không biết gì về các hành vi khác nhau.
Cách chúng ta tổ chức mã, chúng ta chỉ phải nhìn vào một nơi để biết các cách
khác nhau mà một bài đăng đã xuất bản có thể hành xử: triển khai của trait
`State` trên struct `Published`.

Nếu chúng ta tạo một triển khai thay thế không sử dụng mẫu trạng thái, thay vào
đó chúng ta có thể sử dụng các biểu thức `match` trong các phương thức trên
`Post` hoặc thậm chí trong mã `main` kiểm tra trạng thái của bài đăng và thay
đổi hành vi ở những nơi đó. Điều đó có nghĩa là chúng ta sẽ phải nhìn vào nhiều
nơi để hiểu tất cả các hàm ý của việc một bài đăng ở trạng thái đã xuất bản.

Với mẫu trạng thái, các phương thức `Post` và những nơi chúng ta sử dụng `Post`
không cần các biểu thức `match`, và để thêm một trạng thái mới, chúng ta chỉ cần
thêm một struct mới và triển khai các phương thức trait trên struct đó ở một vị
trí.

Việc triển khai sử dụng mẫu trạng thái rất dễ mở rộng để thêm chức năng. Để thấy
sự đơn giản của việc duy trì mã sử dụng mẫu trạng thái, hãy thử một vài gợi ý
này:

- Thêm một phương thức `reject` thay đổi trạng thái của bài đăng từ
  `PendingReview` trở lại `Draft`.
- Yêu cầu hai lần gọi `approve` trước khi trạng thái có thể được thay đổi thành
  `Published`.
- Chỉ cho phép người dùng thêm nội dung văn bản khi bài đăng ở trạng thái
  `Draft`. Gợi ý: để đối tượng trạng thái chịu trách nhiệm cho những gì có thể
  thay đổi về nội dung nhưng không chịu trách nhiệm sửa đổi `Post`.

Một nhược điểm của mẫu trạng thái là, vì các trạng thái triển khai các chuyển
đổi giữa các trạng thái, một số trạng thái được kết nối với nhau. Nếu chúng ta
thêm một trạng thái khác giữa `PendingReview` và `Published`, chẳng hạn như
`Scheduled`, chúng ta sẽ phải thay đổi mã trong `PendingReview` để chuyển sang
`Scheduled` thay vì `Published`. Sẽ ít công việc hơn nếu `PendingReview` không
cần thay đổi với việc thêm một trạng thái mới, nhưng điều đó có nghĩa là chuyển
sang một mẫu thiết kế khác.

Một nhược điểm khác là chúng ta đã lặp lại một số logic. Để loại bỏ một số sự
lặp lại, chúng ta có thể thử tạo các triển khai mặc định cho các phương thức
`request_review` và `approve` trên trait `State` trả về `self`. Tuy nhiên, điều
này sẽ không hoạt động: khi sử dụng `State` như một đối tượng trait, trait không
biết `self` cụ thể sẽ là gì chính xác, vì vậy kiểu trả về không được biết tại
thời điểm biên dịch. (Đây là một trong những quy tắc tương thích `dyn` được đề
cập trước đó.)

Sự lặp lại khác bao gồm các triển khai tương tự của các phương thức
`request_review` và `approve` trên `Post`. Cả hai phương thức đều sử dụng
`Option::take` với trường `state` của `Post`, và nếu `state` là `Some`, chúng ủy
quyền cho triển khai của cùng một phương thức trong giá trị được bọc và đặt giá
trị mới của trường `state` thành kết quả. Nếu chúng ta có nhiều phương thức trên
`Post` tuân theo mẫu này, chúng ta có thể cân nhắc định nghĩa một macro để loại
bỏ sự lặp lại (xem ["Macro"][macros]<!-- ignore --> trong Chương 20).

Bằng cách triển khai mẫu trạng thái chính xác như được định nghĩa cho các ngôn
ngữ hướng đối tượng, chúng ta không tận dụng hết các điểm mạnh của Rust như
chúng ta có thể. Hãy xem xét một số thay đổi chúng ta có thể thực hiện đối với
crate `blog` có thể biến các trạng thái và chuyển đổi không hợp lệ thành các lỗi
thời điểm biên dịch.

### Mã hóa Trạng thái và Hành vi dưới dạng Kiểu

Chúng ta sẽ chỉ cho bạn cách suy nghĩ lại về mẫu trạng thái để có được một tập
hợp các đánh đổi khác. Thay vì đóng gói hoàn toàn các trạng thái và chuyển đổi
để mã bên ngoài không có kiến thức về chúng, chúng ta sẽ mã hóa các trạng thái
thành các kiểu khác nhau. Do đó, hệ thống kiểm tra kiểu của Rust sẽ ngăn chặn
các nỗ lực sử dụng bài đăng nháp ở những nơi chỉ có bài đăng đã xuất bản được
phép bằng cách đưa ra lỗi trình biên dịch.

Hãy xem xét phần đầu tiên của `main` trong Danh sách 18-11:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-11/src/main.rs:here}}
```

</Listing>

Chúng ta vẫn cho phép tạo bài đăng mới ở trạng thái nháp bằng `Post::new` và khả
năng thêm văn bản vào nội dung của bài đăng. Nhưng thay vì có một phương thức
`content` trên bài đăng nháp trả về một chuỗi rỗng, chúng ta sẽ làm cho bài đăng
nháp không có phương thức `content` chút nào. Bằng cách đó, nếu chúng ta cố gắng
lấy nội dung của một bài đăng nháp, chúng ta sẽ nhận được lỗi trình biên dịch
cho chúng ta biết rằng phương thức không tồn tại. Kết quả là, sẽ không thể cho
chúng ta vô tình hiển thị nội dung bài đăng nháp trong sản xuất vì mã đó thậm
chí sẽ không biên dịch. Danh sách 18-19 hiển thị định nghĩa của một struct
`Post` và một struct `DraftPost`, cũng như các phương thức trên mỗi struct.

<Listing number="18-19" file-name="src/lib.rs" caption="Một `Post` với một phương thức `content` và `DraftPost` không có phương thức `content`">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-19/src/lib.rs}}
```

</Listing>

Cả hai struct `Post` và `DraftPost` đều có một trường riêng tư `content` lưu trữ
văn bản của bài đăng blog. Các struct không còn có trường `state` vì chúng ta
đang di chuyển việc mã hóa trạng thái vào các kiểu của struct. Struct `Post` sẽ
đại diện cho một bài đăng đã xuất bản, và nó có một phương thức `content` trả về
`content`.

Chúng ta vẫn có một hàm `Post::new`, nhưng thay vì trả về một thực thể của
`Post`, nó trả về một thực thể của `DraftPost`. Vì `content` là riêng tư và
không có bất kỳ hàm nào trả về `Post`, hiện tại không thể tạo một thực thể của
`Post`.

Struct `DraftPost` có một phương thức `add_text`, vì vậy chúng ta có thể thêm
văn bản vào `content` như trước, nhưng lưu ý rằng `DraftPost` không có phương
thức `content` được định nghĩa! Vì vậy, bây giờ chương trình đảm bảo tất cả các
bài đăng bắt đầu là bài đăng nháp, và bài đăng nháp không có nội dung của chúng
sẵn có để hiển thị. Bất kỳ nỗ lực nào để vượt qua các ràng buộc này sẽ dẫn đến
lỗi trình biên dịch.

<!-- Old headings. Do not remove or links may break. -->

<a id="implementing-transitions-as-transformations-into-different-types"></a>

#### Triển khai Chuyển đổi dưới dạng Chuyển đổi thành Các Kiểu Khác nhau

Vậy làm thế nào để chúng ta có được một bài đăng đã xuất bản? Chúng ta muốn thực
thi quy tắc rằng một bài đăng nháp phải được xem xét và phê duyệt trước khi nó
có thể được xuất bản. Một bài đăng trong trạng thái chờ xem xét vẫn không nên
hiển thị bất kỳ nội dung nào. Hãy triển khai các ràng buộc này bằng cách thêm
một struct khác, `PendingReviewPost`, định nghĩa phương thức `request_review`
trên `DraftPost` để trả về một `PendingReviewPost` và định nghĩa một phương thức
`approve` trên `PendingReviewPost` để trả về một `Post`, như được hiển thị trong
Danh sách 18-20.

<Listing number="18-20" file-name="src/lib.rs" caption="Một `PendingReviewPost` được tạo ra bằng cách gọi `request_review` trên `DraftPost` và một phương thức `approve` biến `PendingReviewPost` thành một `Post` đã xuất bản">

```rust,noplayground
{{#rustdoc_include ../listings/ch18-oop/listing-18-20/src/lib.rs:here}}
```

</Listing>

Các phương thức `request_review` và `approve` lấy quyền sở hữu của `self`, do đó
tiêu thụ các thực thể `DraftPost` và `PendingReviewPost` và chuyển đổi chúng
thành một `PendingReviewPost` và một `Post` đã xuất bản, tương ứng. Bằng cách
này, chúng ta sẽ không có bất kỳ thực thể `DraftPost` nào còn tồn tại sau khi
chúng ta đã gọi `request_review` trên chúng, và tương tự. Struct
`PendingReviewPost` không có phương thức `content` được định nghĩa trên nó, vì
vậy việc cố gắng đọc nội dung của nó dẫn đến lỗi trình biên dịch, cũng giống như
với `DraftPost`. Vì cách duy nhất để có được một thực thể `Post` đã xuất bản có
phương thức `content` được định nghĩa là gọi phương thức `approve` trên một
`PendingReviewPost`, và cách duy nhất để có được một `PendingReviewPost` là gọi
phương thức `request_review` trên một `DraftPost`, chúng ta đã mã hóa quy trình
làm việc của bài đăng blog vào hệ thống kiểu.

Nhưng chúng ta cũng phải thực hiện một số thay đổi nhỏ đối với `main`. Các
phương thức `request_review` và `approve` trả về các thực thể mới thay vì sửa
đổi struct mà chúng được gọi trên, vì vậy chúng ta cần thêm nhiều phép gán
`let post =` che khuất để lưu các thực thể trả về. Chúng ta cũng không thể có
các xác nhận về nội dung của bài đăng nháp và bài đăng đang chờ xem xét là các
chuỗi rỗng, và chúng ta cũng không cần chúng: chúng ta không thể biên dịch mã cố
gắng sử dụng nội dung của bài đăng ở những trạng thái đó nữa. Mã đã cập nhật
trong `main` được hiển thị trong Danh sách 18-21.

<Listing number="18-21" file-name="src/main.rs" caption="Các sửa đổi đối với `main` để sử dụng triển khai mới của quy trình làm việc bài đăng blog">

```rust,ignore
{{#rustdoc_include ../listings/ch18-oop/listing-18-21/src/main.rs}}
```

</Listing>

Các thay đổi chúng ta cần thực hiện đối với `main` để gán lại `post` có nghĩa là
việc triển khai này không hoàn toàn tuân theo mẫu trạng thái hướng đối tượng
nữa: các chuyển đổi giữa các trạng thái không còn được đóng gói hoàn toàn trong
triển khai `Post`. Tuy nhiên, lợi ích của chúng ta là các trạng thái không hợp
lệ hiện không thể xảy ra nhờ vào hệ thống kiểu và việc kiểm tra kiểu xảy ra tại
thời điểm biên dịch! Điều này đảm bảo rằng các lỗi nhất định, chẳng hạn như hiển
thị nội dung của một bài đăng chưa xuất bản, sẽ được phát hiện trước khi chúng
xuất hiện trong sản xuất.

Thử các nhiệm vụ được đề xuất ở đầu phần này trên crate `blog` như nó là sau
Danh sách 18-21 để xem bạn nghĩ gì về thiết kế của phiên bản mã này. Lưu ý rằng
một số nhiệm vụ có thể đã được hoàn thành trong thiết kế này.

Chúng ta đã thấy rằng mặc dù Rust có khả năng triển khai các mẫu thiết kế hướng
đối tượng, các mẫu khác, chẳng hạn như mã hóa trạng thái vào hệ thống kiểu, cũng
có sẵn trong Rust. Các mẫu này có các đánh đổi khác nhau. Mặc dù bạn có thể rất
quen thuộc với các mẫu hướng đối tượng, việc suy nghĩ lại về vấn đề để tận dụng
các tính năng của Rust có thể mang lại lợi ích, chẳng hạn như ngăn chặn một số
lỗi tại thời điểm biên dịch. Các mẫu hướng đối tượng sẽ không phải lúc nào cũng
là giải pháp tốt nhất trong Rust do một số tính năng nhất định, như quyền sở
hữu, mà các ngôn ngữ hướng đối tượng không có.

## Tóm tắt

Bất kể việc bạn có cho rằng Rust là một ngôn ngữ hướng đối tượng sau khi đọc
chương này hay không, bây giờ bạn biết rằng bạn có thể sử dụng các đối tượng
trait để có được một số tính năng hướng đối tượng trong Rust. Điều phối động có
thể cung cấp cho mã của bạn một số tính linh hoạt để đổi lấy một chút hiệu suất
thời gian chạy. Bạn có thể sử dụng tính linh hoạt này để triển khai các mẫu
hướng đối tượng có thể giúp cải thiện khả năng bảo trì của mã. Rust cũng có các
tính năng khác, như quyền sở hữu, mà các ngôn ngữ hướng đối tượng không có. Một
mẫu hướng đối tượng sẽ không phải lúc nào cũng là cách tốt nhất để tận dụng các
điểm mạnh của Rust, nhưng đó là một tùy chọn có sẵn.

Tiếp theo, chúng ta sẽ xem xét các mẫu, là một tính năng khác của Rust cho phép
nhiều tính linh hoạt. Chúng ta đã xem xét chúng một cách ngắn gọn trong suốt
cuốn sách nhưng chưa thấy được khả năng đầy đủ của chúng. Hãy tiếp tục!

[more-info-than-rustc]:
  ch09-03-to-panic-or-not-to-panic.html#cases-in-which-you-have-more-information-than-the-compiler
[macros]: ch20-05-macros.html#macros
