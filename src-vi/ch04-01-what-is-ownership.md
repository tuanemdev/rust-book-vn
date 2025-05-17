## Quyền Sở Hữu Là Gì?

_Quyền sở hữu (Ownership)_ là một tập hợp các quy tắc chi phối cách chương trình
Rust quản lý bộ nhớ. Tất cả các chương trình đều phải quản lý cách sử dụng bộ
nhớ máy tính khi chạy. Một số ngôn ngữ có cơ chế thu gom rác (garbage
collection) thường xuyên tìm kiếm bộ nhớ không còn được sử dụng khi chương trình
đang chạy; trong các ngôn ngữ khác, lập trình viên phải cấp phát và giải phóng
bộ nhớ một cách rõ ràng. Rust sử dụng cách tiếp cận thứ ba: bộ nhớ được quản lý
thông qua hệ thống quyền sở hữu với một tập hợp các quy tắc mà trình biên dịch
kiểm tra. Nếu bất kỳ quy tắc nào bị vi phạm, chương trình sẽ không biên dịch
được. Không có tính năng nào của quyền sở hữu sẽ làm chậm chương trình khi nó
đang chạy.

Vì quyền sở hữu là một khái niệm mới đối với nhiều lập trình viên, nên cần một
khoảng thời gian để làm quen. Tin tốt là càng có nhiều kinh nghiệm với Rust và
các quy tắc của hệ thống quyền sở hữu, bạn sẽ càng dễ dàng phát triển một cách
tự nhiên các đoạn mã an toàn và hiệu quả. Hãy cứ tiếp tục!

Khi bạn hiểu quyền sở hữu, bạn sẽ có một nền tảng vững chắc để hiểu các tính
năng khiến Rust trở nên độc đáo. Trong chương này, bạn sẽ học về quyền sở hữu
thông qua làm việc với một số ví dụ tập trung vào một cấu trúc dữ liệu rất phổ
biến: chuỗi (strings).

