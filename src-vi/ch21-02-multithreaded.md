## Chuyển Đổi Máy Chủ Đơn Luồng Thành Máy Chủ Đa Luồng

Hiện tại, máy chủ sẽ xử lý từng yêu cầu lần lượt, nghĩa là nó sẽ không xử lý kết
nối thứ hai cho đến khi kết nối đầu tiên hoàn tất. Nếu máy chủ nhận được ngày
càng nhiều yêu cầu, việc thực thi tuần tự này sẽ ngày càng kém hiệu quả. Nếu máy
chủ nhận được một yêu cầu mất nhiều thời gian để xử lý, các yêu cầu tiếp theo sẽ
phải đợi cho đến khi yêu cầu dài hoàn thành, ngay cả khi các yêu cầu mới có thể
được xử lý nhanh chóng. Chúng ta cần khắc phục điều này, nhưng trước tiên chúng
ta sẽ xem xét vấn đề trong thực tế.

### Mô Phỏng Yêu Cầu Chậm

Chúng ta sẽ xem xét cách một yêu cầu xử lý chậm có thể ảnh hưởng đến các yêu cầu
khác được gửi đến cài đặt máy chủ hiện tại của chúng ta. Listing 21-10 triển
khai việc xử lý yêu cầu đến _/sleep_ với một phản hồi chậm mô phỏng sẽ khiến máy
chủ ngủ trong năm giây trước khi phản hồi.

