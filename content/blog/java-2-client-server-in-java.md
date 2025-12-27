---
title: "Kiến trúc Client – Server trong Java: Cơ chế và hoạt động chi tiết"
date: 2025-01-02
cover: "/images/cover-java-client-server.jpg"
---

## Kiến trúc Client – Server trong Java

Mô hình **Client – Server (Khách – Chủ)** là xương sống của hầu hết
các ứng dụng mạng hiện đại.  
Trong Java, mô hình này được hiện thực hóa mạnh mẽ thông qua gói
**`java.net`**, chủ yếu sử dụng giao thức **TCP**
để đảm bảo độ tin cậy của dữ liệu truyền tải.

<!--more-->

---

## 1. Quy trình hoạt động (Workflow)

Quy trình giao tiếp giữa Client và Server trong Java
diễn ra theo các bước tuần tự và chặt chẽ:

### 1.1 Khởi tạo Server (Binding)
Server sử dụng lớp `ServerSocket` để đăng ký một cổng (*port*)
cụ thể trên máy chủ (ví dụ: **port 8080**).

### 1.2 Lắng nghe kết nối (Listening)
Server gọi phương thức `accept()`.

- Đây là phương thức **blocking**
- Chương trình server sẽ tạm dừng tại đây
- Chỉ tiếp tục khi có client kết nối tới

### 1.3 Client gửi yêu cầu kết nối
Client khởi tạo đối tượng `Socket`
kèm theo địa chỉ IP và port của Server
để gửi yêu cầu **handshake**.

### 1.4 Thiết lập liên kết
Khi Server chấp nhận yêu cầu:

- Một **kênh giao tiếp ảo (Virtual Circuit)** được thiết lập
- `accept()` trả về một đối tượng `Socket` mới
- Socket này dùng riêng cho client đó

### 1.5 Trao đổi dữ liệu (I/O Operations)
Hai phía giao tiếp thông qua:

- `InputStream` – nhận dữ liệu
- `OutputStream` – gửi dữ liệu  

Dữ liệu được truyền dưới dạng byte hoặc ký tự.

---

## 2. Chiến lược xử lý đa luồng (Multi-threading Strategy)

Đây là yếu tố **then chốt** giúp Server phục vụ nhiều Client
mà không bị “đơ”.

### 2.1 Vấn đề
Nếu Server chỉ chạy trên **một luồng chính (Main Thread)**:

- Chỉ phục vụ được **1 Client tại một thời điểm**
- Client tiếp theo phải chờ kết nối trước kết thúc

### 2.2 Giải pháp trong Java
Java giải quyết vấn đề này bằng **đa luồng**:

- Khi `accept()` trả về một `Socket` mới:
  - Server tạo một **Thread** mới (hoặc lấy từ Thread Pool)
- Thread này chịu trách nhiệm giao tiếp riêng với Client
- Luồng chính quay lại trạng thái `accept()` ngay lập tức

### 2.3 Lưu ý kỹ thuật
Trong các hệ thống hiện đại:

- ❌ Không nên tạo `new Thread()` cho mỗi kết nối
- ✅ Nên dùng **`ExecutorService` (Thread Pool)**

Ưu điểm:
- Quản lý luồng hiệu quả
- Tránh quá tải bộ nhớ
- Tăng khả năng mở rộng (scalability)

---

## 3. Các thành phần code cốt lõi

| Thành phần | Lớp Java | Vai trò |
|----------|---------|--------|
| Server Listener | `ServerSocket(port)` | “Cánh cổng” đón Client, chỉ cần một đối tượng |
| Communication Endpoint | `Socket(ip, port)` | Kênh giao tiếp giữa Client và Server |
| Dòng dữ liệu vào | `InputStream` | Nhận dữ liệu từ phía bên kia |
| Dòng dữ liệu ra | `OutputStream` | Gửi dữ liệu đến phía bên kia |

---

## 4. Ứng dụng thực tế

Kiến trúc Client – Server trong Java
là nền tảng của rất nhiều hệ thống thực tế:

- **Web Server** (Tomcat, Jetty)  
  Xử lý hàng nghìn yêu cầu HTTP từ trình duyệt

- **Database Server**  
  Kết nối JDBC tới MySQL, PostgreSQL thực chất là Socket

- **Game Online**  
  Đồng bộ trạng thái nhân vật trong thời gian thực

- **Remote Desktop**  
  Điều khiển máy tính từ xa qua mạng

---

## Kết luận

Kiến trúc Client – Server trong Java
là nền tảng quan trọng để xây dựng
các hệ thống mạng ổn định, an toàn và mở rộng tốt.
Việc hiểu rõ cơ chế hoạt động và chiến lược đa luồng
giúp lập trình viên thiết kế Server hiệu quả ngay từ đầu.
