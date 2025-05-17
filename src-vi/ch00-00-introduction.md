# Giới thiệu

> Lưu ý: Phiên bản này của cuốn sách giống với [The Rust Programming
> Language][nsprust] có sẵn ở dạng in và sách điện tử từ [No Starch Press][nsp].

[nsprust]: https://nostarch.com/rust-programming-language-3rd-edition
[nsp]: https://nostarch.com/

Chào mừng bạn đến với _Ngôn ngữ lập trình Rust_, một cuốn sách nhập môn về Rust.
Ngôn ngữ lập trình Rust giúp bạn viết phần mềm nhanh hơn và đáng tin cậy hơn.
Tính tiện dụng cấp cao và kiểm soát cấp thấp thường mâu thuẫn với nhau trong
thiết kế ngôn ngữ lập trình; Rust thách thức xung đột đó. Thông qua cân bằng
giữa năng lực kỹ thuật mạnh mẽ và trải nghiệm tuyệt vời của lập trình viên, Rust
cho bạn lựa chọn để kiểm soát chi tiết cấp thấp (như sử dụng bộ nhớ) mà không
gặp phải tất cả những rắc rối thường liên quan đến việc kiểm soát như vậy.

## Rust dành cho ai

Rust lý tưởng cho nhiều người vì nhiều lý do khác nhau. Hãy xem xét một vài nhóm
quan trọng nhất.

### Các nhóm lập trình viên

Rust đang chứng minh là một công cụ hiệu quả cho việc hợp tác giữa các nhóm lớn
gồm các lập trình viên với các mức độ kiến thức lập trình hệ thống khác nhau. Mã
cấp thấp dễ bị các lỗi tinh vi, mà trong hầu hết các ngôn ngữ khác chỉ có thể
phát hiện thông qua kiểm tra kỹ lưỡng và xem xét mã cẩn thận bởi các lập trình
viên có kinh nghiệm. Trong Rust, trình biên dịch đóng vai trò như một người gác
cổng bằng cách từ chối biên dịch mã với những lỗi khó nắm bắt này, bao gồm cả
lỗi đồng thời. Bằng cách làm việc cùng với trình biên dịch, nhóm có thể tập
trung thời gian vào logic của chương trình thay vì săn lùng lỗi.

Rust cũng mang đến các công cụ phát triển hiện đại cho thế giới lập trình hệ
thống:

- Cargo, trình quản lý phụ thuộc và công cụ xây dựng đi kèm, giúp thêm, biên
  dịch và quản lý các phụ thuộc một cách dễ dàng và nhất quán trên toàn bộ hệ
  sinh thái Rust.
- Công cụ định dạng `rustfmt` đảm bảo một phong cách viết mã nhất quán giữa các
  lập trình viên.
- Rust Language Server cung cấp tích hợp với Môi trường Phát triển Tích hợp
  (IDE) cho việc hoàn thành mã và báo lỗi trực tiếp.

Bằng cách sử dụng các công cụ này và các công cụ khác trong hệ sinh thái Rust,
các lập trình viên có thể làm việc hiệu quả trong khi viết mã cấp hệ thống.

### Học sinh

Rust dành cho học sinh và những người quan tâm đến việc học về các khái niệm hệ
thống. Sử dụng Rust, nhiều người đã học về các chủ đề như phát triển hệ điều
hành. Cộng đồng rất chào đón và sẵn sàng trả lời các câu hỏi của học sinh. Thông
qua các nỗ lực như cuốn sách này, các nhóm Rust muốn làm cho các khái niệm hệ
thống trở nên dễ tiếp cận hơn với nhiều người, đặc biệt là những người mới bắt
đầu lập trình.

### Các công ty

Hàng trăm công ty, từ nhỏ đến lớn, sử dụng Rust trong môi trường production cho
nhiều nhiệm vụ khác nhau, bao gồm các công cụ dòng lệnh, dịch vụ web, công cụ
DevOps, thiết bị nhúng, phân tích và chuyển mã âm thanh và video, tiền điện tử,
tin sinh học, công cụ tìm kiếm, ứng dụng Internet of Things, học máy, và thậm
chí là các phần quan trọng của trình duyệt web Firefox.

### Các nhà phát triển mã nguồn mở

Rust dành cho những người muốn xây dựng ngôn ngữ lập trình Rust, cộng đồng, công
cụ phát triển và thư viện. Chúng tôi rất mong bạn đóng góp vào ngôn ngữ Rust.

### Những người coi trọng tốc độ và sự ổn định

Rust dành cho những người khao khát tốc độ và sự ổn định trong một ngôn ngữ. Khi
nói đến tốc độ, chúng tôi muốn nói về cả tốc độ chạy của mã Rust và tốc độ mà
Rust cho phép bạn viết chương trình. Các kiểm tra của trình biên dịch Rust đảm
bảo sự ổn định thông qua việc thêm tính năng và tái cấu trúc. Điều này trái
ngược với mã cũ khó sửa đổi trong các ngôn ngữ không có các kiểm tra này, điều
làm các nhà phát triển thường sợ sửa đổi. Bằng cách cố gắng đạt được các trừu
tượng cấp cao không tốn kém biên dịch thành mã cấp thấp nhanh như mã viết thủ
công - Rust cố gắng làm cho mã an toàn cũng trở thành mã nhanh.

