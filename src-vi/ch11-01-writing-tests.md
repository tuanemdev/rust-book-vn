## Cách Viết Các Bài Kiểm Thử

Các bài kiểm thử trong Rust là các hàm xác minh rằng code không phải kiểm thử
đang hoạt động theo cách mong đợi. Thân hàm của các bài kiểm thử thường thực
hiện ba hành động sau:

- Thiết lập dữ liệu hoặc trạng thái cần thiết.
- Chạy code bạn muốn kiểm thử.
- Khẳng định rằng kết quả là những gì bạn mong đợi.

Hãy xem xét các tính năng Rust cung cấp đặc biệt để viết các bài kiểm thử thực
hiện các hành động này, bao gồm thuộc tính `test`, một số macro và thuộc tính
`should_panic`.

### Cấu Trúc của Một Hàm Kiểm Thử

Ở dạng đơn giản nhất, một bài kiểm thử trong Rust là một hàm được chú thích với
thuộc tính `test`. Các thuộc tính là siêu dữ liệu về các phần code Rust; một ví
dụ là thuộc tính `derive` mà chúng ta đã sử dụng với các cấu trúc trong
Chương 5. Để biến một hàm thành hàm kiểm thử, thêm `#[test]` vào dòng trước
`fn`. Khi bạn chạy các bài kiểm thử với lệnh `cargo test`, Rust xây dựng một tệp
thực thi chạy kiểm thử để chạy các hàm được chú thích và báo cáo xem từng hàm
kiểm thử có thành công hay không.

Bất cứ khi nào chúng ta tạo một dự án thư viện mới với Cargo, một module kiểm
thử với một hàm kiểm thử trong đó được tự động tạo ra cho chúng ta. Module này
cung cấp cho bạn một mẫu để viết các bài kiểm thử của bạn, vì vậy bạn không phải
tìm kiếm cấu trúc và cú pháp chính xác mỗi khi bạn bắt đầu một dự án mới. Bạn có
thể thêm bất kỳ hàm kiểm thử bổ sung nào và bất kỳ module kiểm thử nào mà bạn
muốn!

Chúng ta sẽ khám phá một số khía cạnh về cách các bài kiểm thử hoạt động bằng
cách thử nghiệm với mẫu kiểm thử trước khi chúng ta thực sự kiểm thử bất kỳ mã
nào. Sau đó, chúng ta sẽ viết một số bài kiểm thử thực tế gọi một số code mà
chúng ta đã viết và khẳng định rằng hành vi của nó là chính xác.

Hãy tạo một dự án thư viện mới có tên `adder` sẽ cộng hai số:

```console
$ cargo new adder --lib
     Created library `adder` project
$ cd adder
```

Nội dung của tệp _src/lib.rs_ trong thư viện `adder` của bạn sẽ trông giống như
Listing 11-1.

<Listing number="11-1" file-name="src/lib.rs" caption="Mã được tạo tự động bởi `cargo new`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-01/src/lib.rs}}
```

</Listing>

Tệp bắt đầu với một ví dụ hàm `add`, để chúng ta có thứ gì đó để kiểm thử.

Bây giờ, chúng ta hãy tập trung duy nhất vào hàm `it_works`. Lưu ý chú thích
`#[test]`: thuộc tính này chỉ ra rằng đây là một hàm kiểm thử, vì vậy trình chạy
kiểm thử biết xử lý hàm này như một bài kiểm thử. Chúng ta cũng có thể có các
hàm không phải kiểm thử trong module `tests` để giúp thiết lập các kịch bản
chung hoặc thực hiện các thao tác chung, vì vậy chúng ta luôn cần chỉ ra những
hàm nào là kiểm thử.

Thân hàm ví dụ sử dụng vĩ lệnh `assert_eq!` để khẳng định rằng `result`, là kết
quả của việc gọi hàm `add` với 2 và 2, bằng 4. Khẳng định này đóng vai trò như
một ví dụ về định dạng cho một bài kiểm thử điển hình. Hãy chạy nó để xem bài
kiểm thử này có thành công hay không.

