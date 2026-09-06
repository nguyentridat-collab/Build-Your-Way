# 氣 VẬN KHÍ (YOURWAY - YW) ECOSYSTEM

> **Identity & Consumption Hub** kết nối hệ sinh thái ứng dụng học tập, trò chơi 2D và ví điểm độc lập theo mô hình phân tán (Decoupled Ledgers).

[![PHP Version](https://img.shields.io/badge/PHP-8.0%2B-777BB4?style=flat&logo=php&logoColor=white)](#)
[![Database](https://img.shields.io/badge/Database-MySQL%20%7C%20InnoDB-4479A1?style=flat&logo=mysql&logoColor=white)](#)
[![TailwindCSS](https://img.shields.io/badge/UI-Tailwind_CSS-38B2AC?style=flat&logo=tailwind-css&logoColor=white)](#)
[![Google Auth](https://img.shields.io/badge/Auth-Google_Identity_Services-4285F4?style=flat&logo=google&logoColor=white)](#)

---

## 1. Tổng Quan Kiến Trúc (Architecture Overview)

Hệ sinh thái **Vận Khí (YourWay)** giải quyết bài toán đồng bộ dữ liệu và tài sản số giữa nhiều webapp con độc lập (App học tiếng Đức, Minigame thẻ bài, Webgame nông trại 2D...). 

Hệ thống được thiết kế theo mô hình **In-game Currency Exchange / Decoupled Ledgers** nhằm phân tách rủi ro tài chính và tối ưu hiệu năng:

* **App Ví Độc Lập (`vi.hoctiengducdedang.com/wallet/`):** Nơi tích lũy điểm cày được từ việc học tập và chơi game. Mỗi người dùng sở hữu một mã ví độc lập 20 ký tự (dạng `W...`) bảo mật bằng mật khẩu mã hóa Bcrypt.
* **VanKhi Hub (Trung tâm Danh tính & Tiêu thụ):** Quản lý hồ sơ người dùng thông qua Google Identity Services (Gmail), tự động cấp định danh `vankhi_uid` và lưu trữ **Túi Điểm Vận Khí (User Pocket)** độc lập.
* **Cơ chế Kéo Điểm (Pull Mechanism - M2M):** Người dùng chủ động rút điểm từ Ví về Túi trên Hub. Hai máy chủ giao tiếp ngầm thông qua cổng API bảo vệ bởi chữ ký Server-to-Server (`HUB_ACCESS_KEY`), sử dụng Transaction kết hợp `SELECT ... FOR UPDATE` để ngăn ngừa triệt để lỗi Race Condition và Double-spending.

```text
                  [ NGƯỜI DÙNG / TRÌNH DUYỆT ]
                                │
        ┌───────────────────────┴───────────────────────┐
        ▼ (Google OAuth2)                               ▼ (Mã ví 20 ký tự + Mật khẩu)
┌──────────────────────────────┐              ┌──────────────────────────────┐
│       VANKHI HUB (YW)        │              │       APP VÍ ĐỘC LẬP         │
│  (Identity & Consumption)    │              │  (Earning & Source Ledger)   │
├──────────────────────────────┤              ├──────────────────────────────┤
│ * index.html (Zen Dashboard) │              │ * index.html (Giao diện ví)  │
│ * auth_google.php (Login GSI)│              │ * api.php (Credit/Transfer)  │
│ * pull_points.php (Rút điểm) │              │ * dbconnect.php (MySQL Ví)   │
│ * db_vankhi.php (MySQL Hub)  │              │ * port.php (Cổng M2M Bridge) │
└──────────────┬───────────────┘              └──────────────┬───────────────┘
               │                                             │
               │         [ GIAO TIẾP SERVER-TO-SERVER ]      │
               │ ────────── HTTPS POST (pull_points) ──────> │ (Trừ điểm ví)
               │          Header: X-Hub-Key                  │
               │ <──────── 200 OK + wallet_tx_ref ────────── │
               │                                             │
               ▼                                             ▼
┌──────────────────────────────┐              ┌──────────────────────────────┐
│      CSDL VANKHI (vankhi_db) │              │          CSDL VÍ             │
├──────────────────────────────┤              ├──────────────────────────────┤
│ 1. vk_users                  │              │ 1. point_wallets             │
│ 2. vk_user_pockets           │              │ 2. wallet_transactions       │
│ 3. vk_point_receipts         │              └──────────────────────────────┘
└──────────────────────────────┘

2. Cấu Trúc Thư Mục Dự Án (File Mapping)
vankhi-ecosystem/
├── hub/                                # Mã nguồn VanKhi Hub (Identity & Dashboard)
│   ├── index.html                      # Giao diện Zen-Tech, tích hợp Google GSI & Modal rút điểm
│   ├── auth_google.php                 # Backend xác thực Google ID Token & khởi tạo hồ sơ
│   ├── pull_points.php                 # Backend tiếp nhận lệnh rút điểm, gọi sang Port của Ví
│   ├── db_vankhi.php                   # Kết nối CSDL riêng vankhi_db (PDO)
│   └── vankhi_schema.sql               # File khởi tạo Database cho Hub
│
└── wallet/                             # Mã nguồn App Ví Độc Lập (Source of Truth)
    ├── index.html                      # Giao diện quản lý ví người dùng
    ├── api.php                         # API ví (Đăng nhập, tạo ví, chuyển P2P, nhận điểm game)
    ├── port.php                        # Cổng API Server-to-Server phục vụ Hub rút điểm
    └── dbconnect.php                   # Kết nối CSDL của Ví

3. Thiết Kế Cơ Sở Dữ Liệu (Database Schema)
Cơ sở dữ liệu VanKhi Hub (vankhi_db)
vk_users: Lưu thông tin định danh Google (google_sub), email, tên hiển thị, avatar và mã vankhi_uid (định dạng VK-xxxx-xxxx).

vk_user_pockets: Lưu số dư điểm khả dụng trong túi (points) của người dùng tại Hub.

vk_point_receipts: Sổ cái ghi nhận lịch sử rút điểm thành công, lưu vết mã ví nguồn và mã đối soát (wallet_tx_ref).

Cơ sở dữ liệu App Ví
point_wallets: Lưu trữ mã ví công khai 20 ký tự, mật khẩu mã hóa Bcrypt và số dư điểm cày cuốc.

wallet_transactions: Ghi chép biến động số dư chi tiết (balance_before, balance_after, loại giao dịch).

4. Hướng Dẫn Cài Đặt (Quick Start)
Bước 1: Khởi tạo CSDL
Tạo database cho Hub và chạy file hub/vankhi_schema.sql.

Cấu hình thông số database tương ứng trong file hub/db_vankhi.php.

Bước 2: Thiết lập Google Identity Services
Truy cập Google Cloud Console, tạo mới OAuth 2.0 Client ID (loại Web application).

Thêm domain chạy Hub vào mục Authorized JavaScript origins.

Mở hub/index.html và thay thế hằng số GOOGLE_CLIENT_ID bằng Client ID vừa tạo.

Bước 3: Cấu hình Khóa Bí Mật Kết Nối (Secret Key)
Đảm bảo khóa bảo mật Server-to-Server phải trùng khớp giữa hai bên:

Trong hub/pull_points.php: Khai báo define('HUB_ACCESS_KEY', 'CHON_KHOA_BI_MAT_CUA_BAN');

Trong wallet/port.php: Khai báo define('HUB_ACCESS_KEY', 'CHON_KHOA_BI_MAT_CUA_BAN');

5. Lộ Trình Phát Triển (Roadmap)
[x] Giai đoạn 1 (Core Identity & Currency Exchange):

[x] Đăng nhập một chạm bằng Google One-Tap / GSI.

[x] Giao diện Dashboard phong cách Cyber-Fengshui / Zen-Tech.

[x] Module rút điểm an toàn Server-to-Server chống Race Condition.

[ ] Giai đoạn 2 (Consumption & Progression):

[ ] Xây dựng hệ thống Cây Kỹ Năng (Skill Tree / Perks System).

[ ] Tính năng "Gắn liên kết ví" (Link Wallet) lưu cố định mã ví vào hồ sơ người dùng.

[ ] Module Chợ Vật Phẩm (Inventory / Store) tiêu thụ điểm VKP.
