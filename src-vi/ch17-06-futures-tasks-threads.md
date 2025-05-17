## Tổng hợp tất cả: Futures, Tasks, và Threads

Như chúng ta đã thấy trong [Chương 16][ch16]<!-- ignore -->, threads cung cấp
một cách tiếp cận để thực hiện đồng thời. Chúng ta đã thấy một cách tiếp cận
khác trong chương này: sử dụng async với futures và streams. Nếu bạn đang tự hỏi
khi nào nên chọn phương pháp này thay vì phương pháp kia, câu trả lời là: tùy
trường hợp! Và trong nhiều trường hợp, sự lựa chọn không phải là threads _hoặc_
async mà là threads _và_ async.

Nhiều hệ điều hành đã cung cấp các mô hình đồng thời dựa trên threading trong
hàng thập kỷ qua, và nhiều ngôn ngữ lập trình hỗ trợ chúng do đó. Tuy nhiên, các
mô hình này không phải không có những đánh đổi. Trên nhiều hệ điều hành, chúng
sử dụng một lượng khá lớn bộ nhớ cho mỗi thread, và chúng đi kèm với một số chi
phí cho việc khởi động và tắt. Thread cũng chỉ là một lựa chọn khi hệ điều hành
và phần cứng của bạn hỗ trợ chúng. Không giống như máy tính để bàn và di động
phổ biến, một số hệ thống nhúng hoàn toàn không có hệ điều hành, vì vậy chúng
cũng không có threads.

Mô hình async cung cấp một tập hợp các đánh đổi khác—và cuối cùng là bổ sung.
Trong mô hình async, các hoạt động đồng thời không yêu cầu thread riêng của
chúng. Thay vào đó, chúng có thể chạy trên các task, như khi chúng ta sử dụng
`trpl::spawn_task` để khởi động công việc từ một hàm đồng bộ trong phần streams.
Một task tương tự như một thread, nhưng thay vì được quản lý bởi hệ điều hành,
nó được quản lý bởi mã cấp thư viện: runtime.

Trong phần trước, chúng ta đã thấy rằng chúng ta có thể xây dựng một stream bằng
cách sử dụng kênh async và tạo ra một task async mà chúng ta có thể gọi từ mã
đồng bộ. Chúng ta có thể làm điều tương tự với một thread. Trong Listing 17-40,
chúng ta đã sử dụng `trpl::spawn_task` và `trpl::sleep`. Trong Listing 17-41,
chúng ta thay thế chúng bằng các API `thread::spawn` và `thread::sleep` từ thư
viện chuẩn trong hàm `get_intervals`.

