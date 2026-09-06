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
