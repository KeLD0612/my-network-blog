---
title: "Giới thiệu lập trình mạng với Java"
date: 2025-01-01
cover: "/images/cover-java.jpg"
---

## Tổng quan lập trình mạng với Java: Nền tảng và ứng dụng

Lập trình mạng (Network Programming) là xương sống của các hệ thống phân tán,
ứng dụng Client–Server và hạ tầng Internet hiện đại.  
Trong đó, **Java** nổi lên như một lựa chọn hàng đầu nhờ triết lý
**“Write Once, Run Anywhere”**, tính ổn định cao và khả năng bảo mật mạnh mẽ.

<!--more-->

---

## 1. Các thành phần cốt lõi trong Java Network

Java cung cấp các API mạnh mẽ để xử lý kết nối mạng từ mức thấp (*low-level*)
đến mức cao (*high-level*), chủ yếu nằm trong hai gói thư viện sau:

### `java.net` – Blocking I/O (Cổ điển)

Đây là thư viện nền tảng, dễ học và phù hợp cho người mới bắt đầu.

- **Socket**  
  Đại diện cho một điểm kết nối (*endpoint*) để truyền dữ liệu giữa hai máy tính.

- **ServerSocket**  
  Dùng phía Server để “lắng nghe” tại một cổng (*port*) và chấp nhận kết nối từ Client.

- **URL / HttpURLConnection**  
  Các lớp mức cao để làm việc với giao thức HTTP và tải tài nguyên từ web.

### `java.nio` – Non-blocking I/O (Hiện đại)

Được giới thiệu nhằm khắc phục hạn chế hiệu năng của `java.net`.

- Sử dụng **Channel** và **Buffer** thay vì Stream
- Xử lý hàng nghìn kết nối đồng thời
- Không cần tạo quá nhiều Thread
- Tối ưu tài nguyên cho các hệ thống lớn

---

## 2. Hai giao thức vận chuyển chính

Trong lập trình mạng với Java, lập trình viên thường làm việc với
hai giao thức thuộc tầng Transport:

### TCP (Transmission Control Protocol)

- **Đặc điểm**:  
  Kết nối tin cậy, đảm bảo dữ liệu đến nơi đầy đủ và đúng thứ tự.

- **Ứng dụng trong Java**:  
  Sử dụng `Socket` và `ServerSocket`  
  Phù hợp cho chat, truyền file, giao dịch ngân hàng.

### UDP (User Datagram Protocol)

- **Đặc điểm**:  
  Không kết nối, tốc độ nhanh nhưng không đảm bảo dữ liệu đến nơi.

- **Ứng dụng trong Java**:  
  Sử dụng `DatagramSocket` và `DatagramPacket`  
  Phù hợp cho streaming video, game online thời gian thực.

---

## 3. Mô hình hoạt động Client – Server

Mô hình Client–Server trong Java hoạt động dựa trên cơ chế “bắt tay”
(*handshake*) và truyền dữ liệu qua luồng (*Streams*):

1. **Server khởi tạo**  
   Tạo `ServerSocket` gắn với một cổng (*port*) và chờ kết nối  
   (`accept()` sẽ chặn chương trình cho đến khi có Client).

2. **Client kết nối**  
   Tạo `Socket` với địa chỉ IP và cổng của Server.

3. **Thiết lập kênh**  
   Server chấp nhận kết nối và mở kênh giao tiếp hai chiều.

4. **Trao đổi dữ liệu**  
   - `InputStream`: nhận dữ liệu  
   - `OutputStream`: gửi dữ liệu  

5. **Đóng kết nối**  
   Giải phóng tài nguyên, tránh rò rỉ bộ nhớ.

---

## 4. Tại sao Java phù hợp cho người mới bắt đầu?

Java được đánh giá cao trong giảng dạy lập trình mạng vì:

- **Tính trừu tượng hóa cao**  
  Ẩn đi chi tiết phức tạp của hệ điều hành và mạng.

- **Hỗ trợ đa luồng (Multithreading)**  
  Dễ xây dựng Server phục vụ nhiều Client cùng lúc.

- **Xử lý ngoại lệ (Exception Handling)**  
  Buộc lập trình viên xử lý lỗi mạng ngay từ khi viết code,
  hình thành thói quen lập trình cẩn thận.

---

## Kết luận

Java là nền tảng vững chắc để bắt đầu học lập trình mạng,
từ các ứng dụng Client–Server cơ bản
đến những hệ thống phân tán và Backend quy mô lớn.