> ### Stack và Heap
>
> Nhiều ngôn ngữ lập trình không yêu cầu bạn phải suy nghĩ về stack và heap
> thường xuyên. Nhưng trong một ngôn ngữ lập trình hệ thống như Rust, việc một
> giá trị nằm trên stack hay heap ảnh hưởng đến cách ngôn ngữ hoạt động và lý do
> bạn phải đưa ra những quyết định nhất định. Các phần của quyền sở hữu sẽ được
> mô tả liên quan đến stack và heap sau trong chương này, vì vậy đây là một giải
> thích ngắn gọn để chuẩn bị.
>
> Cả stack và heap đều là những phần của bộ nhớ có sẵn để mã của bạn sử dụng
> trong thời gian chạy, nhưng chúng được cấu trúc theo những cách khác nhau.
> Stack lưu trữ các giá trị theo thứ tự nhận được và xóa các giá trị theo thứ tự
> ngược lại. Điều này được gọi là _vào sau, ra trước (last in, first out)_. Hãy
> nghĩ về một chồng đĩa: khi bạn thêm đĩa, bạn đặt chúng lên trên cùng của
> chồng, và khi bạn cần một cái đĩa, bạn lấy một cái từ trên cùng. Việc thêm
> hoặc xóa đĩa từ giữa hoặc đáy sẽ không hiệu quả! Việc thêm dữ liệu được gọi là
> _đẩy vào stack_, và việc xóa dữ liệu được gọi là _lấy ra khỏi stack_. Tất cả
> dữ liệu được lưu trữ trên stack phải có kích thước đã biết và cố định. Dữ liệu
> có kích thước không xác định tại thời điểm biên dịch hoặc kích thước có thể
> thay đổi phải được lưu trữ trên heap thay thế.
>
> Heap ít có tổ chức hơn: khi bạn đặt dữ liệu trên heap, bạn yêu cầu một lượng
> không gian nhất định. Bộ phân bổ bộ nhớ tìm một chỗ trống trong heap đủ lớn,
> đánh dấu nó là đang sử dụng, và trả về một _con trỏ_, đó là địa chỉ của vị trí
> đó. Quá trình này được gọi là _cấp phát trên heap_ và đôi khi được viết tắt là
> chỉ _cấp phát_ (việc đẩy các giá trị vào stack không được coi là cấp phát). Vì
> con trỏ đến heap có kích thước đã biết và cố định, bạn có thể lưu trữ con trỏ
> trên stack, nhưng khi bạn muốn dữ liệu thực tế, bạn phải đi theo con trỏ. Hãy
> nghĩ về việc được sắp xếp chỗ ngồi tại một nhà hàng. Khi bạn vào, bạn nói số
> người trong nhóm của bạn, và người phục vụ tìm một bàn trống vừa đủ cho mọi
> người và dẫn bạn đến đó. Nếu ai đó trong nhóm của bạn đến muộn, họ có thể hỏi
> bạn đã được xếp chỗ ở đâu để tìm bạn.
>
> Đẩy vào stack nhanh hơn cấp phát trên heap vì bộ cấp phát không bao giờ phải
> tìm kiếm vị trí để lưu trữ dữ liệu mới; vị trí đó luôn ở đỉnh của stack. So
> sánh, việc cấp phát không gian trên heap đòi hỏi nhiều công việc hơn vì bộ cấp
> phát phải trước tiên tìm một không gian đủ lớn để chứa dữ liệu và sau đó thực
> hiện các công việc quản lý để chuẩn bị cho lần cấp phát tiếp theo.
>
> Truy cập dữ liệu trong heap thường chậm hơn truy cập dữ liệu trên stack vì bạn
> phải đi theo con trỏ để đến đó. Các bộ xử lý hiện đại nhanh hơn nếu chúng nhảy
> ít hơn trong bộ nhớ. Tiếp tục ví dụ, hãy xem xét một người phục vụ tại nhà
> hàng đang nhận đơn đặt hàng từ nhiều bàn. Hiệu quả nhất là lấy tất cả đơn đặt
> hàng tại một bàn trước khi chuyển sang bàn tiếp theo. Việc nhận đơn đặt hàng
> từ bàn A, sau đó nhận đơn đặt hàng từ bàn B, sau đó quay lại A, và sau đó lại
> đến bàn B sẽ là một quá trình chậm hơn nhiều. Tương tự, bộ xử lý thường có thể
> làm công việc của mình tốt hơn nếu nó làm việc với dữ liệu gần với dữ liệu
> khác (như trên stack) thay vì xa hơn (như có thể trên heap).
>
> Khi mã của bạn gọi một hàm, các giá trị được truyền vào hàm (bao gồm cả các
> con trỏ đến dữ liệu trên heap) và các biến cục bộ của hàm được đẩy vào stack.
> Khi hàm kết thúc, những giá trị này được lấy ra khỏi stack.
>
> Theo dõi những phần mã nào đang sử dụng dữ liệu nào trên heap, giảm thiểu
> lượng dữ liệu trùng lặp trên heap, và dọn sạch dữ liệu không sử dụng trên heap
> để bạn không bị hết không gian, tất cả đều là những vấn đề mà quyền sở hữu
> giải quyết. Một khi bạn hiểu quyền sở hữu, bạn sẽ không cần phải nghĩ về stack
> và heap thường xuyên nữa, nhưng biết rằng mục đích chính của quyền sở hữu là
> quản lý dữ liệu heap có thể giúp giải thích lý do tại sao nó hoạt động như
> vậy.

### Các Quy Tắc Quyền Sở Hữu

Đầu tiên, hãy xem xét các quy tắc quyền sở hữu. Ghi nhớ những quy tắc này khi
chúng ta làm việc với các ví dụ minh họa:

- Mỗi giá trị trong Rust có một _chủ sở hữu_.
- Tại một thời điểm chỉ có thể có một chủ sở hữu.
- Khi chủ sở hữu ra khỏi phạm vi, giá trị sẽ bị hủy.

### Phạm Vi Biến

Bây giờ chúng ta đã vượt qua cú pháp Rust cơ bản, chúng ta sẽ không bao gồm tất
cả mã `fn main() {` trong các ví dụ, vì vậy nếu bạn đang làm theo, hãy đảm bảo
đặt các ví dụ sau vào hàm `main` một cách thủ công. Do đó, các ví dụ của chúng
ta sẽ ngắn gọn hơn một chút, cho phép chúng ta tập trung vào các chi tiết thực
tế hơn là mã boilerplate.

Như một ví dụ đầu tiên về quyền sở hữu, chúng ta sẽ xem xét _phạm vi_ của một số
biến. Một phạm vi là phạm vi trong một chương trình mà một mục có giá trị. Xét
biến sau:

