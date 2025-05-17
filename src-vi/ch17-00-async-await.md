# Cơ Bản về Lập Trình Bất Đồng Bộ: Async, Await, Futures, và Streams

Nhiều thao tác mà chúng ta yêu cầu máy tính thực hiện có thể mất một khoảng thời
gian để hoàn thành. Sẽ thật tốt nếu chúng ta có thể làm việc khác trong khi đang
chờ đợi những quá trình chạy lâu đó hoàn thành. Máy tính hiện đại cung cấp hai
kỹ thuật để làm nhiều thao tác cùng một lúc: song song và đồng thời. Tuy nhiên,
một khi chúng ta bắt đầu viết các chương trình liên quan đến các thao tác song
song hoặc đồng thời, chúng ta nhanh chóng gặp phải những thách thức mới vốn có
trong _lập trình bất đồng bộ_, nơi các thao tác có thể không hoàn thành tuần tự
theo thứ tự chúng được bắt đầu. Chương này phát triển từ việc sử dụng thread
trong Chương 16 cho tính song song và đồng thời bằng cách giới thiệu một cách
tiếp cận thay thế cho lập trình bất đồng bộ: Futures, Streams của Rust, cú pháp
`async` và `await` hỗ trợ chúng, và các công cụ để quản lý và điều phối giữa các
thao tác bất đồng bộ.

Hãy xem xét một ví dụ. Giả sử bạn đang xuất một video về buổi lễ gia đình mà bạn
đã tạo, một thao tác có thể mất từ vài phút đến vài giờ. Việc xuất video sẽ sử
dụng càng nhiều năng lực CPU và GPU càng tốt. Nếu bạn chỉ có một lõi CPU và hệ
điều hành của bạn không tạm dừng việc xuất đó cho đến khi nó hoàn thành — nghĩa
là, nếu nó thực hiện việc xuất _đồng bộ_ — bạn không thể làm bất cứ điều gì khác
trên máy tính của mình trong khi tác vụ đó đang chạy. Đó sẽ là một trải nghiệm
khá khó chịu. May mắn thay, hệ điều hành của máy tính bạn có thể, và thực sự,
ngắt quãng việc xuất một cách vô hình thường xuyên để cho phép bạn làm việc khác
đồng thời.

Bây giờ giả sử bạn đang tải xuống một video được chia sẻ bởi người khác, điều
này cũng có thể mất một thời gian nhưng không chiếm nhiều thời gian CPU. Trong
trường hợp này, CPU phải đợi dữ liệu đến từ mạng. Mặc dù bạn có thể bắt đầu đọc
dữ liệu khi nó bắt đầu đến, nhưng có thể mất một thời gian để tất cả hiện lên.
Ngay cả khi dữ liệu đã có đầy đủ, nếu video khá lớn, có thể mất ít nhất một hoặc
hai giây để tải tất cả. Điều đó có thể không nghe có vẻ nhiều, nhưng đó là một
khoảng thời gian rất dài đối với bộ xử lý hiện đại, có thể thực hiện hàng tỷ
thao tác mỗi giây. Một lần nữa, hệ điều hành của bạn sẽ ngắt quãng chương trình
của bạn một cách vô hình để cho phép CPU thực hiện công việc khác trong khi chờ
cuộc gọi mạng hoàn thành.

Việc xuất video là một ví dụ về thao tác _bị ràng buộc bởi CPU_ hoặc _bị ràng
buộc bởi tính toán_. Nó bị giới hạn bởi tốc độ xử lý dữ liệu tiềm năng của máy
tính trong CPU hoặc GPU, và bao nhiêu trong số tốc độ đó nó có thể dành cho thao
tác đó. Việc tải xuống video là một ví dụ về thao tác _bị ràng buộc bởi IO_, bởi
vì nó bị giới hạn bởi tốc độ của _đầu vào và đầu ra_ của máy tính; nó chỉ có thể
nhanh bằng tốc độ dữ liệu có thể được gửi qua mạng.

Trong cả hai ví dụ này, sự gián đoạn vô hình của hệ điều hành cung cấp một hình
thức của tính đồng thời. Tuy nhiên, tính đồng thời đó chỉ xảy ra ở cấp độ của
toàn bộ chương trình: hệ điều hành ngắt quãng một chương trình để cho phép các
chương trình khác thực hiện công việc. Trong nhiều trường hợp, bởi vì chúng ta
hiểu các chương trình của mình ở một mức độ chi tiết hơn nhiều so với hệ điều
hành, chúng ta có thể phát hiện các cơ hội cho tính đồng thời mà hệ điều hành
không thể thấy.

