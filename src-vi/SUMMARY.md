# Ngôn ngữ lập trình Rust

[Ngôn ngữ lập trình Rust](title-page.md)
[Lời nói đầu](foreword.md)
[Giới thiệu](ch00-00-introduction.md)

## Bắt đầu

- [Bắt đầu](ch01-00-getting-started.md)
  - [Cài đặt](ch01-01-installation.md)
  - [Xin chào, Thế giới!](ch01-02-hello-world.md)
  - [Xin chào, Cargo!](ch01-03-hello-cargo.md)

- [Lập trình trò chơi đoán số](ch02-00-guessing-game-tutorial.md)

- [Các khái niệm lập trình chung](ch03-00-common-programming-concepts.md)
  - [Biến và Tính khả biến](ch03-01-variables-and-mutability.md)
  - [Kiểu dữ liệu](ch03-02-data-types.md)
  - [Hàm](ch03-03-how-functions-work.md)
  - [Bình luận](ch03-04-comments.md)
  - [Luồng điều khiển](ch03-05-control-flow.md)

- [Hiểu về Quyền sở hữu](ch04-00-understanding-ownership.md)
  - [Quyền sở hữu là gì?](ch04-01-what-is-ownership.md)
  - [Tham chiếu và Mượn](ch04-02-references-and-borrowing.md)
  - [Kiểu Slice](ch04-03-slices.md)

- [Sử dụng Structs để cấu trúc dữ liệu liên quan](ch05-00-structs.md)
  - [Định nghĩa và Khởi tạo Structs](ch05-01-defining-structs.md)
  - [Một chương trình mẫu sử dụng Structs](ch05-02-example-structs.md)
  - [Cú pháp phương thức](ch05-03-method-syntax.md)

- [Enums và Khớp mẫu](ch06-00-enums.md)
  - [Định nghĩa Enum](ch06-01-defining-an-enum.md)
  - [Cấu trúc luồng điều khiển `match`](ch06-02-match.md)
  - [Luồng điều khiển ngắn gọn với `if let` và `let else`](ch06-03-if-let.md)

## Kiến thức cơ bản về Rust

- [Quản lý dự án đang phát triển với Packages, Crates và Modules](ch07-00-managing-growing-projects-with-packages-crates-and-modules.md)
  - [Packages và Crates](ch07-01-packages-and-crates.md)
  - [Định nghĩa Modules để kiểm soát phạm vi và quyền truy cập](ch07-02-defining-modules-to-control-scope-and-privacy.md)
  - [Đường dẫn để tham chiếu đến các phần tử trong cây Module](ch07-03-paths-for-referring-to-an-item-in-the-module-tree.md)
  - [Đưa các đường dẫn vào phạm vi với từ khóa `use`](ch07-04-bringing-paths-into-scope-with-the-use-keyword.md)
  - [Tách các Module thành các tệp khác nhau](ch07-05-separating-modules-into-different-files.md)

- [Các Collection thông dụng](ch08-00-common-collections.md)
  - [Lưu trữ danh sách giá trị với Vectors](ch08-01-vectors.md)
  - [Lưu trữ văn bản mã hóa UTF-8 với Strings](ch08-02-strings.md)
  - [Lưu trữ khóa với giá trị liên kết trong Hash Maps](ch08-03-hash-maps.md)

- [Xử lý lỗi](ch09-00-error-handling.md)
  - [Các lỗi không thể khôi phục với `panic!`](ch09-01-unrecoverable-errors-with-panic.md)
  - [Các lỗi có thể khôi phục với `Result`](ch09-02-recoverable-errors-with-result.md)
  - [Nên hay không nên dùng `panic!`](ch09-03-to-panic-or-not-to-panic.md)

- [Kiểu Generic, Traits và Lifetimes](ch10-00-generics.md)
  - [Kiểu dữ liệu Generic](ch10-01-syntax.md)
  - [Traits: Định nghĩa hành vi dùng chung](ch10-02-traits.md)
  - [Xác thực tham chiếu với Lifetimes](ch10-03-lifetime-syntax.md)

- [Viết kiểm thử tự động](ch11-00-testing.md)
  - [Cách viết kiểm thử](ch11-01-writing-tests.md)
  - [Kiểm soát cách chạy kiểm thử](ch11-02-running-tests.md)
  - [Tổ chức kiểm thử](ch11-03-test-organization.md)

- [Dự án I/O: Xây dựng chương trình dòng lệnh](ch12-00-an-io-project.md)
  - [Chấp nhận đối số từ dòng lệnh](ch12-01-accepting-command-line-arguments.md)
  - [Đọc tệp](ch12-02-reading-a-file.md)
  - [Tái cấu trúc để tăng tính module hóa và xử lý lỗi](ch12-03-improving-error-handling-and-modularity.md)
  - [Phát triển chức năng thư viện với Phát triển theo hướng kiểm thử](ch12-04-testing-the-librarys-functionality.md)
  - [Làm việc với biến môi trường](ch12-05-working-with-environment-variables.md)
  - [Ghi thông báo lỗi vào Standard Error thay vì Standard Output](ch12-06-writing-to-stderr-instead-of-stdout.md)

## Tư duy trong Rust