```rust
let s = "hello";
```

Biến `s` tham chiếu đến một chuỗi chữ, trong đó giá trị của chuỗi được mã hóa
cứng vào văn bản của chương trình của chúng ta. Biến có giá trị từ điểm mà nó
được khai báo cho đến khi kết thúc _phạm vi_ hiện tại. Listing 4-1 cho thấy một
chương trình với các chú thích cho biết nơi biến `s` sẽ có giá trị.

<Listing number="4-1" caption="Một biến và phạm vi mà nó có giá trị">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-01/src/main.rs:here}}
```

</Listing>

Nói cách khác, có hai điểm quan trọng về thời gian ở đây:

- Khi `s` _vào_ phạm vi, nó có giá trị.
- Nó vẫn có giá trị cho đến khi nó _ra khỏi_ phạm vi.

Ở điểm này, mối quan hệ giữa phạm vi và thời điểm các biến có giá trị tương tự
như trong các ngôn ngữ lập trình khác. Bây giờ chúng ta sẽ xây dựng dựa trên
hiểu biết này bằng cách giới thiệu kiểu dữ liệu `String`.

### Kiểu `String`

Để minh họa các quy tắc của quyền sở hữu, chúng ta cần một kiểu dữ liệu phức tạp
hơn những gì chúng ta đã đề cập trong phần ["Kiểu Dữ
Liệu"][data-types]<!-- ignore --> của Chương 3. Các kiểu được đề cập trước đó có
kích thước đã biết, có thể được lưu trữ trên stack và lấy ra khỏi stack khi phạm
vi của chúng kết thúc, và có thể được sao chép nhanh chóng và dễ dàng để tạo một
phiên bản độc lập mới nếu một phần khác của mã cần sử dụng cùng giá trị trong
phạm vi khác. Nhưng chúng ta muốn xem xét dữ liệu được lưu trữ trên heap và khám
phá cách Rust biết khi nào dọn dẹp dữ liệu đó, và kiểu `String` là một ví dụ
tuyệt vời.

Chúng ta sẽ tập trung vào các phần của `String` liên quan đến quyền sở hữu.
Những khía cạnh này cũng áp dụng cho các kiểu dữ liệu phức tạp khác, dù chúng
được cung cấp bởi thư viện chuẩn hay được tạo bởi bạn. Chúng ta sẽ thảo luận về
`String` sâu hơn trong [Chương 8][ch8]<!-- ignore -->.

Chúng ta đã thấy các chuỗi chữ, trong đó giá trị chuỗi được mã hóa cứng vào
chương trình của chúng ta. Chuỗi chữ rất tiện lợi, nhưng chúng không phù hợp cho
mọi tình huống mà chúng ta muốn sử dụng văn bản. Một lý do là chúng không thay
đổi được. Một lý do khác là không phải mọi giá trị chuỗi đều có thể biết khi
chúng ta viết mã của mình: ví dụ, nếu chúng ta muốn lấy đầu vào từ người dùng và
lưu trữ nó? Đối với những tình huống này, Rust có một kiểu chuỗi thứ hai,
`String`. Kiểu này quản lý dữ liệu được cấp phát trên heap và do đó có thể lưu
trữ một lượng văn bản không xác định đối với chúng ta tại thời điểm biên dịch.
Bạn có thể tạo một `String` từ một chuỗi chữ bằng cách sử dụng hàm `from`, như
sau:

```rust
let s = String::from("hello");
```

Toán tử hai dấu hai chấm `::` cho phép chúng ta đặt tên hàm `from` cụ thể này
dưới kiểu `String` thay vì sử dụng một loại tên như `string_from`. Chúng ta sẽ
thảo luận về cú pháp này nhiều hơn trong phần ["Cú pháp Phương
thức"][method-syntax]<!-- ignore --> của Chương 5, và khi chúng ta nói về không
gian tên với các mô-đun trong ["Đường dẫn để Tham chiếu đến một Mục trong Cây
Mô-đun"][paths-module-tree]<!-- ignore --> trong Chương 7.

Loại chuỗi này _có thể_ thay đổi:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-01-can-mutate-string/src/main.rs:here}}
```

