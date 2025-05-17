## Sử Dụng Trao Đổi Tin Nhắchúng ta sẽ gửi các giá trị đơn giản giữa các thread bằng một kênh để minh họa tính
năng này. Khi bạn đã quen với kỹ thuật này, bạn có thể sử dụng các kênh cho bất
kỳ thread nào cần giao tiếp với nhau, chẳng hạn như một hệ thống chat hoặc một
hệ thống mà nhiều thread thực hiện các phần của một phép tính và gửi các phần đó
đến một thread tổng hợp kết quả.Truyền Dữ Liệu Giữa Các Thread

Một cách tiếp cận ngày càng phổ biến để đảm bảo đồng thời an toàn là _trao đổi
tin nhắn_ (message passing), trong đó các thread hoặc actor giao tiếp bằng cách
gửi tin nhắn chứa dữ liệu cho nhau. Đây là ý tưởng được tóm tắt trong một khẩu
hiệu từ
[tài liệu của ngôn ngữ Go](https://golang.org/doc/effective_go.html#concurrency):
"Đừng giao tiếp bằng cách chia sẻ bộ nhớ; thay vào đó, hãy chia sẻ bộ nhớ bằng
cách giao tiếp."

Để thực hiện tính đồng thời bằng cách gửi tin nhắn, thư viện chuẩn của Rust cung
cấp một cài đặt của các kênh (channels). Một _kênh_ là một khái niệm lập trình
chung mà qua đó dữ liệu được gửi từ một thread sang thread khác.

Bạn có thể tưởng tượng một kênh trong lập trình giống như một kênh nước một
chiều, chẳng hạn như một dòng suối hoặc một con sông. Nếu bạn đặt một vật gì đó
như một con vịt cao su vào sông, nó sẽ trôi xuôi dòng đến cuối đường thủy.

Một kênh có hai nửa: một bộ phát (transmitter) và một bộ thu (receiver). Nửa bộ
phát là vị trí thượng nguồn nơi bạn đặt con vịt cao su vào sông, và nửa bộ thu
là nơi con vịt cao su kết thúc ở hạ nguồn. Một phần của mã của bạn gọi các
phương thức trên bộ phát với dữ liệu bạn muốn gửi, và một phần khác kiểm tra đầu
nhận để tìm các tin nhắn đến. Một kênh được gọi là _đóng_ (closed) nếu một trong
hai nửa bộ phát hoặc bộ thu bị loại bỏ.

Ở đây, chúng ta sẽ xây dựng một chương trình có một thread để tạo giá trị và gửi
chúng qua một kênh, và một thread khác sẽ nhận các giá trị và in chúng ra. Chúng
ta sẽ gửi các giá trị đơn giản giữa các thread bằng một kênh để minh họa tính
năng này. Khi bạn đã quen với kỹ thuật này, bạn có thể sử dụng các kênh cho bất
kỳ thread nào cần giao tiếp với nhau, chẳng hạn như một hệ thống chat hoặc một
hệ thống mà nhiều thread thực hiện các phần của một phép tính và gửi các phần đó
đến một thread tổng hợp kết quả.

Đầu tiên, trong Listing 16-6, chúng ta sẽ tạo một kênh nhưng chưa làm gì với nó.
Lưu ý rằng đoạn mã này chưa thể biên dịch được vì Rust không thể xác định loại
giá trị mà chúng ta muốn gửi qua kênh.

<Listing number="16-6" file-name="src/main.rs" caption="Tạo một kênh và gán hai nửa cho `tx` và `rx`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-06/src/main.rs}}
```

</Listing>

Chúng ta tạo một kênh mới bằng hàm `mpsc::channel`; `mpsc` là viết tắt của
_multiple producer, single consumer_ (nhiều người sản xuất, một người tiêu
dùng). Nói ngắn gọn, cách thư viện chuẩn của Rust triển khai các kênh có nghĩa
là một kênh có thể có nhiều đầu _gửi_ tạo ra giá trị nhưng chỉ một đầu _nhận_
tiêu thụ những giá trị đó. Hãy tưởng tượng nhiều dòng suối chảy vào một con sông
lớn: mọi thứ được gửi xuống bất kỳ dòng suối nào đều sẽ kết thúc trong một con
sông ở cuối. Hiện tại chúng ta sẽ bắt đầu với một nhà sản xuất duy nhất, nhưng
chúng ta sẽ thêm nhiều nhà sản xuất khi làm cho ví dụ này hoạt động.

Hàm `mpsc::channel` trả về một tuple, phần tử đầu tiên là đầu gửi - bộ phát - và
phần tử thứ hai là đầu nhận - bộ thu. Các chữ viết tắt `tx` và `rx` theo truyền
thống được sử dụng trong nhiều lĩnh vực cho _transmitter_ (bộ phát) và
_receiver_ (bộ thu) tương ứng, vì vậy chúng ta đặt tên cho các biến của mình như vậy để
chỉ ra mỗi đầu. Chúng ta đang sử dụng câu lệnh `let` với một mẫu phân rã tuple;
chúng ta sẽ thảo luận về việc sử dụng các mẫu trong câu lệnh `let` và phân rã
trong Chương 19. Hiện tại, hãy biết rằng sử dụng câu lệnh `let` như thế này là
một phương pháp thuận tiện để trích xuất các phần của tuple được trả về bởi
`mpsc::channel`.

Hãy chuyển đầu phát vào một thread được tạo ra và có nó gửi một chuỗi để thread
được tạo ra giao tiếp với thread chính, như được hiển thị trong Listing
16-7. Điều này giống như đặt một con vịt cao su vào sông ở thượng nguồn hoặc gửi
một tin nhắn trò chuyện từ một thread sang thread khác.

<Listing number="16-7" file-name="src/main.rs" caption='Di chuyển `tx` đến một thread được tạo ra và gửi `"hi"`'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-07/src/main.rs}}
```

</Listing>

Một lần nữa, chúng ta đang sử dụng `thread::spawn` để tạo một thread mới và sau
đó sử dụng `move` để di chuyển `tx` vào closure để thread được tạo ra sở hữu
`tx`. Thread được tạo ra cần sở hữu bộ phát để có thể gửi tin nhắn thông qua
kênh.

Bộ phát có một phương thức `send` nhận giá trị mà chúng ta muốn gửi. Phương thức
`send` trả về một kiểu `Result<T, E>`, vì vậy nếu bộ thu đã bị loại bỏ và không
có nơi nào để gửi giá trị, thao tác gửi sẽ trả về một lỗi. Trong ví dụ này,
chúng ta đang gọi `unwrap` để panic trong trường hợp có lỗi. Nhưng trong một ứng
dụng thực tế, chúng ta sẽ xử lý nó đúng cách: quay lại Chương 9 để xem lại các
chiến lược xử lý lỗi đúng cách.

Trong Listing 16-8, chúng ta sẽ nhận giá trị từ bộ thu trong thread chính. Điều
này giống như lấy con vịt cao su từ nước ở cuối sông hoặc nhận một tin nhắn trò
chuyện.

<Listing number="16-8" file-name="src/main.rs" caption='Nhận giá trị `"hi"` trong thread chính và in nó ra'>

```rust
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-08/src/main.rs}}
```

</Listing>

Bộ thu có hai phương thức hữu ích: `recv` và `try_recv`. Chúng ta đang sử dụng
`recv`, viết tắt của _receive_, sẽ chặn thực thi của thread chính và đợi cho đến
khi một giá trị được gửi xuống kênh. Khi một giá trị được gửi, `recv` sẽ trả về
nó trong một `Result<T, E>`. Khi bộ phát đóng, `recv` sẽ trả về một lỗi để báo
hiệu rằng không có thêm giá trị nào sẽ đến.

Phương thức `try_recv` không chặn, nhưng thay vào đó sẽ trả về một
`Result<T, E>` ngay lập tức: một giá trị `Ok` chứa một tin nhắn nếu có sẵn và
một giá trị `Err` nếu không có tin nhắn nào vào lúc này. Sử dụng `try_recv` rất
hữu ích nếu thread này có công việc khác cần làm trong khi chờ tin nhắn: chúng
ta có thể viết một vòng lặp gọi `try_recv` thường xuyên, xử lý một tin nhắn nếu
có, và nếu không thì làm việc khác trong một thời gian ngắn cho đến khi kiểm tra
lại.

Chúng ta đã sử dụng `recv` trong ví dụ này để đơn giản hóa; chúng ta không có
công việc nào khác cho thread chính để làm ngoài việc đợi tin nhắn, vì vậy việc
chặn thread chính là phù hợp.

Khi chúng ta chạy mã trong Listing 16-8, chúng ta sẽ thấy giá trị được in ra từ
thread chính:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
```

Hoàn hảo!

### Kênh và Chuyển Giao Quyền Sở Hữu

Các quy tắc sở hữu đóng vai trò quan trọng trong việc gửi tin nhắn vì chúng giúp
bạn viết mã đồng thời an toàn. Ngăn chặn lỗi trong lập trình đồng thời là lợi
thế của việc suy nghĩ về quyền sở hữu trong toàn bộ chương trình Rust của bạn.
Hãy thực hiện một thí nghiệm để hiển thị cách kênh và quyền sở hữu hoạt động
cùng nhau để ngăn chặn vấn đề: chúng ta sẽ cố gắng sử dụng một giá trị `val`
trong thread được tạo ra _sau khi_ chúng ta đã gửi nó xuống kênh. Hãy thử biên
dịch mã trong Listing 16-9 để xem tại sao mã này không được phép.

<Listing number="16-9" file-name="src/main.rs" caption="Cố gắng sử dụng `val` sau khi chúng ta đã gửi nó xuống kênh">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-09/src/main.rs}}
```

</Listing>

Ở đây, chúng ta cố gắng in `val` sau khi đã gửi nó xuống kênh thông qua
`tx.send`. Cho phép điều này sẽ là một ý tưởng tồi: một khi giá trị đã được gửi
đến một thread khác, thread đó có thể sửa đổi hoặc loại bỏ nó trước khi chúng ta
cố gắng sử dụng lại giá trị. Có thể, các sửa đổi của thread khác có thể gây ra
lỗi hoặc kết quả không mong muốn do dữ liệu không nhất quán hoặc không tồn tại.
Tuy nhiên, Rust đưa ra một lỗi nếu chúng ta cố gắng biên dịch mã trong Listing
16-9:

```console
{{#include ../listings/ch16-fearless-concurrency/listing-16-09/output.txt}}
```

Lỗi đồng thời của chúng ta đã gây ra một lỗi thời điểm biên dịch. Hàm `send` lấy quyền sở
hữu tham số của nó, và khi giá trị được chuyển đi bộ thu nhận quyền sở hữu của
nó. Điều này ngăn chúng ta vô tình sử dụng lại giá trị sau khi gửi nó; hệ thống
sở hữu kiểm tra rằng mọi thứ đều ổn.

### Gửi Nhiều Giá Trị và Theo Dõi Bộ Thu Đang Chờ Đợi

Mã trong Listing 16-8 đã được biên dịch và chạy, nhưng nó không hiển thị rõ ràng
cho chúng ta thấy rằng hai thread riêng biệt đang nói chuyện với nhau qua kênh.

Trong Listing 16-10, chúng ta đã thực hiện một số sửa đổi sẽ chứng minh rằng mã
trong Listing 16-8 đang chạy đồng thời: thread được tạo ra sẽ gửi nhiều tin nhắn
và tạm dừng một giây giữa mỗi tin nhắn.

<Listing number="16-10" file-name="src/main.rs" caption="Gửi nhiều tin nhắn và tạm dừng giữa mỗi tin">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-10/src/main.rs}}
```

</Listing>

Lần này, thread được tạo ra có một vector chuỗi mà chúng ta muốn gửi đến thread
chính. Chúng ta lặp qua chúng, gửi từng cái riêng lẻ, và tạm dừng giữa mỗi lần
bằng cách gọi hàm `thread::sleep` với một giá trị `Duration` một giây.

Trong thread chính, chúng ta không còn gọi hàm `recv` một cách rõ ràng nữa: thay
vào đó, chúng ta đang xử lý `rx` như một iterator. Đối với mỗi giá trị nhận
được, chúng ta đang in nó ra. Khi kênh đóng, việc lặp sẽ kết thúc.

Khi chạy mã trong Listing 16-10, bạn sẽ thấy đầu ra sau với khoảng dừng một
giây giữa mỗi dòng:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: from
Got: the
Got: thread
```

