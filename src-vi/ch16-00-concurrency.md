# Lập Trình Đồng Thời Không Lo Lắng

Xử lý lập trình đồng thời một cách an toàn và hiệu quả là một mục tiêu chính
khác của Rust. _Lập trình đồng thời_ (Concurrent programming), trong đó các phần
khác nhau của chương trình thực thi độc lập, và _lập trình song song_ (parallel
programming), trong đó các phần khác nhau của chương trình thực thi cùng một
lúc, đang ngày càng trở nên quan trọng khi ngày càng nhiều máy tính tận dụng
nhiều bộ xử lý của chúng. Trong lịch sử, lập trình trong những bối cảnh này
thường khó khăn và dễ gây lỗi. Rust hy vọng sẽ thay đổi điều đó.

Ban đầu, nhóm phát triển Rust nghĩ rằng đảm bảo an toàn bộ nhớ và ngăn chặn các
vấn đề đồng thời là hai thách thức riêng biệt cần được giải quyết bằng các
phương pháp khác nhau. Theo thời gian, nhóm đã phát hiện ra rằng hệ thống quyền
sở hữu (ownership) và kiểu dữ liệu là một bộ công cụ mạnh mẽ để giúp quản lý cả
an toàn bộ nhớ _và_ các vấn đề đồng thời! Bằng cách tận dụng quyền sở hữu và
kiểm tra kiểu, nhiều lỗi đồng thời trở thành lỗi thời điểm biên dịch trong Rust
thay vì lỗi thời gian chạy. Do đó, thay vì khiến bạn tốn nhiều thời gian cố gắng
tái hiện chính xác các trường hợp mà lỗi đồng thời thời gian chạy xảy ra, mã
không đúng sẽ từ chối biên dịch và hiển thị lỗi giải thích vấn đề. Kết quả là,
bạn có thể sửa mã của mình trong khi bạn đang làm việc với nó thay vì có thể
phải sửa sau khi nó đã được triển khai vào sản xuất. Chúng tôi đã đặt biệt danh
cho khía cạnh này của Rust là _lập trình đồng thời không lo lắng_ (fearless
concurrency). Lập trình đồng thời không lo lắng cho phép bạn viết mã không có
lỗi tinh vi và dễ dàng tái cấu trúc mà không gây ra lỗi mới.

> Lưu ý: Để đơn giản hóa, chúng tôi sẽ gọi nhiều vấn đề là _đồng thời_
> (concurrent) thay vì chính xác hơn bằng cách nói _đồng thời và/hoặc song song_
> (concurrent and/or parallel). Trong chương này, vui lòng thay thế trong đầu
> _đồng thời và/hoặc song song_ bất cứ khi nào chúng tôi sử dụng _đồng thời_.
> Trong chương tiếp theo, nơi sự phân biệt quan trọng hơn, chúng tôi sẽ cụ thể
> hơn.

Nhiều ngôn ngữ rất giáo điều về các giải pháp họ cung cấp để xử lý các vấn đề
đồng thời. Ví dụ, Erlang có chức năng thanh lịch cho đồng thời truyền tin nhắn
nhưng chỉ có cách mơ hồ để chia sẻ trạng thái giữa các luồng. Việc chỉ hỗ trợ
một tập con của các giải pháp có thể là một chiến lược hợp lý cho các ngôn ngữ
cấp cao hơn bởi vì một ngôn ngữ cấp cao hơn hứa hẹn lợi ích từ việc từ bỏ một số
kiểm soát để có được sự trừu tượng. Tuy nhiên, các ngôn ngữ cấp thấp hơn được kỳ
vọng sẽ cung cấp giải pháp có hiệu suất tốt nhất trong bất kỳ tình huống nào và
có ít sự trừu tượng hơn trên phần cứng. Do đó, Rust cung cấp nhiều công cụ để mô
hình hóa vấn đề theo cách phù hợp với tình huống và yêu cầu của bạn.

Dưới đây là các chủ đề chúng ta sẽ đề cập trong chương này:

- Làm thế nào để tạo luồng để chạy nhiều phần mã cùng một lúc
- _Đồng thời truyền tin_ (Message-passing concurrency), nơi các kênh gửi tin
  nhắn giữa các luồng
- _Đồng thời trạng thái chia sẻ_ (Shared-state concurrency), nơi nhiều luồng có
  quyền truy cập vào một số phần dữ liệu
- Các trait `Sync` và `Send`, mở rộng các đảm bảo đồng thời của Rust cho các
  kiểu do người dùng định nghĩa cũng như các kiểu được cung cấp bởi thư viện
  chuẩn