Lệnh `cargo test` chạy tất cả các bài kiểm thử trong dự án của chúng ta, như
được hiển thị trong Listing 11-2.

<Listing number="11-2" caption="Đầu ra từ việc chạy bài kiểm thử được tạo tự động">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-01/output.txt}}
```

</Listing>

Cargo đã biên dịch và chạy bài kiểm thử. Chúng ta thấy dòng `running 1 test`.
Dòng tiếp theo hiển thị tên của hàm kiểm thử được tạo ra, gọi là
`tests::it_works`, và kết quả chạy bài kiểm thử đó là `ok`. Tóm lược tổng thể
`test result: ok.` có nghĩa là tất cả các bài kiểm thử đều thành công, và phần
đọc `1 passed; 0 failed` tổng hợp số bài kiểm thử đã thành công hoặc thất bại.

Có thể đánh dấu một bài kiểm thử là bị bỏ qua để nó không chạy trong một trường
hợp cụ thể; chúng ta sẽ đề cập điều này trong phần ["Bỏ Qua Một Số Bài Kiểm Thử
Trừ Khi Có Yêu Cầu Cụ Thể"][ignoring]<!-- ignore --> sau này trong chương này.
Vì chúng ta chưa làm điều đó ở đây, tóm lược hiển thị `0 ignored`. Chúng ta cũng
có thể truyền một đối số cho lệnh `cargo test` để chỉ chạy các bài kiểm thử có
tên khớp với một chuỗi; điều này được gọi là _lọc_, và chúng ta sẽ đề cập đến nó
trong phần ["Chạy Một Tập Con của Các Bài Kiểm Thử theo
Tên"][subset]<!-- ignore -->. Ở đây, chúng ta chưa lọc các bài kiểm thử đang
chạy, vì vậy phần cuối của tóm lược hiển thị `0 filtered out`.

Chỉ số `0 measured` là dành cho các bài kiểm thử chuẩn đo lường hiệu suất. Các
bài kiểm thử chuẩn, tại thời điểm viết bài này, chỉ có sẵn trong Rust nightly.
Xem [tài liệu về các bài kiểm thử chuẩn][bench] để tìm hiểu thêm.

Phần tiếp theo của kết quả kiểm thử bắt đầu từ `Doc-tests adder` là dành cho kết
quả của bất kỳ bài kiểm thử tài liệu nào. Chúng ta chưa có bài kiểm thử tài liệu
nào, nhưng Rust có thể biên dịch bất kỳ ví dụ code nào xuất hiện trong tài liệu
API của chúng ta. Tính năng này giúp duy trì tài liệu và code của bạn đồng bộ!
Chúng ta sẽ thảo luận về cách viết các bài kiểm thử tài liệu trong phần ["Bình
Luận Tài Liệu như Các Bài Kiểm Thử"][doc-comments]<!-- ignore --> của Chương 14.
Hiện tại, chúng ta sẽ bỏ qua kết quả `Doc-tests`.

Hãy bắt đầu tùy chỉnh bài kiểm thử theo nhu cầu của riêng mình. Đầu tiên, thay
đổi tên của hàm `it_works` thành một cái tên khác, chẳng hạn như `exploration`,
như sau:

<span class="filename">Tên tệp: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/src/lib.rs}}
```

Sau đó chạy `cargo test` lần nữa. Kết quả bây giờ hiển thị `exploration` thay vì
`it_works`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-01-changing-test-name/output.txt}}
```

Bây giờ chúng ta sẽ thêm một bài kiểm thử khác, nhưng lần này chúng ta sẽ tạo
một bài kiểm thử thất bại! Các bài kiểm thử thất bại khi có gì đó trong hàm kiểm
thử hoảng loạn (panic). Mỗi bài kiểm thử được chạy trong một thread mới, và khi
thread chính thấy rằng một thread kiểm thử đã chết, bài kiểm thử được đánh dấu
là đã thất bại. Trong Chương 9, chúng ta đã nói về cách đơn giản nhất để hoảng
loạn là gọi vĩ lệnh `panic!`. Hãy nhập bài kiểm thử mới dưới dạng một hàm có tên
là `another`, để tệp _src/lib.rs_ của bạn trông giống như Listing 11-3.

<Listing number="11-3" file-name="src/lib.rs" caption="Thêm một bài kiểm thử thứ hai sẽ thất bại vì chúng ta gọi vĩ lệnh `panic!`">

```rust,panics,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-03/src/lib.rs}}
```

</Listing>

Chạy các bài kiểm thử một lần nữa bằng `cargo test`. Kết quả sẽ trông giống như
Listing 11-4, cho thấy bài kiểm thử `exploration` của chúng ta đã thành công và
bài kiểm thử `another` đã thất bại.

<Listing number="11-4" caption="Kết quả kiểm thử khi một bài kiểm thử thành công và một bài kiểm thử thất bại">

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-03/output.txt}}
```

