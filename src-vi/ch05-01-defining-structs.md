## Định nghĩa và Khởi tạo Struct

Struct tương tự như tuple, đã được thảo luận trong phần ["Kiểu
Tuple"][tuples]<!--
ignore -->, ở chỗ cả hai đều chứa nhiều giá trị liên quan. Giống như tuple, các thành
phần của một struct có thể có kiểu khác nhau. Khác với tuple, trong một struct, bạn
sẽ đặt tên cho từng phần dữ liệu để làm rõ ý nghĩa của các giá trị. Việc thêm các
tên này có nghĩa là struct linh hoạt hơn tuple: bạn không phải dựa vào thứ tự của
dữ liệu để xác định hoặc truy cập các giá trị của một thực thể.

Để định nghĩa một struct, chúng ta nhập từ khóa `struct` và đặt tên cho toàn bộ
struct. Tên của struct nên mô tả ý nghĩa của các phần dữ liệu được nhóm lại với
nhau. Sau đó, bên trong dấu ngoặc nhọn, chúng ta định nghĩa tên và kiểu của các
phần dữ liệu, mà chúng ta gọi là _trường_ (fields). Ví dụ, Listing 5-1 hiển thị
một struct lưu trữ thông tin về tài khoản người dùng.

<Listing number="5-1" file-name="src/main.rs" caption="Định nghĩa struct `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-01/src/main.rs:here}}
```

</Listing>

Để sử dụng struct sau khi đã định nghĩa nó, chúng ta tạo ra một _instance_ (thực
thể) của struct đó bằng cách xác định giá trị cụ thể cho mỗi trường. Chúng ta
tạo một thực thể bằng cách nêu tên của struct và sau đó thêm dấu ngoặc nhọn chứa
các cặp _`key: value`_, trong đó key là tên của các trường và value là dữ liệu
mà chúng ta muốn lưu trữ trong các trường đó. Chúng ta không cần phải chỉ định
các trường theo cùng thứ tự mà chúng ta đã khai báo trong struct. Nói cách khác,
định nghĩa struct giống như một khuôn mẫu chung cho kiểu dữ liệu, và các thực
thể điền vào khuôn mẫu đó với dữ liệu cụ thể để tạo ra các giá trị của kiểu đó.
Ví dụ, chúng ta có thể khai báo một người dùng cụ thể như trong Listing 5-2.

<Listing number="5-2" file-name="src/main.rs" caption="Tạo một thực thể của struct `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-02/src/main.rs:here}}
```

</Listing>

Để lấy một giá trị cụ thể từ struct, chúng ta sử dụng ký hiệu dấu chấm. Ví dụ,
để truy cập địa chỉ email của người dùng này, chúng ta sử dụng `user1.email`.
Nếu thực thể là có thể thay đổi, chúng ta có thể thay đổi giá trị bằng cách sử
dụng ký hiệu dấu chấm và gán vào một trường cụ thể. Listing 5-3 cho thấy cách
thay đổi giá trị trong trường `email` của một thực thể `User` có thể thay đổi.

<Listing number="5-3" file-name="src/main.rs" caption="Thay đổi giá trị trong trường `email` của một thực thể `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-03/src/main.rs:here}}
```

</Listing>

Lưu ý rằng toàn bộ thực thể phải là có thể thay đổi; Rust không cho phép chúng
ta đánh dấu chỉ một số trường nhất định là có thể thay đổi. Giống như với bất kỳ
biểu thức nào, chúng ta có thể tạo một thực thể mới của struct làm biểu thức
cuối cùng trong thân hàm để ngầm định trả về thực thể mới đó.

Listing 5-4 hiển thị một hàm `build_user` trả về một thực thể `User` với email
và tên người dùng đã cho. Trường `active` nhận giá trị là `true`, và
`sign_in_count` nhận giá trị là `1`.

<Listing number="5-4" file-name="src/main.rs" caption="Hàm `build_user` nhận email và tên người dùng và trả về một thực thể `User`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-04/src/main.rs:here}}
```

</Listing>

