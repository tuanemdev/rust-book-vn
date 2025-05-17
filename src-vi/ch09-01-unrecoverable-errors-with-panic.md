## Lỗi Không Thể Khôi Phục với `panic!`

Đôi khi những điều không mong muốn xảy ra trong code của bạn, và bạn không thể
làm gì để ngăn chặn nó. Trong những trường hợp này, Rust có macro `panic!`. Có
hai cách để gây ra panic trong thực tế: thực hiện một hành động khiến code của
chúng ta panic (ví dụ như truy cập vượt quá giới hạn của mảng) hoặc gọi trực
tiếp macro `panic!`. Trong cả hai trường hợp, chúng ta gây ra panic trong chương
trình. Theo mặc định, những panic này sẽ hiển thị thông báo lỗi, unwind (tháo
gỡ), dọn dẹp stack, và thoát. Thông qua một biến môi trường, bạn cũng có thể yêu
cầu Rust hiển thị call stack khi xảy ra panic để dễ dàng theo dõi nguồn gốc của
panic.

> ### Tháo Gỡ Stack hoặc Hủy Bỏ Khi Xảy Ra Panic
>
> Theo mặc định, khi panic xảy ra, chương trình bắt đầu _tháo gỡ (unwinding)_,
> có nghĩa là Rust đi ngược lại stack và dọn dẹp dữ liệu từ mỗi hàm mà nó gặp
> phải. Tuy nhiên, việc đi ngược lại và dọn dẹp tốn rất nhiều công sức. Vì vậy,
> Rust cho phép bạn chọn phương án thay thế là _hủy bỏ (aborting)_ ngay lập tức,
> điều này kết thúc chương trình mà không dọn dẹp gì cả.
>
> Bộ nhớ mà chương trình đang sử dụng sau đó sẽ cần được dọn dẹp bởi hệ điều
> hành. Nếu trong dự án của bạn cần làm cho tệp nhị phân kết quả nhỏ nhất có
> thể, bạn có thể chuyển từ tháo gỡ sang hủy bỏ khi panic bằng cách thêm
> `panic = 'abort'` vào các phần `[profile]` thích hợp trong tệp _Cargo.toml_.
> Ví dụ, nếu bạn muốn hủy bỏ khi panic trong chế độ phát hành (release mode),
> hãy thêm điều này:
>
> ```toml
> [profile.release]
> panic = 'abort'
> ```

Hãy thử gọi `panic!` trong một chương trình đơn giản:

<Listing file-name="src/main.rs">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/no-listing-01-panic/src/main.rs}}
```

</Listing>

Khi bạn chạy chương trình, bạn sẽ thấy điều gì đó như thế này:

```console
{{#include ../listings/ch09-error-handling/no-listing-01-panic/output.txt}}
```

Lệnh gọi `panic!` gây ra thông báo lỗi trong hai dòng cuối cùng. Dòng đầu tiên
hiển thị thông báo panic của chúng ta và vị trí trong mã nguồn nơi panic xảy ra:
_src/main.rs:2:5_ chỉ ra rằng đó là dòng thứ hai, ký tự thứ năm trong tệp
_src/main.rs_ của chúng ta.

Trong trường hợp này, dòng được chỉ ra là một phần của code chúng ta, và nếu
chúng ta đến dòng đó, chúng ta sẽ thấy lời gọi macro `panic!`. Trong các trường
hợp khác, lời gọi `panic!` có thể nằm trong code mà code của chúng ta gọi, và
tên tệp và số dòng được báo cáo bởi thông báo lỗi sẽ là code của người khác nơi
macro `panic!` được gọi, không phải dòng code của chúng ta cuối cùng dẫn đến lời
gọi `panic!`.

<!-- Old heading. Do not remove or links may break. -->

<a id="using-a-panic-backtrace"></a>

Chúng ta có thể sử dụng backtrace của các hàm từ lời gọi `panic!` để tìm ra phần
code của chúng ta đang gây ra vấn đề. Để hiểu cách sử dụng backtrace của
`panic!`, hãy xem xét một ví dụ khác và xem nó như thế nào khi lời gọi `panic!`
đến từ một thư viện do lỗi trong code của chúng ta thay vì từ code của chúng ta
gọi trực tiếp macro. Listing 9-1 có một số code cố gắng truy cập một chỉ mục
trong vector vượt quá phạm vi các chỉ mục hợp lệ.

<Listing number="9-1" file-name="src/main.rs" caption="Cố gắng truy cập một phần tử vượt quá giới hạn của vector, điều này sẽ gây ra lời gọi đến `panic!`">

```rust,should_panic,panics
{{#rustdoc_include ../listings/ch09-error-handling/listing-09-01/src/main.rs}}
```

</Listing>

Ở đây, chúng ta đang cố gắng truy cập phần tử thứ 100 của vector (ở chỉ mục 99
vì chỉ mục bắt đầu từ 0), nhưng vector chỉ có ba phần tử. Trong trường hợp này,
Rust sẽ panic. Sử dụng `[]` được cho là để trả về một phần tử, nhưng nếu bạn
truyền một chỉ mục không hợp lệ, không có phần tử nào mà Rust có thể trả về ở
đây một cách chính xác.

Trong C, việc cố gắng đọc vượt quá giới hạn của một cấu trúc dữ liệu là hành vi
không xác định. Bạn có thể nhận được bất cứ thứ gì ở vị trí trong bộ nhớ mà sẽ
tương ứng với phần tử đó trong cấu trúc dữ liệu, mặc dù bộ nhớ không thuộc về
cấu trúc đó. Điều này được gọi là _buffer overread_ (đọc quá bộ đệm) và có thể
dẫn đến lỗ hổng bảo mật nếu kẻ tấn công có thể thao túng chỉ mục theo cách để
đọc dữ liệu mà họ không được phép truy cập, được lưu trữ sau cấu trúc dữ liệu.

Để bảo vệ chương trình của bạn khỏi loại lỗ hổng này, nếu bạn cố gắng đọc một
phần tử ở chỉ mục không tồn tại, Rust sẽ dừng thực thi và từ chối tiếp tục. Hãy
thử và xem:

```console
{{#include ../listings/ch09-error-handling/listing-09-01/output.txt}}
```

Lỗi này chỉ đến dòng 4 của tệp _main.rs_ nơi chúng ta cố gắng truy cập chỉ mục
`99` của vector trong `v`.

Dòng `note:` cho chúng ta biết rằng chúng ta có thể thiết lập biến môi trường
`RUST_BACKTRACE` để nhận backtrace chính xác về những gì đã xảy ra để gây ra
lỗi. Một _backtrace_ là danh sách tất cả các hàm đã được gọi để đến thời điểm
này. Backtraces trong Rust hoạt động giống như trong các ngôn ngữ khác: chìa
khóa để đọc backtrace là bắt đầu từ đầu và đọc cho đến khi bạn thấy các tệp bạn
đã viết. Đó là nơi vấn đề bắt nguồn. Các dòng phía trên đó là code mà code của
bạn đã gọi; các dòng bên dưới là code đã gọi code của bạn. Các dòng trước và sau
này có thể bao gồm code Rust cốt lõi, code thư viện chuẩn, hoặc các crate mà bạn
đang sử dụng. Hãy thử lấy backtrace bằng cách thiết lập biến môi trường
`RUST_BACKTRACE` thành bất kỳ giá trị nào ngoại trừ `0`. Listing 9-2 hiển thị
đầu ra tương tự như những gì bạn sẽ thấy.

<!-- manual-regeneration
cd listings/ch09-error-handling/listing-09-01
RUST_BACKTRACE=1 cargo run
copy the backtrace output below
check the backtrace number mentioned in the text below the listing
-->

<Listing number="9-2" caption="Backtrace được tạo ra bởi lời gọi đến `panic!` hiển thị khi biến môi trường `RUST_BACKTRACE` được thiết lập">

```console
$ RUST_BACKTRACE=1 cargo run
thread 'main' panicked at src/main.rs:4:6:
index out of bounds: the len is 3 but the index is 99
stack backtrace:
   0: rust_begin_unwind
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/std/src/panicking.rs:692:5
   1: core::panicking::panic_fmt
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:75:14
   2: core::panicking::panic_bounds_check
             at /rustc/4d91de4e48198da2e33413efdcd9cd2cc0c46688/library/core/src/panicking.rs:273:5
   3: <usize as core::slice::index::SliceIndex<[T]>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:274:10
   4: core::slice::index::<impl core::ops::index::Index<I> for [T]>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/slice/index.rs:16:9
   5: <alloc::vec::Vec<T,A> as core::ops::index::Index<I>>::index
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/alloc/src/vec/mod.rs:3361:9
   6: panic::main
             at ./src/main.rs:4:6
   7: core::ops::function::FnOnce::call_once
             at file:///home/.rustup/toolchains/1.85/lib/rustlib/src/rust/library/core/src/ops/function.rs:250:5
note: Some details are omitted, run with `RUST_BACKTRACE=full` for a verbose backtrace.
```

</Listing>

Đó là rất nhiều thông tin! Đầu ra chính xác bạn thấy có thể khác tùy thuộc vào
hệ điều hành và phiên bản Rust của bạn. Để nhận được backtrace với thông tin
này, các ký hiệu gỡ lỗi phải được bật. Các ký hiệu gỡ lỗi được bật theo mặc định
khi sử dụng `cargo build` hoặc `cargo run` mà không có cờ `--release`, như chúng
ta đang làm ở đây.

Trong đầu ra ở Listing 9-2, dòng 6 của backtrace chỉ đến dòng trong dự án chúng
ta đang gây ra vấn đề: dòng 4 của _src/main.rs_. Nếu chúng ta không muốn chương
trình của chúng ta panic, chúng ta nên bắt đầu điều tra ở vị trí được chỉ ra bởi
dòng đầu tiên đề cập đến tệp chúng ta đã viết. Trong Listing 9-1, nơi chúng ta
cố ý viết code sẽ panic, cách để sửa panic là không yêu cầu một phần tử vượt quá
phạm vi chỉ mục của vector. Khi code của bạn panic trong tương lai, bạn sẽ cần
phải tìm ra hành động nào mà code đang thực hiện với những giá trị nào để gây ra
panic và code nên làm gì thay thế.

Chúng ta sẽ quay lại `panic!` và khi nào chúng ta nên và không nên sử dụng
`panic!` để xử lý các điều kiện lỗi trong phần ["To `panic!` or Not to
`panic!`"][to-panic-or-not-to-panic]<!-- ignore --> sau này trong chương này.
Tiếp theo, chúng ta sẽ xem xét cách khôi phục từ lỗi sử dụng `Result`.

[to-panic-or-not-to-panic]:
  ch09-03-to-panic-or-not-to-panic.html#to-panic-or-not-to-panic
