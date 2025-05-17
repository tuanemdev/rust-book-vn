## Xử lý Một Chuỗi Các Item với Iterators

Mẫu iterator cho phép bạn thực hiện một nhiệm vụ nào đó trên một chuỗi các phần
tử lần lượt. Một iterator chịu trách nhiệm cho logic lặp qua từng phần tử và xác
định khi nào chuỗi đã kết thúc. Khi bạn sử dụng iterators, bạn không phải tái
thực hiện logic đó cho chính mình.

Trong Rust, iterators là _lười biếng_, nghĩa là chúng không có tác dụng gì cho
đến khi bạn gọi các phương thức tiêu thụ iterator để sử dụng nó. Ví dụ, mã trong
Listing 13-10 tạo một iterator trên các phần tử trong vector `v1` bằng cách gọi
phương thức `iter` được định nghĩa trên `Vec<T>`. Mã này tự nó không làm gì có
ích.

<Listing number="13-10" file-name="src/main.rs" caption="Tạo một iterator">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-10/src/main.rs:here}}
```

</Listing>

Iterator được lưu trữ trong biến `v1_iter`. Một khi chúng ta đã tạo một
iterator, chúng ta có thể sử dụng nó theo nhiều cách khác nhau. Trong Listing
3-5, chúng ta đã lặp qua một mảng bằng vòng lặp `for` để thực thi một số mã trên
mỗi phần tử của nó. Bên dưới, điều này ngầm tạo ra và sau đó tiêu thụ một
iterator, nhưng chúng ta đã bỏ qua cách nó hoạt động chính xác cho đến bây giờ.

Trong ví dụ ở Listing 13-11, chúng ta tách việc tạo iterator khỏi việc sử dụng
iterator trong vòng lặp `for`. Khi vòng lặp `for` được gọi sử dụng iterator
trong `v1_iter`, mỗi phần tử trong iterator được sử dụng trong một lần lặp của
vòng lặp, điều này in ra mỗi giá trị.

<Listing number="13-11" file-name="src/main.rs" caption="Sử dụng một iterator trong vòng lặp `for`">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-11/src/main.rs:here}}
```

</Listing>

Trong các ngôn ngữ không có iterators được cung cấp bởi thư viện chuẩn của
chúng, bạn có thể sẽ viết cùng một chức năng này bằng cách bắt đầu với một biến
ở chỉ số 0, sử dụng biến đó để truy cập vào vector để lấy một giá trị, và tăng
giá trị biến trong một vòng lặp cho đến khi nó đạt đến tổng số phần tử trong
vector.

Iterators xử lý tất cả logic đó cho bạn, cắt giảm mã lặp lại mà bạn có thể làm
rối. Iterators cho bạn thêm sự linh hoạt để sử dụng cùng một logic với nhiều
loại chuỗi khác nhau, không chỉ các cấu trúc dữ liệu mà bạn có thể truy cập theo
chỉ mục, như vectors. Hãy xem cách iterators làm điều đó.

### Trait `Iterator` và Phương thức `next`

Tất cả iterators đều thực hiện một trait có tên `Iterator` được định nghĩa trong
thư viện chuẩn. Định nghĩa của trait trông như thế này:

```rust
pub trait Iterator {
    type Item;

    fn next(&mut self) -> Option<Self::Item>;

    // phương thức với các triển khai mặc định được lược bỏ
}
```

Lưu ý rằng định nghĩa này sử dụng một số cú pháp mới: `type Item` và
`Self::Item`, đang định nghĩa một _kiểu liên kết_ với trait này. Chúng ta sẽ nói
về các kiểu liên kết một cách sâu sắc trong Chương 20. Hiện tại, tất cả những gì
bạn cần biết là mã này nói rằng việc thực hiện trait `Iterator` yêu cầu bạn cũng
phải định nghĩa một kiểu `Item`, và kiểu `Item` này được sử dụng trong kiểu trả
về của phương thức `next`. Nói cách khác, kiểu `Item` sẽ là kiểu được trả về từ
iterator.

Trait `Iterator` chỉ yêu cầu những người thực hiện định nghĩa một phương thức:
phương thức `next`, trả về một phần tử của iterator mỗi lần, được bọc trong
`Some`, và, khi lặp kết thúc, trả về `None`.

