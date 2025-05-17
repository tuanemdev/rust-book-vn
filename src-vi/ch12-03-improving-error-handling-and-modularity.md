## Tái cấu trúc để cải thiện tính module hóa và xử lý lỗi

Để cải thiện chương trình của chúng ta, chúng ta sẽ sửa bốn vấn đề liên quan đến
cấu trúc chương trình và cách nó xử lý các lỗi tiềm ẩn. Đầu tiên, hàm `main` của
chúng ta hiện thực hiện hai nhiệm vụ: phân tích đối số và đọc tệp. Khi chương
trình của chúng ta phát triển, số lượng nhiệm vụ riêng biệt mà hàm `main` xử lý
sẽ tăng lên. Khi một hàm có thêm nhiều trách nhiệm, nó trở nên khó hiểu hơn, khó
kiểm thử hơn, và khó thay đổi mà không làm hỏng một trong những phần của nó. Tốt
nhất là tách biệt chức năng để mỗi hàm chịu trách nhiệm cho một nhiệm vụ.

Vấn đề này cũng liên quan đến vấn đề thứ hai: mặc dù `query` và `file_path` là
các biến cấu hình cho chương trình của chúng ta, các biến như `contents` được sử
dụng để thực hiện logic của chương trình. Càng dài `main` trở nên, càng nhiều
biến chúng ta sẽ cần đưa vào phạm vi; càng nhiều biến chúng ta có trong phạm vi,
càng khó để theo dõi mục đích của mỗi biến. Tốt nhất là nhóm các biến cấu hình
vào một cấu trúc để làm rõ mục đích của chúng.

Vấn đề thứ ba là chúng ta đã sử dụng `expect` để in một thông báo lỗi khi đọc
tệp thất bại, nhưng thông báo lỗi chỉ in ra
`Should have been able to read the file`. Đọc một tệp có thể thất bại vì nhiều
lý do: ví dụ, tệp có thể không tồn tại, hoặc chúng ta có thể không có quyền để
mở nó. Hiện tại, bất kể tình huống nào, chúng ta sẽ in cùng một thông báo lỗi
cho mọi thứ, điều này sẽ không cung cấp cho người dùng bất kỳ thông tin nào!

Thứ tư, chúng ta sử dụng `expect` để xử lý lỗi, và nếu người dùng chạy chương
trình của chúng ta mà không chỉ định đủ đối số, họ sẽ nhận được lỗi
`index out of bounds` từ Rust không giải thích rõ ràng vấn đề. Sẽ tốt nhất nếu
tất cả mã xử lý lỗi đều ở một nơi để những người bảo trì trong tương lai chỉ có
một nơi để tham khảo mã nếu logic xử lý lỗi cần thay đổi. Có tất cả mã xử lý lỗi
ở một nơi cũng sẽ đảm bảo rằng chúng ta đang in các thông báo có ý nghĩa đối với
người dùng cuối của chúng ta.

Hãy giải quyết bốn vấn đề này bằng cách tái cấu trúc dự án của chúng ta.

### Tách biệt các mối quan tâm cho các dự án nhị phân

Vấn đề tổ chức của việc phân bổ trách nhiệm cho nhiều nhiệm vụ cho hàm `main` là
phổ biến đối với nhiều dự án nhị phân. Kết quả là, cộng đồng Rust đã phát triển
các hướng dẫn để tách các mối quan tâm riêng biệt của một chương trình nhị phân
khi `main` bắt đầu trở nên lớn. Quá trình này có các bước sau:

- Chia chương trình của bạn thành một tệp _main.rs_ và một tệp _lib.rs_ và di
  chuyển logic chương trình của bạn sang _lib.rs_.
- Miễn là logic phân tích dòng lệnh của bạn nhỏ, nó có thể ở lại _main.rs_.
- Khi logic phân tích dòng lệnh bắt đầu trở nên phức tạp, hãy trích xuất nó từ
  _main.rs_ và di chuyển nó đến _lib.rs_.

Các trách nhiệm còn lại trong hàm `main` sau quá trình này nên được giới hạn
trong các nội dung sau:

- Gọi logic phân tích dòng lệnh với các giá trị đối số
- Thiết lập bất kỳ cấu hình nào khác
- Gọi hàm `run` trong _lib.rs_
- Xử lý lỗi nếu `run` trả về lỗi

Mẫu này là về việc tách biệt các mối quan tâm: _main.rs_ xử lý việc chạy chương
trình và _lib.rs_ xử lý tất cả logic của nhiệm vụ hiện tại. Bởi vì bạn không thể
kiểm tra hàm `main` trực tiếp, cấu trúc này cho phép bạn kiểm tra tất cả logic
chương trình của bạn bằng cách di chuyển nó vào các hàm trong _lib.rs_. Mã còn
lại trong _main.rs_ sẽ đủ nhỏ để xác minh tính đúng đắn của nó bằng cách đọc nó.
Hãy làm lại chương trình của chúng ta bằng cách theo quy trình này.

#### Trích xuất trình phân tích đối số

Chúng ta sẽ trích xuất chức năng phân tích đối số vào một hàm mà `main` sẽ gọi
để chuẩn bị cho việc di chuyển logic phân tích dòng lệnh đến _src/lib.rs_.
Listing 12-5 hiển thị phần bắt đầu mới của `main` gọi một hàm mới
`parse_config`, mà chúng ta sẽ định nghĩa trong _src/main.rs_ tạm thời.

<Listing number="12-5" file-name="src/main.rs" caption="Trích xuất hàm `parse_config` từ `main`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-05/src/main.rs:here}}
```

</Listing>

Chúng ta vẫn đang thu thập các đối số dòng lệnh vào một vector, nhưng thay vì
gán giá trị đối số tại chỉ số 1 cho biến `query` và giá trị đối số tại chỉ số 2
cho biến `file_path` trong hàm `main`, chúng ta truyền toàn bộ vector cho hàm
`parse_config`. Hàm `parse_config` sau đó giữ logic xác định đối số nào vào biến
nào và truyền các giá trị trở lại cho `main`. Chúng ta vẫn tạo các biến `query`
và `file_path` trong `main`, nhưng `main` không còn có trách nhiệm xác định cách
các đối số dòng lệnh và các biến tương ứng.

Việc tái cấu trúc này có vẻ như là quá mức cần thiết cho chương trình nhỏ của
chúng ta, nhưng chúng ta đang tái cấu trúc theo các bước nhỏ, tăng dần. Sau khi
thực hiện thay đổi này, chạy chương trình lần nữa để xác minh rằng việc phân
tích đối số vẫn hoạt động. Tốt nhất là kiểm tra tiến trình của bạn thường xuyên,
để giúp xác định nguyên nhân của các vấn đề khi chúng xảy ra.

#### Nhóm các giá trị cấu hình

Chúng ta có thể thực hiện một bước nhỏ nữa để cải thiện hàm `parse_config` hơn
nữa. Hiện tại, chúng ta đang trả về một tuple, nhưng sau đó chúng ta ngay lập
tức phân chia tuple đó thành các phần riêng biệt một lần nữa. Đây là một dấu
hiệu cho thấy có lẽ chúng ta chưa có sự trừu tượng hóa đúng đắn.

Một chỉ báo khác cho thấy có chỗ để cải thiện là phần `config` của
`parse_config`, ngụ ý rằng hai giá trị mà chúng ta trả về có liên quan và đều là
một phần của một giá trị cấu hình. Hiện tại, chúng ta không truyền tải ý nghĩa
này trong cấu trúc của dữ liệu ngoài việc nhóm hai giá trị vào một tuple; thay
vào đó, chúng ta sẽ đặt hai giá trị vào một struct và đặt tên cho mỗi trường của
struct một cách có ý nghĩa. Làm như vậy sẽ giúp cho những người bảo trì mã này
trong tương lai dễ dàng hiểu các giá trị khác nhau liên quan đến nhau như thế
nào và mục đích của chúng là gì.

Listing 12-6 hiển thị các cải tiến cho hàm `parse_config`.

<Listing number="12-6" file-name="src/main.rs" caption="Tái cấu trúc `parse_config` để trả về một thực thể của struct `Config`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-06/src/main.rs:here}}
```

