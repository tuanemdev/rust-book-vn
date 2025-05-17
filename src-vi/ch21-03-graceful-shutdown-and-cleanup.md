## Tắt Máy Nhẹ Nhàng và Dọn Dẹp

Đoạn mã trong Listing 21-20 đang phản hồi các yêu cầu bất đồng bộ thông qua việc
sử dụng thread pool, như chúng ta dự định. Chúng ta nhận được một số cảnh báo về
các trường `workers`, `id`, và `thread` mà chúng ta không sử dụng trực tiếp,
điều này nhắc nhở chúng ta rằng chúng ta không dọn dẹp bất cứ thứ gì. Khi chúng
ta sử dụng phương pháp kém thanh lịch <kbd>ctrl</kbd>-<kbd>C</kbd> để dừng luồng
chính, tất cả các luồng khác cũng bị dừng ngay lập tức, ngay cả khi chúng đang
trong quá trình phục vụ một yêu cầu.

Tiếp theo, chúng ta sẽ thực hiện đặc tính `Drop` để gọi `join` trên mỗi luồng
trong pool để chúng có thể hoàn thành các yêu cầu mà chúng đang xử lý trước khi
đóng. Sau đó, chúng ta sẽ thực hiện một cách để cho các luồng biết rằng chúng
nên dừng chấp nhận các yêu cầu mới và tắt. Để xem mã này hoạt động, chúng ta sẽ
sửa đổi máy chủ của mình để chỉ chấp nhận hai yêu cầu trước khi tắt thread pool
một cách nhẹ nhàng.

Một điều cần lưu ý khi chúng ta tiến hành: không có gì trong phần này ảnh hưởng
đến các phần của mã xử lý việc thực thi các closure, vì vậy mọi thứ ở đây sẽ
giống hệt nhau nếu chúng ta đang sử dụng thread pool cho một môi trường thực thi
bất đồng bộ.

### Thực Hiện Đặc Tính `Drop` trên `ThreadPool`

Hãy bắt đầu với việc thực hiện `Drop` trên thread pool của chúng ta. Khi pool bị
huỷ, các luồng của chúng ta đều nên tham gia (join) để đảm bảo chúng hoàn thành
công việc của mình. Listing 21-22 hiển thị nỗ lực đầu tiên để thực hiện `Drop`;
mã này chưa hoạt động hoàn toàn.

