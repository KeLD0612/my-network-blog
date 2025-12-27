---
title: "HTTP và REST API với JavaScript"
date: 2025-01-06
cover: "/images/HTTP-REST_API-with-JavaScript.jpg"
---

Trong hành trình trở thành một lập trình viên Fullstack hoặc Backend, việc thấu hiểu giao thức HTTP và kiến trúc REST là bước ngoặt quan trọng nhất. Nếu như Socket là nền tảng cho các kết nối thời gian thực và liên tục, thì HTTP chính là "ngôn ngữ" tiêu chuẩn để các hệ thống trao đổi dữ liệu rời rạc trên Internet.

Hôm nay, tôi sẽ chia sẻ cách tôi sử dụng JavaScript (cụ thể là Node.js) để làm chủ giao thức này và xây dựng các RESTful API chuẩn mực.

## HTTP: Không chỉ là duyệt web

Trước đây, tôi thường nghĩ HTTP chỉ đơn giản là việc trình duyệt gửi yêu cầu và server trả về trang HTML. Nhưng khi đi sâu vào lập trình mạng, tôi nhận ra HTTP là một giao thức truyền tải cực kỳ linh hoạt. Nó không chỉ tải hình ảnh hay văn bản, mà còn là xương sống để các ứng dụng mobile, các microservices, hay các thiết bị IoT giao tiếp với nhau.

Trong JavaScript (Node.js), module `http` tích hợp sẵn cho phép chúng ta tạo ra server ở mức thấp nhất. Tuy nhiên, để xử lý các logic phức tạp như routing (định tuyến) hay middleware, tôi luôn chọn **Express.js** làm người bạn đồng hành.

## Tư duy về RESTful API

REST (Representational State Transfer) không phải là một công nghệ, nó là một tập hợp các nguyên tắc kiến trúc. Khi thiết kế API, sai lầm phổ biến của người mới là tư duy theo "hành động" (ví dụ: `/createUser`, `/deleteProduct`).

Với REST, tư duy của tôi chuyển sang hướng **Tài nguyên (Resource)**. Mọi thứ là danh từ, và hành động được định nghĩa bởi các phương thức HTTP (HTTP Verbs).

Giả sử tôi quản lý danh sách người dùng (users), các Endpoint sẽ trông như thế này:

* **GET** `/users`: Lấy danh sách tất cả người dùng.
* **GET** `/users/123`: Lấy thông tin chi tiết người dùng có ID 123.
* **POST** `/users`: Tạo mới một người dùng (dữ liệu nằm trong Body).
* **PUT** `/users/123`: Cập nhật toàn bộ thông tin người dùng 123.
* **DELETE** `/users/123`: Xóa người dùng 123.

## Triển khai với Express.js

Express.js giúp việc hiện thực hóa các lý thuyết trên trở nên cực kỳ ngắn gọn. Dưới đây là cấu trúc mẫu mà tôi thường sử dụng để khởi tạo một REST API server đơn giản:

```javascript
const express = require('express');
const app = express();
const port = 3000;

// Middleware để server hiểu được dữ liệu JSON từ client gửi lên
app.use(express.json());

// Giả lập Database
let users = [
    { id: 1, name: "Nguyen Van A" },
    { id: 2, name: "Tran Thi B" }
];

// 1. GET: Lấy dữ liệu
app.get('/api/users', (req, res) => {
    res.status(200).json({
        message: "Lấy danh sách thành công",
        data: users
    });
});

// 2. POST: Tạo mới dữ liệu
app.post('/api/users', (req, res) => {
    const newUser = req.body;
    // Trong thực tế cần validate dữ liệu kỹ càng ở bước này
    users.push(newUser);
    
    // Status 201 nghĩa là "Created" - Đã tạo thành công
    res.status(201).json({
        message: "Tạo người dùng thành công",
        data: newUser
    });
});

app.listen(port, () => {
    console.log(`REST API đang chạy tại http://localhost:${port}`);
});
```
## Kinh nghiệm xương máu khi làm REST API
Sau nhiều dự án triển khai API cho cả web và mobile app, tôi rút ra được vài kinh nghiệm quan trọng để API của bạn trở nên chuyên nghiệp hơn:

1. Sử dụng đúng HTTP Status Code
Đừng bao giờ trả về HTTP 200 (OK) cho mọi trường hợp. Client cần biết chính xác chuyện gì đang xảy ra thông qua mã lỗi:

 - 200: Thành công.

 - 201: Tạo mới thành công (dùng cho POST).

 - 400: Bad Request (Lỗi do client gửi dữ liệu sai định dạng).

 - 401: Unauthorized (Chưa đăng nhập).

 - 403: Forbidden (Đã đăng nhập nhưng không có quyền truy cập).

 - 404: Not Found (Không tìm thấy tài nguyên).

 - 500: Internal Server Error (Lỗi logic phía server).

2. Stateless (Phi trạng thái)
REST API nên là phi trạng thái. Điều này có nghĩa là server không nên lưu trạng thái của client (như session) trong bộ nhớ. Mỗi request gửi lên phải chứa đầy đủ thông tin để server xác thực (thường là qua Token trong Header). Điều này giúp hệ thống dễ dàng mở rộng (scale) ra nhiều server khác nhau mà không lo mất đồng bộ dữ liệu phiên làm việc.

3. Quy chuẩn dữ liệu JSON
Luôn trả về dữ liệu dưới dạng JSON và thống nhất cấu trúc phản hồi. Ví dụ, dù thành công hay thất bại, tôi luôn cố gắng giữ một format chung để phía Frontend dễ xử lý:

```JSON

{
  "success": true,
  "data": { ... },
  "error": null
}

```

---

## Lời kết
JavaScript kết hợp với Node.js và Express đã đơn giản hóa rất nhiều việc xây dựng REST API. Tuy nhiên, việc viết code chạy được là chưa đủ, viết code đúng chuẩn REST và dễ bảo trì mới là đích đến của một lập trình viên backend thực thụ. Hy vọng những chia sẻ này giúp bạn có cái nhìn rõ ràng hơn khi bắt tay vào xây dựng API của riêng mình.