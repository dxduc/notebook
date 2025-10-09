# Install-LEMP-stack-Ubuntu 22.4-Wordpress
<b><i> Lưu ý: Hệ điều hành Ubuntu 22.4. Sử dụng gói cài đặt APT đã tích hợp những phiên bản phần mềm mới nhất và phù hợp với OS.</i></b>

## 1. Cài đặt Nginx:
### 1.1 Thêm nameserver
- Mục đích: Server có thể phân giải được tên miền để có thể tải và cài đặt các service có trong kho của OS
- Chỉnh sửa nội dung trong /etc/resolv.conf
```sh
nano /etc/resolv.conf
```
- Thêm các dòng sau vào file:
```sh
nameserver 8.8.8.8
nameserver 1.1.1.1
```
### 1.2 Cài đặt Nginx
```sh
sudo apt install nginx
```
Apt sẽ cài đặt phiên bản nginx mới nhất trong repo mà OS Ubuntu đang có sẵn (không phải phiên bản mới nhất thực tế). Phiên bản nginx cài bằng apt sẽ phù hợp OS đang sử dụng. Nếu muốn cài phiên bản khác 
- Kiểm tra phiên bản của nginx
```sh
nginx -v
```
Phiên bản đang sử dụng: <b>nginx/1.18.0</b>
- Mặc định nginx ban đầu chạy qua port 80
- Ubuntu ban đầu thiết lập không sử dụng firewalld mà sử dụng ufw
- Ban đầu, ufw không được enable nên không cần thiết lập gì thêm
- Nếu ufw được enable, cần cấu hình cho phép truy cập từ mọi IP
```sh
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
```

## 2. Cài đặt MariaDB
- Cài đặt
```sh
sudo apt install mariadb-server -y
```
- Apt sẽ cài đặt phiên bản MariaDB mới nhất trong repo mà OS Ubuntu đang có sẵn (không phải phiên bản mới nhất thực tế). Phiên bản MariaDB cài bằng apt sẽ phù hợp OS đang sử dụng
- Lần đầu khi cài đặt mariadb, user <b>root</b> dùng để truy cập DB đang không thiết lập password
- Kiểm tra phiên bản đang sử dụng
```sh
mysql --version
```
Phiên bản đang sử dụng: <b>mysql  Ver 15.1 Distrib 10.6.22-MariaDB, for debian-linux-gnu (x86_64) using  EditLine wrapper</b>
- Truy cập vào database
```sh
sudo mysql -u root
```
- Tạo database mới
```sh
CREATE DATABASE wordpress;
```
- Tạo user mới
```sh
CREATE USER 'wordpress'@'%' IDENTIFIED BY 'password';
```
`'wordpress'` : tên USER
`'%'` : cho phép truy cập DB từ mọi địa chỉ IP không chỉ mỗi localhost
`'password'` : mật khẩu truy cập vào DB
- Cấp quyền cho user
```sh
GRANT ALL PRIVILEGES ON *.* TO 'wordpress'@'%';
```
- Xác nhận
```sh
flush privileges;
```
- Thoát khỏi DB:
```sh
exit
```

