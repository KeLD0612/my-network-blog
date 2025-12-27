---
title: "Node.js và mô hình bất đồng bộ"
date: 2025-01-05
cover: "/images/Nodejs-asynchronousModel.jpg"
---

Khi mới chuyển từ các ngôn ngữ server-side truyền thống như PHP hay Java sang Node.js, điều khiến tôi bối rối nhất không phải là cú pháp JavaScript (vì nó quá quen thuộc ở frontend), mà chính là tư duy "Bất đồng bộ" (Asynchronous).

Node.js không đơn thuần là đem JavaScript lên server chạy. Nó là một môi trường runtime được xây dựng trên nền tảng V8 engine cực mạnh của Google, nhưng "vũ khí bí mật" giúp nó xử lý hàng nghìn kết nối đồng thời lại nằm ở kiến trúc đơn luồng (Single-threaded) kết hợp với Event Loop.
<!--more-->
Hôm nay, hãy cùng tôi mổ xẻ xem mô hình này hoạt động như thế nào và tại sao nó lại thay đổi cách chúng ta xây dựng các ứng dụng mạng.

## Câu chuyện về "Người phục vụ" (The Waiter Analogy)

Để hiểu rõ nhất về sự khác biệt giữa mô hình truyền thống (Blocking) và Node.js (Non-blocking), tôi thường dùng ví dụ về một quán cà phê.

**Mô hình truyền thống (Multi-threaded/Blocking):**
Hãy tưởng tượng một quán cà phê mà mỗi khi có khách bước vào, quán phải thuê riêng một nhân viên phục vụ để chăm sóc khách đó từ A-Z. Nhân viên này order, đứng chờ pha chế làm xong, rồi bưng ra, rồi chờ khách thanh toán. Trong lúc chờ pha chế, nhân viên này đứng chơi và không làm gì cả. Nếu có 100 khách, quán cần 100 nhân viên. Đây là cách các server truyền thống tạo ra một Thread mới cho mỗi request. Tài nguyên bị lãng phí rất lớn cho việc chờ đợi (I/O blocking).

**Mô hình Node.js (Single-threaded/Non-blocking):**
Quán chỉ có **một** nhân viên phục vụ duy nhất tại quầy (đây chính là Main Thread).
1. Khách A vào order. Nhân viên ghi nhận, chuyển phiếu xuống bếp (System Kernel/Worker Pool), và ngay lập tức quay sang hỏi khách B: "Anh dùng gì ạ?".
2. Khách B order. Nhân viên lại chuyển phiếu xuống bếp, rồi quay sang khách C.
3. Khi bếp làm xong đồ uống của khách A, bếp bấm chuông (Callback/Event). Nhân viên phục vụ lúc này mới quay lại bưng đồ cho khách A.

Nhân viên phục vụ không bao giờ đứng chơi. Đó chính là cách Node.js hoạt động: Không bao giờ chặn luồng chính (Don't block the Event Loop).



## Event Loop: Trái tim của Node.js

Về mặt kỹ thuật, Node.js sử dụng cơ chế **Event Loop** để điều phối các tác vụ.

Khi có một yêu cầu nặng về I/O (như đọc file, truy vấn Database, gọi API bên thứ 3), Node.js sẽ không tự xử lý ngay tại luồng chính. Nó sẽ đẩy tác vụ đó xuống hệ thống (C++ APIs, libuv) để xử lý ngầm. Luồng chính tiếp tục nhận các yêu cầu khác.

Khi tác vụ ngầm hoàn tất, kết quả sẽ được đẩy vào một hàng đợi (Callback Queue). Event Loop có nhiệm vụ liên tục kiểm tra: "Luồng chính có đang rảnh không?". Nếu rảnh, nó sẽ lấy kết quả từ hàng đợi đưa lên để thực thi callback.

Đây là ví dụ minh họa bằng code để thấy sự khác biệt:

```javascript
const fs = require('fs');

// Cách viết Blocking (Đồng bộ) - Ít dùng trong Node.js
// Dòng code phía sau phải chờ dòng này chạy xong
const data = fs.readFileSync('/file.md'); 
console.log(data);
console.log('Dòng này chỉ chạy sau khi đọc file xong');

// Cách viết Non-blocking (Bất đồng bộ) - Chuẩn Node.js
fs.readFile('/file.md', (err, data) => {
    if (err) throw err;
    console.log(data); // Chạy sau cùng khi file đọc xong
});
console.log('Dòng này sẽ chạy NGAY LẬP TỨC, không cần chờ đọc file');

```

## Tại sao mô hình này lại phù hợp cho ứng dụng mạng lớn?
Trong quá trình phát triển các dự án thực tế, tôi nhận thấy Node.js tỏa sáng rực rỡ nhất ở các tác vụ I/O Bound (thiên về nhập xuất dữ liệu).

 - Tiết kiệm tài nguyên: Vì không phải tạo hàng nghìn thread cho hàng nghìn user, Node.js tiêu tốn rất ít RAM.

 - Tốc độ phản hồi cao: Ứng dụng không bị "treo" khi chờ dữ liệu từ database.

 - Realtime: Với các ứng dụng chat, thông báo, hay dashboard chứng khoán, việc duy trì kết nối liên tục (thông qua WebSocket) trên Node.js nhẹ nhàng hơn rất nhiều so với các nền tảng khác.

**Khi nào không nên dùng?**
Tuy nhiên, "không có viên đạn bạc". Kinh nghiệm xương máu của tôi là đừng dùng Node.js cho các tác vụ tính toán nặng (CPU Bound).

Ví dụ: Xử lý nén video, tính toán ma trận phức tạp. Vì Node.js chỉ có một luồng chính, nếu bạn bắt nó tính toán quá lâu, nó sẽ bận rộn với phép tính đó và không thể nhận request từ các user khác (giống như nhân viên phục vụ bị kẹt lại giải toán cho khách A, và cả quán phải chờ).

---

## Kết luận
Hiểu sâu về mô hình bất đồng bộ và Event Loop là chìa khóa để làm chủ Node.js. Nó đòi hỏi chúng ta thay đổi tư duy lập trình tuần tự sang tư duy sự kiện (Event-driven). Một khi đã nắm bắt được nó, bạn sẽ thấy việc xây dựng các hệ thống API chịu tải cao trở nên thú vị và hiệu quả hơn bao giờ hết.