Chúng ta có thể gọi phương thức `next` trực tiếp trên iterators; Listing 13-12
minh họa các giá trị nào được trả về từ các lệnh gọi lặp lại đến `next` trên
iterator được tạo từ vector.

<Listing number="13-12" file-name="src/lib.rs" caption="Gọi phương thức `next` trên một iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-12/src/lib.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cần làm cho `v1_iter` có thể thay đổi: gọi phương thức
`next` trên một iterator thay đổi trạng thái nội bộ mà iterator sử dụng để theo
dõi vị trí của nó trong chuỗi. Nói cách khác, mã này _tiêu thụ_, hoặc sử dụng
hết, iterator. Mỗi lệnh gọi đến `next` tiêu thụ một phần tử từ iterator. Chúng
ta không cần làm cho `v1_iter` có thể thay đổi khi chúng ta sử dụng vòng lặp
`for` vì vòng lặp đã lấy quyền sở hữu của `v1_iter` và làm cho nó có thể thay
đổi đằng sau hậu trường.

Cũng lưu ý rằng các giá trị chúng ta nhận được từ các lệnh gọi đến `next` là các
tham chiếu bất biến đến các giá trị trong vector. Phương thức `iter` tạo ra một
iterator trên các tham chiếu bất biến. Nếu chúng ta muốn tạo một iterator lấy
quyền sở hữu của `v1` và trả về các giá trị thuộc sở hữu, chúng ta có thể gọi
`into_iter` thay vì `iter`. Tương tự, nếu chúng ta muốn lặp qua các tham chiếu
có thể thay đổi, chúng ta có thể gọi `iter_mut` thay vì `iter`.

### Các Phương thức Tiêu thụ Iterator

Trait `Iterator` có một số phương thức khác nhau với các triển khai mặc định
được cung cấp bởi thư viện chuẩn; bạn có thể tìm hiểu về các phương thức này
bằng cách xem tài liệu API thư viện chuẩn cho trait `Iterator`. Một số phương
thức này gọi phương thức `next` trong định nghĩa của chúng, đó là lý do tại sao
bạn được yêu cầu thực hiện phương thức `next` khi thực hiện trait `Iterator`.

Các phương thức gọi `next` được gọi là _bộ điều hợp tiêu thụ_, vì gọi chúng sử
dụng hết iterator. Một ví dụ là phương thức `sum`, lấy quyền sở hữu của iterator
và lặp qua các phần tử bằng cách liên tục gọi `next`, do đó tiêu thụ iterator.
Khi nó lặp qua, nó thêm mỗi phần tử vào một tổng đang chạy và trả về tổng khi
lặp hoàn thành. Listing 13-13 có một test minh họa việc sử dụng phương thức
`sum`.