## 3. Cài đặt php-fpm
- Khác với Apache (có mod_php) có thể xử lý trực tiếp mã PHP, Nginx không thể xử lý trực tiếp mã PHP, cần sử dụng một công cụ trung gian để xử lý mã PHP - <b>PHP-FPM</b>
- PHP-FPM (PHP - FastCGI Process Manager) là trình dùng để quản lý và xử lý tiến hình PHP. PHP-FPM có nhiệm vụ xử lý các yêu cầu PHP từ webserver (Nginx, Apache) và trả dữ liệu lại webserver thông qua socker unix hoặc TCP port
- Cài đặt
```sh
sudo apt install php-fpm
```
Thường sẽ cài đặt phiên bản 8.1
- Kiểm tra trạng thái
```sh
systemctl status php8.1-fpm.service
```
- Thay đổi cấu hình
```sh
nano /etc/php/8.1/fpm/pool.d/www.conf
```
- Thêm vào cuối file các dòng sau:
```conf
listen = 127.0.0.1:9000 # PHP-FPM lắng nghe trên TCP port 127.0.0.1:9000
listen.allowed_clients = 127.0.0.1 # chỉ cho phép lắng nghe các request gửi từ webserver có IP 127.0.0.1, chỉ dùng cho khi lắng nghe trên TCP port. Giới hạn IP được phép kết nối PHP-FPM (bảo mật)
listen.owner = www-data # user của webserver được phép kết nối vào PHP-FPM
listen.group = www-data # group user của webserver được phép kết nối vào PHP-FPM
listen.mode = 0660 # Đặt quyền (permission) cho file UNIX socket mà PHP-FPM tạo ra
slowlog = /var/log/php-fpm/www-slow.log # Đường dẫn đến file ghi lại log các script PHP chạy chậm khi bật cấu hình request_slowlog_timeout
php_admin_value[error_log] = /var/log/php-fpm/www-error.log # đường dẫn file ghi lỗi PHP cho pool này
php_admin_flag[log_errors] = on # Bật ghi log lỗi PHP
php_value[session.save_handler] = files # Chỉ định cách lưu session trong PHP. Lưu vào file trên ổ đĩa thay vì lưu vào memory, database, Redis,...
php_value[session.save_path] = /var/lib/php/session # thư mục lưu session file (đi kèm với session.save_handler = files
security.limit_extensions = .php .php3 .php4 .php5 .php # Giới hạn các phần mở rộng file (extensions) mà PHP-FPM cho phép thực thi
```
- Cập nhật giá trị ở phía trên:
```conf
pm = dynamic
pm.max_children = 50
pm.start_servers = 5
pm.min_spare_servers = 5
pm.max_spare_servers = 35
```
- Kết nối với webserver qua UNIX socket
```conf
listen = /run/php/php8.1-fpm.sock # kết nối với webserver qua UNIX socket
```
<b><i>nginx giao tiếp trực tiếp với PHP trên cùng một máy chủ qua socket unix có đường dẫn: /run/php/php8.1-fpm.sock</i></b> 

- Cài đặt thêm công cụ để truy vấn vào mysql
```sh
sudo apt install php8.1-mysql  
```
- Restart php-fpm
```sh
sudo systemctl restart php8.1-fpm
```

## 4. Cài đặt Wordpress
- Truy cập vào thư mục được dùng để chứa mã nguồn
```sh
cd /var/www/html
```
- Tải Wordpress về
```sh
sudo curl -O https://wordpress.org/latest.tar.gz
```
- Giải nén file
```sh
sudo tar -xvf latest.tar.gz
sudo rm latest.tar.gz
```
- Thiết lập wordpress
```sh
cd wordpress
sudo cp wp-config-sample.php wp-config.php
# Sửa file wp-config.php
sudo nano wp-config.php
```
Cập nhật thông tin database
```php
define('DB_NAME', 'wordpress');
define('DB_USER', 'wordpress');
define('DB_PASSWORD', 'password');
define('DB_HOST', 'localhost');
```

- Đặt lại quyền và user
```sh
sudo chown www-data:www-data -R wordpress
sudo chmod 755 -R wordpress
```

