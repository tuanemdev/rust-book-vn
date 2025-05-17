## Phụ lục G - Cách Rust Được Tạo Ra và "Rust Nightly"

Phụ lục này nói về cách Rust được tạo ra và điều đó ảnh hưởng như thế nào đến
bạn với tư cách là một nhà phát triển Rust.

### Sự Ổn Định Không Đình Trệ

Là một ngôn ngữ, Rust quan tâm _rất nhiều_ về tính ổn định của mã của bạn. Chúng
tôi muốn Rust trở thành nền tảng vững chắc mà bạn có thể xây dựng trên đó, và
nếu mọi thứ liên tục thay đổi, điều đó sẽ là không thể. Đồng thời, nếu chúng ta
không thể thử nghiệm với các tính năng mới, chúng ta có thể không phát hiện ra
những lỗi quan trọng cho đến sau khi phát hành, khi chúng ta không thể thay đổi
mọi thứ nữa.

Giải pháp của chúng tôi cho vấn đề này là điều chúng tôi gọi là "sự ổn định
không đình trệ", và nguyên tắc hướng dẫn của chúng tôi là: bạn không bao giờ
phải lo sợ việc nâng cấp lên phiên bản mới của Rust ổn định. Mỗi lần nâng cấp
nên diễn ra suôn sẻ, nhưng cũng nên mang đến cho bạn các tính năng mới, ít lỗi
hơn và thời gian biên dịch nhanh hơn.

### Choo, Choo! Các Kênh Phát Hành và Đi Theo Các Đoàn Tàu

Sự phát triển của Rust hoạt động theo một _lịch trình đoàn tàu_. Nghĩa là, tất
cả sự phát triển được thực hiện trong nhánh main của kho lưu trữ Rust. Các phiên
bản phát hành tuân theo mô hình đoàn tàu phát hành phần mềm, đã được sử dụng bởi
Cisco IOS và các dự án phần mềm khác. Có ba _kênh phát hành_ cho Rust:

- Nightly
- Beta
- Stable

Hầu hết các nhà phát triển Rust chủ yếu sử dụng kênh ổn định, nhưng những người
muốn thử các tính năng mới có thể sử dụng nightly hoặc beta.

Dưới đây là một ví dụ về cách quá trình phát triển và phát hành hoạt động: giả
sử rằng nhóm Rust đang làm việc trên phiên bản Rust 1.5. Phiên bản đó đã được
phát hành vào tháng 12 năm 2015, nhưng nó sẽ cung cấp cho chúng ta các số phiên
bản thực tế. Một tính năng mới được thêm vào Rust: một commit mới được đưa vào
nhánh main. Mỗi đêm, một phiên bản nightly mới của Rust được tạo ra. Mỗi ngày là
một ngày phát hành, và các phiên bản này được tạo ra bởi cơ sở hạ tầng phát hành
của chúng tôi một cách tự động. Vì vậy, theo thời gian, các phiên bản của chúng
tôi trông như thế này, mỗi đêm một lần:

```text
nightly: * - - * - - *
```

Cứ mỗi sáu tuần, đã đến lúc chuẩn bị một phiên bản mới! Nhánh `beta` của kho lưu
trữ Rust tách ra từ nhánh main được sử dụng bởi nightly. Bây giờ, có hai phiên
bản:

```text
nightly: * - - * - - *
                     |
beta:                *
```

Hầu hết người dùng Rust không sử dụng các phiên bản beta một cách tích cực,
nhưng kiểm tra chống lại beta trong hệ thống CI của họ để giúp Rust phát hiện
các lỗi có thể xảy ra. Trong khi đó, vẫn có một phiên bản nightly mỗi đêm:

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

Giả sử một lỗi được phát hiện. Thật may mắn là chúng tôi đã có thời gian để kiểm
tra phiên bản beta trước khi lỗi này lẻn vào phiên bản ổn định! Bản sửa lỗi được
áp dụng cho nhánh main, để nightly được sửa, và sau đó bản sửa lỗi được chuyển
lại cho nhánh `beta`, và một phiên bản beta mới được tạo ra:

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

