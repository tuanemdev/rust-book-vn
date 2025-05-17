## Không gian làm việc Cargo

Trong Chương 12, chúng ta đã xây dựng một gói bao gồm một crate nhị phân và một
crate thư viện. Khi dự án của bạn phát triển, bạn có thể thấy rằng crate thư
viện tiếp tục trở nên lớn hơn và bạn muốn chia gói của mình thành nhiều crate
thư viện hơn. Cargo cung cấp một tính năng gọi là _workspaces_ (không gian làm
việc) có thể giúp quản lý nhiều gói liên quan được phát triển cùng nhau.

### Tạo một Workspace

Một _workspace_ là một tập hợp các gói chia sẻ cùng một _Cargo.lock_ và thư mục
đầu ra. Hãy tạo một dự án sử dụng workspace—chúng ta sẽ sử dụng mã đơn giản để
có thể tập trung vào cấu trúc của workspace. Có nhiều cách để cấu trúc một
workspace, vì vậy chúng ta sẽ chỉ trình bày một cách phổ biến. Chúng ta sẽ có
một workspace chứa một crate nhị phân và hai crate thư viện. Crate nhị phân,
cung cấp chức năng chính, sẽ phụ thuộc vào hai thư viện. Một thư viện sẽ cung
cấp hàm `add_one` và thư viện khác cung cấp hàm `add_two`. Ba crate này sẽ là
một phần của cùng một workspace. Chúng ta sẽ bắt đầu bằng cách tạo một thư mục
mới cho workspace:

```console
$ mkdir add
$ cd add
```

Tiếp theo, trong thư mục _add_, chúng ta tạo tệp _Cargo.toml_ sẽ cấu hình toàn
bộ workspace. Tệp này sẽ không có phần `[package]`. Thay vào đó, nó sẽ bắt đầu
với phần `[workspace]` cho phép chúng ta thêm thành viên vào workspace. Chúng ta
cũng lưu ý sử dụng phiên bản mới nhất và tốt nhất của thuật toán resolver của
Cargo trong workspace của chúng ta bằng cách đặt giá trị `resolver` thành `"3"`.

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-01-workspace/add/Cargo.toml}}
```

Tiếp theo, chúng ta sẽ tạo crate nhị phân `adder` bằng cách chạy `cargo new`
trong thư mục _add_:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-01-adder-crate/add
remove `members = ["adder"]` from Cargo.toml
rm -rf adder
cargo new adder
copy output below
-->

```console
$ cargo new adder
     Created binary (application) `adder` package
      Adding `adder` as member of workspace at `file:///projects/add`
```

Chạy `cargo new` bên trong một workspace cũng tự động thêm gói mới tạo vào khóa
`members` trong định nghĩa `[workspace]` trong _Cargo.toml_ của workspace, như
thế này:

```toml
{{#include ../listings/ch14-more-about-cargo/output-only-01-adder-crate/add/Cargo.toml}}
```

Tại thời điểm này, chúng ta có thể xây dựng workspace bằng cách chạy
`cargo build`. Các tệp trong thư mục _add_ của bạn sẽ trông như thế này:

```text
├── Cargo.lock
├── Cargo.toml
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Workspace có một thư mục _target_ ở cấp cao nhất nơi các sản phẩm biên dịch sẽ
được đặt vào; gói `adder` không có thư mục _target_ riêng. Ngay cả khi chúng ta
chạy `cargo build` từ bên trong thư mục _adder_, các sản phẩm biên dịch vẫn sẽ
kết thúc trong _add/target_ thay vì _add/adder/target_. Cargo cấu trúc thư mục
_target_ trong một workspace như thế này vì các crate trong một workspace được
thiết kế để phụ thuộc vào nhau. Nếu mỗi crate có thư mục _target_ riêng, mỗi
crate sẽ phải biên dịch lại từng crate khác trong workspace để đặt các sản phẩm
vào thư mục _target_ riêng của nó. Bằng cách chia sẻ một thư mục _target_, các
crate có thể tránh việc xây dựng lại không cần thiết.

### Tạo Gói Thứ Hai trong Workspace

Tiếp theo, hãy tạo một gói thành viên khác trong workspace và gọi nó là
`add_one`. Tạo một crate thư viện mới có tên `add_one`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-02-add-one/add
remove `"add_one"` from `members` list in Cargo.toml
rm -rf add_one
cargo new add_one --lib
copy output below
-->

```console
$ cargo new add_one --lib
     Created library `add_one` package
      Adding `add_one` as member of workspace at `file:///projects/add`
```

_Cargo.toml_ cấp cao nhất sẽ bao gồm đường dẫn _add_one_ trong danh sách
`members`:

<span class="filename">Tên tệp: Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/Cargo.toml}}
```

Thư mục _add_ của bạn bây giờ sẽ có các thư mục và tệp này:

```text
├── Cargo.lock
├── Cargo.toml
├── add_one
│   ├── Cargo.toml
│   └── src
│       └── lib.rs
├── adder
│   ├── Cargo.toml
│   └── src
│       └── main.rs
└── target
```

Trong tệp _add_one/src/lib.rs_, hãy thêm một hàm `add_one`:

<span class="filename">Tên tệp: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/add_one/src/lib.rs}}
```

Bây giờ chúng ta có thể để gói `adder` với chương trình nhị phân của chúng ta
phụ thuộc vào gói `add_one` chứa thư viện của chúng ta. Trước tiên, chúng ta sẽ
cần thêm một phụ thuộc đường dẫn vào `add_one` trong _adder/Cargo.toml_.

<span class="filename">Tên tệp: adder/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-02-workspace-with-two-crates/add/adder/Cargo.toml:6:7}}
```

