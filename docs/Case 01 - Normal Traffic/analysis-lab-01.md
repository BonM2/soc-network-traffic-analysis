# Case 01 — Phân tích lưu lượng DNS, TCP/TLS và ICMP

## 1. Tóm tắt
- Báo cáo phân tích các trao đổi DNS, TCP/TLS và ICMP trong phiên capture trên máy Windows nhằm kiểm tra quá trình phân giải tên miền, thiết lập kết nối TCP, trao đổi TLS và phản hồi ICMP Echo. Trong các trao đổi được lựa chọn, quan sát được truy vấn và phản hồi DNS cho example.com, TCP three-way handshake hoàn chỉnh đến 172.66.147.243:443, TLS Client Hello/Server Hello và bốn cặp ICMP Echo Request/Reply với 1.1.1.1. Nội dung HTTPS chưa được giải mã nên chưa xác định kết quả giao dịch HTTP từ capture. Các kết quả chỉ phản ánh những trao đổi được phân tích trong khoảng thời gian thu thập.
## 2. Thông tin dữ liệu

| Thuộc tính | Giá trị |
|---|---|
| File phân tích | `normal-traffic.pcapng` |
| Nguồn dữ liệu | Tự capture trên máy Windows thực hiện lab |
| Điểm capture | Wi-Fi, Interface ID 0 |
| Hệ điều hành khi capture | Windows 11 26H2|
| Công cụ thu thập | Dumpcap (Wireshark) 4.6.9 |
| Công cụ phân tích | Dumpcap (Wireshark) 4.6.9 |
| Thời điểm packet đầu tiên | `13:59:46 10-11-2026` |
| Thời điểm packet cuối cùng | `14:10:59 10-11-2026` |
| Múi giờ hiển thị | UTC+07:00 |
| Khoảng thời gian được ghi nhận | `11 phút 03 giây` |
| Tổng số packet trong file | 29 536 |
| IPv4 máy thực hiện lab | 192.168.1.39 |
## 3. Phạm vi và phương pháp

### 3.1. Phạm vi phân tích

| Đối tượng | Câu hỏi cần được trả lời |
|---|---|
| DNS cho example.com | Có query/response tương ứng không? Bản ghi A và AAAA trả về gì? |
| TCP giữa 192.168.1.39:53299 và 172.66.147.243:443 | Có đủ ba bước thiết lập kết nối không? |
| TLS trong kết nối TCP trên | Có Client Hello/Server Hello không? Metadata nào quan sát được? |
| ICMP giữa máy kiểm thử và 1.1.1.1 | Mỗi Echo Request có Echo Reply tương ứng không? |

Phạm vi phân tích tập trung vào các trao đổi DNS cho `example.com`, quá trình thiết lập kết nối TCP và trao đổi TLS với 172.66.147.243, cùng các lượt ICMP Echo giữa máy kiểm thử và 1.1.1.1. Báo cáo kiểm tra sự tương ứng của DNS query/response, tính đầy đủ của TCP three-way handshake, metadata TLS quan sát được và các cặp ICMP Echo Request/Reply.

### 3.2. Phương pháp phân tích

Mô tả thứ tự thao tác: File capture được mở bằng Wireshark và các packet được sắp xếp theo thứ tự thời gian tăng dần. Các display filter được sử dụng để lựa chọn những trao đổi trong phạm vi phân tích. Sau đó, các trường trong Packet Details được kiểm tra và đối chiếu theo từng giao thức. Frame number và ảnh bằng chứng được ghi lại để tham chiếu trong phần kết quả.

| Giao thức | Filter và cách đối chiếu |
|---|---|
| DNS | `dns.qry.name == "example.com"`; đối chiếu endpoint, Transaction ID, Query Name và Query Type |
| TCP/TLS | `tcp.stream == 342`; kiểm tra IP/port, flags, Seq/Ack và thông điệp TLS |
| ICMP | `icmp && ip.addr == 1.1.1.1`; đối chiếu Identifier, Sequence Number và hướng truyền |

### 3.3. Giới hạn phương pháp
- Chỉ phân tích các trao đổi được lựa chọn trong khoảng thời gian capture
- Nội dung HTTPS chưa được giải mã
- Capture tại interface máy kiểm thử chỉ phản ánh dữ liệu quan sát được tại điểm đó.
- Chưa đối chiếu với log tiến trình nên không xác định ứng dụng tạo kết nối chỉ từ IP và port.

## 4. Kết quả phân tích
### 4.1. DNS
### 4.2. TCP
### 4.3. TLS
### 4.4. ICMP

## 5. Timeline

## 6. Kết luận và giới hạn

## 7. Phụ lục

### 7.1. Danh mục bằng chứng

Các số frame dưới đây tham chiếu đến file gốc `normal-traffic.pcapng`.

| Mã | Bằng chứng | Frame liên quan | Nội dung tra cứu |
|---|---|---|---|
| E01 | [DNS query/response](../../labs/01-normal-traffic/screenshots/dns-query-response.png) | A: 28606–28607 và 28963–28965; AAAA: 28608–28609 và 28964–28967 | Các cặp truy vấn/phản hồi cho `example.com`. Với các cặp không liền nhau, chỉ tham chiếu hai frame ở hai đầu, không phải toàn bộ khoảng. |
| E02 | [TCP three-way handshake](../../labs/01-normal-traffic/screenshots/tcp-three-way-handshake.png) | 28966, 28968, 28969 | SYN, SYN-ACK và ACK giữa `192.168.1.39:53299` và `172.66.147.243:443`, thuộc TCP stream 342. |
| E03 | [TLS Client Hello/Server Hello](../../labs/01-normal-traffic/screenshots/tls-handshake.png) | 28970, 28972 | Client Hello có SNI `example.com` và Server Hello trong TCP stream 342. |
| E04 | [ICMP Echo Request/Reply](../../labs/01-normal-traffic/screenshots/icmp-echo-request-reply.png) | 29120–29121; 29127–29128; 29143–29144; 29150–29151 | Bốn cặp Echo Request từ `192.168.1.39` đến `1.1.1.1` và Echo Reply theo chiều ngược lại. |

### 7.2. Display filter phục vụ đối chiếu

| Mục đích | Display filter |
|---|---|
| DNS cho tên miền kiểm thử | `dns.qry.name == "example.com"` |
| DNS loại A | `dns.qry.name == "example.com" && dns.qry.type == 1` |
| DNS loại AAAA | `dns.qry.name == "example.com" && dns.qry.type == 28` |
| Toàn bộ kết nối TCP được phân tích | `tcp.stream == 342` |
| TLS Client Hello trong kết nối | `tcp.stream == 342 && tls.handshake.type == 1` |
| TLS Server Hello trong kết nối | `tcp.stream == 342 && tls.handshake.type == 2` |
| ICMP giữa máy kiểm thử và đích ping | `icmp && ip.addr == 192.168.1.39 && ip.addr == 1.1.1.1` |

### 7.3. Lưu ý khi tra cứu

- Stream index và frame number được sử dụng trong phụ lục là các giá trị trong file gốc; cần kiểm tra lại nếu xuất một file capture con.
- Timestamp hiển thị trong các ảnh bằng chứng là UTC. Khi đối chiếu với thời gian UTC+07:00, cộng 7 giờ; frame number không thay đổi.
- Các ảnh chỉ thể hiện những packet được lựa chọn. Sử dụng file capture gốc và Packet Details để kiểm tra đầy đủ các trường liên quan.
