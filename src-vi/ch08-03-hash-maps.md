## Lưu Trữ Khóa với Giá Trị Liên Kết trong Bảng Băm

Loại bộ sưu tập phổ biến cuối cùng của chúng ta là _bảng băm_. Kiểu
`HashMap<K, V>` lưu trữ một ánh xạ từ khóa kiểu `K` đến giá trị kiểu `V` bằng
cách sử dụng một _hàm băm_, xác định cách nó đặt các khóa và giá trị này vào bộ
nhớ. Nhiều ngôn ngữ lập trình hỗ trợ cấu trúc dữ liệu này, nhưng chúng thường sử
dụng tên khác nhau, chẳng hạn như _hash_, _map_, _object_, _hash table_,
_dictionary_, hoặc _associative array_, chỉ để kể vài tên.

Bảng băm hữu ích khi bạn muốn tra cứu dữ liệu không bằng cách sử dụng chỉ mục,
như bạn có thể làm với vector, mà bằng cách sử dụng khóa có thể thuộc bất kỳ
kiểu nào. Ví dụ, trong một trò chơi, bạn có thể theo dõi điểm số của mỗi đội
trong một bảng băm trong đó mỗi khóa là tên của một đội và giá trị là điểm số
của mỗi đội. Cho tên đội, bạn có thể lấy ra điểm số của nó.

Chúng ta sẽ xem qua API cơ bản của bảng băm trong phần này, nhưng còn nhiều thứ
hữu ích khác ẩn trong các hàm được định nghĩa trên `HashMap<K, V>` bởi thư viện
chuẩn. Như mọi khi, hãy kiểm tra tài liệu của thư viện chuẩn để biết thêm thông
tin.

### Tạo Bảng Băm Mới

Một cách để tạo bảng băm rỗng là sử dụng `new` và thêm các phần tử bằng
`insert`. Trong Listing 8-20, chúng ta đang theo dõi điểm số của hai đội có tên
là _Blue_ và _Yellow_. Đội Blue bắt đầu với 10 điểm, và đội Yellow bắt đầu
với 50.

<Listing number="8-20" caption="Tạo một bảng băm mới và chèn một số khóa và giá trị">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-20/src/main.rs:here}}
```

</Listing>

Lưu ý rằng chúng ta cần `use` `HashMap` từ phần collections của thư viện chuẩn
trước. Trong ba bộ sưu tập phổ biến của chúng ta, bộ sưu tập này được sử dụng ít
nhất, nên nó không được đưa vào phạm vi tự động trong phần prelude. Bảng băm
cũng nhận được ít hỗ trợ hơn từ thư viện chuẩn; chẳng hạn, không có macro tích
hợp sẵn để xây dựng chúng.

Giống như vector, bảng băm lưu trữ dữ liệu của chúng trên heap. `HashMap` này có
khóa kiểu `String` và giá trị kiểu `i32`. Giống như vector, bảng băm là đồng
nhất: tất cả các khóa phải có cùng kiểu, và tất cả các giá trị phải có cùng
kiểu.

### Truy Cập Giá Trị trong Bảng Băm

Chúng ta có thể lấy một giá trị từ bảng băm bằng cách cung cấp khóa của nó cho
phương thức `get`, như được hiển thị trong Listing 8-21.

<Listing number="8-21" caption="Truy cập điểm số của đội Blue được lưu trữ trong bảng băm">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-21/src/main.rs:here}}
```

</Listing>

Ở đây, `score` sẽ có giá trị được liên kết với đội Blue, và kết quả sẽ là `10`.
Phương thức `get` trả về một `Option<&V>`; nếu không có giá trị cho khóa đó
trong bảng băm, `get` sẽ trả về `None`. Chương trình này xử lý `Option` bằng
cách gọi `copied` để lấy một `Option<i32>` thay vì một `Option<&i32>`, sau đó
`unwrap_or` để đặt `score` thành không nếu `scores` không có mục cho khóa.

Chúng ta có thể lặp qua từng cặp khóa-giá trị trong một bảng băm theo cách tương
tự như chúng ta làm với vector, sử dụng vòng lặp `for`:

```rust
{{#rustdoc_include ../listings/ch08-common-collections/no-listing-03-iterate-over-hashmap/src/main.rs:here}}
```

Mã này sẽ in mỗi cặp theo thứ tự ngẫu nhiên:

```text
Yellow: 50
Blue: 10
```

### Bảng Băm và Quyền Sở Hữu

Đối với các kiểu triển khai trait `Copy`, như `i32`, các giá trị được sao chép
vào bảng băm. Đối với các giá trị được sở hữu như `String`, các giá trị sẽ bị di
chuyển và bảng băm sẽ là chủ sở hữu của các giá trị đó, như được minh họa trong
Listing 8-22.