Ví dụ, nếu chúng ta đang xây dựng một công cụ để quản lý việc tải xuống tệp,
chúng ta nên có thể viết chương trình của mình để việc bắt đầu một lần tải xuống
sẽ không khóa giao diện người dùng, và người dùng nên có thể bắt đầu nhiều lần
tải xuống cùng một lúc. Tuy nhiên, nhiều API hệ điều hành để tương tác với mạng
là _chặn_, nghĩa là chúng chặn tiến trình của chương trình cho đến khi dữ liệu
mà chúng đang xử lý hoàn toàn sẵn sàng.

> Lưu ý: Đây là cách hoạt động của _hầu hết_ các lệnh gọi hàm, nếu bạn nghĩ về
> nó. Tuy nhiên, thuật ngữ _chặn_ thường được dành riêng cho các lệnh gọi hàm
> tương tác với tệp, mạng, hoặc các tài nguyên khác trên máy tính, bởi vì đó là
> những trường hợp mà một chương trình riêng lẻ sẽ được hưởng lợi từ thao tác
> _không_ chặn.

Chúng ta có thể tránh chặn thread chính của mình bằng cách tạo ra một thread
riêng để tải xuống mỗi tệp. Tuy nhiên, chi phí phụ của những thread đó cuối cùng
sẽ trở thành một vấn đề. Sẽ tốt hơn nếu lệnh gọi không chặn ngay từ đầu. Cũng sẽ
tốt hơn nếu chúng ta có thể viết theo cùng một kiểu trực tiếp mà chúng ta sử
dụng trong mã chặn, tương tự như sau:

```rust,ignore,does_not_compile
let data = fetch_data_from(url).await;
println!("{data}");
```

Đó chính xác là những gì mà trừu tượng _async_ (viết tắt của _asynchronous_) của
Rust cung cấp cho chúng ta. Trong chương này, bạn sẽ tìm hiểu tất cả về async
khi chúng ta đề cập đến các chủ đề sau:

- Cách sử dụng cú pháp `async` và `await` của Rust
- Cách sử dụng mô hình async để giải quyết một số thách thức tương tự mà chúng
  ta đã xem xét trong Chương 16
- Cách đa luồng và async cung cấp các giải pháp bổ sung cho nhau, mà bạn có thể
  kết hợp trong nhiều trường hợp

Tuy nhiên, trước khi chúng ta thấy cách async hoạt động trong thực tế, chúng ta
cần có một đoạn ngắn để thảo luận về sự khác biệt giữa tính song song và tính
đồng thời.

### Tính Song Song và Tính Đồng Thời

Chúng ta đã coi tính song song và tính đồng thời là phần lớn có thể hoán đổi cho
nhau cho đến nay. Bây giờ chúng ta cần phân biệt giữa chúng một cách chính xác
hơn, bởi vì sự khác biệt sẽ xuất hiện khi chúng ta bắt đầu làm việc.

Hãy xem xét các cách khác nhau mà một nhóm có thể chia nhỏ công việc trên một dự
án phần mềm. Bạn có thể giao cho một thành viên nhiều nhiệm vụ, giao cho mỗi
thành viên một nhiệm vụ, hoặc sử dụng kết hợp của hai phương pháp.

Khi một cá nhân làm việc trên nhiều nhiệm vụ khác nhau trước khi bất kỳ nhiệm vụ
nào hoàn thành, đây là _tính đồng thời_. Có thể bạn có hai dự án khác nhau được
kiểm tra trên máy tính của bạn, và khi bạn cảm thấy chán hoặc bị kẹt trên một dự
án, bạn chuyển sang dự án khác. Bạn chỉ là một người, vì vậy bạn không thể tiến
triển trên cả hai nhiệm vụ cùng một lúc chính xác, nhưng bạn có thể đa nhiệm,
tiến triển trên một nhiệm vụ tại một thời điểm bằng cách chuyển đổi giữa chúng
(xem Hình 17-1).

<figure>

<img src="img/trpl17-01.svg" class="center" alt="Một sơ đồ với các hộp được dán nhãn Task A và Task B, với các viên kim cương trong chúng đại diện cho các nhiệm vụ phụ. Có các mũi tên chỉ từ A1 đến B1, B1 đến A2, A2 đến B2, B2 đến A3, A3 đến A4, và A4 đến B3. Các mũi tên giữa các nhiệm vụ phụ vượt qua các hộp giữa Task A và Task B." />

<figcaption>Hình 17-1: Một luồng công việc đồng thời, chuyển đổi giữa Nhiệm vụ A và Nhiệm vụ B</figcaption>