Vậy, sự khác biệt ở đây là gì? Tại sao `String` có thể thay đổi được nhưng các
chuỗi chữ thì không? Sự khác biệt nằm ở cách hai loại này xử lý bộ nhớ.

### Bộ Nhớ và Cấp Phát

Trong trường hợp của chuỗi chữ, chúng ta biết nội dung tại thời điểm biên dịch,
vì vậy văn bản được mã hóa cứng trực tiếp vào tệp thực thi cuối cùng. Đây là lý
do tại sao chuỗi chữ nhanh chóng và hiệu quả. Nhưng những thuộc tính này chỉ có
từ tính không thay đổi của chuỗi chữ. Tiếc thay, chúng ta không thể đặt một khối
bộ nhớ vào tệp nhị phân cho mỗi đoạn văn bản có kích thước không xác định tại
thời điểm biên dịch và có kích thước có thể thay đổi trong khi chạy chương
trình.

Với kiểu `String`, để hỗ trợ một đoạn văn bản có thể thay đổi và phát triển,
chúng ta cần phân bổ một lượng bộ nhớ trên heap, không xác định tại thời điểm
biên dịch, để chứa nội dung. Điều này có nghĩa:

- Bộ nhớ phải được yêu cầu từ bộ phân bổ bộ nhớ tại thời điểm chạy.
- Chúng ta cần một cách để trả lại bộ nhớ này cho bộ cấp phát khi chúng ta đã
  xong với `String` của mình.

Phần đầu tiên được thực hiện bởi chúng ta: khi chúng ta gọi `String::from`, việc
triển khai của nó yêu cầu bộ nhớ mà nó cần. Điều này khá phổ biến trong lập
trình ngôn ngữ.

Tuy nhiên, phần thứ hai là khác nhau. Trong các ngôn ngữ có _bộ thu gom rác
(GC)_, GC theo dõi và dọn dẹp bộ nhớ không còn được sử dụng nữa, và chúng ta
không cần phải nghĩ về nó. Trong hầu hết các ngôn ngữ không có GC, trách nhiệm
của chúng ta là xác định khi nào bộ nhớ không còn được sử dụng và gọi mã để giải
phóng nó một cách rõ ràng, giống như chúng ta đã yêu cầu nó. Làm điều này một
cách chính xác về mặt lịch sử là một vấn đề lập trình khó khăn. Nếu chúng ta
quên, chúng ta sẽ lãng phí bộ nhớ. Nếu chúng ta làm quá sớm, chúng ta sẽ có một
biến không hợp lệ. Nếu chúng ta làm hai lần, đó cũng là một lỗi. Chúng ta cần
ghép chính xác một `allocate` với chính xác một `free`.

Rust đi theo một con đường khác: bộ nhớ được trả lại tự động một khi biến sở hữu
nó ra khỏi phạm vi. Dưới đây là một phiên bản của ví dụ phạm vi từ Listing 4-1
sử dụng một `String` thay vì một chuỗi chữ:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-02-string-scope/src/main.rs:here}}
```

Có một điểm tự nhiên mà chúng ta có thể trả lại bộ nhớ mà `String` của chúng ta
cần cho bộ cấp phát: khi `s` ra khỏi phạm vi. Khi một biến ra khỏi phạm vi, Rust
gọi một hàm đặc biệt cho chúng ta. Hàm này được gọi là
[`drop`][drop]<!-- ignore -->, và đó là nơi tác giả của `String` có thể đặt mã
để trả lại bộ nhớ. Rust gọi `drop` tự động tại dấu ngoặc nhọn đóng.

> Lưu ý: Trong C++, mô hình này của việc phân bổ tài nguyên tại cuối của vòng
> đời của một mục đôi khi được gọi là _Resource Acquisition Is Initialization
> (RAII)_. Chức năng `drop` trong Rust sẽ quen thuộc với bạn nếu bạn đã sử dụng
> các mẫu RAII.

Mô hình này có một tác động sâu sắc đến cách mã Rust được viết. Nó có vẻ đơn
giản ngay bây giờ, nhưng hành vi của mã có thể bất ngờ trong các tình huống phức
tạp hơn khi chúng ta muốn có nhiều biến sử dụng dữ liệu chúng ta đã cấp phát
trên heap. Hãy khám phá một số tình huống đó ngay bây giờ.

<!-- Old heading. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-move"></a>

#### Các Biến và Dữ Liệu Tương Tác với Move

Nhiều biến có thể tương tác với cùng một dữ liệu theo những cách khác nhau trong
Rust. Hãy xem một ví dụ sử dụng một số nguyên trong Listing 4-2.

<Listing number="4-2" caption="Gán giá trị số nguyên của biến `x` cho `y`">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-02/src/main.rs:here}}
```