Ngôn ngữ Rust hy vọng hỗ trợ nhiều người dùng khác nữa; những người được đề cập
ở đây chỉ là một số bên liên quan nhất. Nhìn chung, tham vọng lớn nhất của Rust
là loại bỏ các sự đánh đổi mà các lập trình viên đã chấp nhận trong nhiều thập
kỷ bằng cách cung cấp sự an toàn _và_ năng suất, tốc độ _và_ tính tiện dụng. Hãy
thử Rust và xem liệu các lựa chọn của nó có phù hợp với bạn không.

## Cuốn sách này dành cho ai

Cuốn sách này giả định rằng bạn đã viết mã trong một ngôn ngữ lập trình khác,
nhưng không giả định bạn đã viết mã trong ngôn ngữ nào. Chúng tôi đã cố gắng làm
cho tài liệu này dễ tiếp cận rộng rãi với những người có nền tảng lập trình đa
dạng. Chúng tôi không dành nhiều thời gian để nói về lập trình _là gì_ hoặc cách
suy nghĩ về nó. Nếu bạn hoàn toàn mới với lập trình, bạn sẽ được phục vụ tốt hơn
bằng cách đọc một cuốn sách cung cấp một giới thiệu về lập trình.

## Cách sử dụng cuốn sách này

Nói chung, cuốn sách này giả định rằng bạn đang đọc nó theo thứ tự từ đầu đến
cuối. Các chương sau xây dựng trên các khái niệm trong các chương trước, và các
chương trước có thể không đi sâu vào chi tiết về một chủ đề cụ thể nhưng sẽ xem
xét lại chủ đề đó trong một chương sau.

Bạn sẽ tìm thấy hai loại chương trong cuốn sách này: các chương khái niệm và các
chương dự án. Trong các chương khái niệm, bạn sẽ học về một khía cạnh của Rust.
Trong các chương dự án, chúng ta sẽ cùng nhau xây dựng các chương trình nhỏ, áp
dụng những gì bạn đã học được cho đến nay. Chương 2, Chương 12 và Chương 21 là
các chương dự án; phần còn lại là các chương khái niệm.

**Chương 1** giải thích cách cài đặt Rust, cách viết một chương trình “Hello,
world!”, và cách sử dụng Cargo, trình quản lý gói và công cụ xây dựng của Rust.
**Chương 2** là một phần giới thiệu thực hành về việc viết một chương trình
trong Rust, cho bạn xây dựng một trò chơi đoán số. Ở đây, chúng tôi đề cập đến
các khái niệm ở mức cao, và các chương sau sẽ cung cấp thêm chi tiết. Nếu bạn
muốn bắt tay vào làm ngay lập tức, Chương 2 là nơi dành cho bạn. Nếu bạn là một
người học tỉ mỉ đặc biệt thích học mọi thứ chi tiết trước khi tiếp tục, bạn có
thể muốn bỏ qua Chương 2 và đi thẳng đến **Chương 3**, đề cập đến các đặc điểm
của Rust tương tự như các ngôn ngữ lập trình khác; sau đó, bạn có thể quay lại
Chương 2 khi bạn muốn làm việc trên một dự án áp dụng các chi tiết bạn đã học.

Trong **Chương 4** bạn sẽ học về hệ thống quyền sở hữu của Rust. **Chương 5**
thảo luận về struct và phương thức, và **Chương 6** đề cập đến enum, biểu thức
`match`, và cấu trúc luồng điều khiển `if let` và `let...else`. Bạn sẽ sử dụng
struct và enum để tạo các kiểu dữ liệu tùy chỉnh trong Rust.

Trong **Chương 7**, bạn sẽ học về hệ thống module của Rust và về các quy tắc bảo
mật để tổ chức mã của bạn và Giao diện Lập trình Ứng dụng (API) công khai của
nó. **Chương 8** thảo luận về một số cấu trúc dữ liệu collection phổ biến mà thư
viện tiêu chuẩn cung cấp: vector, chuỗi và hash map. **Chương 9** khám phá triết
lý và kỹ thuật xử lý lỗi của Rust.

**Chương 10** đi sâu vào generics, traits và lifetimes, cho bạn khả năng để định
nghĩa mã áp dụng cho nhiều kiểu dữ liệu. **Chương 11** là tất cả về kiểm thử, mà
ngay cả với các đảm bảo an toàn của Rust cũng cần thiết để đảm bảo logic chương
trình của bạn là chính xác. Trong **Chương 12**, chúng ta sẽ triển khai xây dựng
cho riêng mình cho một phần chức năng của công cụ dòng lệnh `grep` tìm kiếm văn
bản trong các tệp. Để làm điều này, chúng ta sẽ sử dụng nhiều khái niệm đã thảo
luận trong các chương trước.

