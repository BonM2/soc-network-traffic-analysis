# Case 02 — Phân tích lưu lượng HTTP với Wireshark

## 1. Tóm tắt

Báo cáo sử dụng Wireshark để xác định tài nguyên được yêu cầu và phản hồi tương ứng trong một giao dịch HTTP của file mẫu `http.cap`. Trong TCP stream `0`, client gửi `GET /download.html` tại frame 4 và máy chủ trả về `HTTP/1.1 200 OK` tại frame 38 với nội dung được khai báo là HTML. Kết quả phản ánh giao dịch ở tầng HTTP trong capture, còn việc xác nhận trang đã hiển thị đầy đủ cần thêm bằng chứng từ trình duyệt.

## 2. Thông tin dữ liệu

| Thuộc tính | Giá trị |
|---|---|
| File phân tích | `http.cap` |
| Nguồn dữ liệu | `https://wiki.wireshark.org/uploads/27707187aeb30df68e70c8fb9d614981/http.cap` |
| Định dạng capture | `PCAP` |
| Số frame | 43 |
| Thời lượng capture | 30.394s |
| Công cụ phân tích | Wireshark 4.6.10 |

## 3. Phạm vi và phương pháp

| Đối tượng | Câu hỏi phân tích |
|---|---|
| HTTP Request | Client dùng method nào để yêu cầu tài nguyên nào? Request được gửi đến địa chỉ IP và port nào? |
| HTTP Response | Server trả về mã trạng thái và loại nội dung nào nếu quan sát được? Bằng chứng nào cho thấy response tương ứng với request đã chọn? |

1. Dùng display filter `http.request` để tìm request tại frame 4. Trong **Packet Details → Hypertext Transfer Protocol**, đọc Request Method, Request URI, Request Version và Host nếu có.
2. Kiểm tra **Response in frame** của frame 4, sau đó dùng `tcp.stream == 0` để xem các gói trong kết nối. Trong frame 38, kiểm tra **Request in frame** và đối chiếu IP/port theo hai chiều.
3. Dùng **Follow → TCP Stream** để đọc lại request line, response line và các header quan sát được.

## 4. Kết quả phân tích

### 4.1. HTTP Request

| Trường | Giá trị quan sát được |
|---|---|
| Frame | Số frame request: 4 |
| TCP stream | `tcp.stream == 0`|
| IP và port | `Source IP Address: 145.254.160.237`, `Destination IP Address: 65.208.228.223`, `Source port: 3372`, `Destination port: 80` |
| Request line | Method: `GET`, URI: `/download.html` và phiên bản HTTP: `HTTP/1.1` |
| Host | `www.ethereal.com` |
| Header đáng chú ý | Mozilla/5.0 (Windows; U; Windows NT 5.1; en-US; rv:1.6) Gecko/20040113 |

Tại frame 4, client `145.254.160.237:3372` gửi yêu cầu đến `65.208.228.223:80`. Dòng `GET /download.html HTTP/1.1` cho thấy client dùng phương thức `GET` để yêu cầu tài nguyên `/download.html`. Header `Host` xác định tên máy chủ mà yêu cầu hướng tới là `www.ethereal.com`.

### 4.2. HTTP Response

| Trường | Giá trị quan sát được |
|---|---|
| Frame | Số frame response: 38 |
| IP và port | `Source IP Address: 145.254.160.237`, `Destination IP Address: 65.208.228.223`, `Source port: 80`, `Destination port: 3372` |
| Status line | Phiên bản HTTP: `HTTP/1.1`, mã trạng thái: `200`, mô tả: `OK` |
| Content-Type | `text/html; charset=ISO-8859-1` |

Tại frame 38, máy chủ `65.208.228.223:80` gửi phản hồi `HTTP/1.1 200 OK` về client `145.254.160.237:3372`. Trường **Request in frame** trỏ về frame 4, nơi client gửi yêu cầu `GET /download.html`.

Mã `200 OK` cho biết máy chủ báo đã xử lý thành công yêu cầu. Header `Content-Type: text/html; charset=ISO-8859-1` cho biết máy chủ khai báo nội dung phản hồi là HTML với bộ ký tự `ISO-8859-1`.
### 4.3. Đối chiếu giao dịch HTTP

| Thông điệp | Frame | Hướng truyền | Liên kết trong Wireshark |
|---|---:|---|---|
| Request `GET /download.html` | 4 | `145.254.160.237:3372` → `65.208.228.223:80` | **Response in frame:** `38` |
| Response `HTTP/1.1 200 OK` | 38 | `65.208.228.223:80` → `145.254.160.237:3372` | **Request in frame:** `4` |

Cả hai frame thuộc TCP stream `0`. Cặp IP và port đảo chiều giữa hai frame, cho thấy request đi từ client đến server và response quay về cùng client. Hai trường **Response in frame** và **Request in frame** trỏ qua lại, xác nhận frame 38 là phản hồi HTTP cho yêu cầu `GET /download.html` tại frame 4.

Xem [E01 — HTTP request](../screenshots/packet-detail-http-request.png) và [E02 — HTTP response](../screenshots/packet-detail-http-response.png) trong danh mục bằng chứng ở mục 7.

## 5. Timeline

| Thời gian | Frame | Sự kiện |
|---|---|---|
| 2004-05-13 10:17:08 | 4 | Client gửi HTTP request |
| 2004-05-13 10:17:12 | 38 | Server gửi HTTP response tương ứng |

## 6. Kết luận và giới hạn

**Kết luận**

Trong file mẫu `http.cap`, client gửi yêu cầu `GET /download.html HTTP/1.1` đến `www.ethereal.com` tại frame 4. Máy chủ trả về `HTTP/1.1 200 OK` tại frame 38 và khai báo nội dung phản hồi là HTML. Hai frame thuộc TCP stream `0`. Trường **Response in frame** tại frame 4 trỏ đến frame 38, còn **Request in frame** tại frame 38 trỏ về frame 4, xác nhận đây là hai thông điệp của cùng một giao dịch HTTP.

**Giới hạn**

Kết luận chỉ áp dụng cho giao dịch đã chọn trong capture mẫu. Mã `200 OK` phản ánh kết quả xử lý mà máy chủ báo ở tầng HTTP. Để xác nhận trình duyệt đã hiển thị trang đầy đủ, cần thêm bằng chứng từ phía trình duyệt.

## 7. Phụ lục

| Mã | Bằng chứng | Nguồn |
|---|---|---|
| E01 | Ảnh Packet Details của HTTP request | [packet-detail-http-request.png](../screenshots/packet-detail-http-request.png) |
| E02 | Ảnh Packet Details của HTTP response | [packet-detail-http-response.png](../screenshots/packet-detail-http-response.png) |