</Listing>

Chúng ta có thể đoán được đoạn mã này đang làm gì: "gán giá trị `5` cho `x`; sau
đó tạo một bản sao của giá trị trong `x` và gán nó cho `y`." Bây giờ chúng ta có
hai biến, `x` và `y`, và cả hai đều bằng `5`. Đây thực sự là những gì đang xảy
ra, bởi vì số nguyên là các giá trị đơn giản với kích thước đã biết, cố định, và
hai giá trị `5` này được đẩy vào stack.

Bây giờ hãy xem phiên bản `String`:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-03-string-move/src/main.rs:here}}
```

Điều này trông rất giống, vì vậy chúng ta có thể giả định rằng cách nó hoạt động
sẽ giống nhau: nghĩa là, dòng thứ hai sẽ tạo một bản sao của giá trị trong `s1`
và gán nó cho `s2`. Nhưng điều này không hoàn toàn là những gì xảy ra.

Hãy xem Hình 4-1 để xem điều gì đang xảy ra với `String` bên dưới bề mặt. Một
`String` bao gồm ba phần, được hiển thị ở bên trái: một con trỏ đến bộ nhớ chứa
nội dung của chuỗi, một độ dài, và một dung lượng. Nhóm dữ liệu này được lưu trữ
trên stack. Ở bên phải là bộ nhớ trên heap chứa nội dung.

<img alt="Hai bảng: bảng đầu tiên chứa biểu diễn của s1 trên
stack, bao gồm độ dài (5), dung lượng (5), và một con trỏ đến giá trị đầu tiên
trong bảng thứ hai. Bảng thứ hai chứa biểu diễn của
dữ liệu chuỗi trên heap, byte theo byte." src="img/trpl04-01.svg" class="center"
style="width: 50%;" />

<span class="caption">Hình 4-1: Biểu diễn trong bộ nhớ của một `String` chứa giá
trị `"hello"` được gắn với `s1`</span>

Độ dài là lượng bộ nhớ, tính bằng byte, mà nội dung của `String` đang sử dụng
hiện tại. Dung lượng là tổng lượng bộ nhớ, tính bằng byte, mà `String` đã nhận
từ bộ cấp phát. Sự khác biệt giữa độ dài và dung lượng là quan trọng, nhưng
không phải trong ngữ cảnh này, vì vậy hiện tại, việc bỏ qua dung lượng là ổn.

Khi chúng ta gán `s1` cho `s2`, dữ liệu `String` được sao chép, nghĩa là chúng
ta sao chép con trỏ, độ dài và dung lượng nằm trên stack. Chúng ta không sao
chép dữ liệu trên heap mà con trỏ trỏ tới. Nói cách khác, biểu diễn dữ liệu
trong bộ nhớ trông giống như Hình 4-2.

<img alt="Ba bảng: bảng s1 và s2 đại diện cho các chuỗi đó trên
stack, tương ứng, và cả hai đều trỏ đến cùng một dữ liệu chuỗi trên heap."
src="img/trpl04-02.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 4-2: Biểu diễn trong bộ nhớ của biến `s2` có một bản
sao của con trỏ, độ dài và dung lượng của `s1`</span>

Biểu diễn _không_ trông giống như Hình 4-3, đây là cách bộ nhớ sẽ trông như thế
nào nếu Rust cũng sao chép dữ liệu heap. Nếu Rust làm điều này, thao tác
`s2 = s1` có thể rất tốn kém về hiệu suất thời gian chạy nếu dữ liệu trên heap
lớn.

<img alt="Bốn bảng: hai bảng đại diện cho dữ liệu stack cho s1 và s2,
và mỗi bảng trỏ đến bản sao riêng của dữ liệu chuỗi trên heap."
src="img/trpl04-03.svg" class="center" style="width: 50%;" />

<span class="caption">Hình 4-3: Một khả năng khác cho những gì `s2 = s1` có thể
làm nếu Rust cũng sao chép dữ liệu heap</span>

Trước đó, chúng ta đã nói rằng khi một biến ra khỏi phạm vi, Rust tự động gọi
hàm `drop` và dọn sạch bộ nhớ heap cho biến đó. Nhưng Hình 4-2 cho thấy cả hai
con trỏ dữ liệu đều trỏ đến cùng một vị trí. Đây là một vấn đề: khi `s2` và `s1`
ra khỏi phạm vi, cả hai sẽ cố gắng giải phóng cùng một bộ nhớ. Điều này được gọi
là lỗi _giải phóng bộ nhớ hai lần_ và là một trong những lỗi an toàn bộ nhớ mà
chúng ta đã đề cập trước đó. Giải phóng bộ nhớ hai lần có thể dẫn đến hư hỏng bộ
nhớ, điều này có thể dẫn đến lỗ hổng bảo mật.

Để đảm bảo an toàn bộ nhớ, sau dòng `let s2 = s1;`, Rust coi `s1` như không còn
hợp lệ. Do đó, Rust không cần phải giải phóng bất cứ thứ gì khi `s1` ra khỏi
phạm vi. Kiểm tra những gì xảy ra khi bạn cố gắng sử dụng `s1` sau khi `s2` được
tạo; nó sẽ không hoạt động:

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/src/main.rs:here}}
```