</Listing>

Thay vì `ok`, dòng `test tests::another` hiển thị `FAILED`. Hai phần mới xuất
hiện giữa các kết quả cá nhân và tóm lược: phần đầu tiên hiển thị lý do chi tiết
cho mỗi bài kiểm thử thất bại. Trong trường hợp này, chúng ta nhận được chi tiết
rằng `tests::another` đã thất bại vì nó panicked với thông báo
`Make this test fail` trên dòng 17 trong tệp _src/lib.rs_. Phần tiếp theo chỉ
liệt kê tên của tất cả các bài kiểm thử thất bại, điều này hữu ích khi có nhiều
bài kiểm thử và nhiều kết quả kiểm thử thất bại chi tiết. Chúng ta có thể sử
dụng tên của bài kiểm thử thất bại để chỉ chạy bài kiểm thử đó để gỡ lỗi dễ dàng
hơn; chúng ta sẽ nói thêm về cách chạy các bài kiểm thử trong phần ["Kiểm Soát
Cách Chạy Các Bài Kiểm Thử"][controlling-how-tests-are-run]<!-- ignore -->.

Dòng tóm lược hiển thị ở cuối: nhìn chung, kết quả kiểm thử của chúng ta là
`FAILED`. Chúng ta có một bài kiểm thử thành công và một bài kiểm thử thất bại.

Bây giờ bạn đã thấy kết quả kiểm thử trông như thế nào trong các tình huống khác
nhau, hãy xem các vĩ lệnh khác ngoài `panic!` hữu ích trong các bài kiểm thử.

### Kiểm Tra Kết Quả với Vĩ Lệnh `assert!`

Vĩ lệnh `assert!`, được cung cấp bởi thư viện chuẩn, hữu ích khi bạn muốn đảm
bảo rằng một số điều kiện trong bài kiểm thử được đánh giá là `true`. Chúng ta
cung cấp cho vĩ lệnh `assert!` một đối số được đánh giá thành một giá trị
Boolean. Nếu giá trị là `true`, không có gì xảy ra và bài kiểm thử thành công.
Nếu giá trị là `false`, vĩ lệnh `assert!` gọi `panic!` để khiến bài kiểm thử
thất bại. Việc sử dụng vĩ lệnh `assert!` giúp chúng ta kiểm tra xem code của
mình có hoạt động theo cách chúng ta dự định hay không.

Trong Chương 5, Listing 5-15, chúng ta đã sử dụng cấu trúc `Rectangle` và phương
thức `can_hold`, được lặp lại ở đây trong Listing 11-5. Hãy đặt code này vào tệp
_src/lib.rs_, sau đó viết một số bài kiểm thử cho nó bằng vĩ lệnh `assert!`.

<Listing number="11-5" file-name="src/lib.rs" caption="Cấu trúc `Rectangle` và phương thức `can_hold` từ Chương 5">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-05/src/lib.rs}}
```

</Listing>

Phương thức `can_hold` trả về một giá trị Boolean, điều này có nghĩa là nó là
một trường hợp sử dụng hoàn hảo cho vĩ lệnh `assert!`. Trong Listing 11-6, chúng
ta viết một bài kiểm thử thực hiện phương thức `can_hold` bằng cách tạo một
instance `Rectangle` có chiều rộng là 8 và chiều cao là 7 và khẳng định rằng nó
có thể chứa một instance `Rectangle` khác có chiều rộng là 5 và chiều cao là 1.

<Listing number="11-6" file-name="src/lib.rs" caption="Một bài kiểm thử cho `can_hold` kiểm tra xem một hình chữ nhật lớn hơn có thể chứa một hình chữ nhật nhỏ hơn hay không">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-06/src/lib.rs:here}}
```