</Listing>

Chúng ta đã thêm một struct có tên `Config` được định nghĩa để có các trường tên
là `query` và `file_path`. Chữ ký của `parse_config` bây giờ chỉ ra rằng nó trả
về một giá trị `Config`. Trong thân của `parse_config`, nơi mà chúng ta đã từng
trả về các slice chuỗi tham chiếu đến các giá trị `String` trong `args`, bây giờ
chúng ta định nghĩa `Config` để chứa các giá trị `String` sở hữu. Biến `args`
trong `main` là chủ sở hữu của các giá trị đối số và chỉ cho phép hàm
`parse_config` mượn chúng, điều đó có nghĩa là chúng ta sẽ vi phạm quy tắc mượn
của Rust nếu `Config` cố gắng nắm quyền sở hữu các giá trị trong `args`.

Có một số cách chúng ta có thể quản lý dữ liệu `String`; cách dễ nhất, mặc dù
không hiệu quả lắm, là gọi phương thức `clone` trên các giá trị. Điều này sẽ tạo
một bản sao đầy đủ của dữ liệu cho thể hiện `Config` để sở hữu, mà tốn nhiều
thời gian và bộ nhớ hơn việc lưu trữ một tham chiếu đến dữ liệu chuỗi. Tuy
nhiên, việc sao chép dữ liệu cũng làm cho mã của chúng ta rất đơn giản vì chúng
ta không phải quản lý thời gian sống của các tham chiếu; trong trường hợp này,
việc hi sinh một chút hiệu suất để đạt được sự đơn giản là một sự đánh đổi đáng
giá.

> ### Sự đánh đổi khi sử dụng `clone`
>
> Có một xu hướng trong nhiều người dùng Rust là tránh sử dụng `clone` để sửa
> vấn đề sở hữu vì chi phí thời gian chạy của nó. Trong [Chương
> 13][ch13]<!-- ignore -->, bạn sẽ học cách sử dụng các phương pháp hiệu quả hơn
> trong loại tình huống này. Nhưng hiện tại, sao chép một vài chuỗi để tiếp tục
> tiến triển là ổn vì bạn sẽ chỉ tạo các bản sao này một lần và đường dẫn tệp
> cùng chuỗi truy vấn của bạn rất nhỏ. Tốt hơn là có một chương trình hoạt động
> mà hơi kém hiệu quả một chút so với việc cố gắng tối ưu hóa quá mức mã trong
> lần đầu tiên của bạn. Khi bạn có nhiều kinh nghiệm hơn với Rust, sẽ dễ dàng
> hơn để bắt đầu với giải pháp hiệu quả nhất, nhưng hiện tại, việc gọi `clone`
> là hoàn toàn chấp nhận được.

Chúng ta đã cập nhật `main` để nó đặt thể hiện `Config` được trả về bởi
`parse_config` vào một biến tên là `config`, và chúng ta đã cập nhật mã mà trước
đây sử dụng các biến `query` và `file_path` riêng biệt để bây giờ nó sử dụng các
trường trên struct `Config` thay thế.

Bây giờ mã của chúng ta truyền tải rõ ràng hơn rằng `query` và `file_path` có
liên quan và mục đích của chúng là để cấu hình cách chương trình sẽ hoạt động.
Bất kỳ mã nào sử dụng các giá trị này đều biết tìm chúng trong thể hiện `config`
trong các trường được đặt tên theo mục đích của chúng.

#### Tạo một hàm khởi tạo cho `Config`

Cho đến giờ, chúng ta đã trích xuất logic chịu trách nhiệm phân tích các đối số
dòng lệnh từ `main` và đặt nó trong hàm `parse_config`. Làm như vậy đã giúp
chúng ta thấy rằng các giá trị `query` và `file_path` có liên quan, và mối quan
hệ đó nên được truyền tải trong mã của chúng ta. Sau đó, chúng ta đã thêm một
struct `Config` để đặt tên cho mục đích liên quan của `query` và `file_path` và
để có thể trả lại tên của các giá trị dưới dạng tên trường struct từ hàm
`parse_config`.

