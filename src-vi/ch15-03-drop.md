## Chạy Mã khi Dọn dẹp với Trait `Drop`

Trait quan trọng thứ hai cho mẫu con trỏ thông minh là `Drop`, cho phép bạn tùy
chỉnh những gì xảy ra khi một giá trị sắp ra khỏi phạm vi. Bạn có thể cung cấp
một triển khai cho trait `Drop` trên bất kỳ kiểu nào, và mã đó có thể được sử
dụng để giải phóng tài nguyên như tệp hoặc kết nối mạng.

Chúng ta giới thiệu `Drop` trong ngữ cảnh của con trỏ thông minh vì chức năng
của trait `Drop` gần như luôn được sử dụng khi triển khai một con trỏ thông
minh. Ví dụ, khi một `Box<T>` bị hủy, nó sẽ giải phóng không gian trên heap mà
box trỏ tới.

Trong một số ngôn ngữ, đối với một số kiểu, lập trình viên phải gọi mã để giải
phóng bộ nhớ hoặc tài nguyên mỗi khi họ sử dụng xong một thể hiện của những kiểu
đó. Ví dụ bao gồm file handles, sockets và locks. Nếu họ quên, hệ thống có thể
bị quá tải và gặp sự cố. Trong Rust, bạn có thể chỉ định một đoạn mã cụ thể được
chạy bất cứ khi nào một giá trị ra khỏi phạm vi, và trình biên dịch sẽ tự động
chèn mã này. Kết quả là, bạn không cần phải cẩn thận đặt mã dọn dẹp ở mọi nơi
trong chương trình mà một thể hiện của một kiểu cụ thể đã hoàn thành—bạn vẫn sẽ
không rò rỉ tài nguyên!

Bạn chỉ định mã để chạy khi một giá trị ra khỏi phạm vi bằng cách triển khai
trait `Drop`. Trait `Drop` yêu cầu bạn triển khai một phương thức có tên `drop`
nhận một tham chiếu có thể thay đổi tới `self`. Để xem khi nào Rust gọi `drop`,
hãy triển khai `drop` với các câu lệnh `println!` tạm thời.

Listing 15-14 hiển thị một struct `CustomSmartPointer` mà chức năng tùy chỉnh
duy nhất của nó là sẽ in `Dropping CustomSmartPointer!` khi thể hiện ra khỏi
phạm vi, để hiển thị khi Rust chạy phương thức `drop`.

<Listing number="15-14" file-name="src/main.rs" caption="Một struct `CustomSmartPointer` triển khai trait `Drop` nơi chúng ta sẽ đặt mã dọn dẹp của mình">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-14/src/main.rs}}
```

</Listing>

Trait `Drop` được bao gồm trong prelude, vì vậy chúng ta không cần phải đưa nó
vào phạm vi. Chúng ta triển khai trait `Drop` trên `CustomSmartPointer` và cung
cấp một triển khai cho phương thức `drop` gọi `println!`. Phần thân của phương
thức `drop` là nơi bạn sẽ đặt bất kỳ logic nào mà bạn muốn chạy khi một thể hiện
của kiểu của bạn ra khỏi phạm vi. Chúng ta đang in một số văn bản ở đây để minh
họa trực quan khi Rust sẽ gọi `drop`.

Trong `main`, chúng ta tạo hai thể hiện của `CustomSmartPointer` và sau đó in
`CustomSmartPointers created`. Ở cuối `main`, các thể hiện của
`CustomSmartPointer` của chúng ta sẽ ra khỏi phạm vi, và Rust sẽ gọi mã mà chúng
ta đặt trong phương thức `drop`, in thông báo cuối cùng của chúng ta. Lưu ý rằng
chúng ta không cần phải gọi phương thức `drop` một cách rõ ràng.

Khi chúng ta chạy chương trình này, chúng ta sẽ thấy đầu ra sau:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-14/output.txt}}
```

Rust tự động gọi `drop` cho chúng ta khi các thể hiện của chúng ta ra khỏi phạm
vi, gọi mã mà chúng ta đã chỉ định. Các biến được hủy theo thứ tự ngược lại so
với việc tạo, vì vậy `d` đã bị hủy trước `c`. Mục đích của ví dụ này là cung cấp
cho bạn một hướng dẫn trực quan về cách phương thức `drop` hoạt động; thông
thường, bạn sẽ chỉ định mã dọn dẹp mà kiểu của bạn cần chạy thay vì một thông
báo in.

<!-- Old link, do not remove -->

<a id="dropping-a-value-early-with-std-mem-drop"></a>