<Listing number="8-22" caption="Hiển thị rằng khóa và giá trị được sở hữu bởi bảng băm sau khi chúng được chèn">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-22/src/main.rs:here}}
```

</Listing>

Chúng ta không thể sử dụng các biến `field_name` và `field_value` sau khi chúng
đã được di chuyển vào bảng băm bằng lệnh gọi đến `insert`.

Nếu chúng ta chèn tham chiếu đến các giá trị vào bảng băm, các giá trị sẽ không
bị di chuyển vào bảng băm. Các giá trị mà các tham chiếu trỏ đến phải hợp lệ ít
nhất miễn là bảng băm còn hợp lệ. Chúng ta sẽ nói thêm về các vấn đề này trong
["Xác thực Tham chiếu với Vòng
đời"][validating-references-with-lifetimes]<!-- ignore --> trong Chương 10.

### Cập Nhật Bảng Băm

Mặc dù số lượng cặp khóa và giá trị có thể tăng lên, mỗi khóa duy nhất chỉ có
thể có một giá trị liên kết với nó tại một thời điểm (nhưng không ngược lại: ví
dụ, cả đội Blue và đội Yellow đều có thể có giá trị `10` được lưu trữ trong bảng
băm `scores`).

Khi bạn muốn thay đổi dữ liệu trong một bảng băm, bạn phải quyết định cách xử lý
trường hợp khi một khóa đã có một giá trị được gán. Bạn có thể thay thế giá trị
cũ bằng giá trị mới, hoàn toàn bỏ qua giá trị cũ. Bạn có thể giữ giá trị cũ và
bỏ qua giá trị mới, chỉ thêm giá trị mới nếu khóa _không_ đã có một giá trị.
Hoặc bạn có thể kết hợp giá trị cũ và giá trị mới. Hãy xem cách thực hiện mỗi
điều này!

#### Ghi Đè Một Giá Trị

Nếu chúng ta chèn một khóa và một giá trị vào một bảng băm và sau đó chèn cùng
khóa đó với một giá trị khác, giá trị liên kết với khóa đó sẽ bị thay thế. Mặc
dù mã trong Listing 8-23 gọi `insert` hai lần, bảng băm sẽ chỉ chứa một cặp
khóa-giá trị vì chúng ta đang chèn giá trị cho khóa của đội Blue cả hai lần.

<Listing number="8-23" caption="Thay thế giá trị được lưu trữ với một khóa cụ thể">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-23/src/main.rs:here}}
```

</Listing>

Mã này sẽ in `{"Blue": 25}`. Giá trị ban đầu `10` đã bị ghi đè.

<!-- Old headings. Do not remove or links may break. -->

<a id="only-inserting-a-value-if-the-key-has-no-value"></a>

#### Chỉ Thêm Khóa và Giá Trị Nếu Khóa Không Tồn Tại

Thông thường, người ta kiểm tra xem một khóa cụ thể đã tồn tại trong bảng băm
với một giá trị hay chưa và sau đó thực hiện các hành động sau: nếu khóa đó tồn
tại trong bảng băm, giá trị hiện tại nên giữ nguyên như vậy; nếu khóa không tồn
tại, chèn nó và một giá trị cho nó.

Bảng băm có một API đặc biệt cho việc này gọi là `entry` lấy khóa bạn muốn kiểm
tra làm tham số. Giá trị trả về của phương thức `entry` là một enum gọi là
`Entry` đại diện cho một giá trị có thể tồn tại hoặc không. Giả sử chúng ta muốn
kiểm tra xem khóa cho đội Yellow có giá trị liên kết với nó không. Nếu không,
chúng ta muốn chèn giá trị `50`, và tương tự cho đội Blue. Sử dụng API `entry`,
mã trông giống như Listing 8-24.

