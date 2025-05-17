# Các Kiểu Generic, Traits và Lifetimes

Mọi ngôn ngữ lập trình đều có công cụ để xử lý hiệu quả sự trùng lặp của các
khái niệm. Trong Rust, một công cụ như vậy là _generics_: các đại diện trừu
tượng cho các kiểu dữ liệu cụ thể hoặc các thuộc tính khác. Chúng ta có thể biểu
đạt hành vi của generics hoặc cách chúng liên quan đến các generics khác mà
không cần biết cái gì sẽ ở vị trí của chúng khi biên dịch và chạy mã.

Các hàm có thể nhận tham số của một số kiểu generic, thay vì một kiểu cụ thể như
`i32` hoặc `String`, theo cách tương tự như chúng nhận tham số với giá trị không
xác định để chạy cùng một mã trên nhiều giá trị cụ thể. Thực tế, chúng ta đã sử
dụng generics trong Chương 6 với `Option<T>`, trong Chương 8 với `Vec<T>` và
`HashMap<K, V>`, và trong Chương 9 với `Result<T, E>`. Trong chương này, bạn sẽ
khám phá cách định nghĩa các kiểu, hàm và phương thức của riêng mình với
generics!

Đầu tiên chúng ta sẽ xem xét cách trích xuất một hàm để giảm sự trùng lặp mã.
Sau đó chúng ta sẽ sử dụng cùng một kỹ thuật để tạo một hàm generic từ hai hàm
mà chỉ khác nhau về kiểu của các tham số của chúng. Chúng ta cũng sẽ giải thích
cách sử dụng các kiểu generic trong định nghĩa struct và enum.

Sau đó, bạn sẽ học cách sử dụng _traits_ để định nghĩa hành vi theo cách
generic. Bạn có thể kết hợp traits với các kiểu generic để ràng buộc một kiểu
generic chỉ chấp nhận những kiểu có một hành vi cụ thể, trái ngược với việc chấp
nhận bất kỳ kiểu nào.

Cuối cùng, chúng ta sẽ thảo luận về _lifetimes_: một loại generics cung cấp cho
trình biên dịch thông tin về cách các tham chiếu liên quan đến nhau. Lifetimes
cho phép chúng ta cung cấp cho trình biên dịch đủ thông tin về các giá trị được
mượn để nó có thể đảm bảo các tham chiếu sẽ hợp lệ trong nhiều tình huống hơn so
với khi không có sự trợ giúp của chúng ta.

## Loại bỏ sự trùng lặp bằng cách trích xuất một hàm

Generics cho phép chúng ta thay thế các kiểu cụ thể bằng một giữ chỗ đại diện
cho nhiều kiểu để loại bỏ sự trùng lặp mã. Trước khi đi sâu vào cú pháp
generics, hãy xem trước cách loại bỏ sự trùng lặp theo cách không liên quan đến
các kiểu generic bằng cách trích xuất một hàm thay thế các giá trị cụ thể bằng
một giữ chỗ đại diện cho nhiều giá trị. Sau đó chúng ta sẽ áp dụng cùng một kỹ
thuật để trích xuất một hàm generic! Bằng cách xem xét cách nhận biết mã trùng
lặp mà bạn có thể trích xuất vào một hàm, bạn sẽ bắt đầu nhận ra mã trùng lặp có
thể sử dụng generics.

Chúng ta sẽ bắt đầu với một chương trình ngắn trong Listing 10-1 tìm số lớn nhất
trong một danh sách.

<Listing number="10-1" file-name="src/main.rs" caption="Tìm số lớn nhất trong danh sách các số">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-01/src/main.rs:here}}
```

</Listing>

Chúng ta lưu trữ một danh sách các số nguyên trong biến `number_list` và đặt một
tham chiếu tới số đầu tiên trong danh sách vào một biến có tên `largest`. Sau đó
chúng ta lặp qua tất cả các số trong danh sách, và nếu số hiện tại lớn hơn số
được lưu trữ trong `largest`, chúng ta thay thế tham chiếu trong biến đó. Tuy
nhiên, nếu số hiện tại nhỏ hơn hoặc bằng số lớn nhất đã thấy cho đến nay, biến
không thay đổi, và mã chuyển sang số tiếp theo trong danh sách. Sau khi xem xét
tất cả các số trong danh sách, `largest` sẽ tham chiếu đến số lớn nhất, trong
trường hợp này là 100.

Giờ đây chúng ta được giao nhiệm vụ tìm số lớn nhất trong hai danh sách khác
nhau của các số. Để làm điều đó, chúng ta có thể chọn sao chép mã trong Listing
10-1 và sử dụng cùng một logic tại hai vị trí khác nhau trong chương trình, như
được hiển thị trong Listing 10-2.

<Listing number="10-2" file-name="src/main.rs" caption="Mã để tìm số lớn nhất trong *hai* danh sách số">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-02/src/main.rs}}
```

</Listing>

Mặc dù mã này hoạt động, việc sao chép mã là tẻ nhạt và dễ gây lỗi. Chúng ta
cũng phải nhớ cập nhật mã ở nhiều vị trí khi chúng ta muốn thay đổi nó.

Để loại bỏ sự trùng lặp này, chúng ta sẽ tạo một sự trừu tượng bằng cách định
nghĩa một hàm hoạt động trên bất kỳ danh sách số nguyên nào được truyền vào dưới
dạng tham số. Giải pháp này làm cho mã của chúng ta rõ ràng hơn và cho phép
chúng ta biểu đạt khái niệm tìm số lớn nhất trong một danh sách một cách trừu
tượng.

Trong Listing 10-3, chúng ta trích xuất mã tìm số lớn nhất vào một hàm có tên là
`largest`. Sau đó chúng ta gọi hàm để tìm số lớn nhất trong hai danh sách từ
Listing 10-2. Chúng ta cũng có thể sử dụng hàm này trên bất kỳ danh sách giá trị
`i32` nào khác mà chúng ta có thể có trong tương lai.

<Listing number="10-3" file-name="src/main.rs" caption="Mã trừu tượng để tìm số lớn nhất trong hai danh sách">

```rust
{{#rustdoc_include ../listings/ch10-generic-types-traits-and-lifetimes/listing-10-03/src/main.rs:here}}
```

</Listing>

Hàm `largest` có một tham số gọi là `list`, đại diện cho bất kỳ slice cụ thể nào
của các giá trị `i32` mà chúng ta có thể truyền vào hàm. Kết quả là, khi chúng
ta gọi hàm, mã chạy trên các giá trị cụ thể mà chúng ta truyền vào.

Tóm lại, đây là các bước chúng ta đã thực hiện để thay đổi mã từ Listing 10-2
sang Listing 10-3:

1. Xác định mã trùng lặp.
2. Trích xuất mã trùng lặp vào thân hàm, và chỉ định các đầu vào và giá trị trả
   về của mã đó trong chữ ký hàm.
3. Cập nhật hai trường hợp của mã trùng lặp để gọi hàm thay thế.

Tiếp theo, chúng ta sẽ sử dụng các bước tương tự với generics để giảm sự trùng
lặp mã. Cùng cách mà thân hàm có thể hoạt động trên một `list` trừu tượng thay
vì các giá trị cụ thể, generics cho phép mã hoạt động trên các kiểu trừu tượng.

Ví dụ, giả sử chúng ta có hai hàm: một hàm tìm phần tử lớn nhất trong một slice
của các giá trị `i32` và một hàm tìm phần tử lớn nhất trong một slice của các
giá trị `char`. Làm thế nào để chúng ta loại bỏ sự trùng lặp đó? Hãy tìm hiểu!