Việc đặt tên các tham số của hàm giống với tên trường của struct là hợp lý,
nhưng việc phải lặp lại tên trường `email` và `username` cùng với biến là hơi tẻ
nhạt. Nếu struct có nhiều trường hơn, việc lặp lại mỗi tên sẽ càng phiền phức
hơn. May mắn thay, có một cách viết tắt thuận tiện!

<!-- Old heading. Do not remove or links may break. -->

<a id="using-the-field-init-shorthand-when-variables-and-fields-have-the-same-name"></a>

### Sử dụng Field Init Shorthand

Vì tên tham số và tên trường struct là hoàn toàn giống nhau trong Listing 5-4,
chúng ta có thể sử dụng cú pháp _field init shorthand_ để viết lại `build_user`
để nó hoạt động chính xác như cũ nhưng không có sự lặp lại của `username` và
`email`, như trong Listing 5-5.

<Listing number="5-5" file-name="src/main.rs" caption="Hàm `build_user` sử dụng field init shorthand vì tham số `username` và `email` có cùng tên với trường struct">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-05/src/main.rs:here}}
```

</Listing>

Ở đây, chúng ta đang tạo một thực thể mới của struct `User`, nó có một trường
tên là `email`. Chúng ta muốn đặt giá trị của trường `email` thành giá trị trong
tham số `email` của hàm `build_user`. Vì trường `email` và tham số `email` có
cùng tên, chúng ta chỉ cần viết `email` thay vì `email: email`.

### Tạo Thực thể từ Các Thực thể Khác với Cú pháp Cập nhật Struct

Thường rất hữu ích khi tạo một thực thể mới của một struct bao gồm hầu hết các
giá trị từ một thực thể khác của cùng kiểu, nhưng thay đổi một số. Bạn có thể
làm điều này bằng cách sử dụng _cú pháp cập nhật struct_ (struct update syntax).

Đầu tiên, trong Listing 5-6, chúng ta thấy cách tạo một thực thể `User` mới
trong `user2` theo cách thông thường, không sử dụng cú pháp cập nhật. Chúng ta
đặt một giá trị mới cho `email` nhưng sử dụng các giá trị giống nhau từ `user1`
mà chúng ta đã tạo ra trong Listing 5-2.

<Listing number="5-6" file-name="src/main.rs" caption="Tạo một thực thể `User` mới sử dụng tất cả trừ một giá trị từ `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-06/src/main.rs:here}}
```

</Listing>

Sử dụng cú pháp cập nhật struct, chúng ta có thể đạt được cùng kết quả với ít mã
hơn, như trong Listing 5-7. Cú pháp `..` chỉ định rằng các trường còn lại không
được đặt rõ ràng sẽ có cùng giá trị với các trường trong thực thể đã cho.

<Listing number="5-7" file-name="src/main.rs" caption="Sử dụng cú pháp cập nhật struct để đặt một giá trị `email` mới cho một thực thể `User` nhưng để sử dụng phần còn lại của các giá trị từ `user1`">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/listing-05-07/src/main.rs:here}}
```

</Listing>

Mã trong Listing 5-7 cũng tạo ra một thực thể trong `user2` có giá trị khác cho
`email` nhưng có cùng giá trị cho các trường `username`, `active`, và
`sign_in_count` từ `user1`. `..user1` phải đặt cuối cùng để chỉ định rằng bất kỳ
trường còn lại nào nên lấy giá trị của chúng từ các trường tương ứng trong
`user1`, nhưng chúng ta có thể chọn chỉ định giá trị cho nhiều trường tùy ý theo
bất kỳ thứ tự nào, bất kể thứ tự của các trường trong định nghĩa struct.

