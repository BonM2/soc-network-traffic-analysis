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
**IPv4**
| Trường | Giá trị |
|---|---|
| DNS Client | `2405:4802:80ca:7d60:34c2:9b65:96b3:f918` |
| DNS Server | `2405:4802:80ca:7d60::1` |
| Query Name | `example.com` |
| Query Type | `A`|
| Transaction ID | `0x0004` |
| Response Code | `No error (0)` |
| Answers | `172.66.147.243`, `104.20.23.154` |
| Evidence | `Frame query: 28606`, `Frame response: 28607` |

**IPv6**
| Trường | Giá trị |
|---|---|
| DNS Client | `2405:4802:80ca:7d60:34c2:9b65:96b3:f918` |
| DNS Server | `2405:4802:80ca:7d60::1` |
| Query Name | `example.com` |
| Query Type | `AAAA`|
| Transaction ID | `0x0005` |
| Response Code | `No error (0)` |
| Answers | `2606:4700:10::ac42:93f3`, `2606:4700:10::6814:179a` |
| Evidence | `Frame query: 28608`, `Frame response: 28609` |

**Câu hỏi và trả lời**

1. **Những trường nào cho thấy query và response tương ứng?**

   Đối chiếu Transaction ID, Query Name, Query Type và địa chỉ/port của hai endpoint theo chiều ngược nhau, kết hợp thời gian xuất hiện. Cột Info hỗ trợ đọc nhanh nội dung packet.

2. **Truy vấn A và AAAA trả về loại địa chỉ nào?**

   Bản ghi A chứa địa chỉ IPv4; bản ghi AAAA chứa địa chỉ IPv6. Trong các phản hồi đang phân tích, cả hai loại truy vấn đều nhận được địa chỉ tương ứng.

3. **Mã NOERROR cùng các Answers cho thấy điều gì?**

   Phản hồi có mã `NOERROR (0)`, cho biết DNS server không báo lỗi khi xử lý truy vấn. Phần Answers chứa các bản ghi địa chỉ trả lời cho tên miền được hỏi.

4. **Địa chỉ đích của TCP ở mục tiếp theo có xuất hiện trong Answers không?**

   Có. Địa chỉ đích TCP `172.66.147.243` xuất hiện trong phản hồi A tại frame 28607, cùng với `104.20.23.154`. Kết nối TCP ở stream 342 sử dụng một trong các địa chỉ IPv4 được trả về.

5. **Kết quả DNS chưa chứng minh được điều gì về giao dịch HTTPS?**

   Kết quả DNS chưa chứng minh kết nối TCP hoặc phiên TLS đã được thiết lập, dữ liệu ứng dụng được mã hóa, hay giao dịch HTTPS hoàn tất thành công. Những kết luận này cần bằng chứng từ các bước phân tích TCP, TLS và ứng dụng tương ứng.

**Kết quả phân tích:** DNS client gửi truy vấn A cho `example.com` tại frame 28606 và nhận phản hồi tại frame 28607, cùng Transaction ID `0x0004`. Tương tự, truy vấn AAAA tại frame 28608 có phản hồi tại frame 28609, cùng Transaction ID `0x0005`. Các cặp được đối chiếu thêm bằng endpoint, Query Name và Query Type.

Cả hai phản hồi đều có Response Code `NOERROR (0)`, cho biết DNS server không báo lỗi khi xử lý truy vấn. Phản hồi A chứa hai địa chỉ IPv4 `172.66.147.243` và `104.20.23.154`; phản hồi AAAA chứa hai địa chỉ IPv6 `2606:4700:10::ac42:93f3` và `2606:4700:10::6814:179a`. Kết nối TCP được phân tích tiếp theo ở stream 342 sử dụng địa chỉ đích IPv4 `172.66.147.243`, nằm trong phản hồi A.

**Giới hạn:** Các phản hồi cho thấy client đã nhận được bản ghi địa chỉ cho tên miền được hỏi. Riêng bằng chứng DNS chưa xác nhận kết nối TCP được thiết lập, quá trình thương lượng TLS hoàn tất hoặc giao dịch HTTPS thành công.

### 4.2. TCP

**Kết nối được phân tích:**

- Client: `192.168.1.39:53299`.
- Server: `172.66.147.243:443`.
- TCP Stream index: `342`.
- Display filter: `tcp.stream == 342`.

**Bằng chứng handshake**

| Frame | Hướng truyền | Flags | Relative Seq | Relative Ack |
|---|---|---|---|---|
| 28966 | Client → Server | SYN | 0 | Không áp dụng: cờ ACK chưa bật |
| 28968 | Server → Client | SYN, ACK | 0 | 1 |
| 28969 | Client → Server | ACK | 1 | 1 |

**Kết quả phân tích:**
Client chủ động mở kết nối từ địa chỉ 192.168.1.39 đến Server có địa chỉ 172.66.147.243. Tại frame 28966, quan sát được Client gửi packet có cờ SYN với `Seq = 0` đến Server. Server phản hồi với packet có cờ [SYN, ACK] với `Seq = 0, ACK = 1` tại frame 28968. Cuối cùng Client gửi packet có cờ [ACK] với `Seq = 1, ACK = 1` đến Server tại frame 28969. Ba packet thuộc cùng kết nối vì có cùng `Stream Index = 342`. Chuỗi trao đổi này cho thấy Client và Server đã hoàn thành bắt tay 3 bước (Three-way handshake).

**Giới hạn:** Việc thiết lập kết nối TCP thành công chưa đủ
để xác nhận quá trình thương lượng TLS hoặc giao dịch HTTPS
hoàn tất thành công.

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
