## Tùy chỉnh Quá trình Xây dựng với Hồ sơ Phát hành

Trong Rust, _hồ sơ phát hành_ (release profiles) là các hồ sơ được định nghĩa
trước và có thể tùy chỉnh với các cấu hình khác nhau cho phép lập trình viên có
thêm quyền kiểm soát đối với các tùy chọn biên dịch mã. Mỗi hồ sơ được cấu hình
độc lập với các hồ sơ khác.

Cargo có hai hồ sơ chính: hồ sơ `dev` mà Cargo sử dụng khi bạn chạy
`cargo build`, và hồ sơ `release` mà Cargo sử dụng khi bạn chạy
`cargo build --release`. Hồ sơ `dev` được định nghĩa với các giá trị mặc định
tốt cho quá trình phát triển, và hồ sơ `release` có các giá trị mặc định tốt cho
các bản dựng phát hành.

Những tên hồ sơ này có thể quen thuộc từ kết quả đầu ra của quá trình xây dựng
của bạn:

<!-- manual-regeneration
anywhere, run:
cargo build
cargo build --release
and ensure output below is accurate
-->

```console
$ cargo build
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
$ cargo build --release
    Finished `release` profile [optimized] target(s) in 0.32s
```

`dev` và `release` là các hồ sơ khác nhau được sử dụng bởi trình biên dịch.

Cargo có các cài đặt mặc định cho mỗi hồ sơ, áp dụng khi bạn chưa thêm rõ ràng
bất kỳ phần `[profile.*]` nào vào tệp _Cargo.toml_ của dự án. Bằng cách thêm các
phần `[profile.*]` cho bất kỳ hồ sơ nào bạn muốn tùy chỉnh, bạn ghi đè lên bất
kỳ tập hợp con nào của các cài đặt mặc định. Ví dụ, đây là các giá trị mặc định
cho cài đặt `opt-level` của các hồ sơ `dev` và `release`:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 0

[profile.release]
opt-level = 3
```

Cài đặt `opt-level` kiểm soát số lượng tối ưu hóa mà Rust sẽ áp dụng cho mã của
bạn, với phạm vi từ 0 đến 3. Áp dụng nhiều tối ưu hóa hơn sẽ kéo dài thời gian
biên dịch, vì vậy nếu bạn đang trong quá trình phát triển và thường xuyên biên
dịch mã của mình, bạn sẽ muốn ít tối ưu hóa hơn để biên dịch nhanh hơn, ngay cả
khi mã thu được chạy chậm hơn. Do đó, `opt-level` mặc định cho `dev` là `0`. Khi
bạn đã sẵn sàng phát hành mã của mình, tốt nhất là dành nhiều thời gian hơn cho
việc biên dịch. Bạn sẽ chỉ biên dịch ở chế độ phát hành một lần, nhưng bạn sẽ
chạy chương trình đã biên dịch nhiều lần, vì vậy chế độ phát hành đánh đổi thời
gian biên dịch lâu hơn lấy mã chạy nhanh hơn. Đó là lý do tại sao `opt-level`
mặc định cho hồ sơ `release` là `3`.

Bạn có thể ghi đè cài đặt mặc định bằng cách thêm một giá trị khác vào
_Cargo.toml_. Ví dụ, nếu chúng ta muốn sử dụng mức tối ưu hóa 1 trong hồ sơ phát
triển, chúng ta có thể thêm hai dòng này vào tệp _Cargo.toml_ của dự án:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
[profile.dev]
opt-level = 1
```

Mã này ghi đè lên cài đặt mặc định là `0`. Bây giờ khi chúng ta chạy
`cargo build`, Cargo sẽ sử dụng các giá trị mặc định cho hồ sơ `dev` cộng với
tùy chỉnh của chúng ta cho `opt-level`. Vì chúng ta đặt `opt-level` thành `1`,
Cargo sẽ áp dụng nhiều tối ưu hóa hơn so với mặc định, nhưng không nhiều như
trong bản dựng phát hành.

Để biết danh sách đầy đủ các tùy chọn cấu hình và giá trị mặc định cho mỗi hồ
sơ, hãy xem
[tài liệu của Cargo](https://doc.rust-lang.org/cargo/reference/profiles.html).