Do đó, bây giờ mục đích của hàm `parse_config` là để tạo một thể hiện `Config`,
chúng ta có thể thay đổi `parse_config` từ một hàm đơn giản thành một hàm có tên
`new` được liên kết với struct `Config`. Thực hiện thay đổi này sẽ làm cho mã
trở nên thông dụng hơn. Chúng ta có thể tạo các thể hiện của các kiểu trong thư
viện chuẩn, chẳng hạn như `String`, bằng cách gọi `String::new`. Tương tự, bằng
cách thay đổi `parse_config` thành một hàm `new` liên kết với `Config`, chúng ta
sẽ có thể tạo các thể hiện của `Config` bằng cách gọi `Config::new`. Listing
12-7 hiển thị các thay đổi mà chúng ta cần thực hiện.

<Listing number="12-7" file-name="src/main.rs" caption="Thay đổi `parse_config` thành `Config::new`">

```rust,should_panic,noplayground
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-07/src/main.rs:here}}
```

</Listing>

Chúng ta đã cập nhật `main` nơi chúng ta đã gọi `parse_config` để thay vào đó
gọi `Config::new`. Chúng ta đã thay đổi tên của `parse_config` thành `new` và di
chuyển nó trong một khối `impl`, vốn liên kết hàm `new` với `Config`. Hãy thử
biên dịch mã này lần nữa để đảm bảo rằng nó hoạt động.

### Sửa chữa việc xử lý lỗi

Bây giờ chúng ta sẽ làm việc để sửa chữa việc xử lý lỗi của mình. Hãy nhớ lại
rằng việc cố gắng truy cập các giá trị trong vector `args` tại chỉ số 1 hoặc chỉ
số 2 sẽ khiến chương trình panic nếu vector chứa ít hơn ba phần tử. Hãy thử chạy
chương trình mà không có bất kỳ đối số nào; nó sẽ trông như thế này:

```console
{{#include ../listings/ch12-an-io-project/listing-12-07/output.txt}}
```

Dòng `index out of bounds: the len is 1 but the index is 1` là một thông báo lỗi
dành cho lập trình viên. Nó sẽ không giúp người dùng cuối của chúng ta hiểu họ
nên làm gì thay thế. Hãy sửa điều đó ngay bây giờ.

#### Cải thiện thông báo lỗi

Trong Listing 12-8, chúng ta thêm một kiểm tra trong hàm `new` sẽ xác minh rằng
slice đủ dài trước khi truy cập chỉ số 1 và chỉ số 2. Nếu slice không đủ dài,
chương trình panic và hiển thị một thông báo lỗi tốt hơn.

