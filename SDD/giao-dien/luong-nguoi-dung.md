# Luồng người dùng v1

## Lần mở app

```mermaid
flowchart TD
  A[Mở app] --> B{Đã đăng nhập?}
  B -->|Không| C[Màn chào]
  C --> L[Đăng ký AUTH-01 / Đăng nhập AUTH-02]
  L --> T{Đã xem hướng dẫn?}
  B -->|Có| T
  T -->|Chưa| O[Guide tour 4 trang]
  O --> D[Tab Bản đồ]
  T -->|Rồi| D
  D --> E{Quyền vị trí}
  E -->|Cho| F[Nearby + marker MAP-01..04 DISCOVERY-01]
  E -->|Từ chối| G[Tìm khu vực MAP-05]
  F --> H[Chi tiết PLACE-01]
  G --> H
```

**Bắt buộc đăng nhập** — app không có chế độ khách (quyết định 2026-10-27). Chưa đăng nhập,
hoặc phiên hết hạn (401 sau khi refresh thất bại), hay đăng xuất → về màn chào.

Quyền vị trí chỉ xin **sau** guide tour, lúc vào tab Bản đồ. Cờ "đã xem hướng dẫn" lưu theo
thiết bị: đăng ký tài khoản thứ hai trên cùng máy không xem lại tour.

## Từ bản đồ tới đánh giá

```mermaid
flowchart LR
  M[Bản đồ / Khám phá] --> P[Chi tiết quán]
  P --> R[Viết review]
  R --> V{Email verified?}
  V -->|Chưa| N[Nhắc xác minh]
  V -->|Rồi| W[Form review PENDING]
```

Đã có review: form chế độ cập nhật (REVIEW-04). Upload lỗi: thử lại **từng file**.

## Chatbot

```mermaid
flowchart TD
  A[Mở chat] --> D[Nhập / gửi]
  D --> E[SSE + tool PostGIS]
  E --> F[Thẻ quán]
  F --> G[Chi tiết PLACE-01]
```

## Kiểm duyệt

```mermaid
flowchart TD
  A[Hàng chờ PENDING] --> B{Quyết định}
  B -->|Duyệt| C[APPROVED + tính rating + NOTI]
  B -->|Từ chối| D[Bắt buộc lý do]
  D --> E[REJECTED + NOTI]
  B -->|Ẩn| F[HIDDEN + tính lại rating]
  C --> G[audit_logs]
  E --> G
  F --> G
```