<Listing number="17-41" caption="Sử dụng các API `std::thread` thay vì các API async `trpl` cho hàm `get_intervals`" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-41 code is not available in the listings folder
// This content may be from a different version of the book
// Consider using listing-17-25 as reference
```

</Listing>

Nếu bạn chạy mã này, đầu ra giống hệt với của Listing 17-40. Và chú ý có rất ít
thay đổi ở đây từ góc độ của mã gọi. Hơn nữa, mặc dù một trong các hàm của chúng
ta tạo ra một task async trên runtime và hàm kia tạo ra một thread của hệ điều
hành, các stream kết quả không bị ảnh hưởng bởi sự khác biệt.

Mặc dù có những điểm tương đồng, hai cách tiếp cận này hoạt động rất khác nhau,
mặc dù chúng ta có thể khó đo lường được điều đó trong ví dụ rất đơn giản này.
Chúng ta có thể tạo ra hàng triệu task async trên bất kỳ máy tính cá nhân hiện
đại nào. Nếu chúng ta cố gắng làm điều đó với threads, chúng ta sẽ thực sự hết
bộ nhớ!

Tuy nhiên, có lý do khiến các API này rất giống nhau. Threads hoạt động như một
ranh giới cho các tập hợp các hoạt động đồng bộ; tính đồng thời là có thể _giữa_
các thread. Tasks hoạt động như một ranh giới cho các tập hợp hoạt động _bất
đồng bộ_; tính đồng thời là có thể cả _giữa_ và _trong_ các task, bởi vì một
task có thể chuyển đổi giữa các future trong phần thân của nó. Cuối cùng, future
là đơn vị đồng thời nhỏ nhất của Rust, và mỗi future có thể đại diện cho một cây
các future khác. Runtime—cụ thể là trình thực thi của nó—quản lý tasks, và tasks
quản lý futures. Theo nghĩa đó, tasks tương tự như các thread nhẹ, được quản lý
bởi runtime với các khả năng bổ sung đến từ việc được quản lý bởi runtime thay
vì bởi hệ điều hành.

Điều này không có nghĩa là task async luôn tốt hơn threads (hoặc ngược lại).
Tính đồng thời với threads theo một số cách là một mô hình lập trình đơn giản
hơn tính đồng thời với `async`. Điều đó có thể là một điểm mạnh hoặc một điểm
yếu. Threads hơi "phóng và quên"; chúng không có tương đương tự nhiên với
future, vì vậy chúng chỉ đơn giản chạy đến khi hoàn thành mà không bị gián đoạn
ngoại trừ bởi chính hệ điều hành. Tức là, chúng không có hỗ trợ tích hợp cho
_tính đồng thời nội tác vụ_ như cách futures có. Threads trong Rust cũng không
có cơ chế cho việc hủy bỏ—một chủ đề chúng ta chưa đề cập rõ ràng trong chương
này nhưng đã được ngụ ý bởi thực tế rằng bất cứ khi nào chúng ta kết thúc một
future, trạng thái của nó được dọn dẹp một cách chính xác.

Những hạn chế này cũng khiến threads khó kết hợp hơn futures. Ví dụ, khó khăn
hơn nhiều để sử dụng threads để xây dựng các trợ giúp như các phương thức
`timeout` và `throttle` mà chúng ta đã xây dựng trước đó trong chương này. Việc
futures là các cấu trúc dữ liệu phong phú hơn có nghĩa là chúng có thể được kết
hợp với nhau một cách tự nhiên hơn, như chúng ta đã thấy.

Vậy tasks cho chúng ta kiểm soát _bổ sung_ đối với futures, cho phép chúng ta
chọn nơi và cách nhóm chúng. Và hóa ra threads và tasks thường hoạt động rất tốt
cùng nhau, bởi vì tasks có thể (ít nhất là trong một số runtime) được di chuyển
quanh giữa các thread. Thực tế, bên dưới, runtime mà chúng ta đã sử dụng—bao gồm
các hàm `spawn_blocking` và `spawn_task`—là đa luồng theo mặc định! Nhiều
runtime sử dụng một cách tiếp cận được gọi là _đánh cắp công việc_ để di chuyển
các task một cách trong suốt giữa các thread, dựa trên cách các thread hiện đang
được sử dụng, để cải thiện hiệu suất tổng thể của hệ thống. Cách tiếp cận đó
thực sự yêu cầu cả threads _và_ tasks, và do đó là futures.

Khi suy nghĩ về phương pháp nào để sử dụng khi nào, hãy xem xét những quy tắc
chung sau:

- Nếu công việc _rất có thể song song hóa_, chẳng hạn như xử lý một lượng lớn dữ
  liệu mà mỗi phần có thể được xử lý riêng biệt, threads là lựa chọn tốt hơn.
- Nếu công việc _rất đồng thời_, chẳng hạn như xử lý tin nhắn từ một loạt nguồn
  khác nhau có thể đến theo các khoảng thời gian hoặc tốc độ khác nhau, async là
  lựa chọn tốt hơn.

Và nếu bạn cần cả tính song song và tính đồng thời, bạn không phải chọn giữa
threads và async. Bạn có thể sử dụng chúng cùng nhau tự do, để mỗi cái đóng vai
trò mà nó làm tốt nhất. Ví dụ, Listing 17-42 hiển thị một ví dụ khá phổ biến về
loại kết hợp này trong mã Rust thực tế.

<Listing number="17-42" caption="Gửi tin nhắn với mã chặn trong một thread và đợi tin nhắn trong một async block" file-name="src/main.rs">

```rust,ignore
// Note: Listing 17-42 code is not available in the listings folder
// This content may be from a different version of the book
```

</Listing>

Chúng ta bắt đầu bằng cách tạo một kênh async, sau đó tạo ra một thread lấy
quyền sở hữu của phía gửi của kênh. Trong thread, chúng ta gửi các số từ 1 đến
10, ngủ một giây giữa mỗi số. Cuối cùng, chúng ta chạy một future được tạo với
một async block được truyền vào `trpl::run` giống như chúng ta đã làm trong suốt
chương. Trong future đó, chúng ta đợi những tin nhắn đó, giống như trong các ví
dụ truyền tin nhắn khác mà chúng ta đã thấy.

Quay trở lại kịch bản chúng ta đã mở đầu chương, hãy tưởng tượng chạy một tập
hợp các tác vụ mã hóa video sử dụng một thread chuyên dụng (bởi vì mã hóa video
là tính toán nặng) nhưng thông báo cho UI rằng các hoạt động đó đã hoàn thành
với một kênh async. Có vô số ví dụ về các loại kết hợp này trong các trường hợp
sử dụng thực tế.

## Tóm tắt

Đây không phải là lần cuối bạn sẽ thấy tính đồng thời trong cuốn sách này. Dự án
trong [Chương 21][ch21] sẽ áp dụng các khái niệm này trong một tình huống thực
tế hơn so với các ví dụ đơn giản hơn đã thảo luận ở đây và so sánh việc giải
quyết vấn đề với threading đối với tasks một cách trực tiếp hơn.

Bất kể bạn chọn cách tiếp cận nào trong số này, Rust cung cấp cho bạn các công
cụ bạn cần để viết mã đồng thời an toàn, nhanh chóng— cho dù là cho một máy chủ
web thông lượng cao hay một hệ điều hành nhúng.

Tiếp theo, chúng ta sẽ nói về các cách thành ngữ để mô hình hóa vấn đề và cấu
trúc giải pháp khi các chương trình Rust của bạn trở nên lớn hơn. Ngoài ra,
chúng ta sẽ thảo luận cách các thành ngữ của Rust liên quan đến những thành ngữ
mà bạn có thể quen thuộc từ lập trình hướng đối tượng.

[ch16]: http://localhost:3000/ch16-00-concurrency.html
[combining-futures]:
  ch17-03-more-futures.html#building-our-own-async-abstractions
[streams]: ch17-04-streams.html#composing-streams
[ch21]: ch21-00-final-project-a-web-server.html