</Listing>

Lưu ý dòng `use super::*;` bên trong module `tests`. Module `tests` là một
module bình thường tuân theo các quy tắc khả kiến thông thường mà chúng ta đã đề
cập trong Chương 7 ở phần ["Đường Dẫn cho Việc Tham Chiếu đến Một Mục trong Cây
Module"][paths-for-referring-to-an-item-in-the-module-tree]<!-- ignore -->. Vì
module `tests` là một module bên trong, chúng ta cần đưa code đang được kiểm thử
trong module bên ngoài vào phạm vi của module bên trong. Chúng ta sử dụng glob ở
đây, vì vậy bất cứ thứ gì chúng ta xác định trong module bên ngoài đều có sẵn
cho module `tests` này.

Chúng ta đã đặt tên cho bài kiểm thử của mình là `larger_can_hold_smaller`, và
chúng ta đã tạo hai instance `Rectangle` mà chúng ta cần. Sau đó, chúng ta gọi
vĩ lệnh `assert!` và truyền cho nó kết quả của việc gọi
`larger.can_hold(&smaller)`. Biểu thức này được cho là trả về `true`, vì vậy bài
kiểm thử của chúng ta sẽ thành công. Hãy tìm hiểu!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-06/output.txt}}
```

Nó đã thành công! Hãy thêm một bài kiểm thử khác, lần này khẳng định rằng một
hình chữ nhật nhỏ hơn không thể chứa một hình chữ nhật lớn hơn:

<span class="filename">Tên tệp: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/src/lib.rs:here}}
```

Vì kết quả đúng của hàm `can_hold` trong trường hợp này là `false`, chúng ta cần
phủ định kết quả đó trước khi truyền nó cho vĩ lệnh `assert!`. Kết quả là, bài
kiểm thử của chúng ta sẽ thành công nếu `can_hold` trả về `false`:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-02-adding-another-rectangle-test/output.txt}}
```

Hai bài kiểm thử đều thành công! Bây giờ, hãy xem điều gì xảy ra với kết quả
kiểm thử của chúng ta khi chúng ta đưa vào một lỗi trong code. Chúng ta sẽ thay
đổi triển khai của phương thức `can_hold` bằng cách thay thế dấu lớn hơn bằng
dấu nhỏ hơn khi nó so sánh chiều rộng:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/src/lib.rs:here}}
```

Chạy các bài kiểm thử bây giờ sẽ cho kết quả sau:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-03-introducing-a-bug/output.txt}}
```

Các bài kiểm thử của chúng ta đã phát hiện ra lỗi! Bởi vì `larger.width` là `8`
và `smaller.width` là `5`, phép so sánh chiều rộng trong `can_hold` bây giờ trả
về `false`: 8 không nhỏ hơn 5.

### Kiểm Tra Sự Bằng Nhau với Các Vĩ Lệnh `assert_eq!` và `assert_ne!`

Một cách phổ biến để xác minh chức năng là kiểm tra sự bằng nhau giữa kết quả
của code đang được kiểm thử và giá trị bạn mong đợi code trả về. Bạn có thể làm
điều này bằng cách sử dụng vĩ lệnh `assert!` và truyền cho nó một biểu thức sử
dụng toán tử `==`. Tuy nhiên, đây là một bài kiểm thử rất phổ biến mà thư viện
chuẩn cung cấp một cặp vĩ lệnh — `assert_eq!` và `assert_ne!`—để thực hiện bài
kiểm thử này một cách thuận tiện hơn. Các vĩ lệnh này so sánh hai đối số xem có
bằng nhau hoặc không bằng nhau, tương ứng. Chúng cũng sẽ in ra hai giá trị nếu
khẳng định thất bại, điều này giúp dễ dàng thấy _tại sao_ bài kiểm thử thất bại;
ngược lại, vĩ lệnh `assert!` chỉ chỉ ra rằng nó nhận được giá trị `false` cho
biểu thức `==`, mà không in ra các giá trị dẫn đến giá trị `false`.

Trong Listing 11-7, chúng ta viết một hàm có tên `add_two` cộng thêm `2` vào
tham số của nó, sau đó chúng ta kiểm thử hàm này bằng vĩ lệnh `assert_eq!`.

<Listing number="11-7" file-name="src/lib.rs" caption="Kiểm thử hàm `add_two` bằng vĩ lệnh `assert_eq!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-07/src/lib.rs}}
```

</Listing>

Hãy kiểm tra xem nó có thành công không!

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-07/output.txt}}
```

