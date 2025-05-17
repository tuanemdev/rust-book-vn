# Enums và Pattern Matching

Trong chương này, chúng ta sẽ tìm hiểu về _enumerations_, hay còn được gọi là
_enums_. Enums cho phép bạn định nghĩa một kiểu bằng cách liệt kê các _biến thể_
(variants) có thể có của nó. Đầu tiên chúng ta sẽ định nghĩa và sử dụng một enum
để chỉ ra cách một enum có thể mã hóa ý nghĩa cùng với dữ liệu. Tiếp theo, chúng
ta sẽ khám phá một enum đặc biệt hữu ích là `Option`, nó biểu thị rằng một giá
trị có thể là một cái gì đó hoặc không là gì cả. Sau đó, chúng ta sẽ xem xét
cách mà pattern matching trong biểu thức `match` giúp dễ dàng chạy các đoạn mã
khác nhau cho các giá trị khác nhau của một enum. Cuối cùng, chúng ta sẽ tìm
hiểu cách cấu trúc `if let` là một thành ngữ khác tiện lợi và ngắn gọn có sẵn để
xử lý enums trong mã của bạn.