Lưu ý rằng cú pháp cập nhật struct sử dụng `=` như một phép gán; điều này là vì
nó di chuyển dữ liệu, giống như chúng ta đã thấy trong phần ["Variables and Data
Interacting with Move"][move]<!-- ignore -->. Trong ví dụ này, chúng ta không
còn có thể sử dụng `user1` sau khi tạo `user2` vì `String` trong trường
`username` của `user1` đã được di chuyển vào `user2`. Nếu chúng ta đã cung cấp
cho `user2` giá trị `String` mới cho cả `email` và `username`, và do đó chỉ sử
dụng các giá trị `active` và `sign_in_count` từ `user1`, thì `user1` vẫn sẽ hợp
lệ sau khi tạo ra `user2`. Cả `active` và `sign_in_count` đều là các kiểu thực
hiện trait `Copy`, do đó hành vi mà chúng ta đã thảo luận trong phần
["Stack-Only Data: Copy"][copy]<!-- ignore --> sẽ được áp dụng. Chúng ta cũng
vẫn có thể sử dụng `user1.email` trong ví dụ này, vì giá trị của nó không bị di
chuyển ra khỏi `user1`.

### Sử dụng Tuple Struct Không Có Trường Được Đặt Tên để Tạo Các Kiểu Khác Nhau

Rust cũng hỗ trợ struct trông giống như tuple, được gọi là _tuple struct_. Tuple
struct có thêm ý nghĩa mà tên struct cung cấp nhưng không có tên được liên kết
với các trường của chúng; thay vào đó, chúng chỉ có các kiểu của các trường.
Tuple struct hữu ích khi bạn muốn đặt tên cho toàn bộ tuple và làm cho tuple đó
thành một kiểu khác với các tuple khác, và khi đặt tên cho mỗi trường như trong
một struct thông thường sẽ dài dòng hoặc thừa thãi.

Để định nghĩa một tuple struct, bắt đầu với từ khóa `struct` và tên struct theo
sau là các kiểu trong tuple. Ví dụ, ở đây chúng ta định nghĩa và sử dụng hai
tuple struct có tên là `Color` và `Point`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-01-tuple-structs/src/main.rs}}
```

</Listing>

Lưu ý rằng giá trị `black` và `origin` là các kiểu khác nhau vì chúng là các
thực thể của các tuple struct khác nhau. Mỗi struct bạn định nghĩa là kiểu riêng
của nó, ngay cả khi các trường trong struct có thể có cùng kiểu. Ví dụ, một hàm
nhận một tham số kiểu `Color` không thể nhận một `Point` làm đối số, mặc dù cả
hai kiểu đều được tạo thành từ ba giá trị `i32`. Nếu không, các thực thể tuple
struct tương tự như tuple ở chỗ bạn có thể phân rã chúng thành các phần riêng
lẻ, và bạn có thể sử dụng `.` theo sau bởi chỉ số để truy cập một giá trị riêng
lẻ. Khác với tuple, tuple struct yêu cầu bạn đặt tên kiểu của struct khi bạn
phân rã chúng. Ví dụ, chúng ta sẽ viết `let Point(x, y, z) = origin;` để phân rã
các giá trị trong điểm `origin` thành các biến có tên là `x`, `y`, và `z`.

### Unit-Like Struct Không Có Trường Nào

Bạn cũng có thể định nghĩa struct không có trường nào! Những thứ này được gọi là
_unit-like struct_ vì chúng hoạt động tương tự như `()`, kiểu đơn vị mà chúng ta
đã đề cập trong phần ["The Tuple Type"][tuples]<!-- ignore -->. Unit-like struct
có thể hữu ích khi bạn cần thực hiện một trait trên một kiểu nhưng không có bất
kỳ dữ liệu nào mà bạn muốn lưu trữ trong chính kiểu đó. Chúng ta sẽ thảo luận về
trait trong Chương 10. Đây là một ví dụ về việc khai báo và khởi tạo một unit
struct có tên là `AlwaysEqual`:

<Listing file-name="src/main.rs">

```rust
{{#rustdoc_include ../listings/ch05-using-structs-to-structure-related-data/no-listing-04-unit-like-structs/src/main.rs}}
```

</Listing>

Để định nghĩa `AlwaysEqual`, chúng ta sử dụng từ khóa `struct`, tên mà chúng ta
muốn, và sau đó là dấu chấm phẩy. Không cần dấu ngoặc nhọn hoặc dấu ngoặc tròn!
Sau đó chúng ta có thể lấy một thực thể của `AlwaysEqual` trong biến `subject`
theo cách tương tự: sử dụng tên mà chúng ta đã định nghĩa, không có dấu ngoặc
nhọn hoặc dấu ngoặc tròn nào. Hãy tưởng tượng rằng sau này chúng ta sẽ thực hiện
hành vi cho kiểu này sao cho mọi thực thể của `AlwaysEqual` luôn bằng với mọi
thực thể của bất kỳ kiểu nào khác, có lẽ để có một kết quả đã biết cho mục đích
kiểm tra. Chúng ta sẽ không cần bất kỳ dữ liệu nào để thực hiện hành vi đó! Bạn
sẽ thấy trong Chương 10 cách định nghĩa trait và thực hiện chúng trên bất kỳ
kiểu nào, bao gồm cả unit-like struct.

> ### Quyền sở hữu Dữ liệu trong Struct
>
> Trong định nghĩa struct `User` ở Listing 5-1, chúng ta đã sử dụng kiểu
> `String` có quyền sở hữu thay vì kiểu string slice `&str`. Đây là một lựa chọn
> có chủ đích bởi vì chúng ta muốn mỗi thực thể của struct này sở hữu tất cả dữ
> liệu của nó và cho dữ liệu đó hợp lệ miễn là toàn bộ struct còn hợp lệ.
>
> Cũng có thể để struct lưu trữ các tham chiếu đến dữ liệu thuộc sở hữu của một
> thứ khác, nhưng để làm điều đó đòi hỏi phải sử dụng _lifetimes_, một tính năng
> của Rust mà chúng ta sẽ thảo luận trong Chương 10. Lifetimes đảm bảo rằng dữ
> liệu được tham chiếu bởi một struct là hợp lệ miễn là struct còn tồn tại. Hãy
> giả sử bạn cố gắng lưu trữ một tham chiếu trong một struct mà không chỉ định
> lifetimes, như sau; điều này sẽ không hoạt động:
>
> <Listing file-name="src/main.rs">
>
> <!-- CAN'T EXTRACT SEE https://github.com/rust-lang/mdBook/issues/1127 -->
>
> ```rust,ignore,does_not_compile
> struct User {
>     active: bool,
>     username: &str,
>     email: &str,
>     sign_in_count: u64,
> }
>
> fn main() {
>     let user1 = User {
>         active: true,
>         username: "someusername123",
>         email: "someone@example.com",
>         sign_in_count: 1,
>     };
> }
> ```
>
> </Listing>
>
> Trình biên dịch sẽ phàn nàn rằng nó cần bộ chỉ định lifetime:
>
> ```console
> $ cargo run
>    Compiling structs v0.1.0 (file:///projects/structs)
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:3:15
>   |
> 3 |     username: &str,
>   |               ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 ~     username: &'a str,
>   |
>
> error[E0106]: missing lifetime specifier
>  --> src/main.rs:4:12
>   |
> 4 |     email: &str,
>   |            ^ expected named lifetime parameter
>   |
> help: consider introducing a named lifetime parameter
>   |
> 1 ~ struct User<'a> {
> 2 |     active: bool,
> 3 |     username: &str,
> 4 ~     email: &'a str,
>   |
>
> For more information about this error, try `rustc --explain E0106`.
> error: could not compile `structs` (bin "structs") due to 2 previous errors
> ```
>
> Trong Chương 10, chúng ta sẽ thảo luận về cách sửa những lỗi này để bạn có thể
> lưu trữ tham chiếu trong struct, nhưng hiện tại, chúng ta sẽ sửa các lỗi như
> thế này bằng cách sử dụng các kiểu có quyền sở hữu như `String` thay vì các
> tham chiếu như `&str`.

<!-- manual-regeneration
for the error above
after running update-rustc.sh:
pbcopy < listings/ch05-using-structs-to-structure-related-data/no-listing-02-reference-in-struct/output.txt
paste above
add `> ` before every line -->

[tuples]: ch03-02-data-types.html#the-tuple-type
[move]: ch04-01-what-is-ownership.html#variables-and-data-interacting-with-move
[copy]: ch04-01-what-is-ownership.html#stack-only-data-copy