Chúng ta tạo một biến có tên `result` lưu trữ kết quả của việc gọi `add_two(2)`.
Sau đó, chúng ta truyền `result` và `4` làm đối số cho macro `assert_eq!`. Dòng
kết quả cho bài kiểm thử này là `test tests::it_adds_two ... ok`, và văn bản
`ok` chỉ ra rằng bài kiểm thử của chúng ta đã thành công!

Hãy đưa một lỗi vào code của chúng ta để xem `assert_eq!` trông như thế nào khi
nó thất bại. Thay đổi triển khai của hàm `add_two` để cộng thêm `3` thay vì `2`:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/src/lib.rs:here}}
```

Chạy các bài kiểm thử một lần nữa:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-04-bug-in-add-two/output.txt}}
```

Bài kiểm thử của chúng ta đã phát hiện lỗi! Bài kiểm thử `test::it_adds_two` đã
thất bại, và thông báo cho chúng ta biết rằng khẳng định thất bại là
`left == right` và giá trị `left` và `right` là gì. Thông báo này giúp chúng ta
bắt đầu gỡ lỗi: đối số `left`, nơi chúng ta có kết quả của việc gọi
`add_two(2)`, là `5` nhưng đối số `right` là `4`. Bạn có thể tưởng tượng điều
này sẽ đặc biệt hữu ích khi chúng ta có nhiều bài kiểm thử đang diễn ra.

Lưu ý rằng trong một số ngôn ngữ và framework kiểm thử, các tham số cho hàm
khẳng định về sự bằng nhau được gọi là `expected` và `actual`, và thứ tự mà
chúng ta chỉ định các đối số có vấn đề. Tuy nhiên, trong Rust, chúng được gọi là
`left` và `right`, và thứ tự mà chúng ta chỉ định giá trị mà chúng ta mong đợi
và giá trị mà code tạo ra không quan trọng. Chúng ta có thể viết khẳng định
trong bài kiểm thử này là `assert_eq!(4, result)`, điều này sẽ dẫn đến thông báo
lỗi tương tự hiển thị `` assertion `left == right` failed``.

Vĩ lệnh `assert_ne!` sẽ thành công nếu hai giá trị chúng ta cung cấp cho nó
không bằng nhau và thất bại nếu chúng bằng nhau. Vĩ lệnh này hữu ích nhất cho
các trường hợp khi chúng ta không chắc giá trị _sẽ_ là gì, nhưng chúng ta biết
giá trị đó chắc chắn _không nên_ là gì. Ví dụ, nếu chúng ta đang kiểm thử một
hàm được đảm bảo sẽ thay đổi đầu vào của nó theo cách nào đó, nhưng cách thức
đầu vào được thay đổi phụ thuộc vào ngày trong tuần mà chúng ta chạy các bài
kiểm thử, điều tốt nhất để khẳng định có thể là kết quả của hàm không bằng đầu
vào.