Bạn sẽ nhận được lỗi như thế này vì Rust ngăn bạn sử dụng tham chiếu không hợp
lệ:

```console
{{#include ../listings/ch04-understanding-ownership/no-listing-04-cant-use-after-move/output.txt}}
```

Nếu bạn đã nghe về các thuật ngữ _sao chép nông (shallow copy)_ và _sao chép sâu
(deep copy)_ trong khi làm việc với các ngôn ngữ khác, khái niệm sao chép con
trỏ, độ dài và dung lượng mà không sao chép dữ liệu có thể nghe giống như đang
thực hiện sao chép nông. Nhưng vì Rust cũng làm cho biến đầu tiên không hợp lệ,
nên thay vì được gọi là sao chép nông, nó được gọi là _di chuyển (move)_. Trong
ví dụ này, chúng ta sẽ nói rằng `s1` đã được _di chuyển_ vào `s2`. Vì vậy, những
gì thực sự xảy ra được hiển thị trong Hình 4-4.

<img alt="Ba bảng: bảng s1 và s2 đại diện cho các chuỗi đó trên
stack, tương ứng, và cả hai đều trỏ đến cùng một dữ liệu chuỗi trên heap.
Bảng s1 được tô màu xám vì s1 không còn hợp lệ; chỉ s2 có thể được sử dụng để
truy cập dữ liệu heap." src="img/trpl04-04.svg" class="center" style="width:
50%;" />

<span class="caption">Hình 4-4: Biểu diễn trong bộ nhớ sau khi `s1` đã bị vô
hiệu</span>

Điều đó giải quyết vấn đề của chúng ta! Với chỉ `s2` hợp lệ, khi nó ra khỏi phạm
vi, nó một mình sẽ giải phóng bộ nhớ, và chúng ta đã hoàn thành.

Ngoài ra, có một lựa chọn thiết kế được ngụ ý bởi điều này: Rust sẽ không bao
giờ tự động tạo các bản sao "sâu" của dữ liệu của bạn. Do đó, bất kỳ _tự động_
sao chép nào cũng có thể được giả định là có chi phí thấp về hiệu suất thời gian
chạy.

#### Phạm Vi và Gán

