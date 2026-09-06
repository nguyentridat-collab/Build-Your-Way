1. Sơ Đồ Use Case Tổng Quan (System Boundary)
HỆ SINH THÁI VANKHI (BUILDYOURWAY)
   ┌────────────────────────────────────────────────────────┐
   │                                                        │
   │  [ PHÂN HỆ VANKHI HUB ]                                │
   │  (UC-01: Đăng nhập / Kích hoạt hồ sơ bằng Gmail)       │
   │  (UC-02: Xem Dashboard & Tra cứu VanKhi UID)           │
   │  (UC-03: Rút điểm từ Ví vào Túi Vận Khí)               │
   │  (UC-04: Mở khóa Kỹ năng / Tiêu thụ điểm) [Giai đoạn 2]│
   │                                                        │
   │  [ PHÂN HỆ APP VÍ & APP VỆ TINH ]                      │
   │  (UC-05: Tạo và Đăng nhập Ví điểm độc lập)             │
   │  (UC-06: Hoàn thành bài học/minigame -> Tích điểm ví)  │
   │  (UC-07: Chuyển điểm ngang hàng giữa 2 ví P2P)         │
   │  (UC-08: Phục vụ cổng Port trừ điểm cho Hub - M2M)     │
   │                                                        │
   └────────────────────────────────────────────────────────┘
          ▲                                    ▲
          │                                    │
    [ NGƯỜI DÙNG ]                      [ SERVER HUB ]
 (Tương tác UI Hub/Ví)               (Actor gọi ngầm sang Port)

 