<Listing number="13-13" file-name="src/lib.rs" caption="Gọi phương thức `sum` để lấy tổng của tất cả các phần tử trong iterator">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-13/src/lib.rs:here}}
```

</Listing>

Chúng ta không được phép sử dụng `v1_iter` sau lệnh gọi đến `sum` vì `sum` lấy
quyền sở hữu của iterator mà chúng ta gọi nó trên.

### Các Phương thức tạo ra Iterator Khác

_Bộ điều hợp iterator_ là các phương thức được định nghĩa trên trait `Iterator`
không tiêu thụ iterator. Thay vào đó, chúng tạo ra các iterator khác nhau bằng
cách thay đổi một số khía cạnh của iterator gốc.

Listing 13-14 hiển thị một ví dụ về việc gọi phương thức điều hợp iterator
`map`, lấy một closure để gọi trên mỗi phần tử khi các phần tử được lặp qua.
Phương thức `map` trả về một iterator mới tạo ra các phần tử đã được sửa đổi.
Closure ở đây tạo ra một iterator mới trong đó mỗi phần tử từ vector sẽ được
tăng lên 1.

<Listing number="13-14" file-name="src/main.rs" caption="Gọi bộ điều hợp iterator `map` để tạo một iterator mới">

```rust,not_desired_behavior
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-14/src/main.rs:here}}
```

</Listing>

Tuy nhiên, mã này tạo ra một cảnh báo:

```console
{{#include ../listings/ch13-functional-features/listing-13-14/output.txt}}
```

Mã trong Listing 13-14 không làm gì cả; closure mà chúng ta đã chỉ định không
bao giờ được gọi. Cảnh báo nhắc nhở chúng ta lý do tại sao: bộ điều hợp iterator
là lười biếng, và chúng ta cần tiêu thụ iterator ở đây.

Để sửa cảnh báo này và tiêu thụ iterator, chúng ta sẽ sử dụng phương thức
`collect`, mà chúng ta đã sử dụng với `env::args` trong Listing 12-1. Phương
thức này tiêu thụ iterator và thu thập các giá trị kết quả vào một kiểu dữ liệu
tập hợp.

Trong Listing 13-15, chúng ta thu thập các kết quả của việc lặp qua iterator
được trả về từ lệnh gọi đến `map` vào một vector. Vector này sẽ kết thúc chứa
mỗi phần tử từ vector gốc, tăng lên 1.

<Listing number="13-15" file-name="src/main.rs" caption="Gọi phương thức `map` để tạo một iterator mới, và sau đó gọi phương thức `collect` để tiêu thụ iterator mới và tạo một vector">

```rust
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-15/src/main.rs:here}}
```

</Listing>

Bởi vì `map` lấy một closure, chúng ta có thể chỉ định bất kỳ hoạt động nào mà
chúng ta muốn thực hiện trên mỗi phần tử. Đây là một ví dụ tuyệt vời về cách
closures cho phép bạn tùy chỉnh một số hành vi trong khi tái sử dụng hành vi lặp
lại mà trait `Iterator` cung cấp.

Bạn có thể nối nhiều lệnh gọi đến bộ điều hợp iterator để thực hiện các hoạt
động phức tạp theo cách dễ đọc. Nhưng bởi vì tất cả iterators đều lười biếng,
bạn phải gọi một trong các phương thức bộ điều hợp tiêu thụ để có được kết quả
từ các lệnh gọi đến bộ điều hợp iterator.

### Sử dụng Closures Capture Môi trường của Chúng

Nhiều bộ điều hợp iterator lấy closure làm đối số, và thường các closure mà
chúng ta sẽ chỉ định làm đối số cho bộ điều hợp iterator sẽ là closure capture
môi trường của chúng.

Cho ví dụ này, chúng ta sẽ sử dụng phương thức `filter` nhận một closure.
Closure lấy một phần tử từ iterator và trả về một `bool`. Nếu closure trả về
`true`, giá trị sẽ được bao gồm trong lần lặp được tạo ra bởi `filter`. Nếu
closure trả về `false`, giá trị sẽ không được bao gồm.

Trong Listing 13-16, chúng ta sử dụng `filter` với một closure capture biến
`shoe_size` từ môi trường của nó để lặp qua một tập hợp các phiên bản struct
`Shoe`. Nó sẽ chỉ trả về những đôi giày có kích thước được chỉ định.

<Listing number="13-16" file-name="src/lib.rs" caption="Sử dụng phương thức `filter` với một closure capture `shoe_size`">

```rust,noplayground
{{#rustdoc_include ../listings/ch13-functional-features/listing-13-16/src/lib.rs}}
```

</Listing>

Hàm `shoes_in_size` lấy quyền sở hữu của một vector giày và một kích thước giày
làm tham số. Nó trả về một vector chỉ chứa giày có kích thước được chỉ định.

Trong phần thân của `shoes_in_size`, chúng ta gọi `into_iter` để tạo một
iterator lấy quyền sở hữu của vector. Sau đó, chúng ta gọi `filter` để điều
chỉnh iterator đó thành một iterator mới chỉ chứa các phần tử mà closure trả về
`true`.

Closure capture tham số `shoe_size` từ môi trường và so sánh giá trị đó với kích
thước của mỗi đôi giày, giữ lại chỉ những đôi giày có kích thước được chỉ định.
Cuối cùng, gọi `collect` thu thập các giá trị được trả về bởi iterator đã điều
chỉnh vào một vector được trả về bởi hàm.

Test cho thấy rằng khi chúng ta gọi `shoes_in_size`, chúng ta chỉ nhận lại những
đôi giày có cùng kích thước với giá trị mà chúng ta đã chỉ định.