<Listing number="8-24" caption="Sử dụng phương thức `entry` để chỉ chèn nếu khóa chưa có giá trị">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-24/src/main.rs:here}}
```

</Listing>

Phương thức `or_insert` trên `Entry` được định nghĩa để trả về một tham chiếu có
thể thay đổi đến giá trị cho khóa `Entry` tương ứng nếu khóa đó tồn tại, và nếu
không, nó chèn tham số làm giá trị mới cho khóa này và trả về một tham chiếu có
thể thay đổi đến giá trị mới. Kỹ thuật này sạch sẽ hơn nhiều so với việc tự viết
logic và, ngoài ra, hoạt động tốt hơn với bộ kiểm tra mượn.

Chạy mã trong Listing 8-24 sẽ in `{"Yellow": 50, "Blue": 10}`. Lệnh gọi đầu tiên
đến `entry` sẽ chèn khóa cho đội Yellow với giá trị `50` vì đội Yellow chưa có
giá trị. Lệnh gọi thứ hai đến `entry` sẽ không thay đổi bảng băm vì đội Blue đã
có giá trị `10`.

#### Cập Nhật Giá Trị Dựa trên Giá Trị Cũ

Trường hợp sử dụng phổ biến khác cho bảng băm là tra cứu giá trị của một khóa và
sau đó cập nhật nó dựa trên giá trị cũ. Ví dụ, Listing 8-25 hiển thị mã đếm số
lần xuất hiện của mỗi từ trong một đoạn văn bản. Chúng ta sử dụng bảng băm với
các từ làm khóa và tăng giá trị để theo dõi số lần chúng ta đã thấy từ đó. Nếu
đó là lần đầu tiên chúng ta thấy một từ, chúng ta sẽ chèn giá trị `0` trước.

<Listing number="8-25" caption="Đếm số lần xuất hiện của các từ bằng cách sử dụng bảng băm lưu trữ từ và số lượng">

```rust
{{#rustdoc_include ../listings/ch08-common-collections/listing-08-25/src/main.rs:here}}
```

</Listing>

Mã này sẽ in `{"world": 2, "hello": 1, "wonderful": 1}`. Bạn có thể thấy các cặp
khóa-giá trị giống nhau được in ra theo thứ tự khác nhau: hãy nhớ lại từ ["Truy
cập Giá trị trong Bảng băm"][access]<!-- ignore --> rằng việc lặp qua một bảng
băm xảy ra theo thứ tự ngẫu nhiên.

Phương thức `split_whitespace` trả về một iterator qua các lát cắt con, được
phân tách bởi khoảng trắng, của giá trị trong `text`. Phương thức `or_insert`
trả về một tham chiếu có thể thay đổi (`&mut V`) đến giá trị cho khóa được chỉ
định. Ở đây, chúng ta lưu trữ tham chiếu có thể thay đổi đó trong biến `count`,
vì vậy để gán cho giá trị đó, chúng ta phải giải tham chiếu `count` bằng cách sử
dụng dấu hoa thị (`*`). Tham chiếu có thể thay đổi hết phạm vi tại cuối vòng lặp
`for`, vì vậy tất cả các thay đổi này là an toàn và được cho phép bởi các quy
tắc mượn.

### Hàm Băm

Theo mặc định, `HashMap` sử dụng một hàm băm gọi là _SipHash_ có thể cung cấp
khả năng chống lại các cuộc tấn công từ chối dịch vụ (DoS) liên quan đến bảng
băm[^siphash]<!-- ignore -->. Đây không phải là thuật toán băm nhanh nhất có
sẵn, nhưng sự đánh đổi cho bảo mật tốt hơn đi kèm với sự sụt giảm hiệu suất là
đáng giá. Nếu bạn phân tích mã của mình và thấy rằng hàm băm mặc định quá chậm
cho mục đích của bạn, bạn có thể chuyển sang một hàm khác bằng cách chỉ định một
bộ băm khác. Một _hasher_ là một kiểu triển khai trait `BuildHasher`. Chúng ta
sẽ nói về trait và cách triển khai chúng trong [Chương
10][traits]<!-- ignore -->. Bạn không nhất thiết phải triển khai bộ băm riêng
của bạn từ đầu; [crates.io](https://crates.io/)<!-- ignore --> có các thư viện
được chia sẻ bởi người dùng Rust khác cung cấp các hasher triển khai nhiều thuật
toán băm phổ biến.

[^siphash]:
    [https://en.wikipedia.org/wiki/SipHash](https://en.wikipedia.org/wiki/SipHash)

## Tóm tắt

Vector, chuỗi và bảng băm sẽ cung cấp một lượng lớn chức năng cần thiết trong
các chương trình khi bạn cần lưu trữ, truy cập và sửa đổi dữ liệu. Đây là một số
bài tập mà bạn bây giờ đã có thể giải quyết:

1. Cho một danh sách các số nguyên, sử dụng vector và trả về trung vị (khi được
   sắp xếp, giá trị ở vị trí giữa) và mode (giá trị xuất hiện nhiều nhất; một
   bảng băm sẽ hữu ích ở đây) của danh sách.
1. Chuyển đổi chuỗi sang pig latin. Phụ âm đầu tiên của mỗi từ được di chuyển
   đến cuối từ và thêm _ay_, vì vậy _first_ trở thành _irst-fay_. Các từ bắt đầu
   bằng một nguyên âm được thêm _hay_ vào cuối thay vào đó (_apple_ trở thành
   _apple-hay_). Hãy ghi nhớ các chi tiết về mã hóa UTF-8!
1. Sử dụng bảng băm và vector, tạo một giao diện văn bản để cho phép người dùng
   thêm tên nhân viên vào một phòng ban trong công ty; ví dụ, "Add Sally to
   Engineering" hoặc "Add Amir to Sales." Sau đó cho phép người dùng lấy danh
   sách tất cả mọi người trong một phòng ban hoặc tất cả mọi người trong công ty
   theo phòng ban, được sắp xếp theo thứ tự bảng chữ cái.

Tài liệu API thư viện chuẩn mô tả các phương thức mà vector, chuỗi, và bảng băm
có sẽ hữu ích cho các bài tập này!

Chúng ta đang đi vào các chương trình phức tạp hơn trong đó các hoạt động có thể
thất bại, vì vậy đó là thời điểm hoàn hảo để thảo luận về xử lý lỗi. Chúng ta sẽ
làm điều đó tiếp theo!

[validating-references-with-lifetimes]:
  ch10-03-lifetime-syntax.html#validating-references-with-lifetimes
[access]: #accessing-values-in-a-hash-map
[traits]: ch10-02-traits.html
