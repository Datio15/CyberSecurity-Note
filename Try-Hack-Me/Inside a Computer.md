
Quá trình khởi động máy tính (Booting up)

Quá trình này được chia thành 5 bước tuần tự, giống như cách cơ thể người thức dậy và bắt đầu hoạt động vào buổi sáng. Dưới đây là giải thích chi tiết cho từng bước dựa trên tài liệu "Try Hack Me: Inside a Computer".

Bước 1: Press the Power Button (Nhấn nút nguồn)

Khi bạn nhấn nút nguồn, một tín hiệu điện sẽ được gửi đến PSU để cho phép dòng điện chạy qua hệ thống.

Quá trình này giống như việc cơ thể được bơm máu và nhận oxy khi vừa thức dậy.

Giải nghĩa thuật ngữ: PSU (Power Supply Unit) là Bộ nguồn máy tính, bộ phận chuyển đổi dòng điện xoay chiều (AC) từ lưới điện thành dòng điện một chiều (DC) để cung cấp điện năng phù hợp cho các linh kiện trong máy.

Bước 2: Firmware starts (Khởi động Firmware)

Lúc này các linh kiện cốt lõi đã chạy nhưng hệ thống vẫn chưa có "ý thức" (chưa nạp hệ điều hành).

Máy tính sử dụng một phần mềm cơ sở có sẵn để khởi động các linh kiện, được quản lý bởi một hệ thống trung tâm gọi là UEFI.

Giải nghĩa thuật ngữ:

Firmware: Phần mềm hệ thống được lập trình cố định vào trong phần cứng, làm cầu nối giúp phần cứng giao tiếp với phần mềm.

UEFI (Unified Extensible Firmware Interface): Giao diện phần mềm cơ sở mở rộng hợp nhất. Đây là chuẩn giao diện phần mềm hiện đại dùng để quản lý quá trình khởi động của máy tính.

BIOS (Basic Input/Output System): Hệ thống đầu vào/đầu ra cơ bản. BIOS có chức năng tương tự như UEFI nhưng là chuẩn công nghệ cũ và hiện nay chủ yếu đã bị thay thế bởi UEFI.

Bước 3: POST - Power-On Self Test (Tự kiểm tra khi bật nguồn)

UEFI sẽ chạy một chu trình gọi là POST để kiểm tra xem tất cả các linh kiện thiết yếu có mặt đầy đủ, được cấu hình đúng và đang hoạt động bình thường hay không.

Nếu có bất kỳ thành phần nào bị lỗi (ví dụ: RAM lỏng, quạt CPU không quay), hệ thống sẽ phát ra các tín hiệu cảnh báo (alarm signals) như tiếng bíp dài/ngắn hoặc báo lỗi thẳng trên màn hình.

Bước 4: Select Boot Device (Chọn thiết bị khởi động)

Khi phần cứng đã vượt qua bài kiểm tra và sẵn sàng, hệ thống sẽ tìm kiếm vị trí chứa chu trình khởi động của Hệ điều hành.

UEFI lưu trữ sẵn một danh sách thứ tự ưu tiên (ordered list) để quyết định xem sẽ tìm kiếm chu trình khởi động trên thiết bị lưu trữ nào trước tiên (ví dụ: ưu tiên đọc ổ SSD trước, nếu không có mới tìm đến USB hoặc ở đĩa quang).

Giải nghĩa thuật ngữ: Boot Device là bất kỳ phần cứng lưu trữ nào (SSD, HDD, USB) chứa các tệp tin cần thiết để nạp hệ điều hành.

Bước 5: Initiate Bootloader (Khởi chạy Bootloader)

Trên thiết bị khởi động đã được chọn ở Bước 4, hệ thống sẽ kích hoạt "chu trình nạp" (load routine) gọi là bootloader.

Bootloader có nhiệm vụ chuyển Hệ điều hành (OS) từ thiết bị lưu trữ đó vào thẳng RAM.

Sau khi quá trình truyền tải hoàn tất, UEFI sẽ chính thức trao lại toàn quyền kiểm soát các linh kiện phần cứng cho OS để bạn có một giao diện làm việc hoàn chỉnh.

Giải nghĩa thuật ngữ:

Bootloader: Một đoạn chương trình nhỏ khởi chạy trước tiên để nạp hệ điều hành (Ví dụ: GRUB trên các hệ thống Linux, hoặc Windows Boot Manager trên Windows).

OS (Operating System): Hệ điều hành máy tính.

RAM (Random Access Memory): Bộ nhớ truy cập ngẫu nhiên. Đây là nơi lưu trữ dữ liệu tạm thời với tốc độ siêu nhanh để hệ thống trích xuất và xử lý. Hết điện (tắt máy), dữ liệu trên RAM sẽ mất đi.
