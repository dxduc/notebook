# Cài win bằng USB Boot
### Tạo USB Boot
- Tải file .iso của window về
- Dùng phần mềm `Rufus` để tạo USB Boot
### Cài win
- Cắm USB vào máy và khởi động lại máy. Bấm F12 (Đối với máy Dell) để vào Menu Boot. Các dòng máy khác có cách khác
- Chọn USB chứa dữ liệu để boot win
- Thiết lập theo từng bước trên màn hình
- ***Lưu ý:*** có thể xảy ra lỗi, không đọc được ổ đĩa cứng trên máy => Khởi động lại máy và bấm F2 (DELL) để vào BIOS => Vào tag “storage”, tích vào “AHCI” rồi lưu lại thay đổi.
- Khởi động lại máy và boot lại thì sẽ hiện được ổ cứng của máy
- Có nhiều phân vùng ở ổ cứng cũ, xóa các phân vùng cũ của ổ cứng và tạo phân vùng mới. Chọn phân vùng mới tạo đó để boot lại win
- Boot xong thì rút usb ra tránh bị chạy lại từ đầu
- Win mới cài có thể thiếu driver. Cắm dây mạng (nếu chưa có driver wifi) chạy update win để cập nhật lại các driver