Bởi vì chúng ta không có bất kỳ mã nào tạm dừng hoặc trì hoãn trong vòng lặp
`for` trong thread chính, chúng ta có thể biết rằng thread chính đang đợi để
nhận giá trị từ thread được tạo ra.

### Tạo Nhiều Nhà Sản Xuất bằng cách Sao Chép Bộ Phát

Trước đó chúng ta đã đề cập rằng `mpsc` là từ viết tắt của _multiple producer,
single consumer_ (nhiều nhà sản xuất, một người tiêu dùng). Hãy sử dụng `mpsc`
và mở rộng mã trong Listing 16-10 để tạo nhiều thread đều gửi giá trị đến cùng
một bộ thu. Chúng ta có thể làm như vậy bằng cách sao chép bộ phát, như được
hiển thị trong Listing 16-11.

<Listing number="16-11" file-name="src/main.rs" caption="Gửi nhiều tin nhắn từ nhiều nhà sản xuất">

```rust,noplayground
{{#rustdoc_include ../listings/ch16-fearless-concurrency/listing-16-11/src/main.rs:here}}
```

</Listing>

Lần này, trước khi chúng ta tạo thread đầu tiên, chúng ta gọi `clone` trên bộ
phát. Điều này sẽ cung cấp cho chúng ta một bộ phát mới mà chúng ta có thể
truyền cho thread được tạo ra đầu tiên. Chúng ta truyền bộ phát gốc cho thread
thứ hai được tạo ra. Điều này cung cấp cho chúng ta hai thread, mỗi thread gửi
các tin nhắn khác nhau đến một bộ thu.

Khi bạn chạy mã, đầu ra của bạn sẽ trông giống như thế này:

<!-- Not extracting output because changes to this output aren't significant;
the changes are likely to be due to the threads running differently rather than
changes in the compiler -->

```text
Got: hi
Got: more
Got: from
Got: messages
Got: for
Got: the
Got: thread
Got: you
```

Bạn có thể thấy các giá trị theo thứ tự khác, tùy thuộc vào hệ thống của bạn.
Đây là điều làm cho tính đồng thời vừa thú vị vừa khó khăn. Nếu bạn thử nghiệm
với `thread::sleep`, cung cấp cho nó các giá trị khác nhau trong các thread khác
nhau, mỗi lần chạy sẽ không xác định hơn và tạo ra đầu ra khác nhau mỗi lần.

Bây giờ chúng ta đã xem xét cách kênh hoạt động, hãy xem xét một phương pháp
đồng thời khác.
