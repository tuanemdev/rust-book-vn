# Các Khái Niệm Lập Trình Phổ Biến

Chương này bao gồm các khái niệm xuất hiện trong hầu hết mọi ngôn ngữ lập trình
và cách chúng hoạt động trong Rust. Nhiều ngôn ngữ lập trình có nhiều điểm chung
ở cốt lõi. Không có khái niệm nào được trình bày trong chương này là độc quyền
của Rust, nhưng chúng ta sẽ thảo luận về chúng trong bối cảnh của Rust và giải
thích các quy ước xung quanh việc sử dụng các chúng.

Cụ thể, bạn sẽ học về biến, các kiểu dữ liệu cơ bản, hàm, bình luận, và luồng
điều khiển. Những nền tảng này sẽ có trong mọi chương trình Rust, và việc học
chúng sớm sẽ giúp bạn có một nền tảng vững chắc để bắt đầu.

> #### Từ khóa
>
> Ngôn ngữ Rust có một tập hợp các _từ khóa_ được dành riêng để sử dụng bởi ngôn
> ngữ này, tương tự như trong các ngôn ngữ khác. Hãy lưu ý rằng bạn không thể sử
> dụng những từ này làm tên của biến hoặc hàm. Hầu hết các từ khóa đều có ý
> nghĩa đặc biệt, và bạn sẽ sử dụng chúng để thực hiện các nhiệm vụ khác nhau
> trong chương trình Rust của mình; một số ít không có chức năng cụ thể nào liên
> quan đến chúng nhưng đã được dành riêng cho các chức năng có thể được thêm vào
> Rust trong tương lai. Bạn có thể tìm thấy danh sách các từ khóa trong [Phụ lục
> A][appendix_a]<!-- ignore -->.

[appendix_a]: appendix-01-keywords.md
