---
title: "Mô hình Client – Server trong Java"
date: 2025-01-02
---

Mô hình Client – Server là kiến trúc phổ biến nhất trong lập trình mạng.
Trong mô hình này, client gửi yêu cầu và server xử lý, sau đó trả về kết quả.

Trong Java, server thường được xây dựng bằng lớp `ServerSocket`,
trong khi client sử dụng lớp `Socket` để kết nối.

Server sẽ mở một cổng (port) và chờ client kết nối.
Khi có kết nối, server tạo một luồng riêng để xử lý,
giúp phục vụ nhiều client cùng lúc.

Mô hình này được ứng dụng rộng rãi trong web server,
chat application và các hệ thống quản lý dữ liệu phân tán.