<Listing number="21-10" file-name="src/main.rs" caption="Mô phỏng yêu cầu chậm bằng cách ngủ trong 5 giây">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-10/src/main.rs:here}}
```

</Listing>

Chúng ta đã chuyển từ `if` sang `match` khi đã có ba trường hợp. Chúng ta cần
phải so khớp rõ ràng trên một lát cắt của `request_line` để khớp mẫu với các giá
trị chuỗi; `match` không tự động tham chiếu và hủy tham chiếu, như phương thức
so sánh bằng.

Cánh tay đầu tiên giống như khối `if` từ Listing 21-9. Cánh tay thứ hai khớp với
một yêu cầu đến _/sleep_. Khi yêu cầu đó được nhận, máy chủ sẽ ngủ trong năm
giây trước khi hiển thị trang HTML thành công. Cánh tay thứ ba giống như khối
`else` từ Listing 21-9.

Bạn có thể thấy máy chủ của chúng ta nguyên thủy như thế nào: các thư viện thực
tế sẽ xử lý việc nhận dạng nhiều yêu cầu theo cách ít dài dòng hơn nhiều!

Khởi động máy chủ bằng cách sử dụng `cargo run`. Sau đó mở hai cửa sổ trình
duyệt: một cho _http://127.0.0.1:7878_ và cái kia cho
_http://127.0.0.1:7878/sleep_. Nếu bạn nhập URI _/_ vài lần, như trước đây, bạn
sẽ thấy nó phản hồi nhanh chóng. Nhưng nếu bạn nhập _/sleep_ và sau đó tải _/_,
bạn sẽ thấy _/_ đợi cho đến khi `sleep` đã ngủ đủ năm giây trước khi tải.

Có nhiều kỹ thuật chúng ta có thể sử dụng để tránh các yêu cầu ùn tắc phía sau
một yêu cầu chậm, bao gồm việc sử dụng async như chúng ta đã làm trong Chương
17; kỹ thuật chúng ta sẽ triển khai là một thread pool.

### Cải Thiện Thông Lượng với Thread Pool

Một _thread pool_ là một nhóm các luồng được tạo ra đang chờ đợi và sẵn sàng xử
lý một tác vụ. Khi chương trình nhận được một tác vụ mới, nó gán một trong các
luồng trong pool cho tác vụ đó, và luồng đó sẽ xử lý tác vụ. Các luồng còn lại
trong pool sẵn sàng xử lý bất kỳ tác vụ nào khác đến trong khi luồng đầu tiên
đang xử lý. Khi luồng đầu tiên hoàn thành việc xử lý tác vụ của nó, nó được trả
lại pool của các luồng rảnh, sẵn sàng xử lý một tác vụ mới. Thread pool cho phép
bạn xử lý các kết nối đồng thời, tăng thông lượng của máy chủ của bạn.

Chúng ta sẽ giới hạn số lượng luồng trong pool ở một số nhỏ để bảo vệ chúng ta
khỏi các cuộc tấn công DoS; nếu chúng ta để chương trình của mình tạo một luồng
mới cho mỗi yêu cầu khi nó đến, ai đó gửi 10 triệu yêu cầu đến máy chủ của chúng
ta có thể gây rối bằng cách sử dụng hết tài nguyên của máy chủ và làm cho việc
xử lý yêu cầu dừng lại.

Thay vì tạo ra không giới hạn luồng, chúng ta sẽ có một số lượng cố định luồng
đang chờ trong pool. Các yêu cầu đến được gửi đến pool để xử lý. Pool sẽ duy trì
một hàng đợi các yêu cầu đến. Mỗi luồng trong pool sẽ lấy một yêu cầu từ hàng
đợi này, xử lý yêu cầu, và sau đó hỏi hàng đợi về yêu cầu khác. Với thiết kế
này, chúng ta có thể xử lý đồng thời tối đa _`N`_ yêu cầu, trong đó _`N`_ là số
lượng luồng. Nếu mỗi luồng đang đáp ứng một yêu cầu chạy lâu, các yêu cầu tiếp
theo vẫn có thể ùn tắc trong hàng đợi, nhưng chúng ta đã tăng số lượng yêu cầu
chạy lâu mà chúng ta có thể xử lý trước khi đạt đến điểm đó.

Kỹ thuật này chỉ là một trong nhiều cách để cải thiện thông lượng của máy chủ
web. Các tùy chọn khác mà bạn có thể khám phá là mô hình fork/join, mô hình I/O
không đồng bộ đơn luồng, và mô hình I/O không đồng bộ đa luồng. Nếu bạn quan tâm
đến chủ đề này, bạn có thể đọc thêm về các giải pháp khác và thử triển khai
chúng; với một ngôn ngữ cấp thấp như Rust, tất cả các tùy chọn này đều có thể.

Trước khi bắt đầu triển khai thread pool, hãy nói về việc sử dụng pool sẽ trông
như thế nào. Khi bạn đang cố gắng thiết kế mã, việc viết giao diện khách hàng
trước có thể giúp hướng dẫn thiết kế của bạn. Viết API của mã sao cho nó được
cấu trúc theo cách bạn muốn gọi nó; sau đó triển khai chức năng trong cấu trúc
đó thay vì triển khai chức năng và sau đó thiết kế API công khai.

Tương tự như cách chúng ta đã sử dụng phát triển dựa trên kiểm thử trong dự án
trong Chương 12, chúng ta sẽ sử dụng phát triển dựa trên trình biên dịch ở đây.
Chúng ta sẽ viết mã gọi các hàm mà chúng ta muốn, và sau đó chúng ta sẽ xem xét
các lỗi từ trình biên dịch để xác định những gì chúng ta nên thay đổi tiếp theo
để làm cho mã hoạt động. Tuy nhiên, trước khi làm điều đó, chúng ta sẽ khám phá
kỹ thuật mà chúng ta sẽ không sử dụng làm điểm khởi đầu.

<!-- Old headings. Do not remove or links may break. -->

<a id="code-structure-if-we-could-spawn-a-thread-for-each-request"></a>

#### Tạo một Luồng cho Mỗi Yêu Cầu

Đầu tiên, hãy khám phá cách mã của chúng ta có thể trông như thế nào nếu nó thực
sự tạo ra một luồng mới cho mỗi kết nối. Như đã đề cập trước đó, đây không phải
là kế hoạch cuối cùng của chúng ta do các vấn đề với việc có thể tạo ra một số
lượng không giới hạn các luồng, nhưng nó là một điểm khởi đầu để có được một máy
chủ đa luồng hoạt động trước. Sau đó, chúng ta sẽ thêm thread pool như một cải
tiến, và việc so sánh hai giải pháp sẽ dễ dàng hơn.

Listing 21-11 hiển thị các thay đổi cần thực hiện đối với `main` để tạo một
luồng mới để xử lý mỗi luồng trong vòng lặp `for`.

<Listing number="21-11" file-name="src/main.rs" caption="Tạo một luồng mới cho mỗi luồng">

```rust,no_run
{{#rustdoc_include ../listings/ch21-web-server/listing-21-11/src/main.rs:here}}
```

</Listing>

Như bạn đã học trong Chương 16, `thread::spawn` sẽ tạo một luồng mới và sau đó
chạy mã trong closure trong luồng mới. Nếu bạn chạy mã này và tải _/sleep_ trong
trình duyệt của bạn, sau đó _/_ trong hai tab trình duyệt khác, bạn sẽ thấy rằng
các yêu cầu đến _/_ không phải đợi _/sleep_ hoàn thành. Tuy nhiên, như chúng ta
đã đề cập, điều này cuối cùng sẽ làm quá tải hệ thống vì bạn sẽ tạo ra các luồng
mới mà không có bất kỳ giới hạn nào.

Bạn cũng có thể nhớ lại từ Chương 17 rằng đây chính xác là loại tình huống mà
async và await thực sự tỏa sáng! Hãy nhớ điều đó khi chúng ta xây dựng thread
pool và nghĩ về việc mọi thứ sẽ trông khác hoặc giống nhau như thế nào với
async.

<!-- Old headings. Do not remove or links may break. -->

<a id="creating-a-similar-interface-for-a-finite-number-of-threads"></a>

#### Tạo một Số Lượng Giới Hạn Các Luồng

Chúng ta muốn thread pool của mình hoạt động theo cách tương tự, quen thuộc để
việc chuyển đổi từ luồng sang thread pool không yêu cầu thay đổi lớn đối với mã
sử dụng API của chúng ta. Listing 21-12 hiển thị giao diện giả thuyết cho một
cấu trúc `ThreadPool` mà chúng ta muốn sử dụng thay vì `thread::spawn`.

<Listing number="21-12" file-name="src/main.rs" caption="Giao diện `ThreadPool` lý tưởng của chúng ta">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-12/src/main.rs:here}}
```

</Listing>

Chúng ta sử dụng `ThreadPool::new` để tạo một thread pool mới với một số lượng
luồng có thể cấu hình, trong trường hợp này là bốn. Sau đó, trong vòng lặp
`for`, `pool.execute` có một giao diện tương tự như `thread::spawn` ở chỗ nó
nhận một closure mà pool nên chạy cho mỗi luồng. Chúng ta cần triển khai
`pool.execute` để nó lấy closure và đưa nó cho một luồng trong pool để chạy. Mã
này chưa biên dịch, nhưng chúng ta sẽ thử để trình biên dịch có thể hướng dẫn
chúng ta cách khắc phục nó.

<!-- Old headings. Do not remove or links may break. -->

<a id="building-the-threadpool-struct-using-compiler-driven-development"></a>

#### Xây Dựng `ThreadPool` Sử Dụng Phát Triển Dựa Trên Trình Biên Dịch

Thực hiện các thay đổi trong Listing 21-12 cho _src/main.rs_, và sau đó hãy sử
dụng các lỗi biên dịch từ `cargo check` để hướng dẫn phát triển của chúng ta.
Đây là lỗi đầu tiên chúng ta nhận được:

```console
{{#include ../listings/ch21-web-server/listing-21-12/output.txt}}
```

Tuyệt! Lỗi này cho chúng ta biết rằng chúng ta cần một loại hoặc module
`ThreadPool`, vì vậy chúng ta sẽ xây dựng nó ngay bây giờ. Việc triển khai
`ThreadPool` của chúng ta sẽ độc lập với loại công việc mà máy chủ web của chúng
ta đang làm. Vì vậy, hãy chuyển crate `hello` từ một crate nhị phân thành một
crate thư viện để giữ việc triển khai `ThreadPool` của chúng ta. Sau khi chúng
ta chuyển sang crate thư viện, chúng ta cũng có thể sử dụng thư viện thread pool
riêng biệt cho bất kỳ công việc nào chúng ta muốn làm bằng cách sử dụng thread
pool, không chỉ để phục vụ các yêu cầu web.

Tạo một file _src/lib.rs_ chứa những điều sau, đây là định nghĩa đơn giản nhất
của một cấu trúc `ThreadPool` mà chúng ta có thể có bây giờ:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/lib.rs}}
```

</Listing>

Sau đó chỉnh sửa file _main.rs_ để đưa `ThreadPool` vào phạm vi từ crate thư
viện bằng cách thêm mã sau vào đầu của _src/main.rs_:

<Listing file-name="src/main.rs">

```rust,ignore
{{#rustdoc_include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/src/main.rs:here}}
```

</Listing>

Mã này vẫn chưa hoạt động, nhưng hãy kiểm tra lại để có được lỗi tiếp theo mà
chúng ta cần giải quyết:

```console
{{#include ../listings/ch21-web-server/no-listing-01-define-threadpool-struct/output.txt}}
```

Lỗi này chỉ ra rằng tiếp theo chúng ta cần tạo một hàm liên kết có tên `new` cho
`ThreadPool`. Chúng ta cũng biết rằng `new` cần có một tham số có thể chấp nhận
`4` làm đối số và nên trả về một thể hiện `ThreadPool`. Hãy triển khai hàm `new`
đơn giản nhất có những đặc điểm đó:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/src/lib.rs}}
```

</Listing>

Chúng ta đã chọn `usize` làm loại của tham số `size` vì chúng ta biết rằng một
số âm của luồng không có ý nghĩa. Chúng ta cũng biết rằng chúng ta sẽ sử dụng
`4` này làm số lượng phần tử trong một bộ sưu tập các luồng, đó là mục đích của
loại `usize`, như đã thảo luận trong ["Các Loại Số
Nguyên"][integer-types]<!-- ignore --> trong Chương 3.

Hãy kiểm tra mã một lần nữa:

```console
{{#include ../listings/ch21-web-server/no-listing-02-impl-threadpool-new/output.txt}}
```

Bây giờ lỗi xảy ra vì chúng ta không có phương thức `execute` trên `ThreadPool`.
Nhớ lại từ
["Tạo một Số Lượng Giới Hạn Các Luồng"](#creating-a-finite-number-of-threads)<!-- ignore -->
rằng chúng ta đã quyết định thread pool của mình nên có một giao diện tương tự
như `thread::spawn`. Ngoài ra, chúng ta sẽ triển khai hàm `execute` để nó lấy
closure mà nó được cung cấp và giao nó cho một luồng rảnh trong pool để chạy.

Chúng ta sẽ định nghĩa phương thức `execute` trên `ThreadPool` để nhận một
closure làm tham số. Nhớ lại từ ["Di Chuyển Các Giá Trị Được Bắt Giữ Ra Khỏi
Closure và Các Đặc Điểm `Fn`"][fn-traits]<!-- ignore --> trong Chương 13 rằng
chúng ta có thể lấy closures làm tham số với ba đặc điểm khác nhau: `Fn`,
`FnMut`, và `FnOnce`. Chúng ta cần quyết định loại closure nào để sử dụng ở đây.
Chúng ta biết rằng cuối cùng chúng ta sẽ làm điều gì đó tương tự như việc triển
khai `thread::spawn` của thư viện chuẩn, vì vậy chúng ta có thể xem xét những
ràng buộc nào mà chữ ký của `thread::spawn` có trên tham số của nó. Tài liệu
hiển thị cho chúng ta những điều sau:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Tham số loại `F` là loại chúng ta quan tâm ở đây; tham số loại `T` liên quan đến
giá trị trả về, và chúng ta không quan tâm đến điều đó. Chúng ta có thể thấy
rằng `spawn` sử dụng `FnOnce` làm ràng buộc đặc điểm trên `F`. Đây có lẽ là điều
chúng ta cũng muốn, vì cuối cùng chúng ta sẽ truyền đối số mà chúng ta nhận được
trong `execute` cho `spawn`. Chúng ta có thể tin tưởng hơn rằng `FnOnce` là đặc
điểm mà chúng ta muốn sử dụng vì luồng để chạy một yêu cầu sẽ chỉ thực thi
closure của yêu cầu đó một lần, điều này phù hợp với `Once` trong `FnOnce`.

Tham số loại `F` cũng có ràng buộc đặc điểm `Send` và ràng buộc vòng đời
`'static`, điều này hữu ích trong tình huống của chúng ta: chúng ta cần `Send`
để chuyển closure từ một luồng sang luồng khác và `'static` vì chúng ta không
biết luồng sẽ mất bao lâu để thực thi. Hãy tạo một phương thức `execute` trên
`ThreadPool` sẽ lấy một tham số chung kiểu `F` với những ràng buộc này:

<Listing file-name="src/lib.rs">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/no-listing-03-define-execute/src/lib.rs:here}}
```

</Listing>

Chúng ta vẫn sử dụng `()` sau `FnOnce` vì `FnOnce` này đại diện cho một closure
không nhận tham số nào và trả về kiểu đơn vị `()`. Giống như các định nghĩa hàm,
kiểu trả về có thể được bỏ qua khỏi chữ ký, nhưng ngay cả khi chúng ta không có
tham số nào, chúng ta vẫn cần dấu ngoặc đơn.

Một lần nữa, đây là cách triển khai đơn giản nhất của phương thức `execute`: nó
không làm gì cả, nhưng chúng ta chỉ đang cố gắng làm cho mã của mình biên dịch.
Hãy kiểm tra lại:

```console
{{#include ../listings/ch21-web-server/no-listing-03-define-execute/output.txt}}
```

Nó biên dịch! Nhưng lưu ý rằng nếu bạn thử `cargo run` và thực hiện một yêu cầu
trong trình duyệt, bạn sẽ thấy các lỗi trong trình duyệt mà chúng ta đã thấy ở
đầu chương. Thư viện của chúng ta không thực sự gọi closure được truyền vào
`execute` nữa!

> Lưu ý: Một câu nói mà bạn có thể nghe về các ngôn ngữ với trình biên dịch
> nghiêm ngặt, như Haskell và Rust, là "nếu mã biên dịch, nó hoạt động." Nhưng
> câu nói này không phải là sự thật phổ quát. Dự án của chúng ta biên dịch,
> nhưng nó hoàn toàn không làm gì cả! Nếu chúng ta đang xây dựng một dự án thực
> tế, hoàn chỉnh, đây sẽ là thời điểm tốt để bắt đầu viết các bài kiểm tra đơn
> vị để kiểm tra rằng mã biên dịch _và_ có hành vi mà chúng ta muốn.

Hãy xem xét: điều gì sẽ khác ở đây nếu chúng ta sẽ thực thi một future thay vì
một closure?

#### Xác Thực Số Lượng Luồng trong `new`

Chúng ta không làm gì với các tham số cho `new` và `execute`. Hãy triển khai các
thân của các hàm này với hành vi mà chúng ta muốn. Để bắt đầu, hãy nghĩ về
`new`. Trước đó, chúng ta đã chọn một loại không dấu cho tham số `size` vì một
pool với số lượng luồng âm không có ý nghĩa. Tuy nhiên, một pool với số luồng là
không cũng không có ý nghĩa, nhưng không là một `usize` hoàn toàn hợp lệ. Chúng
ta sẽ thêm mã để kiểm tra rằng `size` lớn hơn không trước khi chúng ta trả về
một thể hiện `ThreadPool` và có chương trình hoảng sợ nếu nó nhận được một số
không bằng cách sử dụng macro `assert!`, như được hiển thị trong Listing 21-13.

<Listing number="21-13" file-name="src/lib.rs" caption="Triển khai `ThreadPool::new` để hoảng sợ nếu `size` là không">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-13/src/lib.rs:here}}
```

</Listing>

Chúng ta cũng đã thêm một số tài liệu cho `ThreadPool` của chúng ta với các chú
thích tài liệu. Lưu ý rằng chúng ta đã tuân theo các thực hành tài liệu tốt bằng
cách thêm một phần nêu ra các tình huống mà hàm của chúng ta có thể hoảng sợ,
như đã thảo luận trong Chương 14. Hãy thử chạy `cargo doc --open` và nhấp vào
cấu trúc `ThreadPool` để xem tài liệu được tạo ra cho `new` trông như thế nào!

Thay vì thêm macro `assert!` như chúng ta đã làm ở đây, chúng ta có thể thay đổi
`new` thành `build` và trả về một `Result` như chúng ta đã làm với
`Config::build` trong dự án I/O trong Listing 12-9. Nhưng chúng ta đã quyết định
trong trường hợp này rằng việc cố gắng tạo một thread pool mà không có bất kỳ
luồng nào nên là một lỗi không thể khôi phục. Nếu bạn cảm thấy tham vọng, hãy
thử viết một hàm có tên `build` với chữ ký sau để so sánh với hàm `new`:

```rust,ignore
pub fn build(size: usize) -> Result<ThreadPool, PoolCreationError> {
```

#### Tạo Không Gian Để Lưu Trữ Các Luồng

Bây giờ chúng ta có một cách để biết chúng ta có một số lượng luồng hợp lệ để
lưu trữ trong pool, chúng ta có thể tạo các luồng đó và lưu trữ chúng trong cấu
trúc `ThreadPool` trước khi trả về cấu trúc. Nhưng làm thế nào để chúng ta "lưu
trữ" một luồng? Hãy xem lại chữ ký của `thread::spawn`:

```rust,ignore
pub fn spawn<F, T>(f: F) -> JoinHandle<T>
    where
        F: FnOnce() -> T,
        F: Send + 'static,
        T: Send + 'static,
```

Hàm `spawn` trả về một `JoinHandle<T>`, trong đó `T` là loại mà closure trả về.
Hãy thử sử dụng `JoinHandle` và xem điều gì xảy ra. Trong trường hợp của chúng
ta, các closure mà chúng ta đang truyền cho thread pool sẽ xử lý kết nối và
không trả về bất cứ điều gì, vì vậy `T` sẽ là kiểu đơn vị `()`.

Mã trong Listing 21-14 sẽ biên dịch nhưng chưa tạo bất kỳ luồng nào. Chúng ta đã
thay đổi định nghĩa của `ThreadPool` để chứa một vector của các thể hiện
`thread::JoinHandle<()>`, khởi tạo vector với dung lượng là `size`, thiết lập
một vòng lặp `for` sẽ chạy một số mã để tạo các luồng, và trả về một thể hiện
`ThreadPool` chứa chúng.

<Listing number="21-14" file-name="src/lib.rs" caption="Tạo một vector cho `ThreadPool` để chứa các luồng">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-14/src/lib.rs:here}}
```

</Listing>

Chúng ta đã đưa `std::thread` vào phạm vi trong crate thư viện vì chúng ta đang
sử dụng `thread::JoinHandle` làm kiểu của các mục trong vector trong
`ThreadPool`.

Khi một kích thước hợp lệ được nhận, `ThreadPool` của chúng ta tạo một vector
mới có thể chứa `size` mục. Hàm `with_capacity` thực hiện cùng một nhiệm vụ như
`Vec::new` nhưng với một sự khác biệt quan trọng: nó cấp phát trước không gian
trong vector. Vì chúng ta biết chúng ta cần lưu trữ `size` phần tử trong vector,
việc thực hiện cấp phát này trước là hiệu quả hơn một chút so với việc sử dụng
`Vec::new`, việc này tự điều chỉnh kích thước khi các phần tử được chèn vào.

Khi bạn chạy `cargo check` một lần nữa, nó sẽ thành công.

<a id ="a-worker-struct-responsible-for-sending-code-from-the-threadpool-to-a-thread"></a>

#### Gửi Mã từ `ThreadPool` đến một Luồng

Chúng ta đã để lại một chú thích trong vòng lặp `for` trong Listing 21-14 liên
quan đến việc tạo các luồng. Ở đây, chúng ta sẽ xem xét cách chúng ta thực sự
tạo ra các luồng. Thư viện chuẩn cung cấp `thread::spawn` như một cách để tạo
các luồng, và `thread::spawn` mong đợi nhận được một số mã mà luồng nên chạy
ngay khi luồng được tạo. Tuy nhiên, trong trường hợp của chúng ta, chúng ta muốn
tạo các luồng và có chúng _đợi_ mã mà chúng ta sẽ gửi sau này. Việc triển khai
các luồng của thư viện chuẩn không bao gồm bất kỳ cách nào để làm điều đó; chúng
ta phải triển khai nó theo cách thủ công.

Chúng ta sẽ triển khai hành vi này bằng cách giới thiệu một cấu trúc dữ liệu mới
giữa `ThreadPool` và các luồng sẽ quản lý hành vi mới này. Chúng ta sẽ gọi cấu
trúc dữ liệu này là _Worker_, đây là một thuật ngữ phổ biến trong các triển khai
pooling. `Worker` nhận mã cần được chạy và chạy mã đó trong luồng của nó.

Hãy nghĩ về những người làm việc trong nhà bếp của một nhà hàng: các công nhân
đợi cho đến khi đơn đặt hàng đến từ khách hàng, và sau đó họ chịu trách nhiệm
lấy các đơn đặt hàng đó và thực hiện chúng.

Thay vì lưu trữ một vector các thể hiện `JoinHandle<()>` trong thread pool,
chúng ta sẽ lưu trữ các thể hiện của cấu trúc `Worker`. Mỗi `Worker` sẽ lưu trữ
một thể hiện `JoinHandle<()>` duy nhất. Sau đó, chúng ta sẽ triển khai một
phương thức trên `Worker` sẽ lấy một closure của mã để chạy và gửi nó đến luồng
đã chạy để thực thi. Chúng ta cũng sẽ cung cấp cho mỗi `Worker` một `id` để
chúng ta có thể phân biệt giữa các thể hiện khác nhau của `Worker` trong pool
khi ghi nhật ký hoặc gỡ lỗi.

Đây là quy trình mới sẽ xảy ra khi chúng ta tạo một `ThreadPool`. Chúng ta sẽ
triển khai mã gửi closure đến luồng sau khi chúng ta đã thiết lập `Worker` theo
cách này:

1. Định nghĩa một cấu trúc `Worker` chứa một `id` và một `JoinHandle<()>`.
2. Thay đổi `ThreadPool` để chứa một vector các thể hiện `Worker`.
3. Định nghĩa một hàm `Worker::new` nhận một số `id` và trả về một thể hiện
   `Worker` chứa `id` và một luồng được tạo ra với một closure rỗng.
4. Trong `ThreadPool::new`, sử dụng bộ đếm vòng lặp `for` để tạo ra một `id`,
   tạo một `Worker` mới với `id` đó, và lưu trữ `Worker` trong vector.

Nếu bạn sẵn sàng cho một thử thách, hãy thử triển khai các thay đổi này một mình
trước khi xem mã trong Listing 21-15.

Sẵn sàng chưa? Đây là Listing 21-15 với một cách để thực hiện các sửa đổi trước
đó.

<Listing number="21-15" file-name="src/lib.rs" caption="Sửa đổi `ThreadPool` để chứa các thể hiện `Worker` thay vì chứa các luồng trực tiếp">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-15/src/lib.rs:here}}
```

</Listing>

Chúng ta đã thay đổi tên của trường trên `ThreadPool` từ `threads` thành
`workers` vì nó hiện đang chứa các thể hiện `Worker` thay vì các thể hiện
`JoinHandle<()>`. Chúng ta sử dụng bộ đếm trong vòng lặp `for` làm đối số cho
`Worker::new`, và chúng ta lưu trữ mỗi `Worker` mới trong vector có tên
`workers`.

Mã bên ngoài (như máy chủ của chúng ta trong _src/main.rs_) không cần biết các
chi tiết triển khai liên quan đến việc sử dụng cấu trúc `Worker` trong
`ThreadPool`, vì vậy chúng ta làm cho cấu trúc `Worker` và hàm `new` của nó là
riêng tư. Hàm `Worker::new` sử dụng `id` mà chúng ta cung cấp và lưu trữ một thể
hiện `JoinHandle<()>` được tạo ra bằng cách tạo một luồng mới bằng một closure
rỗng.

> Lưu ý: Nếu hệ điều hành không thể tạo một luồng vì không có đủ tài nguyên hệ
> thống, `thread::spawn` sẽ hoảng sợ. Điều đó sẽ khiến toàn bộ máy chủ của chúng
> ta hoảng sợ, ngay cả khi việc tạo một số luồng có thể thành công. Để đơn giản,
> hành vi này là tốt, nhưng trong một triển khai thread pool sản xuất, bạn có
> thể muốn sử dụng [`std::thread::Builder`][builder]<!-- ignore --> và phương
> thức [`spawn`][builder-spawn]<!-- ignore --> của nó trả về `Result` thay thế.

Mã này sẽ biên dịch và sẽ lưu trữ số lượng thể hiện `Worker` mà chúng ta đã chỉ
định làm đối số cho `ThreadPool::new`. Nhưng chúng ta _vẫn_ không xử lý closure
mà chúng ta nhận được trong `execute`. Hãy xem cách làm điều đó tiếp theo.

#### Gửi Yêu Cầu đến Các Luồng thông qua Các Kênh

Vấn đề tiếp theo mà chúng ta sẽ giải quyết là các closure được đưa cho
`thread::spawn` hoàn toàn không làm gì cả. Hiện tại, chúng ta nhận được closure
mà chúng ta muốn thực thi trong phương thức `execute`. Nhưng chúng ta cần cung
cấp cho `thread::spawn` một closure để chạy khi chúng ta tạo mỗi `Worker` trong
quá trình tạo `ThreadPool`.

Chúng ta muốn các cấu trúc `Worker` mà chúng ta vừa tạo lấy mã để chạy từ một
hàng đợi được giữ trong `ThreadPool` và gửi mã đó đến luồng của nó để chạy.

Các kênh mà chúng ta đã học trong Chương 16—một cách đơn giản để giao tiếp giữa
hai luồng—sẽ hoàn hảo cho trường hợp sử dụng này. Chúng ta sẽ sử dụng một kênh
để hoạt động như hàng đợi các công việc, và `execute` sẽ gửi một công việc từ
`ThreadPool` đến các thể hiện `Worker`, sẽ gửi công việc đến luồng của nó để
chạy. Đây là kế hoạch:

1. `ThreadPool` sẽ tạo một kênh và giữ người gửi.
2. Mỗi `Worker` sẽ giữ người nhận.
3. Chúng ta sẽ tạo một cấu trúc `Job` mới sẽ chứa các closure mà chúng ta muốn
   gửi qua kênh.
4. Phương thức `execute` sẽ gửi công việc mà nó muốn thực thi thông qua người
   gửi.
5. Trong luồng của nó, `Worker` sẽ lặp qua người nhận của nó và thực thi các
   closure của bất kỳ công việc nào nó nhận được.

Hãy bắt đầu bằng cách tạo một kênh trong `ThreadPool::new` và giữ người gửi
trong thể hiện `ThreadPool`, như được hiển thị trong Listing 21-16. Cấu trúc
`Job` không chứa gì bây giờ nhưng sẽ là loại mục mà chúng ta đang gửi qua kênh.

<Listing number="21-16" file-name="src/lib.rs" caption="Sửa đổi `ThreadPool` để lưu trữ người gửi của một kênh truyền các thể hiện `Job`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-16/src/lib.rs:here}}
```

</Listing>

Trong `ThreadPool::new`, chúng ta tạo kênh mới của mình và có pool giữ người
gửi. Điều này sẽ biên dịch thành công.

Hãy thử truyền người nhận của kênh vào mỗi `Worker` khi thread pool tạo kênh.
Chúng ta biết rằng chúng ta muốn sử dụng người nhận trong luồng mà các thể hiện
`Worker` tạo ra, vì vậy chúng ta sẽ tham chiếu đến tham số `receiver` trong
closure. Mã trong Listing 21-17 chưa biên dịch hoàn toàn.

<Listing number="21-17" file-name="src/lib.rs" caption="Truyền người nhận cho mỗi `Worker`">

```rust,ignore,does_not_compile
{{#rustdoc_include ../listings/ch21-web-server/listing-21-17/src/lib.rs:here}}
```

</Listing>

Chúng ta đã thực hiện một số thay đổi nhỏ và đơn giản: chúng ta truyền người
nhận vào `Worker::new`, và sau đó chúng ta sử dụng nó bên trong closure.

Khi chúng ta cố gắng kiểm tra mã này, chúng ta nhận được lỗi này:

```console
{{#include ../listings/ch21-web-server/listing-21-17/output.txt}}
```

Mã đang cố gắng truyền `receiver` cho nhiều thể hiện `Worker`. Điều này sẽ không
hoạt động, như bạn sẽ nhớ lại từ Chương 16: việc triển khai kênh mà Rust cung
cấp là nhiều _producer_, một _consumer_. Điều này có nghĩa là chúng ta không thể
chỉ sao chép đầu tiêu thụ của kênh để sửa mã này. Chúng ta cũng không muốn gửi
một thông điệp nhiều lần đến nhiều người tiêu dùng; chúng ta muốn một danh sách
các thông điệp với nhiều thể hiện `Worker` sao cho mỗi thông điệp được xử lý một
lần.

Ngoài ra, việc lấy một công việc khỏi hàng đợi kênh liên quan đến việc thay đổi
`receiver`, vì vậy các luồng cần một cách an toàn để chia sẻ và sửa đổi
`receiver`; nếu không, chúng ta có thể gặp phải các điều kiện đua (như đã đề cập
trong Chương 16).

Nhớ lại các con trỏ thông minh an toàn cho luồng được thảo luận trong Chương 16:
để chia sẻ quyền sở hữu trên nhiều luồng và cho phép các luồng thay đổi giá trị,
chúng ta cần sử dụng `Arc<Mutex<T>>`. Loại `Arc` sẽ cho phép nhiều thể hiện
`Worker` sở hữu người nhận, và `Mutex` sẽ đảm bảo rằng chỉ một `Worker` nhận
được một công việc từ người nhận tại một thời điểm. Listing 21-18 hiển thị các
thay đổi chúng ta cần thực hiện.

<Listing number="21-18" file-name="src/lib.rs" caption="Chia sẻ người nhận giữa các thể hiện `Worker` bằng cách sử dụng `Arc` và `Mutex`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-18/src/lib.rs:here}}
```

</Listing>

Trong `ThreadPool::new`, chúng ta đặt người nhận trong một `Arc` và một `Mutex`.
Đối với mỗi `Worker` mới, chúng ta sao chép `Arc` để tăng số đếm tham chiếu để
các thể hiện `Worker` có thể chia sẻ quyền sở hữu của người nhận.

Với những thay đổi này, mã biên dịch! Chúng ta đang tiến gần đến đích!

#### Triển Khai Phương Thức `execute`

Cuối cùng, hãy triển khai phương thức `execute` trên `ThreadPool`. Chúng ta cũng
sẽ thay đổi `Job` từ một cấu trúc thành một bí danh kiểu cho một đối tượng đặc
điểm chứa loại closure mà `execute` nhận được. Như đã thảo luận trong ["Tạo Đồng
Nghĩa Kiểu với Bí Danh
Kiểu"][creating-type-synonyms-with-type-aliases]<!-- ignore --> trong Chương 20,
bí danh kiểu cho phép chúng ta làm cho các kiểu dài ngắn hơn để dễ sử dụng. Hãy
xem Listing 21-19.

<Listing number="21-19" file-name="src/lib.rs" caption="Tạo một bí danh kiểu `Job` cho một `Box` chứa mỗi closure và sau đó gửi công việc qua kênh">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-19/src/lib.rs:here}}
```

</Listing>

Sau khi tạo một thể hiện `Job` mới bằng cách sử dụng closure mà chúng ta nhận
được trong `execute`, chúng ta gửi công việc đó qua đầu gửi của kênh. Chúng ta
đang gọi `unwrap` trên `send` cho trường hợp gửi thất bại. Điều này có thể xảy
ra nếu, ví dụ, chúng ta dừng tất cả các luồng của mình từ việc thực thi, nghĩa
là đầu nhận đã dừng nhận các thông điệp mới. Tại thời điểm này, chúng ta không
thể dừng các luồng của mình từ việc thực thi: các luồng của chúng ta tiếp tục
thực thi miễn là pool tồn tại. Lý do chúng ta sử dụng `unwrap` là vì chúng ta
biết trường hợp thất bại sẽ không xảy ra, nhưng trình biên dịch không biết điều
đó.

Nhưng chúng ta chưa hoàn thành! Trong `Worker`, closure của chúng ta đang được
truyền cho `thread::spawn` vẫn chỉ _tham chiếu_ đến đầu nhận của kênh. Thay vào
đó, chúng ta cần closure để lặp mãi mãi, yêu cầu đầu nhận của kênh một công việc
và chạy công việc khi nó nhận được một. Hãy thực hiện thay đổi được hiển thị
trong Listing 21-20 cho `Worker::new`.

<Listing number="21-20" file-name="src/lib.rs" caption="Nhận và thực thi các công việc trong luồng của thể hiện `Worker`">

```rust,noplayground
{{#rustdoc_include ../listings/ch21-web-server/listing-21-20/src/lib.rs:here}}
```

</Listing>

Ở đây, trước tiên chúng ta gọi `lock` trên `receiver` để có được mutex, và sau
đó chúng ta gọi `unwrap` để hoảng sợ khi có bất kỳ lỗi nào. Việc có được một
khóa có thể không thành công nếu mutex đang ở trạng thái _poisoned_, điều này có
thể xảy ra nếu một số luồng khác hoảng sợ trong khi giữ khóa thay vì giải phóng
khóa. Trong tình huống này, việc gọi `unwrap` để có luồng này hoảng sợ là hành
động đúng đắn để thực hiện. Đừng ngại thay đổi `unwrap` này thành `expect` với
một thông báo lỗi có ý nghĩa đối với bạn.

Nếu chúng ta có được khóa trên mutex, chúng ta gọi `recv` để nhận một `Job` từ
kênh. Một `unwrap` cuối cùng vượt qua bất kỳ lỗi nào ở đây cũng vậy, có thể xảy
ra nếu luồng giữ người gửi đã tắt, tương tự như cách phương thức `send` trả về
`Err` nếu người nhận tắt.

Cuộc gọi đến `recv` chặn, vì vậy nếu chưa có công việc, luồng hiện tại sẽ đợi
cho đến khi một công việc có sẵn. `Mutex<T>` đảm bảo rằng chỉ một luồng `Worker`
tại một thời điểm đang cố gắng yêu cầu một công việc.

Thread pool của chúng ta bây giờ đang ở trạng thái hoạt động! Hãy thử
`cargo run` và thực hiện một số yêu cầu:

<!-- manual-regeneration
cd listings/ch21-web-server/listing-21-20
cargo run
make some requests to 127.0.0.1:7878
Can't automate because the output depends on making requests
-->

```console
$ cargo run
   Compiling hello v0.1.0 (file:///projects/hello)
warning: field `workers` is never read
 --> src/lib.rs:7:5
  |
6 | pub struct ThreadPool {
  |            ---------- field in this struct
7 |     workers: Vec<Worker>,
  |     ^^^^^^^
  |
  = note: `#[warn(dead_code)]` on by default

warning: fields `id` and `thread` are never read
  --> src/lib.rs:48:5
   |
47 | struct Worker {
   |        ------ fields in this struct
48 |     id: usize,
   |     ^^
49 |     thread: thread::JoinHandle<()>,
   |     ^^^^^^

warning: `hello` (lib) generated 2 warnings
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 4.91s
     Running `target/debug/hello`
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
Worker 1 got a job; executing.
Worker 3 got a job; executing.
Worker 0 got a job; executing.
Worker 2 got a job; executing.
```

Thành công! Bây giờ chúng ta có một thread pool thực thi các kết nối không đồng
bộ. Không bao giờ có nhiều hơn bốn luồng được tạo ra, vì vậy hệ thống của chúng
ta sẽ không bị quá tải nếu máy chủ nhận được rất nhiều yêu cầu. Nếu chúng ta
thực hiện một yêu cầu đến _/sleep_, máy chủ sẽ có thể phục vụ các yêu cầu khác
bằng cách có một luồng khác chạy chúng.

> Lưu ý: Nếu bạn mở _/sleep_ trong nhiều cửa sổ trình duyệt cùng một lúc, chúng
> có thể tải một cái một lúc trong khoảng thời gian năm giây. Một số trình duyệt
> web thực thi nhiều phiên bản của cùng một yêu cầu tuần tự vì lý do cache. Hạn
> chế này không phải do máy chủ web của chúng ta gây ra.

Đây là một thời điểm tốt để tạm dừng và xem xét cách mã trong Listings 21-18,
21-19, và 21-20 sẽ khác nhau nếu chúng ta đang sử dụng futures thay vì closure
cho công việc cần thực hiện. Những kiểu nào sẽ thay đổi? Chữ ký phương thức sẽ
khác nhau như thế nào, nếu có? Những phần nào của mã sẽ giữ nguyên?

Sau khi học về vòng lặp `while let` trong Chương 17 và Chương 19, bạn có thể
đang tự hỏi tại sao chúng ta không viết mã luồng công nhân như được hiển thị
trong Listing 21-21.

<Listing number="21-21" file-name="src/lib.rs" caption="Một cách triển khai thay thế của `Worker::new` sử dụng `while let`">

```rust,ignore,not_desired_behavior
{{#rustdoc_include ../listings/ch21-web-server/listing-21-21/src/lib.rs:here}}
```

</Listing>

Mã này biên dịch và chạy nhưng không dẫn đến hành vi luồng mong muốn: một yêu
cầu chậm vẫn sẽ khiến các yêu cầu khác phải đợi để được xử lý. Lý do hơi tinh
tế: cấu trúc `Mutex` không có phương thức `unlock` công khai vì quyền sở hữu của
khóa dựa trên vòng đời của `MutexGuard<T>` trong `LockResult<MutexGuard<T>>` mà
phương thức `lock` trả về. Tại thời điểm biên dịch, trình kiểm tra mượn sau đó
có thể thực thi quy tắc rằng một tài nguyên được bảo vệ bởi `Mutex` không thể
được truy cập trừ khi chúng ta giữ khóa. Tuy nhiên, việc triển khai này cũng có
thể dẫn đến khóa được giữ lâu hơn dự định nếu chúng ta không chú ý đến vòng đời
của `MutexGuard<T>`.

Mã trong Listing 21-20 sử dụng
`let job = receiver.lock().unwrap().recv().unwrap();` hoạt động vì với `let`,
bất kỳ giá trị tạm thời nào được sử dụng trong biểu thức ở phía bên phải của dấu
bằng sẽ bị bỏ ngay lập tức khi câu lệnh `let` kết thúc. Tuy nhiên, `while let`
(và `if let` và `match`) không bỏ các giá trị tạm thời cho đến khi kết thúc khối
liên kết. Trong Listing 21-21, khóa vẫn được giữ trong suốt thời gian gọi
`job()`, có nghĩa là các thể hiện `Worker` khác không thể nhận công việc.

[creating-type-synonyms-with-type-aliases]:
  ch20-03-advanced-types.html#creating-type-synonyms-with-type-aliases
[integer-types]: ch03-02-data-types.html#integer-types
[fn-traits]:
  ch13-01-closures.html#moving-captured-values-out-of-the-closure-and-the-fn-traits
[builder]: ../std/thread/struct.Builder.html
[builder-spawn]: ../std/thread/struct.Builder.html#method.spawn