**Chương 13** khám phá closure và iterator: các đặc điểm của Rust đến từ các
ngôn ngữ lập trình hàm. Trong **Chương 14**, chúng ta sẽ xem xét Cargo sâu hơn
và nói về các thực hành tốt nhất để chia sẻ thư viện của bạn với người khác.
**Chương 15** thảo luận về các con trỏ thông minh mà thư viện tiêu chuẩn cung
cấp và các traits cần thiết cho chức năng của chúng.

Trong **Chương 16**, chúng ta sẽ đi qua các mô hình lập trình đồng thời khác
nhau và nói về cách Rust giúp bạn lập trình trong nhiều luồng một cách không sợ
hãi. Trong **Chương 17**, chúng ta xây dựng trên đó bằng cách khám phá cú pháp
async và await của Rust, cùng với các task, future và stream, và mô hình đồng
thời nhẹ mà chúng cho phép.

**Chương 18** xem xét cách các đặc trưng của Rust so sánh với các nguyên tắc lập
trình hướng đối tượng mà bạn có thể quen thuộc. **Chương 19** là một tài liệu
tham khảo về mẫu và khớp mẫu, là những cách mạnh mẽ để biểu đạt ý tưởng trong
suốt các chương trình Rust. **Chương 20** chứa một loạt các chủ đề nâng cao thú
vị, bao gồm Rust không an toàn, macro, và tìm hiểu sâu hơn về lifetimes, traits,
kiểu dữ liệu, hàm và closure.

Trong **Chương 21**, chúng ta sẽ hoàn thành một dự án trong đó chúng ta sẽ triển
khai một máy chủ web đa luồng cấp thấp!

Cuối cùng, một số phụ lục chứa thông tin hữu ích về ngôn ngữ theo định dạng tham
khảo hơn. **Phụ lục A** đề cập đến các từ khóa của Rust, **Phụ lục B** đề cập
đến các toán tử và ký hiệu của Rust, **Phụ lục C** đề cập đến các traits có thể
được dẫn xuất do thư viện tiêu chuẩn cung cấp, **Phụ lục D** đề cập đến một số
công cụ phát triển hữu ích, và **Phụ lục E** giải thích các phiên bản Rust.
Trong **Phụ lục F**, bạn có thể tìm thấy các bản dịch của cuốn sách, và trong
**Phụ lục G** chúng tôi sẽ đề cập đến cách Rust được tạo ra và Rust nightly là
gì.

Không có cách nào sai để đọc cuốn sách này: Nếu bạn muốn bỏ qua, hãy làm điều
đó! Bạn có thể phải quay lại các chương trước nếu bạn gặp bất kỳ sự nhầm lẫn
nào. Nhưng hãy làm bất cứ điều gì phù hợp với bạn.

<span id="ferris"></span>

Một phần quan trọng của quá trình học Rust là học cách đọc các thông báo lỗi mà
trình biên dịch hiển thị: Những thông báo này sẽ hướng dẫn bạn đến mã chạy được.
Vì vậy, chúng tôi sẽ cung cấp nhiều ví dụ không biên dịch cùng với thông báo lỗi
mà trình biên dịch sẽ hiển thị cho bạn trong mỗi tình huống. Hãy biết rằng nếu
bạn nhập và chạy một ví dụ ngẫu nhiên, nó có thể không biên dịch! Hãy chắc chắn
rằng bạn đọc văn bản xung quanh để xem liệu ví dụ bạn đang cố gắng chạy sẽ gây
lỗi hay không. Trong hầu hết các trường hợp, chúng tôi sẽ hướng dẫn bạn đến
phiên bản chính xác của bất kỳ mã nào không biên dịch được. Ferris cũng sẽ giúp
bạn phân biệt mã không chạy được:

| Ferris                                                                                                               | Ý nghĩa                                |
| -------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| <img src="img/ferris/does_not_compile.svg" class="ferris-explain" alt="Ferris với dấu chấm hỏi"/>                    | Mã này không biên dịch!                |
| <img src="img/ferris/panics.svg" class="ferris-explain" alt="Ferris giơ tay lên"/>                                   | Mã này gây hoảng loạn! (panic!)        |
| <img src="img/ferris/not_desired_behavior.svg" class="ferris-explain" alt="Ferris với một móng vuốt lên, nhún vai"/> | Mã này không tạo ra hành vi mong muốn. |

Trong hầu hết các tình huống, chúng tôi sẽ dẫn bạn đến phiên bản chính xác của
bất kỳ mã nào không biên dịch.

## Mã nguồn

Các tệp mã nguồn mà từ đó cuốn sách này được tạo ra có thể được tìm thấy trên
[GitHub][book].

[book]: https://github.com/tuanemdev/rust-book-vn/tree/main/src-vi