</figure>

Khi nhóm chia nhỏ một nhóm nhiệm vụ bằng cách để mỗi thành viên nhận một nhiệm
vụ và làm việc trên đó một mình, đây là _tính song song_. Mỗi người trong nhóm
có thể tiến triển chính xác cùng một lúc (xem Hình 17-2).

<figure>

<img src="img/trpl17-02.svg" class="center" alt="Một sơ đồ với các hộp được dán nhãn Task A và Task B, với các viên kim cương trong chúng đại diện cho các nhiệm vụ phụ. Có các mũi tên chỉ từ A1 đến A2, A2 đến A3, A3 đến A4, B1 đến B2, và B2 đến B3. Không có mũi tên nào đi qua giữa các hộp cho Task A và Task B." />

<figcaption>Hình 17-2: Một luồng công việc song song, nơi công việc diễn ra trên Nhiệm vụ A và Nhiệm vụ B một cách độc lập</figcaption>

</figure>

Trong cả hai luồng công việc này, bạn có thể phải phối hợp giữa các nhiệm vụ
khác nhau. Có thể bạn _nghĩ_ rằng nhiệm vụ được giao cho một người hoàn toàn độc
lập với công việc của mọi người khác, nhưng trên thực tế nó yêu cầu một người
khác trong nhóm phải hoàn thành nhiệm vụ của họ trước. Một số công việc có thể
được thực hiện song song, nhưng một số công việc lại thực sự là _tuần tự_: nó
chỉ có thể xảy ra trong một chuỗi, một nhiệm vụ sau nhiệm vụ khác, như trong
Hình 17-3.

<figure>

<img src="img/trpl17-03.svg" class="center" alt="Một sơ đồ với các hộp được dán nhãn Task A và Task B, với các viên kim cương trong chúng đại diện cho các nhiệm vụ phụ. Có các mũi tên chỉ từ A1 đến A2, A2 đến một cặp đường thẳng dọc dày như một biểu tượng 'tạm dừng', từ biểu tượng đó đến A3, B1 đến B2, B2 đến B3, nằm bên dưới biểu tượng đó, B3 đến A3, và B3 đến B4." />

<figcaption>Hình 17-3: Một luồng công việc song song một phần, nơi công việc diễn ra trên Nhiệm vụ A và Nhiệm vụ B một cách độc lập cho đến khi Nhiệm vụ A3 bị chặn do kết quả của Nhiệm vụ B3.</figcaption>

</figure>

Tương tự, bạn có thể nhận ra rằng một trong các nhiệm vụ của bạn phụ thuộc vào
một nhiệm vụ khác của bạn. Bây giờ công việc đồng thời của bạn cũng đã trở thành
tuần tự.

Tính song song và tính đồng thời cũng có thể giao nhau. Nếu bạn biết rằng một
đồng nghiệp bị kẹt cho đến khi bạn hoàn thành một trong các nhiệm vụ của mình,
bạn có thể sẽ tập trung tất cả nỗ lực vào nhiệm vụ đó để "mở khóa" cho đồng
nghiệp của bạn. Bạn và đồng nghiệp của bạn không còn có thể làm việc song song,
và bạn cũng không còn có thể làm việc đồng thời trên các nhiệm vụ của riêng bạn.

Cùng một động lực cơ bản cũng áp dụng với phần mềm và phần cứng. Trên một máy
với một lõi CPU duy nhất, CPU chỉ có thể thực hiện một thao tác tại một thời
điểm, nhưng nó vẫn có thể làm việc đồng thời. Sử dụng các công cụ như thread,
tiến trình, và async, máy tính có thể tạm dừng một hoạt động và chuyển sang các
hoạt động khác trước khi cuối cùng quay lại hoạt động đầu tiên đó. Trên một máy
với nhiều lõi CPU, nó cũng có thể làm việc song song. Một lõi có thể thực hiện
một nhiệm vụ trong khi một lõi khác thực hiện một nhiệm vụ hoàn toàn không liên
quan, và những thao tác đó thực sự xảy ra cùng một lúc.

Khi làm việc với async trong Rust, chúng ta luôn đang xử lý tính đồng thời. Tùy
thuộc vào phần cứng, hệ điều hành, và runtime async mà chúng ta đang sử dụng (sẽ
nói thêm về runtime async sau), tính đồng thời đó cũng có thể sử dụng tính song
song bên dưới nắp capo.

Bây giờ, hãy đi sâu vào cách lập trình async trong Rust thực sự hoạt động.
