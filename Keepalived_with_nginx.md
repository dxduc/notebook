# Cài đặt và cấu hình Keepalive với nginx reverse proxy
## Tổng quan về Keepalive
- Keepalive là một phần mềm mã nguồn mở trên Linux dùng để:
    - Đảm bảo tính sẵn sàng cao (High Availability) cho dịch vụ
    - Tự động phát hiện lỗi và chuyển đổi (failover) giữa các server
    - Thường kết hợp với VRRP để chia sẻ địa chỉ IP ảo (Virtual IP – VIP)