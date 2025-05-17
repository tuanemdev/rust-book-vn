# Hiểu về Ownership

Ownership (quyền sở hữu) là tính năng độc đáo nhất của Rust và có ảnh hưởng sâu
sắc đến phần còn lại của ngôn ngữ. Nó cho phép Rust đảm bảo an toàn bộ nhớ mà
không cần bộ thu gom rác (garbage collector), vì vậy việc hiểu cách ownership
hoạt động là rất quan trọng. Trong chương này, chúng ta sẽ nói về ownership cũng
như một số tính năng liên quan: borrowing (mượn), slices (lát cắt), và cách Rust
bố trí dữ liệu trong bộ nhớ.
