---
title: "WebSocket và ứng dụng realtime"
date: 2025-01-08
cover: "/images/js-network-WebSocket-realtime.jpg"
---

Trong các bài viết trước, chúng ta đã bàn về mô hình Request-Response tiêu chuẩn của HTTP. Client hỏi, Server trả lời. Hết chuyện. Mô hình này tuyệt vời cho việc lướt web, đọc tin tức hay gọi API lấy dữ liệu.

Nhưng thế giới web hiện đại đòi hỏi nhiều hơn thế. Người dùng muốn chat và nhận tin nhắn ngay lập tức mà không cần F5. Người chơi game online cần thấy đối thủ di chuyển ngay khi họ bấm nút. Các bảng điều khiển tài chính cần nhảy số theo từng mili-giây.
<!--more-->
Đó là lúc mô hình HTTP truyền thống bộc lộ điểm yếu, và WebSocket xuất hiện như một vị cứu tinh. Hôm nay, tôi sẽ chia sẻ về cách tôi ứng dụng WebSocket để giải quyết bài toán thời gian thực (Real-time).

## Tại sao lại cần WebSocket?

Trước khi WebSocket trở nên phổ biến, để làm chức năng thông báo hay chat, tôi thường phải dùng kỹ thuật **Polling** (hoặc Long-polling).

Cơ chế của Polling khá "ngây thơ": Client cứ mỗi 1-2 giây lại gửi một request lên Server hỏi: *"Có tin nhắn mới không?"*.

* Nếu không: Server trả về "Không".

* Nếu có: Server trả về tin nhắn.

Cách làm này tạo ra hàng nghìn request thừa thãi, gây lãng phí băng thông và tài nguyên server kinh khủng, đồng thời độ trễ vẫn cao.

WebSocket thay đổi hoàn toàn cuộc chơi. Nó thiết lập một đường ống kết nối hai chiều bền vững (persistent connection) giữa Client và Server. Một khi kết nối được thiết lập (thông qua quá trình bắt tay - Handshake), cả hai bên có thể đẩy dữ liệu cho nhau bất cứ lúc nào mà không cần chờ bên kia hỏi.

## Giao thức ws:// và wss://

Khác với `http://` hay `https://`, WebSocket sử dụng giao thức `ws://` (hoặc `wss://` cho kết nối bảo mật).

Điểm thú vị là WebSocket bắt đầu cuộc đời của mình bằng một HTTP Request thông thường. Client gửi yêu cầu với header `Upgrade: websocket`. Nếu Server chấp nhận, kết nối HTTP sẽ được "nâng cấp" lên thành WebSocket và giữ nguyên trạng thái kết nối đó cho đến khi một trong hai bên ngắt kết nối.

## Triển khai với Node.js và Socket.io

Dù JavaScript trình duyệt có sẵn API `WebSocket`, và Node.js có thư viện `ws` rất mạnh, nhưng trong các dự án thực tế, tôi ưu tiên sử dụng **Socket.io**.

Socket.io là một thư viện bọc bên ngoài WebSocket. Nó giải quyết giúp tôi những vấn đề đau đầu:

1.  **Tự động kết nối lại (Auto-reconnect):** Khi mạng chập chờn, client tự tìm cách kết nối lại.
2.  **Fallback:** Nếu trình duyệt quá cũ hoặc mạng chặn WebSocket, nó tự động chuyển về Long-polling để đảm bảo ứng dụng vẫn chạy.
3.  **Room & Namespace:** Hỗ trợ chia phòng chat cực dễ dàng.

Dưới đây là ví dụ đơn giản về một Chat Server tôi thường dựng nhanh bằng Socket.io:

**Server (Node.js):**

```javascript
const express = require('express');
const http = require('http');
const { Server } = require("socket.io");

const app = express();
const server = http.createServer(app);
const io = new Server(server);

io.on('connection', (socket) => {
    console.log('Một người dùng đã kết nối: ' + socket.id);

    // Lắng nghe sự kiện 'chat message' từ client
    socket.on('chat message', (msg) => {
        // Gửi tin nhắn này đến TẤT CẢ mọi người đang kết nối
        io.emit('chat message', msg);
    });

    socket.on('disconnect', () => {
        console.log('Người dùng đã thoát');
    });
});

server.listen(3000, () => {
    console.log('Listening on *:3000');
});
Client (JavaScript):

```

```JavaScript

const socket = io();

// Gửi tin nhắn đi
function sendMessage(text) {
    socket.emit('chat message', text);
}

// Nhận tin nhắn về
socket.on('chat message', (msg) => {
    console.log("Tin nhắn mới: ", msg);
    // Cập nhật giao diện tại đây
});

```

## Thách thức khi vận hành hệ thống Real-time

Lập trình WebSocket thú vị, nhưng vận hành nó ở quy mô lớn (Production) lại là câu chuyện khác. Dưới đây là hai vấn đề lớn tôi từng gặp phải:

1. Vấn đề quy mô (Scaling)
Khi lượng người dùng tăng lên 100.000, một server không thể chịu nổi. Tôi phải chạy nhiều server (Cluster). Vấn đề nảy sinh: Người dùng A kết nối tới Server 1, người dùng B kết nối tới Server 2. Khi A gửi tin nhắn, chỉ Server 1 biết. Server 2 không biết gì cả, nên người dùng B không nhận được tin nhắn.

Giải pháp: Sử dụng Redis Pub/Sub làm trung gian. Khi Server 1 nhận tin nhắn, nó "bắn" tin đó vào Redis. Redis sẽ phân phối tin đó cho tất cả các Server còn lại. Socket.io hỗ trợ socket.io-redis-adapter để làm việc này rất mượt.

2. Quản lý trạng thái kết nối
Không giống như HTTP (gửi xong là quên), WebSocket yêu cầu server phải nhớ ai đang kết nối. Việc phát hiện "kết nối ma" (kết nối bị đứt nhưng server vẫn nghĩ là đang online) rất quan trọng để giải phóng tài nguyên bộ nhớ. Cơ chế Heartbeat (Ping/Pong) là bắt buộc để kiểm tra sức khỏe kết nối định kỳ.

---
## Lời kết

WebSocket mở ra cánh cửa cho các ứng dụng tương tác cao mà HTTP truyền thống không thể làm tốt. Nếu bạn đang định làm app chat, game online, hay hệ thống theo dõi giá coin, chứng khoán, thì WebSocket cùng với Node.js là bộ đôi hoàn hảo để bắt đầu.