# Các câu lệnh xem log
## Dùng `tail` để xem file log
```sh
# Xem log theo thời gian thực, khi nào có log mới sẽ hiện ra
tail -f ten_file
# Theo dõi nhiều file một lúc
tail -f ten_file_1 ten_file_2
```
```sh
# Xem bao nhiêu dòng log mới nhất ở cuối file
tail -n 100 ten_file
```
## Dùng `head` để xem log

## Dùng `cat` để xem log
- Lệnh này xem toàn bộ nội dung file, in toàn bộ nội dung từ đầu tới cuối
```sh
cat ten_file
```
```sh
# xem nội dung file đã được nén mà không cần giải nén
zcat ten_file
```
## Lệnh `tac`
- Lệnh này in file ngược dòng (từ cuối tới đầu)
## Lệnh `more`hiển thị file theo trang, cuộn xuống được
## Lệnh `less` trình đọc file tương tác mạnh, cuộn lên xuống, tìm kiếm,