## 5. Cấu hình Nginx
- Cấu hình để truy cập trong mạng nội bộ, không có domain, IP Public. Và dùng địa chỉ IP Private để truy cập trên trình duyệt khi được kết nối trong mạng nội bộ
```sh
sudo nano /etc/nginx/sites-available/lemp.conf
```
Thêm nội dung sau
```conf
server {

    listen 80;
    
    server_name _; 
    
    root /var/www/html/wordpress;  #thư mục gốc chứa các tệp tĩnh như HTML, CSS, JavaScript, hình ảnh, video và các tài nguyên web khác.
    
    index index.html index.php; #thứ tự yêu tiên đọc file
    
    access_log /var/log/nginx/example.com.access.log;
    
    error_log /var/log/nginx/example.com.error.log;
    
    location / {
    
        try_files $uri $uri/ =404;
	
    }

    location ~ \.php$ {   
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock; 	
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; 
        fastcgi_index index.php; 
    }
} 	
```
- Cấu hình trên dùng để truy cập qua giao thức http với IP private trong mạng nội bộ
- Cấu hình chạy trong mạng nội bộ, không có domain nên không cần thiết lập ssl. Thiết lập lắng nghe qua port 80.
- Loại bỏ cấu hình mặc định:
```sh
sudo unlink /etc/nginx/sites-enabled/default
```
- Cập nhật cấu hình mới
```sh
sudo ln -s /etc/nginx/sites-available/lemp.conf /etc/nginx/sites-enabled/
```
- Restart nginx
```sh
sudo systemctl restart nginx
```
- Để truy cập qua giao thức https với IP private trong mạng nội bộ, cần thiết lập ssl cho server
- Cài đặt openssl để có thể tạo chứng chỉ
```sh
sudo apt install openssl -y
```
- Tạo thư mục lưu chứng chỉ và khóa
```sh
sudo mkdir /etc/nginx/ssl/
```
- Tạo chứng chỉ SSL và khóa riêng self-signed
```sh
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/nginx/ssl/nginx.key -out /etc/nginx/ssl/nginx.crt -subj "/C=VN/ST=Hanoi/L=Hanoi/O=MyCompany/OU=IT/CN=172.16.1.28" -addext "subjectAltName=IP:172.16.1.28"
```
- IP là IP Private trong mạng nội bộ, bắt buộc điền vào
- Cập nhật cấu hình vào file /etc/nginx/sites-available/lemp.conf
```sh
sudo nano /etc/nginx/sites-available/lemp.conf
```
- Thêm nội dung sau:
```conf
server {

    listen 443 ssl;
    
    server_name _;

    ssl_certificate /etc/nginx/ssl/nginx.crt;    
    ssl_certificate_key /etc/nginx/ssl/nginx.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
    ssl_prefer_server_ciphers on;
    
    root /var/www/html/wordpress;  #thư mục gốc chứa các tệp tĩnh như HTML, CSS, JavaScript, hình ảnh, video và các tài nguyên web khác.
    
    index index.html index.php; #thứ tự yêu tiên đọc file
    
    access_log /var/log/nginx/example.com.access.log;
   
    error_log /var/log/nginx/example.com.error.log;
    
    location / {
    
        try_files $uri $uri/ =404; # kiểm tra tuần tự xem file hoặc thư mục có tồn tại không
	
    }

    location ~ \.php$ {   # xử lý tất cả các yêu cầu đối với các URL có phần mở rộng .php.
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; 
        fastcgi_index index.php; 
    }

    error_page 497 https://$host$request_uri; #xử lý lỗi 497, chuyển hướng ng dùng từ http sang https
}

; server {		#chuyển hướng vĩnh viễn (301 redirect) từ HTTP sang HTTPS.
;     listen 80;  
;     server_name test.com www.test.com;
;     return 301 https://$host$request_uri; #host đại diện cho tên miền người dùng yêu cầu (ví dụ: test.com), và $request_uri là phần đường dẫn yêu cầu(ví dụ: /index.html)
; }
```
- Nginx hỗ trợ các location chính:
| Loại                                               | Cú pháp                       | Mô tả                                                                       |
| -------------------------------------------------- | ----------------------------- | --------------------------------------------------------------------------- |
| **Chính xác (Exact match)**                        | `location = /path { ... }`    | Chỉ khớp chính xác với `/path`. Không có thêm ký tự nào phía sau.           |
| **Bắt đầu bằng (Prefix match)**                    | `location /path { ... }`      | Khớp nếu URL bắt đầu bằng `/path`. Đây là loại phổ biến nhất.               |
| **Biểu thức chính quy (Regex match)**              | `location ~ pattern { ... }`  | Sử dụng **regex** (phân biệt chữ hoa–thường) để khớp URL.                   |
| **Biểu thức chính quy không phân biệt hoa–thường** | `location ~* pattern { ... }` | Giống như `~`, nhưng **không phân biệt chữ hoa–thường** (case-insensitive). |

