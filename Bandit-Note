# OverTheWire — Bandit Wargame Write-up

> Ghi chú cá nhân trong quá trình giải [Bandit](https://overthewire.org/wargames/bandit/) (OverTheWire). Mỗi level: **Vấn đề/Yêu cầu** → **Câu lệnh học được** → **Hướng giải quyết**.
>
> Mật khẩu các level **không** được đưa vào bài viết.

## Mục lục

- [Kiến thức nền tảng](#kiến-thức-nền-tảng)
- [Level 0](#level-0--kết-nối-ssh) · [1](#level-1--đọc-file-readme) · [2](#level-2--file-có-tên-là-dấu--) · [3](#level-3--file-có-khoảng-trắng-và-dấu-gạch-ngang) · [4](#level-4--file-ẩn-giữa-nhiều-file-binary) · [5](#level-5--ls--la-và-tìm-file-đọc-được) · [6](#level-6--find-theo-kích-thước--quyền) · [7](#level-7--find-toàn-hệ-thống)
- [Level 8](#level-8--lọc-dòng-bằng-grep) · [9](#level-9--tìm-dòng-duy-nhất-bằng-sort--uniq) · [10](#level-10--lọc-chuỗi-đọc-được-bằng-strings) · [11](#level-11--giải-mã-base64) · [12](#level-12--giải-mã-rot13-bằng-tr) · [13](#level-13--giải-nén-nhiều-tầng)
- [Level 14](#level-14--đăng-nhập-bằng-ssh-private-key) · [15](#level-15--gửi-dữ-liệu-qua-nc-netcat) · [16](#level-16--gửi-dữ-liệu-qua-ssltls-bằng-openssl) · [17](#level-17--quét-cổng-bằng-nmap) · [18](#level-18--chạy-lệnh-từ-xa-qua-ssh-so-sánh-bằng-diff) · [19](#level-19--bỏ-qua-bashrc-độc-hại)
- [Level 20](#level-20--quyền-suid) · [21](#level-21--truyền-dữ-liệu-qua-cổng-cục-bộ) · [22](#level-22--đọc-script-chạy-bởi-cron) · [23](#level-23--tái-tạo-hash-md5-trong-cron-job) · [24](#level-24--lợi-dụng-cron-job-quét-thư-mục-ghi-được) · [25](#level-25--brute-force-mã-pin-4-chữ-số)
- [Tổng hợp câu lệnh đã học](#tổng-hợp-câu-lệnh-đã-học)

> **Level 26 và các level sau** chưa được đưa vào bản này — sẽ bổ sung sau.

---

## Kiến thức nền tảng

**Cây thư mục Linux (một số thư mục quan trọng):**

| Thư mục | Vai trò |
|---|---|
| `/etc` | Tệp cấu hình hệ thống và ứng dụng |
| `/home` | Thư mục cá nhân của từng người dùng |
| `/root` | Thư mục cá nhân của người dùng quản trị (root) |
| `/mnt` | Nơi gắn (mount) tạm thời các hệ thống tệp |
| `/opt` | Phần mềm bổ sung, không thuộc hệ thống mặc định |
| `/usr` | Chương trình, thư viện, tài liệu cho người dùng |
| `/bin` | Các lệnh nhị phân cơ bản cần thiết |

**Encode vs Encrypt:**
- **Encode**: chuyển dữ liệu sang định dạng phù hợp với kênh truyền tải (mục đích *truyền tải/tương thích*, không phải bảo mật) — ví dụ Base64.
- **Encrypt**: bảo vệ dữ liệu bằng cách cản trở việc đảo ngược nếu không có khóa (mục đích *bảo mật*).

**Trạng thái cổng khi quét mạng:**

| Trạng thái | Ý nghĩa |
|---|---|
| `open` | Có dịch vụ đang lắng nghe trên cổng |
| `closed` | Không có dịch vụ lắng nghe |
| `filtered` | Không xác định được mở/đóng do bị chặn (firewall,...) |
| `unfiltered` | Chỉ xuất hiện với kiểu quét ACK, không xác định được mở/đóng |

**Biến môi trường:** các giá trị dùng để cấu hình cách chương trình chạy. Xem một biến: `echo $PATH`, `echo $SHELL`; xem tất cả: `env`.

**`ls -la` — ý nghĩa các cột:**

| Cột | Ý nghĩa |
|---|---|
| 1 | Quyền truy cập theo Owner - Group - Other (r/w/x); ký tự đầu `d` = thư mục, `-` = file thường |
| 2 | Số hardlink |
| 3 | Chủ sở hữu (owner) |
| 4 | Nhóm sở hữu (group) |
| 5 | Kích thước (bytes) |
| 6 | Ngày sửa đổi lần cuối |
| 7 | Tên file |

---

## Level 0 — Kết nối SSH

**Yêu cầu:** Kết nối tới server Bandit qua SSH.

```bash
# ssh -p <port> user@host
ssh -p 2220 bandit0@bandit.labs.overthewire.org
```

`-p` chỉ định cổng (mặc định SSH là 22, Bandit dùng 2220).

---

## Level 1 — Đọc file `readme`

**Yêu cầu:** Mật khẩu nằm trong file `readme` ở thư mục home.

```bash
ls            # xác định vị trí file readme
cat readme    # xem nội dung
```

---

## Level 2 — File có tên là dấu `-`

**Yêu cầu:** Mật khẩu nằm trong file tên chỉ gồm ký tự `-`.

**Vấn đề:** Dự đoán ban đầu là `cat -` sẽ báo lỗi (bị hiểu là option rỗng). Thực tế không lỗi: `-` là quy ước Unix chỉ **stdin**, nên `cat -` chờ đọc từ bàn phím. Kiểm chứng: `cat - > test` rồi nhập `abc` → chuỗi được ghi vào file `test`.

**Giải quyết:**
```bash
cat ./-
```
Thêm `./` để chỉ đường dẫn tương đối tới file, chương trình hiểu đây là **tên file** chứ không phải ký hiệu stdin.

---

## Level 3 — File có khoảng trắng và dấu gạch ngang

**Yêu cầu:** Đọc file tên `--spaces in this filename--`.

**Vấn đề:**
- Gõ trực tiếp → `cat` hiểu phần bắt đầu bằng `--` là option, báo lỗi không nhận diện tham số.
- Dùng gợi ý của shell (tab completion) → nhận diện được tên file nhưng khoảng trắng vẫn chưa được xử lý đúng.

**Giải quyết:**
```bash
cat "./--spaces in this filename--"
```
- `./` để tên file không bị hiểu là option.
- Dấu ngoặc kép (hoặc escape `\ `) để giữ khoảng trắng, gộp thành **một argument**.

---

## Level 4 — File ẩn giữa nhiều file binary

**Yêu cầu:** Trong thư mục `inhere`, tìm file chứa nội dung đọc được (human-readable).

**Kiến thức — file ẩn vs file thường:**

| | Normal file | Hidden file |
|---|---|---|
| Cấu trúc tên | Không bắt đầu bằng `.` | Bắt đầu bằng `.` |
| Mục đích | Dữ liệu người dùng | Cấu hình hệ thống, log,... |
| Ví dụ | `test` | `.config` |

```bash
ls -a inhere          # -a: liệt kê cả file ẩn
file inhere/*         # xác định loại từng file
cat inhere/<file có kiểu ASCII text>
```

**Mẹo:** phím **Tab** vẫn tự hoàn thành tên file kể cả file ẩn.

---

## Level 5 — `ls -la` và tìm file đọc được

**Yêu cầu:** Tìm mật khẩu trong file đọc được, nằm giữa nhiều thư mục/file.

```bash
ls -la      # best practice: xem đầy đủ quyền, owner, size, file ẩn
file ./*    # cách 2: chỉ đọc file có kiểu ASCII text
cat ./*     # cách 1: in nội dung toàn bộ file
```

**Q&A ôn tập:**
- *Làm sao chạy lệnh cho nhiều file cùng lúc?* → Dùng ký tự đại diện `*` khi tên file có chung dạng.
- *Phân biệt directory và file trong `ls -la`?* → Ký tự đầu tiên: `d` là thư mục, `-` là file thường.
- *`file` biết text hay binary bằng cách nào?* → Kiểm tra **magic byte** và nội dung file (không dựa vào phần mở rộng).

---

## Level 6 — `find` theo kích thước & quyền

**Yêu cầu:** Trong `inhere` (có nhiều thư mục con `maybehere00`–`maybehere19`), tìm file thỏa: kích thước **1033 byte**, **không** có quyền thực thi, human-readable.

**Vấn đề:** Quá nhiều thư mục lồng nhau, không thể kiểm tra thủ công.

```bash
# find <path> [option] [expression]
find . -type f -size 1033c ! -executable
```

- `-type f`: chỉ lấy file thường.
- `-size 1033c`: kích thước tính bằng byte (`c`).
- `! -executable`: loại file có quyền thực thi.

---

## Level 7 — `find` toàn hệ thống

**Yêu cầu:** Tìm file thỏa 3 điều kiện (owner, group, size) ở **bất kỳ đâu** trên hệ thống.

**Vấn đề:** `find .` chỉ quét thư mục hiện tại → kết quả rỗng (`NULL`) vì file nằm ngoài. Cần quét từ thư mục gốc `/` — đây là lý do phải hiểu cây thư mục Linux.

```bash
find / -user bandit7 -group bandit6 -size 33c 2>/dev/null
```

`2>/dev/null` bỏ các thông báo "Permission denied" để kết quả sạch.

---

## Level 8 — Lọc dòng bằng `grep`

**Yêu cầu:** Mật khẩu nằm cạnh từ `millionth` trong file `data.txt` rất lớn.

```bash
grep millionth data.txt
```

`grep <pattern> <file>` liệt kê mọi dòng chứa chuỗi `pattern`.

---

## Level 9 — Tìm dòng duy nhất bằng `sort` + `uniq`

**Yêu cầu:** Mật khẩu nằm trên dòng **chỉ xuất hiện đúng một lần**.

```bash
sort data.txt | uniq -u
```

`uniq` chỉ xử lý được các dòng trùng **liền kề**, nên phải `sort` trước.

| Tùy chọn | Ý nghĩa |
|---|---|
| `uniq -u` | Chỉ hiển thị dòng không lặp (duy nhất) |
| `uniq -c` | Đếm số lần xuất hiện của mỗi dòng |
| `uniq -d` | Chỉ hiển thị dòng bị lặp |

---

## Level 10 — Lọc chuỗi đọc được bằng `strings`

**Yêu cầu:** Mật khẩu nằm trong file binary, là chuỗi đọc được và đứng sau ký tự `=`.

```bash
strings data.txt | grep "="
```

`strings` liệt kê các chuỗi in được trong file, kể cả file nhị phân.

---

## Level 11 — Giải mã Base64

**Yêu cầu:** Mật khẩu được mã hóa Base64.

```bash
base64 -d data.txt
```

Base64 biến dữ liệu nhị phân/văn bản thành chuỗi ASCII; `-d` giải mã ngược lại. (Đây là *encode*, không phải *encrypt* — xem phần kiến thức nền.)

---

## Level 12 — Giải mã ROT13 bằng `tr`

**Yêu cầu:** Mật khẩu được mã hóa ROT13.

```bash
cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'
```

ROT13 dịch mỗi chữ cái đi 13 vị trí. Bảng chữ cái có 26 chữ nên dịch 2 lần = quay lại ký tự gốc → giải mã chính là áp dụng ROT13 lần nữa. Bảng ánh xạ cố định:

| Original | Encoded |
|---|---|
| A–M | N–Z |
| N–Z | A–M |
| a–m | n–z |
| n–z | a–m |

`tr set1 set2` ánh xạ từng ký tự của tập 1 sang ký tự tương ứng của tập 2.

---

## Level 13 — Giải nén nhiều tầng

**Yêu cầu:** `data.txt` là hexdump của một file đã bị nén nhiều lớp; cần giải nén lần lượt để lấy dữ liệu cuối.

**Hướng giải quyết:**
1. Tạo thư mục tạm (tên ngẫu nhiên, để không xung đột/không làm rối thư mục home) và copy `data.txt` vào.
2. `file data.txt` → `ASCII text`. Theo gợi ý đề bài đây là **hexdump** (byte nhị phân biểu diễn bằng ký tự hex đọc được).
3. Chuyển về nhị phân bằng `xxd -r`.
4. Dùng `file` xem **magic byte** để biết loại nén, rồi giải nén tương ứng; lặp lại đến khi ra file cuối.

```bash
mkdir "$(mktemp -d)"; cd "$(mktemp -d)"
cp ~/data.txt .
xxd -r data.txt data
file data                       # → gzip / bzip2 / POSIX tar ...
mv data data.gz && gzip -d data.gz
bzip2 -d data
tar -xf data
```

**Lưu ý:**
- `file` kiểm tra nhiều lớp: lớp magic byte để xác định định dạng, lớp kiểm tra nội dung là text hay binary.
- `gzip -d` suy ra tên file đầu ra bằng cách bỏ đuôi `.gz`/`.z`, nên file **phải có đuôi** này → cần `mv` đổi tên trước.
- `bzip2` không yêu cầu đuôi mở rộng cụ thể.
- `tar` chỉ **đóng gói** nhiều file thành một archive (không nhất thiết nén).

---

## Level 14 — Đăng nhập bằng SSH private key

**Yêu cầu:** Dùng private key để đăng nhập vào level kế tiếp thay vì mật khẩu.

```bash
ssh -i private-key -p 2220 user@hostname
```

**Cơ chế SSH gồm 3 bước:**
1. Thiết lập khóa phiên (session key): hai bên trao đổi public key tạm thời để tạo khóa mã hóa chung mà kẻ nghe lén không biết.
2. Xác thực Host.
3. Xác thực User — bằng `password` hoặc `public-key`. Server gửi danh sách phương thức chấp nhận, client thử lần lượt (server cũng có thể yêu cầu nhiều phương thức cùng lúc). Các level trước dùng password; level này yêu cầu public-key.

**Q&A — cơ chế xác thực public-key:** Dù lệnh chỉ có `-i private-key`, hệ điều hành tự trích xuất public key từ private key:
- Client gửi public key tới Host để xác nhận sự tồn tại; nếu có, Host gửi lại một **chuỗi ngẫu nhiên**.
- Client **ký** private key lên chuỗi đó và gửi lại cho Host.
- Host xác thực bằng cách kết hợp chuỗi và public key để xác thực định danh.

---

## Level 15 — Gửi dữ liệu qua `nc` (Netcat)

**Yêu cầu:** Gửi mật khẩu của level 14 (lưu tại `/etc/bandit_pass/bandit14`) tới cổng 30000.

```bash
cat /etc/bandit_pass/bandit14 | nc localhost 30000
```

`Netcat` (`nc`) là tiện ích đọc/ghi dữ liệu qua kết nối TCP/UDP.

**Cổng phổ biến cần nhớ:** `21/ftp`, `22/ssh`, `23/telnet`, `25/smtp`, `53/dns`, `80/http`, `443/https`, `445/smb`.

---

## Level 16 — Gửi dữ liệu qua SSL/TLS bằng `openssl`

**Yêu cầu:** Gửi mật khẩu tới cổng 30001 bằng giao thức SSL/TLS.

```bash
openssl s_client -connect localhost:30001
# rồi dán mật khẩu và Enter
```

`OpenSSL` là bộ công cụ mã nguồn mở dùng SSL/TLS. Sau khi bắt tay TLS thành công, server trả về `Correct!` kèm mật khẩu tiếp theo.

---

## Level 17 — Quét cổng bằng `nmap`

**Yêu cầu:** Trong dải cổng 31000-32000 trên localhost, xác định cổng đang mở và dùng dịch vụ SSL, rồi gửi mật khẩu tới đó.

```bash
nmap -sV localhost -p 31000-32000
openssl s_client -connect localhost:<port> -quiet
```

`-sV` cho biết dịch vụ/phiên bản đang chạy trên từng cổng.

**Vấn đề gặp phải:** Khi kết nối không dùng `-quiet`, `openssl` đọc dữ liệu gửi vào như **lệnh tương tác nội bộ** — vì chữ cái đầu của mật khẩu là `k`, nó bị hiểu là yêu cầu gửi lại khóa. Thêm cờ `-quiet` để tắt chế độ tương tác này, sau đó dùng private key nhận được để đăng nhập level kế.

**Q&A ôn tập:**
- *Nếu không chỉ định cổng, nmap quét gì?* → 1000 cổng phổ biến (21/ftp, 22/ssh, 23/telnet, 25/smtp, 53/dns, 80/http, 443/https, 445/smb,...).
- *Default scan của nmap:* Nếu chạy quyền root (`sudo nmap` hoặc `-sS`): gửi gói SYN, host phản hồi RST → quét nhanh, ít log. Nếu chạy quyền thường (`nmap` hoặc `-sT`): ý nghĩa ngược lại (hoàn tất bắt tay 3 bước, chậm hơn, để lại log).

---

## Level 18 — Chạy lệnh từ xa qua SSH, so sánh bằng `diff`

**Yêu cầu:** Tìm dòng duy nhất khác nhau giữa `password.new` và `password.old` — đó là mật khẩu.

```bash
diff password.old password.new
```

`diff` là lệnh so sánh sự khác biệt giữa các tệp tin hoặc thư mục dưới dạng văn bản.

---

## Level 19 — Bỏ qua `.bashrc` độc hại

**Yêu cầu:** File `.bashrc` bị thay đổi khiến chương trình dừng lại và in `Bye!` mỗi lần cố truy cập.

**Phân tích:**
- `.bashrc` được nạp mỗi khi mở một **shell tương tác không đăng nhập**.
- Khi đăng nhập máy chủ từ xa bằng SSH, đây gọi là hành động mở một **shell tương tác có đăng nhập**.
- Hành động đăng nhập gọi shell tương tác có đăng nhập nên chương trình sẽ gọi tệp `.profile`, và tệp này lại gọi `.bashrc` — vì đề bài nói chỉ tệp `.bashrc` bị thay đổi, nên đoạn code dừng chương trình nằm ở đó và chạy ngay khi đăng nhập.

**Hướng giải quyết:** Vì vậy, cách đúng là không được gọi tệp `.bashrc`, nghĩa là đồng thời không được gọi tệp `.profile`. Một cách phù hợp: thay gọi shell mặc định là bash bằng cách ép gọi một shell khác (`.sh`), với điều kiện host không chặn tùy chọn chỉ định shell khác mặc định.

```bash
ssh -p 2220 user@host /bin/sh
```

---

## Level 20 — Quyền SUID

**Yêu cầu:** Chương trình `bandit20-do` có quyền SUID thuộc user kế tiếp, cần tận dụng nó để đọc file mật khẩu mà user hiện tại không đọc trực tiếp được.

```bash
ls -l bandit20-do
./bandit20-do cat /etc/bandit_pass/bandit20
```

**Setuid** là một quyền đặc biệt, cho phép thực thi tệp với quyền của **owner**. Có thể viết lại thành hàm `bandit20-do(arg = commands)` — hàm thực thi câu lệnh với quyền thuộc về `owner = bandit20`.

**Điều kiện cần lưu ý:** cần có quyền execute trên chính file này — quan sát `ls -l` thấy quyền group cho phép execute và user hiện tại thuộc group đó.

---

## Level 21 — Truyền dữ liệu qua cổng cục bộ

**Yêu cầu:** Một chương trình (`suconnect`) mở kết nối tới một cổng cụ thể và yêu cầu gửi mật khẩu bandit20 để nhận được mật khẩu của bandit21.

```bash
# đầu 1: mở cổng lắng nghe, đẩy sẵn mật khẩu vào đầu ống truyền tải dữ liệu
echo "<password_bandit20>" | nc -l -p 1234 &
# đầu 2: khởi tạo, xác nhận kết nối để nhận dữ liệu
./suconnect 1234
```

**Dòng dữ liệu:** tiến trình `nc -l -p 1234` gửi mật khẩu bandit20 và chờ đầu còn lại kết nối; tiến trình `suconnect` được khởi tạo, xác nhận kết nối để nhận dữ liệu và trả mật khẩu bandit21 ra terminal.

---

## Level 22 — Đọc script chạy bởi Cron

**Yêu cầu:** Mật khẩu của level tiếp theo được sinh ra bởi một cron job.

```bash
cat /etc/cron.d/cronjob_bandit22       # lịch chạy và user thực thi
cat /usr/bin/cronjob_bandit22.sh        # nội dung script
cat <đường-dẫn-file-tạm-được-ghi-trong-script>
```

`Cron` là trình lập lịch, tự động chạy các lệnh hoặc script vào những thời điểm được cài đặt.

**Phân tích script:** với quyền thực thi của bandit22, script mở tệp chứa mật khẩu tương ứng và chép vào một thư mục, cấp quyền đọc cho user hiện tại. Vì thế password của bandit22 được lưu trong một tệp tạm thời — chỉ cần mở nội dung bằng lệnh `cat` là xem được mật khẩu.

**Kiến thức:**
- Tệp shell có phần mở rộng `.sh`, chứa các lệnh shell, được viết để tự động chạy; nó được **thông dịch trực tiếp** bởi bash/sh, thay vì biên dịch như các ngôn ngữ lập trình phổ biến.
- `Shebang`: khai báo chương trình (shell) có nhiệm vụ thông dịch script.
- *Không có shebang thì có chạy được không?* Shebang chỉ định interpreter để thông dịch chương trình. Nếu tệp thiếu shebang thì phải chỉ rõ trình thông dịch ở lệnh thực thi (ví dụ `bash ./elf`), nếu không hệ điều hành sẽ hiểu là tệp nhị phân.

---

## Level 23 — Tái tạo hash MD5 trong Cron job

**Yêu cầu:** Tương tự Level 22, nhưng thư mục lưu mật khẩu có tên được mã hóa bằng MD5, dựa trên biến `myname` trong script.

```bash
cat /usr/bin/cronjob_bandit23.sh
# thay myname="bandit23" và in giá trị mytarget để biết đường dẫn
cat <tệp đích được tính từ mytarget>
```

**Hướng giải quyết:** Đọc script để thấy công thức tính tên thư mục lưu mật khẩu (MD5 của một chuỗi có chứa biến `myname`). Thay giá trị `myname="bandit23"` và in ra giá trị `mytarget` để biết chính xác đường dẫn mà cron job sẽ ghi mật khẩu vào, sau đó mở tệp đó.

---

## Level 24 — Lợi dụng Cron job quét thư mục ghi được

**Yêu cầu:** Cron liên tục thực thi tệp `cronjob_bandit24`, dẫn tới thực thi liên tục các tệp nằm tại đường dẫn `/var/spool/bandit24/foo` và xóa ngay lập tức nếu thư mục không rỗng.

**Phân tích:**
- Thư mục `foo` cho phép user hiện tại **ghi và thực thi**, nghĩa là được phép ghi tệp script vào thư mục này.
- Dựa vào nội dung `cronjob_bandit24`, tại thời điểm hiện tại thư mục `foo` không chứa script nào, nên chưa có tệp nào in mật khẩu lên terminal.
- Bước tiếp theo là viết shell script có nội dung đẩy mật khẩu bandit24 tới một tệp mà bandit23 có quyền đọc.

```bash
# tạo thư mục tạm để lưu script và tệp lưu password của next level
WORK=$(mktemp -d)
cd "$WORK"
touch password
cat > crack.sh <<'EOF'
#!/bin/bash
cat /etc/bandit_pass/bandit24 > <WORK>/password
EOF

chmod o+x crack.sh     # cấp quyền excute cho nhóm others (bandit24) để thực thi
chmod o+w password     # cấp quyền write để bandit24 ghi được vào tệp password
chmod o+x "$WORK"      # mở rộng quyền cho thư mục cha (xem Q&A)

cp crack.sh /var/spool/bandit24/foo/
cat "$WORK/password"
```

**Q&A — vì sao phải mở rộng quyền thư mục tạm dù đã cấp quyền write cho tệp password?** Để có thể ghi mật khẩu lên `password` thì máy cần mở được thư mục cha, và để mở được thì nhóm others (bandit24) cần được cấp quyền execute. Cụ thể có thể set quyền cho thư mục bằng `chmod o+x $WORK`.

---

## Level 25 — Brute-force mã PIN 4 chữ số

**Yêu cầu:** Brute-force chuỗi pincode có 4 chữ số để tìm secret-pincode và gửi tới cổng 30002 để nhận mật khẩu next-level.

```bash
for i in $(seq -w 0 9999); do
  echo "<password_bandit24> $i"
done | nc localhost 30002 | grep -v "Wrong"
```

**Hướng giải quyết:** Viết script brute-force từng pincode và chuyển tới `localhost:30002`, sau đó lọc bỏ hết các dòng chứa chuỗi `Wrong` để tìm ra chuỗi mật khẩu cần tìm.

---

## Tổng hợp câu lệnh đã học

| Lệnh | Công dụng chính |
|---|---|
| `ssh -p <port> user@host` | Kết nối SSH tới server ở cổng chỉ định |
| `ssh -i <key> user@host` | Đăng nhập SSH bằng private key |
| `cat`, `ls -a`, `ls -la` | Đọc nội dung file, liệt kê file (kể cả file ẩn), xem chi tiết quyền/owner |
| `file <file>` | Xác định loại/định dạng của file |
| `find <path> [expr]` | Tìm file theo nhiều điều kiện (size, quyền, owner, group,...) |
| `grep <pattern> <file>` | Lọc các dòng khớp với mẫu (pattern) |
| `sort` + `uniq -u/-c/-d` | Sắp xếp và lọc dòng trùng/duy nhất |
| `strings <file>` | Liệt kê chuỗi ký tự đọc được trong file (kể cả binary) |
| `base64 -d` | Giải mã Base64 |
| `tr set1 set2` | Ánh xạ/dịch ký tự (dùng để giải ROT13) |
| `xxd -r` | Chuyển hexdump ngược lại thành file nhị phân |
| `gzip -d`, `bzip2 -d`, `tar -xf` | Giải nén / giải đóng gói file |
| `nc` (Netcat) | Gửi/nhận dữ liệu thô qua TCP/UDP |
| `openssl s_client -connect host:port -quiet` | Gửi/nhận dữ liệu qua kết nối SSL/TLS |
| `nmap -sV -p <range>` | Quét cổng và xác định dịch vụ |
| `diff fileA fileB` | So sánh khác biệt giữa hai file |
| `chmod` | Thay đổi quyền truy cập file/thư mục |
| `md5sum` | Tính hash MD5 của một chuỗi/file |
| `seq -w` | Sinh dãy số (giữ định dạng số chữ số cố định) |

---

*Write-up được tổng hợp lại từ ghi chú cá nhân trong quá trình tự học Linux/Bandit — dùng để ôn tập và chia sẻ.*