<Listing number="21-22" file-name="src/lib.rs" caption="Kết hợp mỗi luồng khi thread pool ra khỏi phạm vi">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-22/src/lib.rs:here}}
```

</Listing>

Đầu tiên chúng ta lặp qua từng `worker` trong thread pool. Chúng ta sử dụng
`&mut` cho việc này vì `self` là một tham chiếu có thể thay đổi, và chúng ta
cũng cần có khả năng thay đổi `worker`. Đối với mỗi `worker`, chúng ta in một
thông báo cho biết thể hiện `Worker` cụ thể này đang tắt, sau đó chúng ta gọi
`join` trên luồng của thể hiện `Worker` đó. Nếu lệnh gọi `join` thất bại, chúng
ta sử dụng `unwrap` để làm cho Rust hoảng loạn (panic) và chuyển sang tắt không
nhẹ nhàng.

Đây là lỗi chúng ta nhận được khi biên dịch mã này:

```console
{{#include ../listings/ch21-web-server/listing-21-22/output.txt}}
```

Lỗi cho chúng ta biết rằng chúng ta không thể gọi `join` vì chúng ta chỉ có một
tham chiếu có thể thay đổi của mỗi `worker` và `join` lấy quyền sở hữu của đối
số của nó. Để giải quyết vấn đề này, chúng ta cần di chuyển luồng ra khỏi thể
hiện `Worker` sở hữu `thread` để `join` có thể tiêu thụ luồng. Một cách để làm
điều này là bằng cách sử dụng cùng một phương pháp mà chúng ta đã làm trong
Listing 18-15. Nếu `Worker` giữ một `Option<thread::JoinHandle<()>>`, chúng ta
có thể gọi phương thức `take` trên `Option` để di chuyển giá trị ra khỏi biến
thể `Some` và để lại một biến thể `None` ở vị trí của nó. Nói cách khác, một
`Worker` đang chạy sẽ có một biến thể `Some` trong `thread`, và khi chúng ta
muốn dọn dẹp một `Worker`, chúng ta sẽ thay thế `Some` bằng `None` để `Worker`
không có luồng để chạy.

Tuy nhiên, _chỉ_ thời điểm này sẽ xuất hiện là khi huỷ `Worker`. Để đổi lại,
chúng ta sẽ phải xử lý một `Option<thread::JoinHandle<()>>` bất cứ nơi nào chúng
ta truy cập `worker.thread`. Rust thành ngữ sử dụng `Option` khá nhiều, nhưng
khi bạn thấy mình bọc thứ gì đó mà bạn biết sẽ luôn có mặt trong `Option` như
một giải pháp tạm thời như thế này, đó là một ý tưởng tốt để tìm kiếm các phương
pháp thay thế để làm cho mã của bạn sạch hơn và ít dễ xảy ra lỗi hơn.

Trong trường hợp này, một giải pháp thay thế tốt hơn tồn tại: phương thức
`Vec::drain`. Nó chấp nhận một tham số phạm vi để chỉ định các mục cần xóa khỏi
vector và trả về một iterator của các mục đó. Truyền cú pháp phạm vi `..` sẽ xóa
_mọi_ giá trị khỏi vector.

Vì vậy, chúng ta cần cập nhật việc thực hiện `drop` của `ThreadPool` như thế
này:

<Listing file-name="src/lib.rs">

```rust
{{#rustdoc_include ../listings/ch21-web-server/no-listing-04-update-drop-definition/src/lib.rs:here}}
```

</Listing>

Điều này giải quyết lỗi biên dịch và không yêu cầu bất kỳ thay đổi nào khác cho
mã của chúng ta. Lưu ý rằng, vì drop có thể được gọi khi hoảng loạn, unwrap cũng
có thể hoảng loạn và gây ra hoảng loạn kép, điều này ngay lập tức làm sập chương
trình và kết thúc bất kỳ quá trình dọn dẹp nào đang diễn ra. Điều này ổn đối với
một chương trình ví dụ, nhưng không được khuyến nghị cho mã sản xuất.

### Báo Hiệu cho Các Luồng Dừng Lắng Nghe Công Việc

Với tất cả các thay đổi chúng ta đã thực hiện, mã của chúng ta biên dịch mà
không có bất kỳ cảnh báo nào. Tuy nhiên, tin xấu là mã này chưa hoạt động theo
cách chúng ta muốn. Chìa khóa là logic trong các closure được chạy bởi các luồng
của các thể hiện `Worker`: tại thời điểm này, chúng ta gọi `join`, nhưng điều đó
sẽ không tắt các luồng, vì chúng `loop` mãi mãi tìm kiếm công việc. Nếu chúng ta
thử huỷ `ThreadPool` của mình với cách thực hiện `drop` hiện tại, luồng chính sẽ
bị chặn mãi mãi, chờ đợi luồng đầu tiên hoàn thành.

Để khắc phục vấn đề này, chúng ta sẽ cần thay đổi trong cách thực hiện `drop`
của `ThreadPool` và sau đó thay đổi trong vòng lặp `Worker`.

Đầu tiên, chúng ta sẽ thay đổi cách thực hiện `drop` của `ThreadPool` để rõ ràng
huỷ `sender` trước khi đợi các luồng hoàn thành. Listing 21-23 hiển thị các thay
đổi đối với `ThreadPool` để rõ ràng huỷ `sender`. Không giống như với luồng, ở
đây chúng ta _thực sự_ cần sử dụng `Option` để có thể di chuyển `sender` ra khỏi
`ThreadPool` với `Option::take`.

<Listing number="21-23" file-name="src/lib.rs" caption="Rõ ràng huỷ `sender` trước khi tham gia các luồng `Worker`">

```rust,noplayground,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-23/src/lib.rs:here}}
```

</Listing>

Việc huỷ `sender` đóng kênh, điều này chỉ ra rằng không có thêm thông báo nào sẽ
được gửi. Khi điều đó xảy ra, tất cả các lệnh gọi `recv` mà các thể hiện
`Worker` thực hiện trong vòng lặp vô hạn sẽ trả về một lỗi. Trong Listing 21-24,
chúng ta thay đổi vòng lặp `Worker` để nhẹ nhàng thoát khỏi vòng lặp trong
trường hợp đó, điều đó có nghĩa là các luồng sẽ kết thúc khi thực hiện `drop`
của `ThreadPool` gọi `join` trên chúng.

<Listing number="21-24" file-name="src/lib.rs" caption="Rõ ràng thoát khỏi vòng lặp khi `recv` trả về một lỗi">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-24/src/lib.rs:here}}
```

</Listing>

Để xem mã này hoạt động, hãy sửa đổi `main` để chỉ chấp nhận hai yêu cầu trước
khi tắt máy chủ một cách nhẹ nhàng, như được hiển thị trong Listing 21-25.

<Listing number="21-25" file-name="src/main.rs" caption="Tắt máy chủ sau khi phục vụ hai yêu cầu bằng cách thoát khỏi vòng lặp">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/listing-21-25/src/main.rs:here}}
```

</Listing>

Bạn sẽ không muốn một máy chủ web trong thế giới thực tắt sau khi phục vụ chỉ
hai yêu cầu. Mã này chỉ chứng minh rằng việc tắt nhẹ nhàng và dọn dẹp đang hoạt
động tốt.

Phương thức `take` được định nghĩa trong đặc tính `Iterator` và giới hạn vòng
lặp tối đa là hai mục đầu tiên. `ThreadPool` sẽ ra khỏi phạm vi khi kết thúc
`main`, và việc thực hiện `drop` sẽ chạy.

Khởi động máy chủ với `cargo run`, và thực hiện ba yêu cầu. Yêu cầu thứ ba sẽ bị
lỗi, và trong terminal của bạn, bạn sẽ thấy đầu ra tương tự như thế này:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-25
cargo run
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
curl http://127.0.0.1:7878
third request will error because server will have shut down
copy output below
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.41s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Shutting down.
Shutting down worker 0
Worker 3 got a job; executing.
Worker 1 disconnected; shutting down.
Worker 2 disconnected; shutting down.
Worker 3 disconnected; shutting down.
Worker 0 disconnected; shutting down.
Shutting down worker 1
Shutting down worker 2
Shutting down worker 3
```

