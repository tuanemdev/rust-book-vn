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
hiệu suất ở đây, bởi vì điểm quan trọng không phải để chứng minh rằng hai phiên
bản là tương đương mà để có cảm nhận chung về cách hai triển khai này so sánh về
mặt hiệu suất.

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

Một ví dụ khác, đoạn mã sau được lấy từ một bộ giải mã âm thanh. Thuật toán giải
mã sử dụng phép toán dự đoán tuyến tính để ước tính các giá trị tương lai dựa
trên một hàm tuyến tính của các mẫu trước đó. Mã này sử dụng một chuỗi iterator
để thực hiện một số phép toán trên ba biến trong phạm vi: một slice dữ liệu
`buffer`, một mảng 12 `coefficients`, và một lượng để dịch chuyển dữ liệu trong
`qlp_shift`. Chúng tôi đã khai báo các biến trong ví dụ này nhưng không gán giá
trị cho chúng; mặc dù mã này không có nhiều ý nghĩa ngoài ngữ cảnh của nó, nó
vẫn là một ví dụ ngắn gọn, thực tế về cách Rust chuyển đổi ý tưởng cấp cao thành
mã cấp thấp.

```rust,ignore
let buffer: &mut [i32];
let coefficients: [i64; 12];
let qlp_shift: i16;

for i in 12..buffer.len() {
    let prediction = coefficients.iter()
                                 .zip(&buffer[i - 12..i])
                                 .map(|(&c, &s)| c * s as i64)
                                 .sum::<i64>() >> qlp_shift;
    let delta = buffer[i];
    buffer[i] = prediction as i32 + delta;
}
```

Để tính toán giá trị của `prediction`, mã này lặp qua mỗi trong 12 giá trị trong
`coefficients` và sử dụng phương thức `zip` để ghép cặp các giá trị hệ số với 12
giá trị trước đó trong `buffer`. Sau đó, với mỗi cặp, chúng ta nhân các giá trị
với nhau, tổng hợp tất cả các kết quả, và dịch chuyển các bit trong tổng
`qlp_shift` bit sang phải.

Các phép tính trong ứng dụng như bộ giải mã âm thanh thường ưu tiên hiệu suất
cao nhất. Ở đây, chúng ta đang tạo một iterator, sử dụng hai bộ điều hợp, và sau
đó tiêu thụ giá trị. Mã assembly nào mà mã Rust này sẽ được biên dịch thành? Vào
thời điểm viết bài này, nó được biên dịch thành cùng mã assembly mà bạn sẽ viết
bằng tay. Không có vòng lặp nào tương ứng với việc lặp qua các giá trị trong
`coefficients`: Rust biết rằng có 12 lần lặp, vì vậy nó "mở rộng" vòng lặp. _Mở
rộng vòng lặp_ là một tối ưu hóa loại bỏ chi phí của mã điều khiển vòng lặp và
thay vào đó tạo ra mã lặp lại cho mỗi lần lặp của vòng lặp.

Tất cả các hệ số được lưu trữ trong các thanh ghi, điều này có nghĩa là việc
truy cập các giá trị rất nhanh. Không có kiểm tra giới hạn nào trên truy cập
mảng trong thời gian chạy. Tất cả các tối ưu hóa mà Rust có thể áp dụng làm cho
mã kết quả cực kỳ hiệu quả. Bây giờ bạn đã biết điều này, bạn có thể sử dụng
iterator và closure mà không sợ hãi! Chúng làm cho mã có vẻ như ở cấp độ cao hơn
nhưng không áp đặt hình phạt hiệu suất thời gian chạy khi làm như vậy.

## Tóm tắt

Closure và iterator là các tính năng của Rust lấy cảm hứng từ ý tưởng ngôn ngữ
lập trình hàm. Chúng đóng góp vào khả năng của Rust trong việc biểu đạt rõ ràng
các ý tưởng cấp cao với hiệu suất cấp thấp. Các triển khai của closure và
iterator là như vậy để hiệu suất thời gian chạy không bị ảnh hưởng. Đây là một
phần trong mục tiêu của Rust nhằm cung cấp các trừu tượng không tốn chi phí.

Bây giờ chúng ta đã cải thiện khả năng biểu đạt của dự án I/O của mình, hãy xem
xét một số tính năng khác của `cargo` sẽ giúp chúng ta chia sẻ dự án với thế
giới.
