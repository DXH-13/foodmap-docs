# Luồng người dùng v1

## Lần mở app

```mermaid
flowchart TD
  A[Mở app] --> B{Đã đăng nhập?}
  B -->|Không| C[Tab Bản đồ — khách]
  B -->|Có| D[Tab Bản đồ — user]
  C --> E{Quyền vị trí}
  D --> E
  E -->|Cho| F[Nearby + marker MAP-01..04 DISCOVERY-01]
  E -->|Từ chối| G[Tìm khu vực MAP-05]
  F --> H[Chi tiết PLACE-01]
  G --> H
```

Đăng nhập **không** chắn cửa. Chỉ chặn khi bấm: viết review, yêu thích, mở chatbot, mở thông báo.

Sau login: **quay đúng màn đang dở**.

## Từ bản đồ tới đánh giá

```mermaid
flowchart LR
  M[Bản đồ / Khám phá] --> P[Chi tiết quán]
  P --> R{Viết review / Tim?}
  R -->|Chưa login| L[Login / Đăng ký]
  L --> V{Email verified?}
  R -->|Rồi| V
  V -->|Chưa| N[Nhắc xác minh]
  V -->|Rồi| W[Form review PENDING]
```

Đã có review: form chế độ cập nhật (REVIEW-04). Upload lỗi: thử lại **từng file**.

## Chatbot

```mermaid
flowchart TD
  A[Mở chat] --> B{Login?}
  B -->|Không| C[Login rồi quay lại]
  B -->|Có| D[Nhập / gửi]
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