Điều ngược lại cũng đúng cho mối quan hệ giữa phạm vi, quyền sở hữu và bộ nhớ
được giải phóng thông qua hàm `drop`. Khi bạn gán một giá trị hoàn toàn mới cho
một biến hiện có, Rust sẽ gọi `drop` và giải phóng bộ nhớ của giá trị ban đầu
ngay lập tức. Xem xét mã này, ví dụ:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-04b-replacement-drop/src/main.rs:here}}
```

Ban đầu chúng ta khai báo một biến `s` và gắn nó với một `String` có giá trị
`"hello"`. Sau đó chúng ta ngay lập tức tạo một `String` mới với giá trị
`"ahoy"` và gán nó cho `s`. Tại thời điểm này, không có gì tham chiếu đến giá
trị ban đầu trên heap một chút nào.

<img alt="Một bảng s đại diện cho giá trị chuỗi trên stack, trỏ đến
phần dữ liệu chuỗi thứ hai (ahoy) trên heap, với dữ liệu chuỗi ban đầu
(hello) được tô xám vì không thể truy cập nữa."
src="img/trpl04-05.svg"
class="center"
style="width: 50%;"
/>

<span class="caption">Hình 4-5: Biểu diễn trong bộ nhớ sau khi giá trị ban đầu
đã bị thay thế hoàn toàn.</span>

Vì vậy, chuỗi ban đầu ngay lập tức ra khỏi phạm vi. Rust sẽ chạy hàm `drop` trên
nó và bộ nhớ của nó sẽ được giải phóng ngay lập tức. Khi chúng ta in giá trị vào
cuối, nó sẽ là `"ahoy, world!"`.

<!-- Old heading. Do not remove or links may break. -->

<a id="ways-variables-and-data-interact-clone"></a>

#### Các Biến và Dữ Liệu Tương Tác với Clone

Nếu chúng ta _muốn_ sao chép sâu dữ liệu heap của `String`, không chỉ dữ liệu
stack, chúng ta có thể sử dụng một phương thức phổ biến gọi là `clone`. Chúng ta
sẽ thảo luận về cú pháp phương thức trong Chương 5, nhưng vì các phương thức là
một tính năng phổ biến trong nhiều ngôn ngữ lập trình, bạn có thể đã thấy chúng
trước đây.

Đây là một ví dụ về phương thức `clone` trong hành động:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-05-clone/src/main.rs:here}}
```

Điều này hoạt động tốt và rõ ràng tạo ra hành vi được hiển thị trong Hình 4-3,
nơi dữ liệu heap _thực sự_ được sao chép.

Khi bạn thấy một lệnh gọi đến `clone`, bạn biết rằng một số mã tùy ý đang được
thực thi và mã đó có thể tốn kém. Đó là một dấu hiệu trực quan rằng điều gì đó
khác biệt đang xảy ra.

#### Dữ Liệu Chỉ Trên Stack: Copy

Còn một chi tiết khác mà chúng ta chưa đề cập. Mã này sử dụng số nguyên—một phần
đã được hiển thị trong Listing 4-2—hoạt động và hợp lệ:

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/no-listing-06-copy/src/main.rs:here}}
```

Nhưng mã này dường như trái ngược với những gì chúng ta vừa học: chúng ta không
có lệnh gọi `clone`, nhưng `x` vẫn hợp lệ và không bị di chuyển vào `y`.

Lý do là các kiểu như số nguyên có kích thước đã biết tại thời điểm biên dịch
được lưu trữ hoàn toàn trên stack, nên các bản sao của giá trị thực tế được tạo
nhanh chóng. Điều đó có nghĩa là không có lý do tại sao chúng ta muốn ngăn `x`
không còn hợp lệ sau khi chúng ta tạo biến `y`. Nói cách khác, không có sự khác
biệt giữa sao chép sâu và nông ở đây, vì vậy việc gọi `clone` sẽ không làm gì
khác so với sao chép nông thông thường, và chúng ta có thể bỏ qua nó.

Rust có một chú thích đặc biệt gọi là trait `Copy` mà chúng ta có thể đặt trên
các kiểu được lưu trữ trên stack, như số nguyên (chúng ta sẽ nói nhiều hơn về
trait trong [Chương 10][traits]<!-- ignore -->). Nếu một kiểu triển khai trait
`Copy`, các biến sử dụng nó không bị di chuyển, mà là được sao chép một cách tầm
thường, khiến chúng vẫn hợp lệ sau khi gán cho một biến khác.

Rust sẽ không cho phép chúng ta chú thích một kiểu với `Copy` nếu kiểu đó, hoặc
bất kỳ phần nào của nó, đã triển khai trait `Drop`. Nếu kiểu cần điều gì đó đặc
biệt xảy ra khi giá trị ra khỏi phạm vi và chúng ta thêm chú thích `Copy` vào
kiểu đó, chúng ta sẽ nhận được lỗi biên dịch. Để tìm hiểu về cách thêm chú thích
`Copy` vào kiểu của bạn để triển khai trait, xem ["Các Trait Có thể Dẫn
xuất"][derivable-traits]<!-- ignore --> trong Phụ lục C.

Vậy, các kiểu nào triển khai trait `Copy`? Bạn có thể kiểm tra tài liệu cho kiểu
đã cho để chắc chắn, nhưng theo quy tắc chung, bất kỳ nhóm giá trị vô hướng đơn
giản nào cũng có thể triển khai `Copy`, và không có gì yêu cầu cấp phát hoặc là
một dạng tài nguyên có thể triển khai `Copy`. Đây là một số kiểu triển khai
`Copy`:

- Tất cả các kiểu số nguyên, chẳng hạn như `u32`.
- Kiểu Boolean, `bool`, với giá trị `true` và `false`.
- Tất cả các kiểu số thực, chẳng hạn như `f64`.
- Kiểu ký tự, `char`.
- Tuple, nếu chúng chỉ chứa các kiểu cũng triển khai `Copy`. Ví dụ, `(i32, i32)`
  triển khai `Copy`, nhưng `(i32, String)` thì không.

### Quyền Sở Hữu và Hàm

Cơ chế của việc truyền một giá trị cho một hàm tương tự như khi gán một giá trị
cho một biến. Truyền một biến cho một hàm sẽ di chuyển hoặc sao chép, giống như
gán. Listing 4-3 có một ví dụ với một số chú thích cho thấy các biến đi vào và
ra khỏi phạm vi ở đâu.

<Listing number="4-3" file-name="src/main.rs" caption="Các hàm với quyền sở hữu và phạm vi được chú thích">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-03/src/main.rs}}
```

