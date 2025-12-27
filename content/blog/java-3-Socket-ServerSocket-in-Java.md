---
title: "Socket và ServerSocket trong Java"
date: 2025-01-03
---

Socket là thành phần cốt lõi trong lập trình mạng Java.
Nó đại diện cho một điểm kết nối giữa hai máy tính trong mạng.

`ServerSocket` được sử dụng ở phía server để lắng nghe kết nối từ client.
Khi client kết nối thành công, server tạo ra một đối tượng `Socket`
để giao tiếp với client đó.

Thông qua Socket, dữ liệu được truyền đi dưới dạng luồng
(InputStream và OutputStream).

Việc hiểu rõ Socket giúp lập trình viên xây dựng các ứng dụng mạng
ổn định và hiệu quả.