Không may, việc vô hiệu hóa chức năng `drop` tự động không đơn giản. Vô hiệu hóa
`drop` thường không cần thiết; toàn bộ ý nghĩa của trait `Drop` là nó được xử lý
tự động. Tuy nhiên, đôi khi bạn có thể muốn dọn dẹp một giá trị sớm. Một ví dụ
là khi sử dụng con trỏ thông minh quản lý khóa: bạn có thể muốn buộc phương thức
`drop` giải phóng khóa để mã khác trong cùng phạm vi có thể lấy khóa. Rust không
cho phép bạn gọi phương thức `drop` của trait `Drop` theo cách thủ công; thay
vào đó, bạn phải gọi hàm `std::mem::drop` được cung cấp bởi thư viện chuẩn nếu
bạn muốn buộc một giá trị bị hủy trước khi kết thúc phạm vi của nó.

Nếu chúng ta cố gắng gọi phương thức `drop` của trait `Drop` theo cách thủ công
bằng cách sửa đổi hàm `main` từ Listing 15-14, như trong Listing 15-15, chúng ta
sẽ gặp lỗi trình biên dịch.

<Listing number="15-15" file-name="src/main.rs" caption="Cố gắng gọi phương thức `drop` từ trait `Drop` theo cách thủ công để dọn dẹp sớm">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-15/src/main.rs:here}}
```

</Listing>

Khi chúng ta cố gắng biên dịch mã này, chúng ta sẽ gặp lỗi này:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-15/output.txt}}
```

Thông báo lỗi này nêu rõ rằng chúng ta không được phép gọi `drop` một cách rõ
ràng. Thông báo lỗi sử dụng thuật ngữ _destructor_, đó là thuật ngữ lập trình
chung cho một hàm dọn dẹp một thể hiện. Một _destructor_ tương tự như một
_constructor_, tạo ra một thể hiện. Hàm `drop` trong Rust là một destructor cụ
thể.

Rust không cho phép chúng ta gọi `drop` một cách rõ ràng vì Rust vẫn sẽ tự động
gọi `drop` trên giá trị ở cuối `main`. Điều này sẽ gây ra lỗi _double free_ vì
Rust sẽ cố gắng dọn dẹp cùng một giá trị hai lần.

Chúng ta không thể vô hiệu hóa việc chèn `drop` tự động khi một giá trị ra khỏi
phạm vi, và chúng ta không thể gọi phương thức `drop` một cách rõ ràng. Vì vậy,
nếu chúng ta cần buộc một giá trị được dọn dẹp sớm, chúng ta sử dụng hàm
`std::mem::drop`.

Hàm `std::mem::drop` khác với phương thức `drop` trong trait `Drop`. Chúng ta
gọi nó bằng cách truyền một đối số là giá trị mà chúng ta muốn buộc hủy. Hàm này
nằm trong prelude, vì vậy chúng ta có thể sửa đổi `main` trong Listing 15-15 để
gọi hàm `drop`, như trong Listing 15-16.

<Listing number="15-16" file-name="src/main.rs" caption="Gọi `std::mem::drop` để rõ ràng hủy một giá trị trước khi nó ra khỏi phạm vi">

```rust
{{#rustdoc_include ../listings/ch15-smart-pointers/listing-15-16/src/main.rs:here}}
```

</Listing>

Chạy mã này sẽ in ra như sau:

```console
{{#include ../listings/ch15-smart-pointers/listing-15-16/output.txt}}
```

Văn bản `` Dropping CustomSmartPointer with data `some data`! `` được in giữa
văn bản `CustomSmartPointer created.` và
`CustomSmartPointer dropped before the end of main.`, cho thấy rằng mã phương
thức `drop` được gọi để hủy `c` tại thời điểm đó.

Bạn có thể sử dụng mã được chỉ định trong triển khai trait `Drop` theo nhiều
cách để làm cho việc dọn dẹp trở nên thuận tiện và an toàn: ví dụ, bạn có thể sử
dụng nó để tạo bộ cấp phát bộ nhớ của riêng bạn! Với trait `Drop` và hệ thống sở
hữu của Rust, bạn không phải nhớ dọn dẹp vì Rust làm điều đó tự động.

Bạn cũng không phải lo lắng về các vấn đề do vô tình dọn dẹp các giá trị vẫn
đang được sử dụng: hệ thống sở hữu đảm bảo các tham chiếu luôn hợp lệ cũng đảm
bảo rằng `drop` chỉ được gọi một lần khi giá trị không còn được sử dụng.

Bây giờ chúng ta đã xem xét `Box<T>` và một số đặc điểm của con trỏ thông minh,
hãy xem một vài con trỏ thông minh khác được định nghĩa trong thư viện chuẩn.