Cargo không giả định rằng các crate trong một workspace sẽ phụ thuộc vào nhau,
vì vậy chúng ta cần chỉ định rõ ràng mối quan hệ phụ thuộc.

Tiếp theo, hãy sử dụng hàm `add_one` (từ crate `add_one`) trong crate `adder`.
Mở tệp _adder/src/main.rs_ và thay đổi hàm `main` để gọi hàm `add_one`, như
trong Listing 14-7.

<Listing number="14-7" file-name="adder/src/main.rs" caption="Sử dụng crate thư viện `add_one` từ crate `adder`">

```rust,ignore
{{#rustdoc_include ../listings/ch14-more-about-cargo/listing-14-07/add/adder/src/main.rs}}
```

</Listing>

Hãy xây dựng workspace bằng cách chạy `cargo build` trong thư mục _add_ cấp cao
nhất!

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.22s
```

Để chạy crate nhị phân từ thư mục _add_, chúng ta có thể chỉ định gói nào trong
workspace chúng ta muốn chạy bằng cách sử dụng đối số `-p` và tên gói với
`cargo run`:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/listing-14-07/add
cargo run -p adder
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo run -p adder
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running `target/debug/adder`
Hello, world! 10 plus one is 11!
```

Điều này chạy mã trong _adder/src/main.rs_, mã này phụ thuộc vào crate
`add_one`.

#### Phụ thuộc vào một Gói Bên ngoài trong một Workspace

Lưu ý rằng workspace chỉ có một tệp _Cargo.lock_ ở cấp cao nhất, thay vì có
_Cargo.lock_ trong thư mục của mỗi crate. Điều này đảm bảo rằng tất cả các crate
đều sử dụng cùng phiên bản của tất cả các phụ thuộc. Nếu chúng ta thêm gói
`rand` vào tệp _adder/Cargo.toml_ và _add_one/Cargo.toml_, Cargo sẽ giải quyết
cả hai gói này thành một phiên bản của `rand` và ghi lại trong _Cargo.lock_ duy
nhất. Việc làm cho tất cả các crate trong workspace sử dụng cùng các phụ thuộc
có nghĩa là các crate sẽ luôn tương thích với nhau. Hãy thêm crate `rand` vào
phần `[dependencies]` trong tệp _add_one/Cargo.toml_ để chúng ta có thể sử dụng
crate `rand` trong crate `add_one`:

<!-- When updating the version of `rand` used, also update the version of
`rand` used in these files so they all match:
* ch02-00-guessing-game-tutorial.md
* ch07-04-bringing-paths-into-scope-with-the-use-keyword.md
-->

<span class="filename">Tên tệp: add_one/Cargo.toml</span>

```toml
{{#include ../listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add/add_one/Cargo.toml:6:7}}
```

Bây giờ chúng ta có thể thêm `use rand;` vào tệp _add_one/src/lib.rs_, và việc
xây dựng toàn bộ workspace bằng cách chạy `cargo build` trong thư mục _add_ sẽ
mang `rand` crate vào và biên dịch nó. Chúng ta sẽ nhận được một cảnh báo vì
chúng ta không tham chiếu đến `rand` mà chúng ta đã đưa vào phạm vi:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-03-workspace-with-external-dependency/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
    Updating crates.io index
  Downloaded rand v0.8.5
   --snip--
   Compiling rand v0.8.5
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
warning: unused import: `rand`
 --> add_one/src/lib.rs:1:5
  |
1 | use rand;
  |     ^^^^
  |
  = note: `#[warn(unused_imports)]` on by default

warning: `add_one` (lib) generated 1 warning (run `cargo fix --lib -p add_one` to apply 1 suggestion)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `dev` profile [unoptimized + debuginfo] target(s) in 0.95s
```

_Cargo.lock_ cấp cao nhất bây giờ chứa thông tin về phụ thuộc của `add_one` vào
`rand`. Tuy nhiên, mặc dù `rand` được sử dụng ở đâu đó trong workspace, chúng ta
không thể sử dụng nó trong các crate khác trong workspace trừ khi chúng ta thêm
`rand` vào tệp _Cargo.toml_ của họ. Ví dụ: nếu chúng ta thêm `use rand;` vào tệp
_adder/src/main.rs_ cho gói `adder`, chúng ta sẽ nhận được lỗi:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/output-only-03-use-rand/add
cargo build
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo build
  --snip--
   Compiling adder v0.1.0 (file:///projects/add/adder)
error[E0432]: unresolved import `rand`
 --> adder/src/main.rs:2:5
  |
2 | use rand;
  |     ^^^^ no external crate `rand`
```

