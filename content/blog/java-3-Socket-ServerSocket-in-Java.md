---
title: "Socket và ServerSocket trong Java"
date: 2025-01-03
cover: "/images/Socket-ServerSocket-in-Java.jpg"
---

## Hiểu sâu về Socket trong Java: “Cánh cổng” của thế giới mạng

Chào các bạn,

Nếu bạn đang mày mò xây dựng các ứng dụng **Chat**, **game online**
hay đơn giản là muốn hai máy tính “nói chuyện” được với nhau,
chắc chắn bạn đã từng nghe tới khái niệm **Socket**.

Hồi mới học Java, mình từng nghĩ Socket là thứ gì đó rất cao siêu.
Nhưng thực tế, hãy tưởng tượng nó đơn giản như **một cái giắc cắm điện thoại** –
khi cắm đúng chỗ, hai bên có thể giao tiếp trực tiếp với nhau.

Hôm nay, mình muốn chia sẻ lại những gì mình đúc kết được
về thành phần cốt lõi này trong gói `java.net`.

<!--more-->

---

## 1. Socket và ServerSocket – Phân vai rõ ràng

Trong Java, bạn không thể dùng một class cho tất cả.
Chúng ta có **hai nhân vật chính**, mỗi người một nhiệm vụ.

### ServerSocket – “Ông chủ nhà hiếu khách”

`ServerSocket` chỉ tồn tại ở **phía Server**.

Nhiệm vụ của nó:
- Mở một cổng (*port*) cụ thể (ví dụ: **8080**)
- Ngồi chờ client kết nối bằng phương thức `accept()`

>**Kinh nghiệm thực tế**  
> Khi chương trình chạy đến `serverSocket.accept()`,
> server sẽ **đứng im hoàn toàn (blocking)** cho đến khi có client kết nối.
> Nếu console không in thêm gì, đừng hoảng — nó đang “lắng nghe” đấy.

---

### Socket – Cầu nối liên lạc

`Socket` được dùng cho **cả hai phía**:

- **Client**: tạo `Socket` để chủ động kết nối đến Server
- **Server**: khi có client kết nối,
  `ServerSocket` sẽ “đẻ” ra **một đối tượng Socket riêng**
  để giao tiếp với client đó

>**Tư duy hình ảnh**  
> `ServerSocket` giống như tổng đài viên trực ban.  
> Khi có cuộc gọi đến, tổng đài viên nối bạn sang
> một đường dây riêng (**Socket**) để nói chuyện,
> còn họ quay lại chờ cuộc gọi tiếp theo.

---

## 2. Dòng chảy dữ liệu (Streams) – Câu chuyện cái ống nước

Có kết nối Socket rồi, nhưng làm sao để gửi tin nhắn?

Java **không cho bạn ném thẳng một String**
từ máy A sang máy B.  
Thay vào đó, bạn phải dùng **Streams (Luồng dữ liệu)**.

### Hai luồng cơ bản
- **InputStream** – nhận dữ liệu (đọc)
- **OutputStream** – gửi dữ liệu (ghi)

> **Bài học xương máu**  
> Khi gửi dữ liệu text, thường dùng `PrintWriter`
> hoặc `BufferedWriter`.  
> **Luôn nhớ gọi `.flush()` sau khi ghi!**  
>
> Nếu không, dữ liệu sẽ nằm trong buffer,
> và phía bên kia sẽ… chờ mãi mà không thấy tin nhắn đâu.

---

## 3. Tại sao Socket lại quan trọng đến vậy?

Có thể bạn sẽ hỏi:

> “Giờ người ta dùng HTTP, REST API hết rồi,
> học Socket làm gì cho khổ?”

Thực ra:
> **HTTP cũng chạy trên nền Socket.**

Hiểu Socket giúp bạn:

### Hiểu bản chất
- Biết dữ liệu đi như thế nào
- Hiểu vì sao lại có độ trễ (*latency*)

### Debug tốt hơn
- Gặp lỗi `Connection refused` → biết có thể do Server chưa mở port
- Gặp `Timeout` → nghĩ ngay tới firewall hoặc network

### Làm chủ Real-time
- Ứng dụng chat
- Game online
- Điều khiển thiết bị IoT
- Kết nối liên tục, tốc độ cao

Trong những trường hợp này,
Socket (TCP/UDP) **vẫn tối ưu hơn HTTP rất nhiều**.

---

## Tóm lại

Socket chính là **viên gạch đầu tiên**
để xây dựng mọi ứng dụng mạng.

Chỉ cần nắm vững:
- `Socket`
- `ServerSocket`
- `InputStream` / `OutputStream`

là bạn đã nắm được **khoảng 50% sức mạnh**
của lập trình mạng trong Java.

Ở bài sau, mình sẽ chia sẻ về  
**xử lý đa luồng (Multi-threading)** –  
bí kíp để Server có thể phục vụ **hàng nghìn Client cùng lúc**
mà không bị “đơ”.