Sáu tuần sau khi phiên bản beta đầu tiên được tạo ra, đã đến lúc phát hành phiên
bản ổn định! Nhánh `stable` được tạo ra từ nhánh `beta`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

Hoan hô! Rust 1.5 đã hoàn thành! Tuy nhiên, chúng tôi đã quên một điều: vì sáu
tuần đã trôi qua, chúng tôi cũng cần một phiên bản beta mới của _phiên bản tiếp
theo_ của Rust, 1.6. Vì vậy, sau khi `stable` tách ra từ `beta`, phiên bản tiếp
theo của `beta` lại tách ra từ `nightly`:

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

Điều này được gọi là "mô hình đoàn tàu" vì cứ mỗi sáu tuần, một phiên bản "rời
khỏi nhà ga", nhưng vẫn phải trải qua một hành trình qua kênh beta trước khi nó
đến như một phiên bản ổn định.

Rust phát hành mỗi sáu tuần, như đồng hồ. Nếu bạn biết ngày của một phiên bản
Rust, bạn có thể biết ngày của phiên bản tiếp theo: nó là sáu tuần sau đó. Một
khía cạnh hay của việc có các phiên bản được lên lịch mỗi sáu tuần là đoàn tàu
tiếp theo đang đến sớm. Nếu một tính năng tình cờ bỏ lỡ một phiên bản cụ thể,
không cần phải lo lắng: một phiên bản khác đang diễn ra trong thời gian ngắn!
Điều này giúp giảm áp lực để lén lút các tính năng có thể chưa hoàn thiện vào
gần thời hạn phát hành.

Nhờ quá trình này, bạn luôn có thể kiểm tra bản dựng tiếp theo của Rust và tự
mình xác minh rằng việc nâng cấp là dễ dàng: nếu một phiên bản beta không hoạt
động như mong đợi, bạn có thể báo cáo cho nhóm và sửa lỗi trước khi phiên bản ổn
định tiếp theo diễn ra! Sự cố trong một phiên bản beta là tương đối hiếm, nhưng
`rustc` vẫn là một phần mềm, và lỗi vẫn tồn tại.

### Thời gian bảo trì

Dự án Rust hỗ trợ phiên bản ổn định mới nhất. Khi một phiên bản ổn định mới được
phát hành, phiên bản cũ sẽ đạt đến cuối vòng đời (EOL). Điều này có nghĩa là mỗi
phiên bản được hỗ trợ trong sáu tuần.

### Các Tính Năng Không Ổn Định

Có một điều nữa với mô hình phát hành này: các tính năng không ổn định. Rust sử
dụng một kỹ thuật gọi là "cờ tính năng" để xác định các tính năng nào được kích
hoạt trong một phiên bản nhất định. Nếu một tính năng mới đang được phát triển
tích cực, nó sẽ được đưa vào nhánh main, và do đó, trong nightly, nhưng đằng sau
một _cờ tính năng_. Nếu bạn, với tư cách là người dùng, muốn thử tính năng đang
trong quá trình phát triển, bạn có thể, nhưng bạn phải sử dụng phiên bản nightly
của Rust và chú thích mã nguồn của bạn với cờ thích hợp để chọn tham gia.

Nếu bạn đang sử dụng phiên bản beta hoặc ổn định của Rust, bạn không thể sử dụng
bất kỳ cờ tính năng nào. Đây là chìa khóa cho phép chúng tôi sử dụng thực tế với
các tính năng mới trước khi chúng tôi tuyên bố chúng ổn định mãi mãi. Những
người muốn tham gia vào cạnh tiên tiến có thể làm như vậy, và những người muốn
có trải nghiệm vững chắc có thể gắn bó với ổn định và biết rằng mã của họ sẽ
không bị hỏng. Sự ổn định không đình trệ.