- [Các đặc điểm của ngôn ngữ hàm: Iterators và Closures](ch13-00-functional-features.md)
  - [Closures: Hàm ẩn danh có khả năng bắt giữ môi trường](ch13-01-closures.md)
  - [Xử lý một chuỗi các phần tử với Iterators](ch13-02-iterators.md)
  - [Cải thiện dự án I/O của chúng ta](ch13-03-improving-our-io-project.md)
  - [So sánh hiệu suất: Vòng lặp và Iterators](ch13-04-performance.md)

- [Tìm hiểu thêm về Cargo và Crates.io](ch14-00-more-about-cargo.md)
  - [Tùy chỉnh bản dựng với Release Profiles](ch14-01-release-profiles.md)
  - [Xuất bản một Crate lên Crates.io](ch14-02-publishing-to-crates-io.md)
  - [Cargo Workspaces](ch14-03-cargo-workspaces.md)
  - [Cài đặt các tệp nhị phân từ Crates.io với `cargo install`](ch14-04-installing-binaries.md)
  - [Mở rộng Cargo với các lệnh tùy chỉnh](ch14-05-extending-cargo.md)

- [Con trỏ thông minh](ch15-00-smart-pointers.md)
  - [Sử dụng `Box<T>` để trỏ đến dữ liệu trên Heap](ch15-01-box.md)
  - [Coi con trỏ thông minh như tham chiếu thông thường với `Deref`](ch15-02-deref.md)
  - [Chạy mã khi dọn dẹp với Trait `Drop`](ch15-03-drop.md)
  - [`Rc<T>`, con trỏ thông minh đếm tham chiếu](ch15-04-rc.md)
  - [`RefCell<T>` và mẫu tính khả biến nội tại](ch15-05-interior-mutability.md)
  - [Các vòng tham chiếu có thể gây rò rỉ bộ nhớ](ch15-06-reference-cycles.md)

- [Concurrency không sợ hãi](ch16-00-concurrency.md)
  - [Sử dụng Threads để chạy mã đồng thời](ch16-01-threads.md)
  - [Sử dụng Message Passing để truyền dữ liệu giữa các Threads](ch16-02-message-passing.md)
  - [Concurrency với trạng thái được chia sẻ](ch16-03-shared-state.md)
  - [Concurrency có thể mở rộng với các Trait `Send` và `Sync`](ch16-04-extensible-concurrency-sync-and-send.md)

- [Cơ bản về lập trình bất đồng bộ: Async, Await, Futures và Streams](ch17-00-async-await.md)
  - [Futures và cú pháp Async](ch17-01-futures-and-syntax.md)
  - [Áp dụng Concurrency với Async](ch17-02-concurrency-with-async.md)
  - [Làm việc với bất kỳ số lượng Futures nào](ch17-03-more-futures.md)
  - [Streams: Futures theo chuỗi](ch17-04-streams.md)
  - [Xem xét kỹ hơn về các Trait cho Async](ch17-05-traits-for-async.md)
  - [Futures, Tasks và Threads](ch17-06-futures-tasks-threads.md)

- [Các đặc điểm của lập trình hướng đối tượng trong Rust](ch18-00-oop.md)
  - [Các tính chất của ngôn ngữ hướng đối tượng](ch18-01-what-is-oo.md)
  - [Sử dụng Trait Objects cho phép chứa các giá trị của các kiểu khác nhau](ch18-02-trait-objects.md)
  - [Triển khai một mẫu thiết kế hướng đối tượng](ch18-03-oo-design-patterns.md)

## Chủ đề nâng cao

- [Mẫu và Khớp mẫu](ch19-00-patterns.md)
  - [Tất cả các nơi mà mẫu có thể được sử dụng](ch19-01-all-the-places-for-patterns.md)
  - [Tính bác bỏ: Liệu một mẫu có thể không khớp](ch19-02-refutability.md)
  - [Cú pháp mẫu](ch19-03-pattern-syntax.md)

- [Tính năng nâng cao](ch20-00-advanced-features.md)
  - [Rust không an toàn](ch20-01-unsafe-rust.md)
  - [Traits nâng cao](ch20-02-advanced-traits.md)
  - [Kiểu nâng cao](ch20-03-advanced-types.md)
  - [Hàm và Closures nâng cao](ch20-04-advanced-functions-and-closures.md)
  - [Macros](ch20-05-macros.md)

- [Dự án cuối cùng: Xây dựng một Web Server đa luồng](ch21-00-final-project-a-web-server.md)
  - [Xây dựng một Web Server đơn luồng](ch21-01-single-threaded.md)
  - [Chuyển Server đơn luồng thành Server đa luồng](ch21-02-multithreaded.md)
  - [Tắt và dọn dẹp một cách an toàn](ch21-03-graceful-shutdown-and-cleanup.md)

- [Phụ lục](appendix-00.md)
  - [A - Các từ khóa](appendix-01-keywords.md)
  - [B - Các toán tử và ký hiệu](appendix-02-operators.md)
  - [C - Các Trait có thể dẫn xuất](appendix-03-derivable-traits.md)
  - [D - Các công cụ phát triển hữu ích](appendix-04-useful-development-tools.md)
  - [E - Các phiên bản](appendix-05-editions.md)
  - [F - Các bản dịch của cuốn sách](appendix-06-translation.md)
  - [G - Cách Rust được tạo ra và "Nightly Rust"](appendix-07-nightly-rust.md)