| Dòng lệnh                                                        | Mô tả                                                                                                                 |
| -----------------------------------------------------------------| --------------------------------------------------------------------------------------------------------------------- |
| include fastcgi_params                                           | Nạp file cấu hình chuẩn chứa các biến mặc định của Nginx                                                              |
| fastcgi_pass unix:/run/php/php8.1-fpm.sock                       | Kết nối tới PHP-FPM. `unix:/run/php/php8.1-fpm.sock` dùng unix socket kết nối nội bộ trong cùng máy, tốc độ nhanh hơn |
| fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name | Thiết lập đường dẫn tuyệt đối tới file PHP thực tế trên máy chủ mà PHP-FPM cần chạy                                   |
| fastcgi_index index.php                                          | Đặt file mặc định khi truy cập thư mục                                                                                | 
- Lưu ý: 
|Biến         | Vai trò                                      |
|:-----------:|:--------------------------------------------:|
|$uri         |	Đường dẫn đã chuẩn hóa, không có query string|
|$request_uri |	Đường dẫn gốc client gửi, có query string    |
|$host        |	Tên miền (Server_name) gửi từ trình duyệt    |
- Reload nginx
```sh
sudo systemctl reload nginx
```
- Dùng openssl để tạo SSL thì để có thể xác thực trên trình duyệt thì máy khách cần thêm chứng chỉ trên máy khách.
- Lấy file /etc/nginx/ssl/nginx.crt rồi install trên máy khách.

### Nếu muốn truy cập ở mạng Internet ngoài cần có IP Public và domain để cấu hình các vitual host. Hỗ trợ chạy nhiều trang web trên cùng một máy chủ.
- Truy cập thư mục sau:
```sh
cd /etc/nginx/conf.d/
```
- Thêm mới các file cấu hình: Ví dụ server có IP Public và được trỏ tới tên miền test.com
- Tạo file cấu hình mới trong /etc/nginx/conf.d/
```sh
nano test.com.conf
```
- Thêm cấu hình server block 
```conf
server {

    listen 80;  #lắng nghe yêu cầu trên port 80 http
    
    server_name test.com;
    
    root /var/www/html/wordpress;  #thư mục gốc chứa các tệp tĩnh như HTML, CSS, JavaScript, hình ảnh, video và các tài nguyên web khác.
    
    index index.html index.htm index.php; #thứ tự yêu tiên đọc file
    
    access_log /var/log/nginx/example.com.access.log;
    
    error_log /var/log/nginx/example.com.error.log;
    
    location / {			#trong Nginx là một chỉ thị được sử dụng để xử lý tất cả các yêu cầu có đường dẫn bắt đầu bằng /.
    
        try_files $uri $uri/ =404;  # kiểm tra và trả về tệp yêu cầu nếu có, nếu không thì trả về lỗi 404.
	
    } 
}	
```
- Trường hợp có domain thì có thể cài đặt SSL với Let’s Encrypt
```sh
sudo apt install certbot python3-certbot-nginx -y
sudo certbot --nginx -d yourdomain.com
```
- Cập nhật lại file cấu hình ở thư mục /etc/nginx/conf.d/
```conf
server {

    listen 443 ssl;
    
    server_name test.com;

    ssl_certificate /etc/letsencrypt/live/test.com/fullchain.pem;    
    ssl_certificate_key /etc/letsencrypt/live/test.com/privkey.pem;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers 'HIGH:!aNULL:!MD5';
    ssl_prefer_server_ciphers on;
    
    root /var/www/html/wordpress;  #thư mục gốc chứa các tệp tĩnh như HTML, CSS, JavaScript, hình ảnh, video và các tài nguyên web khác.
    
    index index.html index.php; # thứ tự yêu tiên đọc file
    
    access_log /var/log/nginx/example.com.access.log;
   
    error_log /var/log/nginx/example.com.error.log;
    
    location / {
    
        try_files $uri $uri/ =404;
	
    }

    location ~ \.php$ {   
        include fastcgi_params;
        fastcgi_pass unix:/run/php/php8.1-fpm.sock; 	
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name; 
        fastcgi_index index.php; 
    }

    error_page 497 https://$host$request_uri; #xử lý lỗi 497, chuyển hướng ng dùng từ http sang https
}

server {		#chuyển hướng vĩnh viễn (301 redirect) từ HTTP sang HTTPS.
    listen 80;  
    server_name test.com;
    return 301 https://$host$request_uri; #host đại diện cho tên miền người dùng yêu cầu (ví dụ: test.com), và $request_uri là phần đường dẫn yêu cầu(ví dụ: /index.html)
}
```