Bạn có thể thấy thứ tự ID `Worker` và thông báo được in khác nhau. Chúng ta có
thể thấy cách mã này hoạt động từ các thông báo: các thể hiện `Worker` 0 và 3
nhận hai yêu cầu đầu tiên. Máy chủ dừng chấp nhận kết nối sau kết nối thứ hai,
và việc thực hiện `Drop` trên `ThreadPool` bắt đầu thực thi trước khi `Worker` 3
thậm chí bắt đầu công việc của nó. Việc huỷ `sender` ngắt kết nối tất cả các thể
hiện `Worker` và cho chúng biết phải tắt. Các thể hiện `Worker` mỗi thể hiện in
một thông báo khi chúng ngắt kết nối, và sau đó thread pool gọi `join` để đợi
mỗi luồng `Worker` hoàn thành.

Hãy chú ý một khía cạnh thú vị của việc thực thi cụ thể này: `ThreadPool` đã huỷ
`sender`, và trước khi bất kỳ `Worker` nào nhận được lỗi, chúng ta đã cố gắng
tham gia `Worker` 0. `Worker` 0 chưa nhận được lỗi từ `recv`, vì vậy luồng chính
bị chặn, chờ đợi `Worker` 0 hoàn thành. Trong khi đó, `Worker` 3 nhận được một
công việc và sau đó tất cả các luồng nhận được lỗi. Khi `Worker` 0 hoàn thành,
luồng chính đợi phần còn lại của các thể hiện `Worker` hoàn thành. Tại thời điểm
đó, tất cả chúng đều đã thoát khỏi vòng lặp của chúng và dừng lại.

Xin chúc mừng! Bây giờ chúng ta đã hoàn thành dự án của mình; chúng ta có một
máy chủ web cơ bản sử dụng thread pool để phản hồi bất đồng bộ. Chúng ta có thể
thực hiện việc tắt máy chủ một cách nhẹ nhàng, điều này dọn dẹp tất cả các luồng
trong pool.

Đây là toàn bộ mã để tham khảo:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/main.rs}}
```

</Listing>

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-07-final-code/src/lib.rs}}
```

</Listing>

Chúng ta có thể làm nhiều hơn nữa ở đây! Nếu bạn muốn tiếp tục nâng cao dự án
này, đây là một số ý tưởng:

- Thêm nhiều tài liệu vào `ThreadPool` và các phương thức công khai của nó.
- Thêm các bài kiểm tra chức năng của thư viện.
- Thay đổi các lệnh gọi `unwrap` thành xử lý lỗi mạnh mẽ hơn.
- Sử dụng `ThreadPool` để thực hiện một số tác vụ khác ngoài việc phục vụ các
  yêu cầu web.
- Tìm một crate thread pool trên [crates.io](https://crates.io/) và thực hiện
  một máy chủ web tương tự bằng cách sử dụng crate thay thế. Sau đó so sánh API
  và độ mạnh mẽ của nó với thread pool mà chúng ta đã thực hiện.

## Tổng Kết

Làm tốt lắm! Bạn đã đến được cuối sách! Chúng tôi muốn cảm ơn bạn đã tham gia
cùng chúng tôi trong chuyến tham quan Rust này. Bây giờ bạn đã sẵn sàng để thực
hiện các dự án Rust của riêng mình và giúp đỡ với các dự án của người khác. Hãy
nhớ rằng có một cộng đồng chào đón của các Rustacean khác sẵn sàng giúp đỡ bạn
với bất kỳ thách thức nào bạn gặp phải trên hành trình Rust của mình.
