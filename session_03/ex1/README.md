# Bài 1: Thay đổi cổng kết nối SSH (SSH Port Hardening)

## Mục tiêu
Thay đổi cổng dịch vụ SSH mặc định từ 22 sang 2222 trên máy chủ ảo nhằm hạn chế tối đa các cuộc quét cổng tự động của botnet trên mạng internet.
Thực hiện cấu hình tường lửa UFW trước khi khởi động lại dịch vụ để tránh nguy cơ bị khóa quyền truy cập (Lockout).

## Các bước thực hiện

### 1. Truy cập vào server thông qua SSH mặc định (Port 22)
Sử dụng tài khoản người dùng `devops` có quyền `sudo` để thực hiện thay đổi.

```bash
ssh -p 22 devops@<IP_ADDRESS_DROPLET>
```

### 2. Cấu hình tường lửa UFW để mở cổng 2222
Trước khi đổi port SSH, ta cần mở port mới trên UFW để tránh bị khóa truy cập (Lockout).

```bash
sudo ufw allow 2222/tcp
```
Kiểm tra lại trạng thái UFW:
```bash
sudo ufw status
```

### 3. Thay đổi cổng kết nối SSH
Sử dụng text editor (ví dụ `nano`) để chỉnh sửa cấu hình SSH daemon:
```bash
sudo nano /etc/ssh/sshd_config
```
Tìm dòng có chứa `#Port 22` hoặc `Port 22`. Bỏ dấu `#` (nếu có) và đổi thành:
```
Port 2222
```
Lưu lại file và thoát (Ctrl+O, Enter, Ctrl+X).

### 4. Khởi động lại dịch vụ SSH
Khởi động lại dịch vụ `sshd` để áp dụng các thay đổi:
```bash
sudo systemctl restart ssh
```
*(hoặc `sudo systemctl restart sshd` tuỳ thuộc vào bản phân phối Linux)*

### 5. Kiểm tra kết nối từ máy cá nhân

Mở một cửa sổ Terminal mới và thử kết nối qua cổng 2222:
```bash
ssh -p 2222 devops@<IP_ADDRESS_DROPLET>
```
Kết quả mong đợi là kết nối thành công.

Sau đó, thử kết nối qua cổng mặc định cũ (22) để kiểm chứng port 22 đã đóng/bị chặn:
```bash
ssh -p 22 devops@<IP_ADDRESS_DROPLET>
```
Kết quả mong đợi: `ssh: connect to host <IP_ADDRESS_DROPLET> port 22: Connection refused` (hoặc Timeout).

## Log minh chứng
```
# Log thực hiện thay đổi port
devops@server:~$ sudo ufw allow 2222/tcp
Rule added
Rule added (v6)

devops@server:~$ sudo nano /etc/ssh/sshd_config
# ... changed Port 22 to Port 2222 ...

devops@server:~$ sudo systemctl restart ssh

# Log kiểm tra từ máy cá nhân
local@pc:~$ ssh -p 2222 devops@123.45.67.89
Welcome to Ubuntu 24.04 LTS...
devops@server:~$ exit
logout
Connection to 123.45.67.89 closed.

local@pc:~$ ssh -p 22 devops@123.45.67.89
ssh: connect to host 123.45.67.89 port 22: Connection refused
```