</Listing>

Nếu chúng ta cố gắng sử dụng `s` sau cuộc gọi đến `takes_ownership`, Rust sẽ đưa
ra một lỗi biên dịch. Những kiểm tra tĩnh này bảo vệ chúng ta khỏi các sai lầm.
Hãy thử thêm mã vào `main` sử dụng `s` và `x` để xem bạn có thể sử dụng chúng ở
đâu và nơi quy tắc sở hữu ngăn bạn làm như vậy.

### Giá Trị Trả Về và Phạm Vi

Việc trả về giá trị cũng có thể chuyển quyền sở hữu. Listing 4-4 hiển thị một ví
dụ về một hàm trả về một số giá trị, với các chú thích tương tự như trong
Listing 4-3.

<Listing number="4-4" file-name="src/main.rs" caption="Chuyển quyền sở hữu của giá trị trả về">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-04/src/main.rs}}
```

</Listing>

Quyền sở hữu của một biến tuân theo cùng một mô hình mọi lúc: gán một giá trị
cho một biến khác sẽ di chuyển nó. Khi một biến bao gồm dữ liệu trên heap ra
khỏi phạm vi, giá trị sẽ được dọn dẹp bởi `drop` trừ khi quyền sở hữu của dữ
liệu đã được chuyển sang một biến khác.

Mặc dù điều này hoạt động, việc lấy quyền sở hữu và sau đó trả lại quyền sở hữu
với mọi hàm hơi tẻ nhạt. Nếu chúng ta muốn cho một hàm sử dụng một giá trị nhưng
không lấy quyền sở hữu? Khá là phiền toái khi bất cứ thứ gì chúng ta truyền vào
cũng cần được truyền lại nếu chúng ta muốn sử dụng lại, cùng với bất kỳ dữ liệu
nào có từ thân hàm mà chúng ta cũng có thể muốn trả về.

Rust cho phép chúng ta trả về nhiều giá trị bằng cách sử dụng một tuple, như
được hiển thị trong Listing 4-5.

<Listing number="4-5" file-name="src/main.rs" caption="Trả lại quyền sở hữu của các tham số">

```rust
{{#rustdoc_include ../listings/ch04-understanding-ownership/listing-04-05/src/main.rs}}
```

</Listing>

Nhưng đây là quá nhiều nghi thức và rất nhiều công việc cho một khái niệm nên
phổ biến. May mắn thay, Rust có một tính năng để sử dụng một giá trị mà không
chuyển quyền sở hữu, được gọi là _tham chiếu (references)_.

[data-types]: ch03-02-data-types.html#data-types
[ch8]: ch08-02-strings.html
[traits]: ch10-02-traits.html
[derivable-traits]: appendix-03-derivable-traits.html
[method-syntax]: ch05-03-method-syntax.html#method-syntax
[paths-module-tree]:
  ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
[drop]: ../std/ops/trait.Drop.html#tymethod.drop
