# 🧭 Bảng Mã Lỗi HTTP và Ý Nghĩa

## 🟦 1xx – *Informational (Thông tin)*
> Máy chủ đã nhận yêu cầu và đang tiếp tục xử lý.

| Mã | Tên | Ý nghĩa |
|----|------|---------|
| **100** | Continue | Máy chủ đã nhận phần đầu của yêu cầu, client nên tiếp tục gửi phần còn lại. |
| **101** | Switching Protocols | Máy chủ chấp nhận chuyển đổi giao thức (ví dụ: từ HTTP sang WebSocket). |
| **102** | Processing | (WebDAV) Máy chủ đang xử lý nhưng chưa có phản hồi cuối cùng. |
| **103** | Early Hints | Cho phép client tải trước tài nguyên (ví dụ: preload CSS/JS). |

---

## 🟩 2xx – *Success (Thành công)*
> Yêu cầu đã được xử lý thành công.

| Mã | Tên | Ý nghĩa |
|----|------|---------|
| **200** | OK | Thành công, phản hồi chứa nội dung yêu cầu. |
| **201** | Created | Tạo tài nguyên mới thành công (thường dùng với `POST`). |
| **202** | Accepted | Yêu cầu được chấp nhận nhưng đang xử lý. |
| **203** | Non-Authoritative Information | Dữ liệu trả về không hoàn toàn chính xác (thông qua proxy). |
| **204** | No Content | Thành công, nhưng không có nội dung trả về. |
| **205** | Reset Content | Client nên reset form hoặc giao diện nhập liệu. |
| **206** | Partial Content | Trả về một phần dữ liệu (khi dùng `Range` header). |

---

## 🟨 3xx – *Redirection (Chuyển hướng)*
> Client cần thực hiện bước khác để hoàn thành yêu cầu.

| Mã | Tên | Ý nghĩa |
|----|------|---------|
| **300** | Multiple Choices | Có nhiều lựa chọn tài nguyên. |
| **301** | Moved Permanently | Tài nguyên đã được chuyển vĩnh viễn (redirect 301). |
| **302** | Found | Tạm thời chuyển hướng (redirect 302). |
| **303** | See Other | Chuyển hướng đến URL khác dùng `GET`. |
| **304** | Not Modified | Tài nguyên chưa thay đổi (dùng trong cache). |
| **307** | Temporary Redirect | Tạm thời chuyển hướng, giữ nguyên phương thức (POST → POST). |
| **308** | Permanent Redirect | Chuyển hướng vĩnh viễn, giữ nguyên phương thức. |

---

## 🟥 4xx – *Client Error (Lỗi phía Client)*
> Client gửi yêu cầu sai hoặc không được phép.

| Mã | Tên | Ý nghĩa |
|----|------|---------|
| **400** | Bad Request | Cú pháp yêu cầu không hợp lệ. |
| **401** | Unauthorized | Chưa được xác thực (cần login/token). |
| **402** | Payment Required | Dự phòng cho thanh toán (hiếm dùng). |
| **403** | Forbidden | Bị từ chối, không có quyền truy cập. |
| **404** | Not Found | Không tìm thấy tài nguyên. |
| **405** | Method Not Allowed | Phương thức HTTP không được phép (ví dụ: `POST` cho endpoint chỉ hỗ trợ `GET`). |
| **406** | Not Acceptable | Server không thể trả về dữ liệu theo định dạng client yêu cầu. |
| **407** | Proxy Authentication Required | Cần xác thực với proxy. |
| **408** | Request Timeout | Client gửi yêu cầu quá chậm, hết thời gian. |
| **409** | Conflict | Xung đột dữ liệu (ví dụ: ghi đè tài nguyên). |
| **410** | Gone | Tài nguyên đã bị xóa vĩnh viễn. |
| **411** | Length Required | Thiếu header `Content-Length`. |
| **412** | Precondition Failed | Điều kiện trong header (như `If-Match`) không đúng. |
| **413** | Payload Too Large | Dữ liệu gửi quá lớn. |
| **414** | URI Too Long | URL quá dài. |
| **415** | Unsupported Media Type | Kiểu dữ liệu không được hỗ trợ (ví dụ: `application/xml`). |
| **416** | Range Not Satisfiable | Yêu cầu phạm vi không hợp lệ (khi tải từng phần). |
| **417** | Expectation Failed | Header `Expect` không được đáp ứng. |
| **418** | I’m a teapot ☕ | Mã “đùa” theo RFC 2324 (April Fools). |
| **421** | Misdirected Request | Gửi yêu cầu đến sai server. |
| **422** | Unprocessable Entity | Dữ liệu hợp lệ về cú pháp nhưng sai logic (thường gặp trong API). |
| **423** | Locked | Tài nguyên bị khóa. |
| **424** | Failed Dependency | Một yêu cầu trước đó thất bại. |
| **425** | Too Early | Server từ chối xử lý sớm. |
| **426** | Upgrade Required | Cần nâng cấp giao thức (ví dụ: HTTP → HTTPS). |
| **429** | Too Many Requests | Gửi quá nhiều yêu cầu (rate limit). |
| **431** | Request Header Fields Too Large | Header quá lớn. |
| **451** | Unavailable For Legal Reasons | Bị chặn vì lý do pháp lý (VD: nội dung bị cấm). |

---

## ⬛ 5xx – *Server Error (Lỗi phía Server)*
> Máy chủ gặp lỗi trong khi xử lý.

| Mã | Tên | Ý nghĩa |
|----|------|---------|
| **500** | Internal Server Error | Lỗi chung của máy chủ. |
| **501** | Not Implemented | Server chưa hỗ trợ chức năng đó. |
| **502** | Bad Gateway | Proxy/Gateway nhận phản hồi lỗi từ backend. |
| **503** | Service Unavailable | Server đang quá tải hoặc bảo trì. |
| **504** | Gateway Timeout | Proxy không nhận được phản hồi từ backend. |
| **505** | HTTP Version Not Supported | Phiên bản HTTP không được hỗ trợ. |
| **507** | Insufficient Storage | Hết dung lượng lưu trữ. |
| **508** | Loop Detected | Phát hiện vòng lặp nội bộ khi xử lý. |
| **510** | Not Extended | Thiếu phần mở rộng bắt buộc trong yêu cầu. |
| **511** | Network Authentication Required | Cần xác thực mạng (ví dụ captive portal WiFi). |

---

📘 *Nguồn tham khảo:*  
- [RFC 7231](https://datatracker.ietf.org/doc/html/rfc7231)  
- [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status)
