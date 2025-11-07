# Chaper 1: Quản lý user
## Tạo một user mới
### Dùng lệnh đơn giản
```sh
sudo adduser ten_user
```
- Khi dùng lệnh này, hệ thống sẽ yêu cầu nhập mật khẩu, tên đầy đủ
- Tự tạo home directory, set quyền, shell, group
### Dùng lệnh thủ công (ít thân thiện)
```sh
sudo useradd -m -s /bin/bash -c "Họ và tên" ten_user
sudo passwd duc
```
- `-m` = tạo thư mục home

- `-s` = chỉ shell đăng nhập

- `-c` = comment/GECOS

- `passwd` = đặt mật khẩu

### Hệ thống xử lý những gì?
- Khi tạo một user mới thì các file hệ thống liên quan đến danh tính người dùng được cập nhật đồng bộ.
- Ghi dòng mới vào `/etc/passwd`
Cấu trúc: 
```
username:password_placeholder:UID:GID:comment:home_directory:login_shell
```
Ví dụ:
```
duc:x:1001:1001:Duong Xuan Duc,,,:/home/duc:/bin/bash
```
Hệ thống cấp UID mới (1001) và GID chính (1001), rồi ghi vào `/etc/passwd`
`x` → mật khẩu hash nằm trong `/etc/shadow`.
`/home/duc` → thư mục home mặc định.
`/bin/bash` → shell đăng nhập.

- Ghi hash mật khẩu vào `/etc/shadow`. Chỉ root mới đọc được file.
- Tạo group mới trong `/etc/group`. Theo mặc định, hệ thống tạo group trùng tên user
- Tạo thư mục home `/home/<user>`
Nếu bạn tạo user với `useradd` mà không có `-m`, thư mục `home` sẽ không được tạo.
Copy nội dung mẫu từ `/etc/skel`/ (thường chứa `.bashrc`, `.profile`, `.bash_logout`)
- Gán shell mặc định
`/etc/passwd` chỉ định shell.
Nếu bạn dùng adduser, shell mặc định lấy từ `/etc/adduser.conf`:
```
DSHELL=/bin/bash
```
Bạn có thể đổi shell sau:
```sh
sudo chsh -s /bin/zsh duc
```

## Xóa user
```sh
sudo deluser <username>
```
- Một số option khi xóa 
`--remove-home` → xóa thư mục trong `home`

`--remove-all-files` → xóa toàn bộ file user đó sở hữu (trên toàn hệ thống)

`--backup` → tạo file nén backup home trước khi xóa (.tar.gz trong /var/backups/)