Dưới bề mặt, các vĩ lệnh `assert_eq!` và `assert_ne!` sử dụng các toán tử `==`
và `!=`, tương ứng. Khi các khẳng định thất bại, các vĩ lệnh này in ra các đối
số của chúng bằng định dạng debug, điều này có nghĩa là các giá trị đang được so
sánh phải triển khai các trait `PartialEq` và `Debug`. Tất cả các kiểu nguyên
thủy và hầu hết các kiểu thư viện chuẩn đều triển khai các trait này. Đối với
các cấu trúc và enum mà bạn tự định nghĩa, bạn cần triển khai `PartialEq` để
khẳng định bằng nhau cho các kiểu này. Bạn cũng cần triển khai `Debug` để in ra
các giá trị khi khẳng định thất bại. Bởi vì cả hai trait đều là các trait có thể
dẫn xuất, như đã đề cập trong Listing 5-12 của Chương 5, điều này thường đơn
giản như việc thêm chú thích `#[derive(PartialEq, Debug)]` vào định nghĩa cấu
trúc hoặc enum của bạn. Xem Phụ lục C, ["Các Trait Có Thể Dẫn
Xuất,"][derivable-traits]<!-- ignore --> để biết thêm chi tiết về các trait này
và các trait có thể dẫn xuất khác.

### Thêm Thông Báo Thất Bại Tùy Chỉnh

Bạn cũng có thể thêm một thông báo tùy chỉnh để hiển thị với thông báo thất bại
làm đối số tùy chọn cho các vĩ lệnh `assert!`, `assert_eq!` và `assert_ne!`. Bất
kỳ đối số nào được chỉ định sau các đối số bắt buộc đều được chuyển đến vĩ lệnh
`format!` (được thảo luận trong ["Nối Chuỗi với Toán Tử `+` hoặc Vĩ Lệnh
`format!`"][concatenation-with-the--operator-or-the-format-macro]<!-- ignore -->
ở Chương 8), vì vậy bạn có thể truyền một chuỗi định dạng chứa các placeholder
`{}` và các giá trị để điền vào các placeholder đó. Các thông báo tùy chỉnh hữu
ích để ghi lại ý nghĩa của khẳng định; khi một bài kiểm thử thất bại, bạn sẽ có
ý tưởng tốt hơn về vấn đề trong code.

Ví dụ, giả sử chúng ta có một hàm chào hỏi mọi người theo tên và chúng ta muốn
kiểm tra rằng tên mà chúng ta truyền vào hàm xuất hiện trong kết quả:

<span class="filename">Tên tệp: src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-05-greeter/src/lib.rs}}
```

Các yêu cầu cho chương trình này chưa được thống nhất, và chúng ta khá chắc chắn
rằng văn bản `Hello` ở đầu lời chào sẽ thay đổi. Chúng ta quyết định không muốn
phải cập nhật bài kiểm thử khi các yêu cầu thay đổi, vì vậy thay vì kiểm tra sự
bằng nhau chính xác với giá trị trả về từ hàm `greeting`, chúng ta sẽ chỉ khẳng
định rằng kết quả chứa văn bản của tham số đầu vào.

Bây giờ hãy đưa một lỗi vào code này bằng cách thay đổi `greeting` để loại bỏ
`name` để xem lỗi kiểm thử mặc định trông như thế nào:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/src/lib.rs:here}}
```

Chạy bài kiểm thử này sẽ cho kết quả sau:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-06-greeter-with-bug/output.txt}}
```

Kết quả này chỉ chỉ ra rằng khẳng định đã thất bại và dòng nào khẳng định đó.
Một thông báo lỗi hữu ích hơn sẽ in ra giá trị từ hàm `greeting`. Hãy thêm một
thông báo lỗi tùy chỉnh bao gồm một chuỗi định dạng với một placeholder được
điền bằng giá trị thực tế mà chúng ta nhận được từ hàm `greeting`:

```rust,ignore
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/src/lib.rs:here}}
```

Bây giờ khi chúng ta chạy bài kiểm thử, chúng ta sẽ nhận được một thông báo lỗi
thông tin hơn:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-07-custom-failure-message/output.txt}}
```

Chúng ta có thể thấy giá trị mà chúng ta thực sự nhận được trong kết quả kiểm
thử, điều này sẽ giúp chúng ta gỡ lỗi những gì đã xảy ra thay vì những gì chúng
ta mong đợi sẽ xảy ra.

