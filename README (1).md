# Bài 1: Khởi tạo máy chủ Ubuntu trên Cloud & kết nối bằng SSH Key

## Mục tiêu
- Khởi tạo một máy chủ ảo (VPS) chạy Ubuntu Server trên hạ tầng đám mây, cấu hình tối giản để tiết kiệm chi phí.
- Cấu hình xác thực bằng cặp khóa SSH (SSH Keypair) và kết nối từ máy cá nhân bằng tài khoản `root` mà **không cần nhập mật khẩu**.

> **Ghi chú:** Bài thực hành dùng nhà cung cấp **AZVPS** (máy chủ đặt tại Việt Nam) thay cho DigitalOcean. Quy trình thực hiện tương đương: tạo VPS, tạo SSH key, đưa public key lên máy chủ, rồi SSH bằng private key.

## Thông tin máy chủ

| Thông tin | Giá trị |
|---|---|
| Nhà cung cấp | AZVPS (portal.azvps.vn) |
| Gói dịch vụ | Cheap 2 – 100.000 đ/tháng (gói tối giản) |
| Hệ điều hành | Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-198-generic x86_64) |
| Ổ đĩa | 19.59 GB SSD |
| Hostname | `wean2607` |
| IPv4 | `160.187.229.88` |
| Tài khoản | `root` |
| Xác thực | SSH Key **ed25519** |
| Máy cá nhân | Windows 10 – Command Prompt (OpenSSH Client có sẵn) |

---

## Các bước thực hiện

### Bước 1 – Khởi tạo VPS
1. Đăng ký tài khoản tại portal.azvps.vn.
2. Vào **Đăng ký dịch vụ**, chọn VPS gói **Cheap 2** và hệ điều hành **Ubuntu 22.04 LTS**.
3. Thanh toán. Sau khi VPS được cấp, vào **Quản lý dịch vụ → VPS** để lấy địa chỉ IP và mật khẩu `root` ban đầu.

![Trang quản lý VPS](images/01-azvps-dashboard.png)

### Bước 2 – Đăng nhập lần đầu & cập nhật hệ thống
Lần đầu đăng nhập bằng mật khẩu `root` mà nhà cung cấp cấp:

```bat
ssh root@160.187.229.88
```

Sau khi vào được máy chủ, cập nhật các gói phần mềm:

```bash
apt update && apt upgrade -y
```

Kết quả: hệ thống được nâng cấp từ Ubuntu 22.04.1 lên **22.04.5 LTS**, kernel lên `5.15.0-198`. Khởi động lại máy để dùng kernel mới:

```bash
reboot
```

### Bước 3 – Tạo cặp khóa SSH trên máy cá nhân
Chạy trên máy cá nhân (Windows cmd):

```bat
ssh-keygen -t ed25519 -C "phanhtran0501@gmail.com"
```

Nhấn Enter để giữ đường dẫn mặc định và để trống passphrase. Lệnh tạo ra 2 file:
- `C:\Users\ADMIN\.ssh\id_ed25519` là **private key**, chỉ giữ trên máy cá nhân.
- `C:\Users\ADMIN\.ssh\id_ed25519.pub` là **public key**, dùng để đưa lên máy chủ.

![Tạo SSH key](images/02-ssh-keygen.png)

### Bước 4 – Đưa Public Key lên máy chủ
Lệnh dưới đây gửi nội dung public key lên VPS và ghi vào `~/.ssh/authorized_keys`, đồng thời phân quyền đúng cho thư mục và file. Đây là lần cuối cùng cần nhập mật khẩu.

```bat
type %USERPROFILE%\.ssh\id_ed25519.pub | ssh root@160.187.229.88 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
```

### Bước 5 – Kết nối bằng SSH Key

```bat
ssh -i %USERPROFILE%\.ssh\id_ed25519 root@160.187.229.88
```

Kết quả: vào thẳng máy chủ, **không bị hỏi mật khẩu**, và Terminal hiển thị thông điệp chào mừng của Ubuntu Server.

![Kết nối SSH bằng key thành công](images/03-ssh-key-success.png)

## Log kết nối thành công

Log dưới đây được ghi sau khi đã cập nhật và khởi động lại máy chủ:

```text
C:\Users\ADMIN>ssh -i %USERPROFILE%\.ssh\id_ed25519 root@160.187.229.88
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-198-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Thu Oct  1 08:47:26 PM +07 2026

  System load:  0.12               Processes:             99
  Usage of /:   26.2% of 19.59GB   Users logged in:       0
  Memory usage: 9%                 IPv4 address for eth0: 160.187.229.88
  Swap usage:   0%

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Last login: Thu Oct  1 20:36:58 2026 from 171.237.167.12
root@wean2607:~#
```

## Đối chiếu yêu cầu

| Yêu cầu | Kết quả |
|---|---|
| Ubuntu 22.04 LTS (hoặc mới hơn) | ✅ Ubuntu 22.04.5 LTS, đã cập nhật đầy đủ (0 updates) |
| Cấu hình tối giản, tiết kiệm chi phí | ✅ Gói Cheap 2 |
| Datacenter gần người dùng | ✅ Máy chủ đặt tại Việt Nam |
| Xác thực bằng SSH Key thay vì mật khẩu | ✅ ed25519 |
| SSH bằng `root` từ máy cá nhân, không cần mật khẩu | ✅ Xem log ở trên |
| Hiển thị thông điệp chào mừng của Ubuntu | ✅ `Welcome to Ubuntu 22.04.5 LTS` |

## Bảo mật
- Private key (`id_ed25519`) chỉ nằm trên máy cá nhân và không được đưa lên GitHub.
- Ảnh chụp màn hình đã che phần mật khẩu.
- Bước nâng cao (chưa làm trong bài này): tắt đăng nhập bằng mật khẩu bằng cách đặt `PasswordAuthentication no` trong `/etc/ssh/sshd_config`, để máy chủ chỉ chấp nhận SSH key.
