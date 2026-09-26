# Mô hình Client–Server & các khái niệm mạng cơ bản

> Ghi chú học từ TryHackMe. Bài gốc dùng ví dụ **Alice – Bob – Luigi's Pizza** để giải thích; bản này giữ lại phép so sánh đó và bổ sung thêm kiến thức chuyên môn.

## Mục lục

- [Ví dụ minh họa](#ví-dụ-minh-họa)
- [1. Service, Client, Server](#1-service-client-server)
- [2. Request và Response](#2-request-và-response)
- [3. Protocol (Giao thức)](#3-protocol-giao-thức)
- [4. Port (Cổng)](#4-port-cổng)
- [5. DNS](#5-dns)
- [6. Luồng hoạt động khi truy cập một website](#6-luồng-hoạt-động-khi-truy-cập-một-website)
- [7. Các lệnh thực hành](#7-các-lệnh-thực-hành)
- [Tổng kết](#tổng-kết)

---

## Ví dụ minh họa

| Nhân vật trong ví dụ | Tương ứng trong hệ thống máy tính |
|---|---|
| **Alice** | Người dùng / **Client** (ví dụ trình duyệt) |
| **Bob** | **Giao thức (protocol)** và hạ tầng mạng chuyển tin |
| **Luigi's Pizza** | **Server** cung cấp dịch vụ |
| Menu, ngôn ngữ đặt món | Đặc tả giao thức (lệnh, cú pháp) |
| Cửa A / B / C | **Port** của từng dịch vụ |
| GPS (tên quán → tọa độ) | **DNS** (tên miền → địa chỉ IP) |

---

## 1. Service, Client, Server

- **Server**: máy/tiến trình **lắng nghe** (listen) và cung cấp tài nguyên hoặc dịch vụ (web, mail, file, database,...).
- **Client**: máy/chương trình **khởi tạo yêu cầu** để dùng dịch vụ (trình duyệt, `curl`, `ssh`, `nc`,...).
- **Service**: chương trình cụ thể chạy trên server, ví dụ `nginx`/`Apache` (HTTP), `sshd` (SSH), `MySQL` (database).

> **Quy tắc quan trọng:** client luôn là bên **khởi tạo** yêu cầu; server chờ và phản hồi.

**Ghi chú chuyên môn:**
- "Client" và "Server" là **vai trò**, không phải loại thiết bị. Một máy có thể vừa là server (chạy web) vừa là client (gọi API bên ngoài).
- Server thường xử lý **nhiều client cùng lúc** (đa luồng, đa tiến trình hoặc xử lý bất đồng bộ).
- Ngoài mô hình client–server còn có **peer-to-peer (P2P)**, nơi các bên vừa yêu cầu vừa phục vụ (ví dụ BitTorrent).
- Với dịch vụ web, tầng ứng dụng thường gồm nhiều lớp: trình duyệt → reverse proxy/load balancer → web server → application server → database.

---

## 2. Request và Response

Trong ví dụ, Alice gọi *một pizza pepperoni lớn*. Yêu cầu này **gửi tới Luigi's**, không phải tới Bob — Bob chỉ là bên **chuyển** yêu cầu.

Trong hệ thống máy tính: client gửi **request**, server xử lý và trả về **response**. Nếu request sai định dạng hoặc tài nguyên không có → server trả về **lỗi**.

### Ví dụ HTTP thực tế

**Request:**
```http
GET /index.html HTTP/1.1
Host: example.com
User-Agent: Mozilla/5.0
Accept: text/html
```

**Response:**
```http
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 1256

<!DOCTYPE html>
<html> ... </html>
```

### Các nhóm mã trạng thái HTTP

| Mã | Nhóm | Ý nghĩa | Tương ứng ví dụ Pizza |
|---|---|---|---|
| `1xx` | Informational | Đã nhận, đang xử lý | "Đã nhận đơn" |
| `2xx` | Success | Thành công (`200 OK`) | Giao đúng pizza |
| `3xx` | Redirection | Chuyển hướng (`301`, `302`) | "Quán đã chuyển địa chỉ" |
| `4xx` | Client error | Lỗi từ phía client (`400`, `403`, `404`) | Gọi món không có trong menu / sai cú pháp |
| `5xx` | Server error | Lỗi từ phía server (`500`, `503`) | Bếp bị hỏng, không làm được |

Ví dụ: hết pizza pepperoni ≈ `404 Not Found`; server không hiểu đơn hàng ≈ `400 Bad Request`.

> **Dưới góc nhìn bảo mật:** thông tin trong request/response (header, cookie, tham số, thông báo lỗi) thường lộ ra phiên bản phần mềm, cấu trúc thư mục hoặc logic ứng dụng. Đây là nguồn dữ liệu quan trọng khi **thu thập thông tin (enumeration)**.

---

## 3. Protocol (Giao thức)

Alice dùng đúng **ngôn ngữ** mà Luigi's hiểu và đặt món theo **menu**. Bob hiểu yêu cầu, đem đi và mang phản hồi về. Tương tự, **giao thức** là bộ quy tắc mà hai bên phải tuân theo để hiểu nhau.

Một giao thức định nghĩa:

| Nội dung | Ví dụ (ẩn dụ Pizza) | Ví dụ thực tế (HTTP) |
|---|---|---|
| Các **lệnh** được hiểu | Lệnh "lấy" (get) | `GET`, `POST`, `PUT`, `DELETE` |
| **Cấu trúc** của request | Lệnh trước, rồi đến món | Request line → headers → (dòng trống) → body |
| **Cú pháp** dùng | Tiếng Anh | Văn bản ASCII, header dạng `Tên: Giá trị` |
| **Phản hồi** cho mỗi loại yêu cầu | Yêu cầu pizza → trả pizza có sẵn | `GET /` → `200 OK` + nội dung trang |
| Xử lý **yêu cầu sai** | "Không có pizza pepperoni" | `404 Not Found`, `400 Bad Request` |

### Một số giao thức phổ biến

| Giao thức | Chức năng | Cổng mặc định | Tầng vận chuyển |
|---|---|---|---|
| **HTTP** | Truyền tải trang web | 80 | TCP |
| **HTTPS** | HTTP mã hóa qua TLS | 443 | TCP |
| **FTP** | Truyền file | 21 (điều khiển), 20 (dữ liệu) | TCP |
| **SSH** | Truy cập/điều khiển từ xa an toàn | 22 | TCP |
| **Telnet** | Điều khiển từ xa (không mã hóa) | 23 | TCP |
| **SMTP** | Gửi email | 25 (587 có xác thực) | TCP |
| **POP3 / IMAP** | Nhận email | 110 / 143 | TCP |
| **DNS** | Phân giải tên miền | 53 | UDP (và TCP) |
| **DHCP** | Cấp phát địa chỉ IP tự động | 67/68 | UDP |
| **SMB** | Chia sẻ file/máy in (Windows) | 445 | TCP |

### Các tầng (mô hình TCP/IP)

Các giao thức hoạt động ở nhiều tầng khác nhau:

| Tầng | Vai trò | Ví dụ |
|---|---|---|
| Ứng dụng | Giao tiếp với phần mềm | HTTP, DNS, SSH, SMTP |
| Vận chuyển | Truyền dữ liệu giữa các tiến trình, dùng **port** | TCP, UDP |
| Internet | Định tuyến gói tin theo địa chỉ IP | IP, ICMP |
| Truy cập mạng | Truyền vật lý trong mạng cục bộ | Ethernet, Wi-Fi |

### TCP và UDP

| | **TCP** | **UDP** |
|---|---|---|
| Kết nối | Có (bắt tay 3 bước: SYN → SYN/ACK → ACK) | Không |
| Độ tin cậy | Đảm bảo thứ tự và chống mất gói | Không đảm bảo |
| Tốc độ | Chậm hơn | Nhanh hơn |
| Dùng cho | Web, email, SSH, truyền file | DNS, streaming, game, VoIP |

> **Lưu ý bảo mật:** nhiều giao thức cũ (**HTTP, FTP, Telnet**) truyền dữ liệu **dạng văn bản thuần (plaintext)**, kể cả mật khẩu — kẻ nghe lén trên đường truyền có thể đọc được. Nên dùng bản mã hóa tương ứng: **HTTPS, SFTP/FTPS, SSH**.

---

## 4. Port (Cổng)

**Port** là số nguyên **16 bit** (từ `0` đến `65535`) dùng để xác định **một dịch vụ cụ thể** đang chạy trên một máy. Địa chỉ IP xác định *máy nào*, còn port xác định *dịch vụ nào* trên máy đó.

Ẩn dụ: mọi khách đặt mang đi đều vào cùng một cửa. Nếu Luigi's có nhiều dịch vụ (mang đi, ăn tại chỗ, giao hàng) thì mỗi dịch vụ dùng **một cửa khác nhau** (A, B, C) — tương tự một server chạy nhiều dịch vụ cùng lúc, mỗi dịch vụ ở một port riêng.

```text
IP Server: 203.0.113.10
 ├── :22   → SSH
 ├── :80   → HTTP
 ├── :443  → HTTPS
 └── :3306 → MySQL
```

Một **kết nối** được xác định bởi bộ năm (5-tuple): `giao thức, IP nguồn, port nguồn, IP đích, port đích`.

### Phân loại port

| Dải | Tên | Ghi chú |
|---|---|---|
| `0 – 1023` | **Well-known ports** | Dành cho dịch vụ chuẩn (80, 443, 22,...); thường cần quyền root để lắng nghe |
| `1024 – 49151` | **Registered ports** | Đăng ký cho ứng dụng cụ thể (ví dụ 3306 MySQL, 8080 HTTP thay thế) |
| `49152 – 65535` | **Dynamic / ephemeral ports** | Client tự động chọn làm port nguồn khi tạo kết nối |

> Khi bạn duyệt web, trình duyệt dùng một **port ngẫu nhiên** (ví dụ 52814) làm port nguồn, còn port đích là `443`.

### Trạng thái port (khi quét)

| Trạng thái | Ý nghĩa |
|---|---|
| `open` | Có dịch vụ đang lắng nghe |
| `closed` | Không có dịch vụ lắng nghe (nhưng máy phản hồi) |
| `filtered` | Không xác định được, thường do firewall chặn gói |

> **Dưới góc nhìn bảo mật:** mỗi port mở là một **bề mặt tấn công (attack surface)**. Vì vậy cần: chỉ mở port thực sự cần, cập nhật phần mềm và cấu hình firewall. Bước **quét cổng** (ví dụ bằng `nmap`) là một phần cơ bản của trinh sát.

---

## 5. DNS

Khi Alice chỉ biết **tên** quán, Bob nhập tên vào GPS để lấy **tọa độ**. **DNS (Domain Name System)** hoạt động tương tự: chuyển **tên miền** (ví dụ `tryhackme.com`) thành **địa chỉ IP** (ví dụ `104.x.x.x`) để máy có thể kết nối.

Địa chỉ **IP** giống địa chỉ nhà (đường, số nhà, mã bưu chính, thành phố, quốc gia) nhưng dành cho máy tính.

### Quy trình phân giải tên miền

```text
Trình duyệt → Cache trình duyệt / hệ điều hành / file hosts
            → Recursive Resolver (ISP, 8.8.8.8, 1.1.1.1)
                 → Root Server        ("." → chỉ tới máy chủ .com)
                 → TLD Server         (".com" → chỉ tới authoritative)
                 → Authoritative NS   (trả về IP cuối cùng)
            ← IP trả về, được lưu vào cache theo TTL
```

### Các loại bản ghi DNS thường gặp

| Bản ghi | Chức năng | Ví dụ |
|---|---|---|
| `A` | Tên miền → địa chỉ **IPv4** | `example.com → 93.184.216.34` |
| `AAAA` | Tên miền → địa chỉ **IPv6** | `example.com → 2606:2800:...` |
| `CNAME` | Bí danh (alias) trỏ tới tên khác | `www → example.com` |
| `MX` | Máy chủ nhận email của tên miền | `mail.example.com` |
| `NS` | Name server có thẩm quyền của tên miền | `ns1.example.com` |
| `TXT` | Văn bản tùy ý (SPF, DKIM, xác minh sở hữu) | `v=spf1 ...` |
| `PTR` | Phân giải ngược: IP → tên miền | `34.216.184.93.in-addr.arpa` |
| `SOA` | Thông tin quản trị vùng (zone) | serial, refresh,... |

### Khái niệm liên quan

- **TTL (Time To Live):** thời gian (giây) bản ghi được **cache**; TTL thấp thì cập nhật nhanh hơn nhưng tạo nhiều truy vấn hơn.
- **File `hosts`:** ánh xạ tên → IP cục bộ, được ưu tiên kiểm tra trước khi hỏi DNS (`/etc/hosts` trên Linux, `C:\Windows\System32\drivers\etc\hosts` trên Windows).
- **DNS dùng UDP cổng 53** cho truy vấn thông thường và **TCP cổng 53** cho phản hồi lớn hoặc **zone transfer**.

> **Dưới góc nhìn bảo mật:** DNS là nguồn thông tin trinh sát quan trọng (tìm **subdomain**, bản ghi TXT/MX). Các rủi ro thường gặp gồm **DNS spoofing/cache poisoning**, **zone transfer bị cấu hình sai** (lộ toàn bộ bản ghi) và **DNS tunneling** (giấu dữ liệu trong truy vấn DNS). Giải pháp phòng thủ có **DNSSEC**, **DoH/DoT** (mã hóa truy vấn).

---

## 6. Luồng hoạt động khi truy cập một website

Khi Alice (người dùng) gõ `https://example.com` vào trình duyệt:

1. **DNS:** trình duyệt hỏi DNS để chuyển `example.com` thành địa chỉ IP.
2. **TCP handshake:** client kết nối tới `IP:443` bằng bắt tay 3 bước (SYN → SYN/ACK → ACK).
3. **TLS handshake:** thống nhất phiên bản/cipher, server gửi chứng chỉ, hai bên tạo khóa phiên để mã hóa.
4. **HTTP request:** trình duyệt gửi `GET / HTTP/1.1` (kèm header `Host`, `User-Agent`, cookie,...).
5. **Xử lý phía server:** web server nhận yêu cầu, có thể chuyển cho ứng dụng/database để tạo nội dung.
6. **HTTP response:** server trả về mã trạng thái (`200 OK`) cùng nội dung HTML.
7. **Hiển thị:** trình duyệt phân tích HTML và gửi thêm request cho CSS, JavaScript, hình ảnh,... rồi render trang.

---

## 7. Các lệnh thực hành

```bash
# --- DNS ---
nslookup tryhackme.com               # tra IP của một tên miền
dig tryhackme.com A                  # truy vấn bản ghi A
dig tryhackme.com MX +short          # bản ghi mail, chỉ in kết quả
dig @8.8.8.8 tryhackme.com           # chỉ định DNS server để hỏi
dig -x 8.8.8.8                       # phân giải ngược (PTR)
host tryhackme.com

# --- Gửi request (client) ---
curl -v http://example.com           # xem cả request và response header
curl -I https://example.com          # chỉ lấy header phản hồi
curl -X POST -d "user=a&pass=b" http://example.com/login

# --- Kết nối thủ công tới một port ---
nc example.com 80                    # gõ tay: GET / HTTP/1.1  + Host: example.com
telnet example.com 80

# --- Xem port đang mở ---
ss -tulpn                            # cổng đang lắng nghe trên máy hiện tại
nmap -sV <IP>                        # quét cổng và dịch vụ trên máy đích

# --- Theo dõi đường đi & gói tin ---
ping example.com
traceroute example.com
sudo tcpdump -i any port 53          # bắt gói tin DNS
```

> Chỉ quét và thực hành trên hệ thống **của mình** hoặc môi trường được phép (như TryHackMe, HackTheBox).

---

## Tổng kết

| Khái niệm | Vai trò | Ẩn dụ Pizza |
|---|---|---|
| **Client** | Khởi tạo yêu cầu | Alice |
| **Server** | Lắng nghe & phục vụ | Luigi's Pizza |
| **Request / Response** | Yêu cầu và phản hồi (có thể là lỗi) | Đơn hàng / pizza hoặc "hết hàng" |
| **Protocol** | Quy tắc giao tiếp (lệnh, cấu trúc, cú pháp, xử lý lỗi) | Ngôn ngữ và menu |
| **Port** | Xác định dịch vụ trên máy (0–65535) | Cửa A / B / C |
| **DNS** | Tên miền → địa chỉ IP | GPS |
| **IP** | Địa chỉ của máy trên mạng | Địa chỉ nhà |

**Ghi nhớ nhanh:** *DNS cho biết tìm ở đâu (IP) → Port cho biết gõ cửa nào → Protocol cho biết nói ngôn ngữ nào → Client gửi Request → Server trả Response.*
