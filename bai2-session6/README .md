# Bài 2: Cấu hình phân quyền Nhóm và sudoers bằng visudo

## 1. Mục tiêu

- Tạo nhóm người dùng mới trên Linux.
- Tạo tài khoản `deployer` và thêm vào nhóm `devops-admin`.
- Cấu hình quyền `sudo` giới hạn bằng `visudo`.
- Cho phép nhóm `devops-admin` sử dụng các lệnh quản lý dịch vụ `systemctl` gồm:
  - `start`
  - `stop`
  - `restart`
  - `status`
- Các lệnh trên được chạy với `sudo` mà không cần nhập mật khẩu.

---

## 2. Môi trường thực hiện

- Hệ điều hành: Ubuntu Server
- Tài khoản quản trị thực hiện cấu hình: `devops`
- Nhóm được tạo: `devops-admin`
- User được tạo: `deployer`

---

## 3. Các bước thực hiện

### Bước 1: Tạo nhóm `devops-admin`

```bash
sudo groupadd devops-admin
```

Kiểm tra nhóm đã được tạo:

```bash
getent group devops-admin
```

Kết quả mong đợi:

```text
devops-admin:x:1002:
```

---

### Bước 2: Tạo user `deployer`

```bash
sudo adduser deployer
```

Sau đó đặt mật khẩu cho tài khoản `deployer`.

---

### Bước 3: Thêm user `deployer` vào nhóm `devops-admin`

```bash
sudo usermod -aG devops-admin deployer
```

Kiểm tra:

```bash
groups deployer
```

Kết quả mong đợi:

```text
deployer : deployer devops-admin
```

---

### Bước 4: Kiểm tra đường dẫn của `systemctl`

```bash
which systemctl
```

Kết quả:

```text
/usr/bin/systemctl
```

---

### Bước 5: Cấu hình quyền sudo bằng `visudo`

Mở file sudoers:

```bash
sudo visudo
```

Thêm dòng sau vào cuối file:

```text
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

Ý nghĩa:

- `%devops-admin`: áp dụng cho toàn bộ user thuộc nhóm `devops-admin`.
- `ALL=(ALL)`: cho phép thực thi với quyền nâng cao.
- `NOPASSWD`: không yêu cầu nhập mật khẩu khi chạy các lệnh được cấp quyền.
- Chỉ cấp quyền với `systemctl start`, `stop`, `restart`, `status`.

---

## 4. Kiểm tra kết quả

### Bước 1: Chuyển sang user `deployer`

```bash
su - deployer
```

Kiểm tra user hiện tại:

```bash
whoami
```

Kết quả:

```text
deployer
```

Kiểm tra nhóm:

```bash
groups
```

Kết quả mong đợi có:

```text
deployer devops-admin
```

---

### Bước 2: Kiểm tra quyền sudo

```bash
sudo -l
```

Kết quả mong đợi hiển thị quyền tương tự:

```text
User deployer may run the following commands on this host:

    (ALL) NOPASSWD: /usr/bin/systemctl start *,
                    /usr/bin/systemctl stop *,
                    /usr/bin/systemctl restart *,
                    /usr/bin/systemctl status *
```

Điều này chứng minh user `deployer` được phép quản lý dịch vụ thông qua `systemctl` mà không cần nhập mật khẩu.

---

### Bước 3: Restart dịch vụ cron

```bash
sudo systemctl restart cron
```

Nếu lệnh chạy thành công và không yêu cầu mật khẩu thì cấu hình đã đúng.

Kiểm tra trạng thái dịch vụ:

```bash
sudo systemctl status cron
```

Kết quả mong đợi:

```text
Active: active (running)
```

Nhấn `q` để thoát màn hình trạng thái.

---

## 5. Kiểm tra giới hạn quyền

Thử chạy một lệnh không được cấp quyền:

```bash
sudo apt update
```

User `deployer` không được phép chạy lệnh này nếu không có quyền sudo khác.

Điều này cho thấy tài khoản chỉ được cấp quyền quản lý dịch vụ cần thiết thay vì toàn quyền `root`.

---

## 6. Minh chứng

### Ảnh 1: User `deployer` thuộc nhóm `devops-admin`

Chèn ảnh kết quả lệnh:

```bash
groups deployer
```

Ví dụ:

```markdown
![Groups](images/groups.png)
```

---

### Ảnh 2: Kết quả `sudo -l`

Chèn ảnh kết quả:

```bash
sudo -l
```

Ví dụ:

```markdown
![Sudo List](images/sudo-l.png)
```

---

### Ảnh 3: Restart và kiểm tra dịch vụ cron

Chèn ảnh kết quả:

```bash
sudo systemctl restart cron
sudo systemctl status cron
```

Ví dụ:

```markdown
![Cron Status](images/cron-status.png)
```

---

## 7. Cấu trúc thư mục nộp bài

```text
homework/
└── session_06/
    └── ex2/
        ├── README.md
        └── images/
            ├── groups.png
            ├── sudo-l.png
            └── cron-status.png
```

---

## 8. Kết luận

Bài thực hành đã hoàn thành các yêu cầu:

- Tạo nhóm `devops-admin`.
- Tạo user `deployer`.
- Thêm `deployer` vào nhóm `devops-admin`.
- Cấu hình quyền bằng `visudo`.
- User `deployer` có thể sử dụng `systemctl start`, `stop`, `restart`, `status`.
- Các lệnh được cấp quyền không yêu cầu nhập mật khẩu.
- User không được cấp toàn quyền `root`, giúp giảm rủi ro bảo mật.
