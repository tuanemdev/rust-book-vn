## So sánh Hiệu suất: Vòng lặp và Iterator

Để quyết định nên sử dụng vòng lặp hay iterator, bạn cần biết triển khai nào
nhanh hơn: phiên bản của hàm `search` với vòng lặp `for` rõ ràng hay phiên bản
với iterator.

Chúng tôi đã chạy một bài kiểm tra hiệu suất bằng cách tải toàn bộ nội dung của
_The Adventures of Sherlock Holmes_ của Sir Arthur Conan Doyle vào một `String`
và tìm kiếm từ _the_ trong nội dung. Dưới đây là kết quả kiểm tra hiệu suất trên
phiên bản `search` sử dụng vòng lặp `for` và phiên bản sử dụng iterator:

```text
test bench_search_for  ... bench:  19,620,300 ns/iter (+/- 915,700)
test bench_search_iter ... bench:  19,234,900 ns/iter (+/- 657,200)
```

Hai triển khai có hiệu suất tương tự! Chúng tôi sẽ không giải thích mã kiểm tra
hiệu suất ở đây vì điểm quan trọng không phải để chứng minh rằng hai phiên bản
là tương đương mà để có cảm nhận chung về cách hai triển khai này so sánh về mặt
hiệu suất.

Để có một bài kiểm tra hiệu suất toàn diện hơn, bạn nên kiểm tra sử dụng các văn
bản khác nhau với kích thước khác nhau làm `contents`, các từ khác nhau và các
từ có độ dài khác nhau làm `query`, và tất cả các biến thể khác. Điểm quan trọng
là: iterator, mặc dù là một trừu tượng cấp cao, được biên dịch thành mã gần
giống như khi bạn tự viết mã cấp thấp. Iterator là một trong những _trừu tượng
không tốn chi phí_ của Rust, nghĩa là việc sử dụng trừu tượng không áp đặt thêm
chi phí thời gian chạy. Điều này tương tự như cách Bjarne Stroustrup, nhà thiết
kế và người triển khai ban đầu của C++, định nghĩa _không tốn chi phí_ trong
"Foundations of C++" (2012):

> Nói chung, các triển khai C++ tuân theo nguyên tắc không tốn chi phí: Những gì
> bạn không sử dụng, bạn không phải trả giá. Và hơn nữa: Những gì bạn sử dụng,
> bạn không thể tự viết mã tốt hơn được.

Trong nhiều trường hợp, mã Rust sử dụng iterator được biên dịch thành cùng
assembly mà bạn sẽ viết bằng tay. Các tối ưu hóa như mở rộng vòng lặp và loại bỏ
kiểm tra giới hạn trên truy cập mảng được áp dụng và làm cho mã kết quả cực kỳ
hiệu quả. Bây giờ bạn đã biết điều này, bạn có thể sử dụng iterator và closure
mà không sợ hãi! Chúng làm cho mã có vẻ như ở cấp độ cao hơn nhưng không áp đặt
hình phạt hiệu suất thời gian chạy khi làm như vậy.

## Tóm tắt

Closure và iterator là các tính năng của Rust lấy cảm hứng từ ý tưởng ngôn ngữ
lập trình hàm. Chúng đóng góp vào khả năng của Rust trong việc biểu đạt rõ ràng
các ý tưởng cấp cao với hiệu suất cấp thấp. Các triển khai của closure và
iterator là như vậy để hiệu suất thời gian chạy không bị ảnh hưởng. Đây là một
phần trong mục tiêu của Rust nhằm cung cấp các trừu tượng không tốn chi phí.

Bây giờ chúng ta đã cải thiện khả năng biểu đạt của dự án I/O của mình, hãy xem
xét một số tính năng khác của `cargo` sẽ giúp chúng ta chia sẻ dự án với thế
giới.