Để sửa điều này, hãy chỉnh sửa tệp _Cargo.toml_ cho gói `adder` và chỉ ra rằng
`rand` cũng là một phụ thuộc cho nó. Việc xây dựng gói `adder` sẽ thêm `rand`
vào danh sách phụ thuộc cho `adder` trong _Cargo.lock_, nhưng không có bản sao
bổ sung nào của `rand` sẽ được tải xuống. Cargo sẽ đảm bảo rằng mọi crate trong
mọi gói trong workspace sử dụng gói `rand` sẽ sử dụng cùng một phiên bản miễn là
họ chỉ định các phiên bản tương thích của `rand`, tiết kiệm không gian cho chúng
ta và đảm bảo rằng các crate trong workspace sẽ tương thích với nhau.

Nếu các crate trong workspace chỉ định các phiên bản không tương thích của cùng
một phụ thuộc, Cargo sẽ giải quyết từng crate, nhưng vẫn cố gắng giải quyết càng
ít phiên bản càng tốt.

#### Thêm một Bài Kiểm thử vào Workspace

Để một cải tiến khác, hãy thêm một bài kiểm thử cho hàm `add_one::add_one` trong
crate `add_one`:

<span class="filename">Tên tệp: add_one/src/lib.rs</span>

```rust,noplayground
{{#rustdoc_include ../listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add/add_one/src/lib.rs}}
```

Bây giờ chạy `cargo test` trong thư mục _add_ cấp cao nhất. Chạy `cargo test`
trong một workspace có cấu trúc như thế này sẽ chạy các bài kiểm thử cho tất cả
các crate trong workspace:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test
copy output below; the output updating script doesn't handle subdirectories in
paths properly
-->

```console
$ cargo test
   Compiling add_one v0.1.0 (file:///projects/add/add_one)
   Compiling adder v0.1.0 (file:///projects/add/adder)
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.20s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

     Running unittests src/main.rs (target/debug/deps/adder-3a47283c568d2b6a)

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Phần đầu tiên của đầu ra cho thấy bài kiểm thử `it_works` trong crate `add_one`
đã thành công. Phần tiếp theo cho thấy không tìm thấy bài kiểm thử nào trong
crate `adder`, và sau đó phần cuối cùng cho thấy không tìm thấy bài kiểm thử tài
liệu nào trong crate `add_one`.

Chúng ta cũng có thể chạy các bài kiểm thử cho một crate cụ thể trong workspace
từ thư mục cấp cao nhất bằng cách sử dụng cờ `-p` và chỉ định tên của crate mà
chúng ta muốn kiểm thử:

<!-- manual-regeneration
cd listings/ch14-more-about-cargo/no-listing-04-workspace-with-tests/add
cargo test -p add_one
copy output below; the output updating script doesn't handle subdirectories in paths properly
-->

```console
$ cargo test -p add_one
    Finished `test` profile [unoptimized + debuginfo] target(s) in 0.00s
     Running unittests src/lib.rs (target/debug/deps/add_one-93c49ee75dc46543)

running 1 test
test tests::it_works ... ok

test result: ok. 1 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s

   Doc-tests add_one

running 0 tests

test result: ok. 0 passed; 0 failed; 0 ignored; 0 measured; 0 filtered out; finished in 0.00s
```

Đầu ra này hiển thị `cargo test` chỉ chạy các bài kiểm thử cho crate `add_one`
và không chạy các bài kiểm thử crate `adder`.

Nếu bạn xuất bản các crate trong workspace lên
[crates.io](https://crates.io/)<!-- ignore -->, mỗi crate trong workspace sẽ cần
được xuất bản riêng. Giống như `cargo test`, chúng ta có thể xuất bản một crate
cụ thể trong workspace của chúng ta bằng cách sử dụng cờ `-p` và chỉ định tên
của crate mà chúng ta muốn xuất bản.

Để thực hành thêm, hãy thêm một crate `add_two` vào workspace này theo cách
tương tự như crate `add_one`!

Khi dự án của bạn phát triển, hãy cân nhắc sử dụng một workspace: nó cho phép
bạn làm việc với các thành phần nhỏ hơn, dễ hiểu hơn so với một đống mã lớn. Hơn
nữa, việc giữ các crate trong một workspace có thể làm cho việc phối hợp giữa
các crate dễ dàng hơn nếu chúng thường xuyên thay đổi cùng một lúc.
