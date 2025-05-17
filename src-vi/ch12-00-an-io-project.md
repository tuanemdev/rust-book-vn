# Dự án I/O: Xây dựng chương trình dòng lệnh

Chương này là phần tổng hợp về nhiều kỹ năng mà bạn đã học được và một khám phá
thêm một số tính năng thư viện chuẩn. Chúng ta sẽ xây dựng một công cụ dòng lệnh
tương tác với tệp và đầu vào/đầu ra dòng lệnh để thực hành các khái niệm Rust mà
bạn đã nắm vững.

Tốc độ, tính an toàn, đầu ra nhị phân đơn và hỗ trợ đa nền tảng của Rust làm cho
nó trở thành ngôn ngữ lý tưởng để tạo công cụ dòng lệnh, vì vậy cho dự án của
chúng ta, chúng ta sẽ tạo phiên bản riêng của công cụ tìm kiếm dòng lệnh cổ điển
`grep` (**g**lobally search a **r**egular **e**xpression and **p**rint - tìm
kiếm **b**iểu **t**hức **c**hính **q**uy toàn cục và **i**n). Trong trường hợp
sử dụng đơn giản nhất, `grep` tìm kiếm một chuỗi cụ thể trong một tệp cụ thể. Để
làm được điều này, `grep` nhận đường dẫn tệp và một chuỗi làm đối số. Sau đó nó
đọc tệp, tìm các dòng trong tệp đó có chứa chuỗi đối số, và in các dòng đó.

Trong quá trình này, chúng ta sẽ chỉ ra cách làm cho công cụ dòng lệnh của chúng
ta sử dụng các tính năng terminal mà nhiều công cụ dòng lệnh khác sử dụng. Chúng
ta sẽ đọc giá trị của một biến môi trường để cho phép người dùng cấu hình hành
vi của công cụ của chúng ta. Chúng ta cũng sẽ in thông báo lỗi đến luồng console
lỗi tiêu chuẩn (`stderr`) thay vì đầu ra tiêu chuẩn (`stdout`) để, ví dụ, người
dùng có thể chuyển hướng đầu ra thành công đến một tệp trong khi vẫn nhìn thấy
thông báo lỗi trên màn hình.

Một thành viên cộng đồng Rust, Andrew Gallant, đã tạo ra một phiên bản `grep`
đầy đủ tính năng, rất nhanh, gọi là `ripgrep`. So với phiên bản của anh ấy,
phiên bản của chúng ta sẽ khá đơn giản, nhưng chương này sẽ cung cấp cho bạn một
số kiến thức nền tảng cần thiết để hiểu một dự án trong thực tế như `ripgrep`.

Dự án `grep` của chúng ta sẽ kết hợp một số khái niệm mà bạn đã học được:

- Tổ chức mã ([Chương 7][ch7]<!-- ignore -->)
- Sử dụng vector và chuỗi ([Chương 8][ch8]<!-- ignore -->)
- Xử lý lỗi ([Chương 9][ch9]<!-- ignore -->)
- Sử dụng trait và lifetime khi thích hợp ([Chương 10][ch10]<!-- ignore -->)
- Viết kiểm thử ([Chương 11][ch11]<!-- ignore -->)

Chúng ta cũng sẽ giới thiệu ngắn gọn về closures, iterators và trait objects, mà
[Chương 13][ch13]<!-- ignore --> và [Chương 18][ch18]<!-- ignore --> sẽ đề cập
chi tiết.

[ch7]: ch07-00-managing-growing-projects-with-packages-crates-and-modules.html
[ch8]: ch08-00-common-collections.html
[ch9]: ch09-00-error-handling.html
[ch10]: ch10-00-generics.html
[ch11]: ch11-00-testing.html
[ch13]: ch13-00-functional-features.html
[ch18]: ch18-00-oop.html
