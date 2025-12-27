---
title: "Node.js và mô hình bất đồng bộ"
date: 2025-01-05
---

Node.js là môi trường chạy JavaScript phía server,
được xây dựng trên nền tảng V8 của Google.

Điểm mạnh của Node.js là mô hình bất đồng bộ (asynchronous),
cho phép xử lý nhiều kết nối cùng lúc mà không bị chặn luồng.

Thay vì tạo nhiều thread như Java truyền thống,
Node.js sử dụng event loop để xử lý yêu cầu.

Mô hình này rất phù hợp cho các ứng dụng mạng có lượng truy cập lớn
như API server hoặc ứng dụng realtime.