### Kiểm Tra Hoảng Loạn với `should_panic`

Ngoài việc kiểm tra giá trị trả về, điều quan trọng là phải kiểm tra rằng code
của chúng ta xử lý các tình huống lỗi như mong đợi. Ví dụ, hãy xem xét kiểu
`Guess` mà chúng ta đã tạo trong Chương 9, Listing 9-13. Các code khác sử dụng
`Guess` phụ thuộc vào sự đảm bảo rằng các instance `Guess` sẽ chỉ chứa các giá
trị từ 1 đến 100. Chúng ta có thể viết một bài kiểm thử để đảm bảo rằng việc cố
gắng tạo một instance `Guess` với một giá trị nằm ngoài phạm vi đó sẽ hoảng
loạn.

Chúng ta làm điều này bằng cách thêm thuộc tính `should_panic` vào hàm kiểm thử
của mình. Bài kiểm thử thành công nếu code bên trong hàm hoảng loạn; bài kiểm
thử thất bại nếu code bên trong hàm không hoảng loạn.

Listing 11-8 cho thấy một bài kiểm thử kiểm tra rằng các điều kiện lỗi của
`Guess::new` xảy ra khi chúng ta mong đợi.

<Listing number="11-8" file-name="src/lib.rs" caption="Kiểm thử rằng một điều kiện sẽ gây ra `panic!`">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-08/src/lib.rs}}
```

</Listing>

Chúng ta đặt thuộc tính `#[should_panic]` sau thuộc tính `#[test]` và trước hàm
kiểm thử mà nó áp dụng. Hãy xem kết quả khi bài kiểm thử này thành công:

```console
{{#include ../listings/ch11-writing-automated-tests/listing-11-08/output.txt}}
```

Trông tốt! Bây giờ hãy đưa một lỗi vào code của chúng ta bằng cách loại bỏ điều
kiện mà hàm `new` sẽ hoảng loạn nếu giá trị lớn hơn 100:

```rust,not_desired_behavior,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/src/lib.rs:here}}
```

Khi chúng ta chạy bài kiểm thử trong Listing 11-8, nó sẽ thất bại:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-08-guess-with-bug/output.txt}}
```

Chúng ta không nhận được thông báo rất hữu ích trong trường hợp này, nhưng khi
chúng ta nhìn vào hàm kiểm thử, chúng ta thấy nó được chú thích với
`#[should_panic]`. Lỗi mà chúng ta nhận được có nghĩa là code trong hàm kiểm thử
không gây ra hoảng loạn.

Các bài kiểm thử sử dụng `should_panic` có thể không chính xác. Một bài kiểm thử
`should_panic` sẽ thành công ngay cả khi bài kiểm thử hoảng loạn vì một lý do
khác với lý do chúng ta mong đợi. Để làm cho các bài kiểm thử `should_panic`
chính xác hơn, chúng ta có thể thêm một tham số `expected` tùy chọn vào thuộc
tính `should_panic`. Công cụ kiểm thử sẽ đảm bảo rằng thông báo lỗi chứa văn bản
được cung cấp. Ví dụ, hãy xem xét code đã sửa đổi cho `Guess` trong Listing
11-9, trong đó hàm `new` hoảng loạn với các thông báo khác nhau tùy thuộc vào
việc giá trị quá nhỏ hay quá lớn.

<Listing number="11-9" file-name="src/lib.rs" caption="Kiểm thử một `panic!` với thông báo hoảng loạn chứa một chuỗi con được chỉ định">

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/listing-11-09/src/lib.rs:here}}
```

</Listing>

Bài kiểm thử này sẽ thành công vì giá trị mà chúng ta đặt trong tham số
`expected` của thuộc tính `should_panic` là một chuỗi con của thông báo mà hàm
`Guess::new` hoảng loạn. Chúng ta có thể đã chỉ định toàn bộ thông báo hoảng
loạn mà chúng ta mong đợi, trong trường hợp này sẽ là
`Guess value must be less than or equal to 100, got 200`. Những gì bạn chọn để
chỉ định phụ thuộc vào phần nào của thông báo hoảng loạn là duy nhất hoặc động
và mức độ chính xác bạn muốn cho bài kiểm thử của mình. Trong trường hợp này,
một chuỗi con của thông báo hoảng loạn là đủ để đảm bảo rằng code trong hàm kiểm
thử thực thi trường hợp `else if value > 100`.

Để xem điều gì xảy ra khi một bài kiểm thử `should_panic` với một thông báo
`expected` thất bại, hãy một lần nữa đưa một lỗi vào code của chúng ta bằng cách
hoán đổi các thân của khối `if value < 1` và `else if value > 100`:

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/src/lib.rs:here}}
```