Cuốn sách này chỉ chứa thông tin về các tính năng ổn định, vì các tính năng đang
trong quá trình phát triển vẫn đang thay đổi, và chắc chắn chúng sẽ khác nhau
giữa khi cuốn sách này được viết và khi chúng được kích hoạt trong các bản dựng
ổn định. Bạn có thể tìm thấy tài liệu cho các tính năng chỉ có trong nightly
trực tuyến.

### Rustup và Vai Trò của Rust Nightly

Rustup giúp dễ dàng chuyển đổi giữa các kênh phát hành khác nhau của Rust, trên
toàn cầu hoặc theo dự án. Theo mặc định, bạn sẽ có Rust ổn định được cài đặt. Để
cài đặt nightly, ví dụ:

```console
$ rustup toolchain install nightly
```

Bạn cũng có thể xem tất cả các _toolchains_ (các phiên bản của Rust và các thành
phần liên quan) mà bạn đã cài đặt với `rustup`. Dưới đây là một ví dụ trên máy
tính Windows của một trong những tác giả của bạn:

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

Như bạn có thể thấy, toolchain ổn định là mặc định. Hầu hết người dùng Rust sử
dụng ổn định hầu hết thời gian. Bạn có thể muốn sử dụng ổn định hầu hết thời
gian, nhưng sử dụng nightly trên một dự án cụ thể, vì bạn quan tâm đến một tính
năng tiên tiến. Để làm điều đó, bạn có thể sử dụng `rustup override` trong thư
mục của dự án đó để đặt toolchain nightly là toolchain mà `rustup` nên sử dụng
khi bạn ở trong thư mục đó:

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

Bây giờ, mỗi khi bạn gọi `rustc` hoặc `cargo` bên trong
_~/projects/needs-nightly_, `rustup` sẽ đảm bảo rằng bạn đang sử dụng Rust
nightly, thay vì mặc định của bạn là Rust ổn định. Điều này rất hữu ích khi bạn
có nhiều dự án Rust!

### Quy Trình RFC và Các Nhóm

Vậy làm thế nào để bạn biết về những tính năng mới này? Mô hình phát triển của
Rust tuân theo một _Quy Trình Yêu Cầu Bình Luận (RFC)_. Nếu bạn muốn cải thiện
Rust, bạn có thể viết một đề xuất, gọi là RFC.

Bất kỳ ai cũng có thể viết RFC để cải thiện Rust, và các đề xuất được xem xét và
thảo luận bởi nhóm Rust, bao gồm nhiều nhóm chủ đề. Có một danh sách đầy đủ các
nhóm [trên trang web của Rust](https://www.rust-lang.org/governance), bao gồm
các nhóm cho mỗi lĩnh vực của dự án: thiết kế ngôn ngữ, triển khai trình biên
dịch, cơ sở hạ tầng, tài liệu và nhiều hơn nữa. Nhóm thích hợp đọc đề xuất và
các bình luận, viết một số bình luận của riêng họ, và cuối cùng, có sự đồng
thuận để chấp nhận hoặc từ chối tính năng.

Nếu tính năng được chấp nhận, một vấn đề được mở trên kho lưu trữ Rust, và ai đó
có thể triển khai nó. Người triển khai nó rất có thể không phải là người đề xuất
tính năng ban đầu! Khi việc triển khai đã sẵn sàng, nó sẽ được đưa vào nhánh
main đằng sau một cờ tính năng, như chúng tôi đã thảo luận trong phần
[“Các Tính Năng Không Ổn Định”](#unstable-features)<!-- ignore -->.

Sau một thời gian, khi các nhà phát triển Rust sử dụng các phiên bản nightly đã
có thể thử tính năng mới, các thành viên nhóm sẽ thảo luận về tính năng, cách nó
hoạt động trên nightly, và quyết định xem nó có nên được đưa vào Rust ổn định
hay không. Nếu quyết định là tiến lên, cờ tính năng sẽ được gỡ bỏ, và tính năng
này bây giờ được coi là ổn định! Nó sẽ đi theo các đoàn tàu vào một phiên bản ổn
định mới của Rust.
