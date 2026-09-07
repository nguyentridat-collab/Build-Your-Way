# 📋 TÀI LIỆU ĐẶC TẢ TRƯỜNG HỢP SỬ DỤNG (USE CASE SPECIFICATION)
## HỆ SINH THÁI VẬN KHÍ (YOURWAY - YW)

> **Phiên bản:** 1.0.0  
> **Trạng thái:** Đã triển khai Giai đoạn 1 (Core Identity & Currency Exchange)  
> **Mô hình kiến trúc:** Decoupled Ledgers / Pull-on-Demand Exchange

---

## 1. SƠ ĐỒ TỔNG QUAN HỆ THỐNG (SYSTEM BOUNDARY)

```text
               +------------------------------------------------------------------+
               |                  HỆ SINH THÁI VẬN KHÍ (YOURWAY)                  |
               |                                                                  |
               |  [ PHÂN HỆ VANKHI HUB ]                                          |
               |  +-- UC-01: Đăng nhập & Tạo hồ sơ tự động qua Gmail (Google GSI) |
               |  +-- UC-02: Tra cứu Hồ sơ & Quản lý Mã Định Danh (VanKhi UID)    |
               |  +-- UC-03: Rút điểm từ App Ví vào Túi Vận Khí (Pull Points)     |
               |  +-- UC-04: Tiêu thụ điểm mở khóa Kỹ năng (Skill Tree) [Pha 2]   |
               |                                                                  |
               |  [ PHÂN HỆ APP VÍ & APP VỆ TINH ]                                |
               |  +-- UC-05: Tạo & Quản lý Ví điểm độc lập                        |
               |  +-- UC-06: Hoàn thành bài tập/minigame tích lũy điểm ví         |
               |  +-- UC-07: Chuyển điểm ngang hàng P2P giữa hai ví               |
               |  +-- UC-08: Cổng ngầm trừ điểm ví cho Hub (M2M Port)             |
               +------------------------------------------------------------------+
                        ^                                        ^
                        |                                        |
                 [ NGƯỜI DÙNG ]                           [ SERVER HUB ]
              (Thao tác qua Web UI)                    (Máy chủ gọi ngầm M2M)

https://docs.google.com/document/d/e/2PACX-1vRypczVTtGqJZZx3iBOPPE3eb49iSh-jE4wM7Lpk7e_QiInfg92k5QqauOgQzBoZaWCP-8MRZflEEqU/pub