Lần này khi chúng ta chạy bài kiểm thử `should_panic`, nó sẽ thất bại:

```console
{{#include ../listings/ch11-writing-automated-tests/no-listing-09-guess-with-panic-msg-bug/output.txt}}
```

Thông báo lỗi cho biết rằng bài kiểm thử này thực sự hoảng loạn như chúng ta
mong đợi, nhưng thông báo hoảng loạn không chứa chuỗi mong đợi
`less than or equal to 100`. Thông báo hoảng loạn mà chúng ta nhận được trong
trường hợp này là `Guess value must be greater than or equal to 1, got 200.` Bây
giờ chúng ta có thể bắt đầu tìm ra lỗi ở đâu!

### Sử Dụng `Result<T, E>` trong Các Bài Kiểm Thử

Cho đến nay, các bài kiểm thử của chúng ta đều hoảng loạn khi chúng thất bại.
Chúng ta cũng có thể viết các bài kiểm thử sử dụng `Result<T, E>`! Đây là bài
kiểm thử từ Listing 11-1, được viết lại để sử dụng `Result<T, E>` và trả về một
`Err` thay vì hoảng loạn:

```rust,noplayground
{{#rustdoc_include ../listings/ch11-writing-automated-tests/no-listing-10-result-in-tests/src/lib.rs:here}}
```

Hàm `it_works` bây giờ có kiểu trả về là `Result<(), String>`. Trong thân hàm,
thay vì gọi vĩ lệnh `assert_eq!`, chúng ta trả về `Ok(())` khi bài kiểm thử
thành công và một `Err` với một `String` bên trong khi bài kiểm thử thất bại.

Viết các bài kiểm thử sao cho chúng trả về `Result<T, E>` cho phép bạn sử dụng
toán tử dấu hỏi trong thân các bài kiểm thử, điều này có thể là một cách thuận
tiện để viết các bài kiểm thử nên thất bại nếu bất kỳ thao tác nào trong chúng
trả về biến thể `Err`.

Bạn không thể sử dụng chú thích `#[should_panic]` trên các bài kiểm thử sử dụng
`Result<T, E>`. Để khẳng định rằng một thao tác trả về biến thể `Err`, _đừng_ sử
dụng toán tử dấu hỏi trên giá trị `Result<T, E>`. Thay vào đó, hãy sử dụng
`assert!(value.is_err())`.

Bây giờ bạn đã biết một số cách để viết các bài kiểm thử, hãy xem những gì xảy
ra khi chúng ta chạy các bài kiểm thử của mình và khám phá các tùy chọn khác
nhau mà chúng ta có thể sử dụng với `cargo test`.

[concatenation-with-the--operator-or-the-format-macro]:
  ch08-02-strings.html#concatenation-with-the--operator-or-the-format-macro
[bench]: ../unstable-book/library-features/test.html
[ignoring]:
  ch11-02-running-tests.html#ignoring-some-tests-unless-specifically-requested
[subset]: ch11-02-running-tests.html#running-a-subset-of-tests-by-name
[controlling-how-tests-are-run]:
  ch11-02-running-tests.html#controlling-how-tests-are-run
[derivable-traits]: appendix-03-derivable-traits.html
[doc-comments]:
  ch14-02-publishing-to-crates-io.html#documentation-comments-as-tests
[paths-for-referring-to-an-item-in-the-module-tree]:
  ch07-03-paths-for-referring-to-an-item-in-the-module-tree.html
