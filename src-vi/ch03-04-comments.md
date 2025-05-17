## Chú thích

Tất cả các lập trình viên đều cố gắng làm cho mã của họ dễ hiểu, nhưng đôi khi
cần có những giải thích thêm. Trong những trường hợp này, lập trình viên để lại
_chú thích (comments)_ trong mã nguồn mà trình biên dịch sẽ bỏ qua nhưng người
đọc mã nguồn có thể thấy hữu ích.

Đây là một chú thích đơn giản:

```rust
// xin chào, thế giới
```

Trong Rust, phong cách chú thích thông dụng bắt đầu bằng hai dấu gạch chéo, và
chú thích tiếp tục đến hết dòng. Đối với các chú thích kéo dài hơn một dòng, bạn
cần phải thêm `//` ở mỗi dòng, như thế này:

```rust
// Chúng ta đang làm một điều gì đó phức tạp ở đây, đủ dài để chúng ta cần
// nhiều dòng chú thích để làm điều đó! Ồ! Hy vọng rằng, chú thích này sẽ
// giải thích những gì đang diễn ra.
```

Chú thích cũng có thể được đặt ở cuối dòng chứa mã:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-24-comments-end-of-line/src/main.rs}}
```

Nhưng bạn sẽ thường thấy chúng được sử dụng trong định dạng này, với chú thích
trên một dòng riêng phía trên mã mà nó đang chú thích:

<span class="filename">Tên tệp: src/main.rs</span>

```rust
{{#rustdoc_include ../listings/ch03-common-programming-concepts/no-listing-25-comments-above-line/src/main.rs}}
```

Rust cũng có một loại chú thích khác, chú thích tài liệu, mà chúng ta sẽ thảo
luận trong phần ["Xuất bản một Crate lên Crates.io"][publishing]<!-- ignore -->
của Chương 14.

[publishing]: ch14-02-publishing-to-crates-io.html
