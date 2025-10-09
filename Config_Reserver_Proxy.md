# Cài đặt và cấu hình Nginx dùng làm proxy và load balancer
- Hệ điều hành: Ubuntu 22.04.5 LTS
- Thiết lập mô hình revered proxy trỏ tới 2 server backend wordpress
## Chuẩn bị
- Clone một server wordpress từ server wordpress gốc
- Trỏ wordpress mới tới DB nằm trên server của wordpress gốc.
- Trên ***server wordpress gốc***, mở port 3306 trên database và cho phép truy cập từ IP ngoài
```sh
sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
```
- Tìm dòng `bind-address` thay `127.0.0.1` bằng `0.0.0.0`
```sh
sudo nano /etc/mysql/mariadb.cnf
```
- Bỏ comment ỏ dòng `port=3306`
- Restart lại database
```sh
sudo systemctl restart mariadb
```
## Tổng quan về nginx revered proxy

## 1. Cài đặt Nginx
```sh
sudo apt update
sudo apt install nginx -y
```
## 2. Cấu hình Nginx
- Cấu hình được thiết lập để chạy trong mạng nội bộ, với IP Private.
- Cài đặt openssl
```sh
sudo apt install openssl -y
```
- Tạo thư mục lưu chứng chỉ và khóa
```sh
sudo mkdir /etc/nginx/ssl/
```
- Tạo chứng chỉ SSL và khóa riêng self-signed
```sh
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt -subj "/C=VN/ST=Hanoi/L=Hanoi/O=MyCompany/OU=IT/CN=172.16.1.27" -addext "subjectAltName=IP:172.16.1.27"
```
- IP là IP Private của server trong mạng nội bộ, bắt buộc điền vào
- Cập nhật file config trong thư mục /etc/nginx/conf.d
```sh
sudo nano /etc/nginx/conf.d/proxy.conf
```
- Thêm nội dùng sau vào file:
```conf
upstream backend_servers {    #định ngĩa một nhóm server backend
    server 172.16.1.28:443;
    server 172.16.1.29:443;
}

server {
    listen 443 ssl;
    server_name _;

    # Đường dẫn đến chứng chỉ và khóa riêng
    ssl_certificate /etc/nginx/ssl/nginx.crt;
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    # Cấu hình SSL
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
    ssl_prefer_server_ciphers on;

    access_log /var/log/nginx/proxy_access.log upstreamlog; # Ghi lại log balancing

    location / {
    	# Chuyển tiếp yêu cầu tới nhóm máy chủ backend
    	proxy_pass https://backend_servers;

    	# Thiết lập các header cần thiết cho reverse proxy
    	proxy_set_header Host $host;    # Truyền thông tin domain tới backend
    	proxy_set_header X-Real-IP $remote_addr;    # Truyền địa chỉ IP của client
    	proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;    # Truyền thông tin của các proxy trung gian
    	proxy_set_header X-Forwarded-Proto $scheme;    # Truyền giao thức (HTTP hoặc HTTPS) tới backend
    }
}
```
- Loại bỏ site mặc định để cập nhật site mới. Do mình dùng IP để truy cập vào trình duyệt mà config mặc định đang sử dụng IP làm domain để truy cập.
```sh
sudo unlink /etc/nginx/sites-enabled/default
```
- Thêm cấu hình sau vào block `http` của file `/etc/nginx/nginx.conf` để xem đã balancing chưa
```conf
 log_format upstreamlog '$remote_addr - $remote_user [$time_local] '
                      '"$request" $status $body_bytes_sent '
                      '"$http_referer" "$http_user_agent" '
                      'to: $upstream_addr';
```
- Khởi động lại nginx
```sh
sudo systemctl restart nginx
```
- câu lệnh xem log
```sh
sudo tail -f /var/log/nginx/proxy_access.log
```
## 3. Thuật toán load balancer
### Round Robin (Mặc định)
- Phân phối luân phiên, tuần tự request tới các server
- Cấu hình:
```conf
upstream backend_servers {
    server 172.16.1.28;
    server 172.16.1.29;
}
```
### Weighted Round Robin (Cân đối theo trọng số)
- Phân phối theo tỷ lệ trọng số
- Cấu hình:
```conf
upstream backend_servers {
    server 172.16.1.28 weight=3; # Nhận 60% traffic
    server 172.16.1.29 weight=2; # Nhận 40% traffic
}
```
### Least Connections
- Request được gửi tới server có ít active connection nhất
- Cấu hình:
```conf
upstream backend_servers {
    least_conn;
    server 172.16.1.28;
    server 172.16.1.29;
}
```
### IP Hash
- Dựa trên IP của client, Nginx chuyển các request của 1 client chỉ tới 1 server.
- Cấu hình:
```conf
upstream backend_servers {
    ip_hash;
    server 172.16.1.28;
    server 172.16.1.29;
}
```