<Listing number="12-8" file-name="src/main.rs" caption="Thêm kiểm tra cho số lượng đối số">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-08/src/main.rs:here}}
```

</Listing>

Mã này tương tự như [hàm `Guess::new` mà chúng ta đã viết trong Listing
9-13][ch9-custom-types]<!-- ignore -->, nơi chúng ta đã gọi `panic!` khi đối số
`value` nằm ngoài phạm vi các giá trị hợp lệ. Thay vì kiểm tra cho một phạm vi
giá trị ở đây, chúng ta kiểm tra rằng độ dài của `args` ít nhất là `3` và phần
còn lại của hàm có thể hoạt động dưới giả định rằng điều kiện này đã được đáp
ứng. Nếu `args` có ít hơn ba mục, điều kiện này sẽ là `true`, và chúng ta gọi
macro `panic!` để kết thúc chương trình ngay lập tức.

Với một vài dòng mã bổ sung này trong `new`, hãy chạy chương trình mà không có
bất kỳ đối số nào một lần nữa để xem lỗi trông như thế nào bây giờ:

```console
{{#include ../listings/ch12-an-io-project/listing-12-08/output.txt}}
```

Đầu ra này tốt hơn: bây giờ chúng ta có một thông báo lỗi hợp lý. Tuy nhiên,
chúng ta cũng có thông tin không cần thiết mà chúng ta không muốn cung cấp cho
người dùng. Có lẽ kỹ thuật mà chúng ta đã sử dụng trong Listing 9-13 không phải
là cách tốt nhất để sử dụng ở đây: một cuộc gọi đến `panic!` thích hợp hơn cho
một vấn đề lập trình hơn là một vấn đề sử dụng, [như đã thảo luận trong Chương
9][ch9-error-guidelines]<!-- ignore -->. Thay vào đó, chúng ta sẽ sử dụng kỹ
thuật khác mà bạn đã học về trong Chương 9—[trả về một
`Result`][ch9-result]<!-- ignore --> cho biết thành công hoặc một lỗi.

<!-- Tiêu đề cũ. Đừng xóa hoặc các liên kết có thể bị hỏng. -->

<a id="returning-a-result-from-new-instead-of-calling-panic"></a>

#### Trả về một `Result` thay vì gọi `panic!`

Thay vào đó, chúng ta có thể trả về một giá trị `Result` sẽ chứa một thể hiện
`Config` trong trường hợp thành công và sẽ mô tả vấn đề trong trường hợp lỗi.
Chúng ta cũng sẽ thay đổi tên hàm từ `new` thành `build` vì nhiều lập trình viên
mong đợi các hàm `new` không bao giờ thất bại. Khi `Config::build` đang giao
tiếp với `main`, chúng ta có thể sử dụng kiểu `Result` để báo hiệu có một vấn
đề. Sau đó chúng ta có thể thay đổi `main` để chuyển đổi một biến thể `Err`
thành một lỗi thực tế hơn cho người dùng của chúng ta mà không có các văn bản
xung quanh về `thread 'main'` và `RUST_BACKTRACE` mà một cuộc gọi đến `panic!`
gây ra.

Listing 12-9 hiển thị các thay đổi mà chúng ta cần thực hiện cho giá trị trả về
của hàm mà chúng ta hiện đang gọi là `Config::build` và thân của hàm cần thiết
để trả về một `Result`. Lưu ý rằng điều này sẽ không biên dịch cho đến khi chúng
ta cập nhật `main` cũng vậy, điều mà chúng ta sẽ làm trong danh sách tiếp theo.

<Listing number="12-9" file-name="src/main.rs" caption="Trả về một `Result` từ `Config::build`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-09/src/main.rs:here}}
```

</Listing>

Hàm `build` của chúng ta trả về một `Result` với một thể hiện `Config` trong
trường hợp thành công và một chuỗi chữ trong trường hợp lỗi. Các giá trị lỗi của
chúng ta sẽ luôn luôn là các chuỗi chữ có thời gian sống `'static`.

Chúng ta đã thực hiện hai thay đổi trong thân hàm: thay vì gọi `panic!` khi
người dùng không truyền đủ đối số, chúng ta hiện trả về một giá trị `Err`, và
chúng ta đã bọc giá trị trả về `Config` trong một `Ok`. Những thay đổi này làm
cho hàm phù hợp với chữ ký kiểu mới của nó.

Việc trả về một giá trị `Err` từ `Config::build` cho phép hàm `main` xử lý giá
trị `Result` được trả về từ hàm `build` và thoát khỏi quá trình một cách sạch sẽ
hơn trong trường hợp lỗi.

<!-- Tiêu đề cũ. Đừng xóa hoặc các liên kết có thể bị hỏng. -->

<a id="calling-confignew-and-handling-errors"></a>

#### Gọi `Config::build` và xử lý lỗi

Để xử lý trường hợp lỗi và in một thông báo thân thiện với người dùng, chúng ta
cần cập nhật `main` để xử lý `Result` được trả về bởi `Config::build`, như hiển
thị trong Listing 12-10. Chúng ta cũng sẽ lấy trách nhiệm thoát khỏi công cụ
dòng lệnh với một mã lỗi khác không từ `panic!` và thay vào đó thực hiện nó bằng
tay. Một trạng thái thoát khác không là một quy ước để báo hiệu cho quá trình
gọi chương trình của chúng ta rằng chương trình đã thoát với trạng thái lỗi.

<Listing number="12-10" file-name="src/main.rs" caption="Thoát với một mã lỗi nếu việc xây dựng `Config` thất bại">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-10/src/main.rs:here}}
```

</Listing>

Trong danh sách này, chúng ta đã sử dụng một phương thức mà chúng ta chưa đề cập
đến chi tiết: `unwrap_or_else`, nó được định nghĩa trên `Result<T, E>` bởi thư
viện chuẩn. Sử dụng `unwrap_or_else` cho phép chúng ta định nghĩa một số xử lý
lỗi tùy chỉnh, không phải `panic!`. Nếu `Result` là một giá trị `Ok`, hành vi
của phương thức này giống với `unwrap`: nó trả về giá trị bên trong mà `Ok` đang
bọc. Tuy nhiên, nếu giá trị là một giá trị `Err`, phương thức này gọi mã trong
_closure_, đó là một hàm ẩn danh mà chúng ta định nghĩa và truyền làm đối số cho
`unwrap_or_else`. Chúng ta sẽ đề cập đến closure chi tiết hơn trong [Chương
13][ch13]<!-- ignore -->. Hiện tại, bạn chỉ cần biết rằng `unwrap_or_else` sẽ
truyền giá trị bên trong của `Err`, trong trường hợp này là chuỗi tĩnh
`"not enough arguments"` mà chúng ta đã thêm trong Listing 12-9, cho closure
trong đối số `err` vốn xuất hiện giữa các đường dọc. Mã trong closure sau đó có
thể sử dụng giá trị `err` khi nó chạy.

Chúng ta đã thêm một dòng `use` mới để đưa `process` từ thư viện chuẩn vào phạm
vi. Mã trong closure sẽ được chạy trong trường hợp lỗi chỉ có hai dòng: chúng ta
in giá trị `err` và sau đó gọi `process::exit`. Hàm `process::exit` sẽ dừng
chương trình ngay lập tức và trả về số được truyền làm mã trạng thái thoát. Điều
này tương tự như xử lý dựa trên `panic!` mà chúng ta đã sử dụng trong Listing
12-8, nhưng chúng ta không còn nhận được tất cả đầu ra bổ sung. Hãy thử nó:

```console
{{#include ../listings/ch12-an-io-project/listing-12-10/output.txt}}
```

Tuyệt vời! Đầu ra này thân thiện hơn nhiều với người dùng của chúng ta.

### Trích xuất logic từ `main`

Bây giờ chúng ta đã hoàn thành việc tái cấu trúc phân tích cấu hình, hãy chuyển
sang logic của chương trình. Như chúng ta đã nêu trong
["Tách biệt các mối quan tâm cho các dự án nhị phân"](#separation-of-concerns-for-binary-projects)<!-- ignore -->,
chúng ta sẽ trích xuất một hàm tên là `run` sẽ chứa tất cả logic hiện tại trong
hàm `main` không liên quan đến thiết lập cấu hình hoặc xử lý lỗi. Khi chúng ta
hoàn thành, `main` sẽ ngắn gọn và dễ dàng xác minh bằng việc kiểm tra, và chúng
ta sẽ có thể viết các bài kiểm tra cho tất cả các logic khác.

Listing 12-11 hiển thị hàm `run` được trích xuất. Hiện tại, chúng ta chỉ đang
thực hiện sự cải thiện nhỏ, tăng dần của việc trích xuất hàm. Chúng ta vẫn định
nghĩa hàm này trong _src/main.rs_.

<Listing number="12-11" file-name="src/main.rs" caption="Trích xuất một hàm `run` chứa phần còn lại của logic chương trình">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-11/src/main.rs:here}}
```

</Listing>

Hàm `run` hiện chứa tất cả logic còn lại từ `main`, bắt đầu từ việc đọc tệp. Hàm
`run` nhận thể hiện `Config` làm một đối số.

#### Trả về lỗi từ hàm `run`

Với logic chương trình còn lại đã được tách biệt vào hàm `run`, chúng ta có thể
cải thiện xử lý lỗi, như chúng ta đã làm với `Config::build` trong Listing 12-9.
Thay vì cho phép chương trình panic bằng cách gọi `expect`, hàm `run` sẽ trả về
một `Result<T, E>` khi có gì đó không ổn. Điều này sẽ cho phép chúng ta hợp nhất
hơn nữa logic xử lý lỗi vào `main` theo một cách thân thiện với người dùng.
Listing 12-12 hiển thị các thay đổi chúng ta cần thực hiện cho chữ ký và thân
của `run`.

<Listing number="12-12" file-name="src/main.rs" caption="Thay đổi hàm `run` để trả về `Result`">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-12/src/main.rs:here}}
```

</Listing>

Chúng ta đã thực hiện ba thay đổi đáng kể ở đây. Đầu tiên, chúng ta đã thay đổi
kiểu trả về của hàm `run` thành `Result<(), Box<dyn Error>>`. Hàm này trước đây
trả về kiểu đơn vị, `()`, và chúng ta giữ giá trị đó làm giá trị được trả về
trong trường hợp `Ok`.

Đối với kiểu lỗi, chúng ta đã sử dụng _trait object_ `Box<dyn Error>` (và chúng
ta đã đưa `std::error::Error` vào phạm vi với một câu lệnh `use` ở đầu). Chúng
ta sẽ nói về trait object trong [Chương 18][ch18]<!-- ignore -->. Hiện tại, chỉ
cần biết rằng `Box<dyn Error>` có nghĩa là hàm sẽ trả về một kiểu mà thực hiện
trait `Error`, nhưng chúng ta không phải chỉ định kiểu cụ thể mà giá trị trả về
sẽ là. Điều này cung cấp cho chúng ta sự linh hoạt để trả về các giá trị lỗi có
thể thuộc các kiểu khác nhau trong các trường hợp lỗi khác nhau. Từ khóa `dyn`
là viết tắt của _dynamic_.

Thứ hai, chúng ta đã loại bỏ lệnh gọi đến `expect` để ủng hộ toán tử `?`, như
chúng ta đã nói trong [Chương 9][ch9-question-mark]<!-- ignore -->. Thay vì
`panic!` khi có lỗi, `?` sẽ trả về giá trị lỗi từ hàm hiện tại để người gọi xử
lý.

Thứ ba, hàm `run` bây giờ trả về một giá trị `Ok` trong trường hợp thành công.
Chúng ta đã khai báo kiểu thành công của hàm `run` là `()` trong chữ ký, có
nghĩa là chúng ta cần bọc giá trị kiểu đơn vị trong giá trị `Ok`. Cú pháp
`Ok(())` này có vẻ hơi kỳ lạ lúc đầu, nhưng sử dụng `()` như thế này là cách
thông dụng để chỉ ra rằng chúng ta đang gọi `run` chỉ vì các hiệu ứng phụ của
nó; nó không trả về một giá trị mà chúng ta cần.

Khi bạn chạy mã này, nó sẽ biên dịch nhưng sẽ hiển thị một cảnh báo:

```console
{{#include ../listings/ch12-an-io-project/listing-12-12/output.txt}}
```

Rust cho chúng ta biết rằng mã của chúng ta đã bỏ qua giá trị `Result` và giá
trị `Result` có thể chỉ ra rằng một lỗi đã xảy ra. Nhưng chúng ta không kiểm tra
xem liệu có hay không có lỗi, và trình biên dịch nhắc nhở chúng ta rằng có lẽ
chúng ta muốn có mã xử lý lỗi ở đây! Hãy sửa vấn đề đó ngay bây giờ.

#### Xử lý lỗi trả về từ `run` trong `main`

Chúng ta sẽ kiểm tra lỗi và xử lý chúng bằng một kỹ thuật tương tự như một kỹ
thuật chúng ta đã sử dụng với `Config::build` trong Listing 12-10, nhưng với một
chút khác biệt:

<span class="filename">Tên tệp: src/main.rs</span>

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/no-listing-01-handling-errors-in-main/src/main.rs:here}}
```

Chúng ta sử dụng `if let` thay vì `unwrap_or_else` để kiểm tra xem `run` có trả
về một giá trị `Err` hay không và gọi `process::exit(1)` nếu nó trả về. Hàm
`run` không trả về một giá trị mà chúng ta muốn `unwrap` theo cùng cách mà
`Config::build` trả về thể hiện `Config`. Bởi vì `run` trả về `()` trong trường
hợp thành công, chúng ta chỉ quan tâm đến việc phát hiện một lỗi, vì vậy chúng
ta không cần `unwrap_or_else` để trả về giá trị unwrapped, chỉ là `()`.

Thân của `if let` và các hàm `unwrap_or_else` là giống nhau trong cả hai trường
hợp: chúng ta in lỗi và thoát.

### Chia mã thành một Crate thư viện

Dự án `minigrep` của chúng ta đang hoạt động tốt cho đến nay! Bây giờ chúng ta
sẽ chia tệp _src/main.rs_ và đặt một số mã vào tệp _src/lib.rs_. Bằng cách đó,
chúng ta có thể kiểm tra mã và có một tệp _src/main.rs_ với ít trách nhiệm hơn.

Hãy chuyển tất cả mã không ở trong hàm `main` từ _src/main.rs_ đến _src/lib.rs_:

- Định nghĩa hàm `run`
- Các câu lệnh `use` liên quan
- Định nghĩa của `Config`
- Định nghĩa hàm `Config::build`

Nội dung của _src/lib.rs_ nên có các chữ ký được hiển thị trong Listing 12-13
(chúng ta đã bỏ qua thân của các hàm để ngắn gọn). Lưu ý rằng điều này sẽ không
biên dịch cho đến khi chúng ta sửa đổi _src/main.rs_ trong Listing 12-14.

<Listing number="12-13" file-name="src/lib.rs" caption="Di chuyển `Config` và `run` vào *src/lib.rs*">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-13/src/lib.rs:here}}
```

</Listing>

Chúng ta đã sử dụng từ khóa `pub` một cách rộng rãi: trên `Config`, trên các
trường và phương thức `build` của nó, và trên hàm `run`. Bây giờ chúng ta có một
crate thư viện có một API công khai mà chúng ta có thể kiểm tra!

Bây giờ chúng ta cần đưa mã mà chúng ta đã di chuyển đến _src/lib.rs_ vào phạm
vi của crate nhị phân trong _src/main.rs_, như hiển thị trong Listing 12-14.

<Listing number="12-14" file-name="src/main.rs" caption="Sử dụng crate thư viện `minigrep` trong *src/main.rs*">

```rust,ignore
{{#rustdoc_include ../listings/ch12-an-io-project/listing-12-14/src/main.rs:here}}
```

</Listing>

Chúng ta thêm một dòng `use minigrep::Config` để đưa kiểu `Config` từ crate thư
viện vào phạm vi của crate nhị phân, và chúng ta thêm tiền tố `minigrep::` trước
hàm `run`. Bây giờ tất cả chức năng nên được kết nối và nên hoạt động. Chạy
chương trình với `cargo run` và đảm bảo mọi thứ hoạt động đúng.

Chà! Đó là một công việc lớn, nhưng chúng ta đã thiết lập bản thân để thành công
trong tương lai. Bây giờ việc xử lý lỗi dễ dàng hơn nhiều và chúng ta đã làm cho
mã trở nên module hơn. Gần như tất cả công việc của chúng ta sẽ được thực hiện
trong _src/lib.rs_ từ bây giờ.

Hãy tận dụng tính module hóa mới này bằng cách làm một điều mà lẽ ra đã khó với
mã cũ nhưng dễ dàng với mã mới: chúng ta sẽ viết một số bài kiểm tra!

[ch13]: ch13-00-functional-features.html
[ch9-custom-types]:
  ch09-03-to-panic-or-not-to-panic.html#creating-custom-types-for-validation
[ch9-error-guidelines]:
  ch09-03-to-panic-or-not-to-panic.html#guidelines-for-error-handling
[ch9-result]: ch09-02-recoverable-errors-with-result.html
[ch18]: ch18-00-oop.html
[ch9-question-mark]:
  ch09-02-recoverable-errors-with-result.html#a-shortcut-for-propagating-errors-the--operator
