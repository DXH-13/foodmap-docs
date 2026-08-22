# Thiết kế chức năng v1 — theo Feature Breakdown

Nguồn: file *Ứng dụng bản đồ quán, hàng, chợ đồ ăn ngon trên đất nước Việt Nam - Feature Breakdown.pdf* (51 Function ID, MoSCoW).

Tài liệu này trả lời *xây như thế nào* cho từng nhóm. Danh sách màn hình: [`giao-dien/man-hinh.md`](./giao-dien/man-hinh.md). Luồng: [`giao-dien/luong-nguoi-dung.md`](./giao-dien/luong-nguoi-dung.md). Kiến trúc: [`kien-truc/tong-quan.md`](./kien-truc/tong-quan.md). Thực thể: [`du-lieu/thuc-the.md`](./du-lieu/thuc-the.md).

## Phạm vi

| Mức PDF | Số ID | Xử lý trong thiết kế |
|---|---:|---|
| Must Have | 28 (+ AUTH-01) | **v1 bắt buộc.** AUTH-01 không ghi priority trong PDF — xếp Must. |
| Should Have | 15 | **v1.** Làm sau Must cùng module. |
| Could Have | 4 | **v1.1** — không chặn kiến trúc v1, trừ NOTI-03 (xem dưới). |
| Won't Have | 3 | **Cắt.** Không API, không màn hình, không bảng. |

**Ngoại lệ:** `NOTI-03` (đánh dấu đã đọc) PDF ghi Could, nhưng danh sách thông báo (`NOTI-01`) không dùng được nếu không có `readAt`. Đưa vào v1.

**Không có trong PDF** (có trong skill/`SRS` cũ): feedback báo sai quán, check-in/visit, đề xuất place từ user, cấu hình push theo loại. **Không thiết kế vào v1** khi bám breakdown này. `review` ≠ `feedback` — không dùng từ “báo cáo quán” cho review.

Khách (`GUEST`) dùng: bản đồ, nearby, popular, chi tiết quán, media, review đã duyệt, đổi ngôn ngữ UI. Cần `USER` đã đăng nhập: review, yêu thích, chatbot, thông báo. Chặn thật ở API (`AUTH-07`).

---

## Bản đồ — `MAP-*`

**Ý tưởng:** tab Bản đồ là màn mặc định. Google Maps chỉ *vẽ*. Dữ liệu quán và “quanh đây” đến từ PostGIS.

| ID | Thiết kế |
|---|---|
| MAP-01 | Bản đồ toàn màn, camera theo vị trí hoặc khu vực mặc định (TP.HCM nếu chưa có vị trí). |
| MAP-02 | `expo-location`; marker “bạn đang ở đây”; nút recenter. Từ chối quyền: không khoá app, hiện tìm theo khu vực + nút mở Cài đặt hệ thống. |
| MAP-03 | `GET /places/nearby` theo tâm camera hoặc GPS. Chỉ `PUBLISHED` và `TEMPORARILY_CLOSED` (có nhãn). Bán kính mặc định 2 km, tối đa 50 km — vượt trả 400, không cắt thầm. |
| MAP-04 | Marker theo `placeType` (`RESTAURANT`, `STREET_FOOD`, `FOOD_MARKET`, `CAFE`). Clustering khi zoom nhỏ. Bấm marker → sheet rút gọn → `PLACE-01`. |
| MAP-05 | Tìm tên/mô tả không dấu (`f_unaccent` + GIN). Ô tìm trên tab Khám phá và overlay bản đồ. |
| MAP-06 | Query: `placeType`, `categoryId[]`, `minRating`, `openNow`, `priceLevel` (1–4, cột tuỳ chọn trên `places`). |
| MAP-07 (Could) | Không vẽ turn-by-turn. v1.1: mở Google Maps / Apple Maps bằng toạ độ. |

## Khám phá — `DISCOVERY-*`

| ID | Thiết kế |
|---|---|
| DISCOVERY-01 | Cùng API nearby, sort `distanceMeters` tăng dần. |
| DISCOVERY-02 | `GET /places/trending?days=7` — điểm = lượt xem server + lượt yêu thích mới. Không dùng visit vì PDF không có check-in. |
| DISCOVERY-03 | `GET /places?sort=popular` — `averageRating` DESC, `reviewCount` DESC. `averageRating = null` → UI “Chưa có đánh giá”, không hiện 0 sao. |
| DISCOVERY-04 | **Client:** tối đa 20 `placeId` đã mở chi tiết, MMKV/SecureStore. Khách cũng có recent. |
| DISCOVERY-05 | User đã login: nearby ∩ cùng `category` với quán đang yêu thích. Khách: fallback Popular. Không ML. |
| DISCOVERY-06 | Chip danh mục từ `GET /categories`; lọc `place_categories`. |

## Địa điểm — `PLACE-*`

| ID | Thiết kế |
|---|---|
| PLACE-01 | `GET /places/{id}`: tên/mô tả theo locale (fallback `en`→`vi`), loại, danh mục, địa chỉ, điện thoại, giờ mở, `isOpenNow` (múi `Asia/Ho_Chi_Minh`), rating, số review, `priceLevel`. |
| PLACE-02 | Gallery: `place_media` (admin) + media từ review `APPROVED`. Ảnh JPEG/PNG/WebP; video MP4. |
| PLACE-03 | Block rating + danh sách review phân trang trên cùng màn chi tiết. |
| PLACE-04 | `PUT/DELETE /me/favorites/{placeId}` idempotent. Tab Yêu thích. Quán `PERMANENTLY_CLOSED` giữ trong list, gắn nhãn. |

## Đánh giá — `REVIEW-*`

Một user **một** review đang hoạt động cho mỗi place. Gửi lại = cập nhật, không tạo hàng mới.

| ID | Thiết kế |
|---|---|
| REVIEW-01 | `GET /places/{id}/reviews` — chỉ `APPROVED`. Sort: newest / ratingDesc / ratingAsc. |
| REVIEW-02 | `rating` nguyên 1–5, bắt buộc. |
| REVIEW-03 | `POST` (upsert). `content` tối đa 2000 ký tự, tuỳ chọn. Status `PENDING`. Cần email đã xác minh. |
| REVIEW-04 | Sửa = cùng bản ghi. Nếu đang `APPROVED` → về `PENDING`, điểm trung bình tính lại (bỏ review đó). |
| REVIEW-05 | Xoá mềm của chính mình; tính lại `averageRating` / `reviewCount`. |
| REVIEW-06 | Tối đa 5 ảnh (10 MB) + 1 video (60 s, 100 MB). Client xin presigned URL, upload thẳng storage, rồi `confirm`. |
| REVIEW-07 (Could) | Không bảng `review_likes` ở v1. |
| REVIEW-08 (Won't) | Không. Moderator dùng `HIDDEN` trên hàng chờ. |

`averageRating` / `reviewCount` chỉ tính review `APPROVED`.

## Xác thực — `AUTH-*`

| ID | Thiết kế |
|---|---|
| AUTH-01 | Email + mật khẩu (≥8, có chữ và số). Email unique (không phân biệt hoa thường). Gửi mail xác minh. Chưa verify: xem được, **không** viết review. |
| AUTH-02 | Access JWT 15 phút + refresh 30 ngày (hash trên Redis/DB), refresh **dùng một lần**. |
| AUTH-03 (Should) | Bảng `oauth_accounts`. Làm Google trên iOS ⇒ phải có Sign in with Apple. Có thể ship email trước, OAuth sau, schema để chỗ. |
| AUTH-04 | Thu hồi refresh hiện tại. |
| AUTH-05 | Link TTL 30 phút. Email không tồn tại vẫn trả thông báo thành công giống hệt. |
| AUTH-06 | Token reset một lần; mật khẩu mới BCrypt cost 12. |
| AUTH-07 | Role `USER` \| `MODERATOR` \| `ADMIN`. Admin web chỉ MODERATOR/ADMIN. `locked_at` chặn login (ADMIN-02). |

Rate limit: 5 login sai / email / 15 phút; 3 forgot-password / email / giờ → 429.

## Chatbot — `AI-*`

Client **không** gọi Claude. API đăng ký tool: `search_places`, `get_place_detail`, `recommend_nearby`.

| ID | Thiết kế |
|---|---|
| AI-01 | Route `chat/`. Chưa login → CTA đăng nhập, giữ deep-link. |
| AI-02 | Ô nhập + chip gợi ý. |
| AI-03 | Gửi kèm `locale` và toạ độ nếu có. 30 tin / user / giờ. |
| AI-04 | SSE. Token đầu mục tiêu &lt; 2 giây. Lỗi nhà cung cấp: thông báo thân thiện, không lộ stack. |
| AI-05 | Mỗi quán trong câu trả lời có `placeId` để render thẻ. Không tìm thấy → nói không có trong dữ liệu FoodMap. |
| AI-06 | Một phiên hiện tại / user; GET lịch sử; DELETE xoá phiên. Không đa hội thoại song song. |

Trả lời **cùng ngôn ngữ câu hỏi** (vi/en).

## Quản trị — `ADMIN-*`

Mọi bảng phân trang/lọc/sort **phía server**. Hành động không hoàn tác được (xoá quán, khoá user, từ chối review) có bước xác nhận.

| ID | Thiết kế |
|---|---|
| ADMIN-01 | `GET /admin/stats`: place theo status, review theo status, user mới theo ngày. |
| ADMIN-02 | Chỉ `ADMIN`. Tìm user, đổi role, khoá/mở (`locked_at`). Không “thêm user” bằng mật khẩu tạm trừ khi cần seed — ưu tiên user tự đăng ký. |
| ADMIN-03 | CRUD place: marker trên bản đồ, dịch vi/en, giờ mở nhiều khoảng, media, category, `priceLevel`. |
| ADMIN-04 (Should) | Hàng chờ PENDING: thumbnail, tên quán, tác giả, số lần bị từ chối trước. Duyệt / từ chối (bắt buộc `moderationNote`) / ẩn. Audit log append-only. |
| ADMIN-05 | CRUD category + bản dịch + `displayOrder`. |
| ADMIN-06 (Won't) | Không module báo cáo user. |

## Thông báo — `NOTI-*`

| ID | Thiết kế |
|---|---|
| NOTI-01 | In-app bắt buộc. Loại v1: `REVIEW_APPROVED`, `REVIEW_REJECTED`, `SYSTEM_ANNOUNCEMENT`. Push Expo không chặn v1. |
| NOTI-02 | Deep-link `placeId` / `reviewId`. |
| NOTI-03 | `readAt`; đánh dấu một hoặc tất cả — **làm ở v1**. |
| NOTI-04 (Could) | v1.1 xoá mềm. |
| NOTI-05 (Won't) | Không preference theo loại. |

## Ngôn ngữ — `LANG-*`

- Locale chỉ `vi` (mặc định) và `en`.
- UI: file i18n mobile (`i18next`) và admin (`next-intl`); backend `messages.properties` / `messages_en.properties`.
- Nội dung động: `place_translation`, `category_translation`. Thiếu `en` → fallback `vi`, không `null`.
- Review **không** dịch; cột `locale` để gắn nhãn.

---

## Thứ tự triển khai (phụ thuộc)

```
Auth (AUTH-01..07 trừ OAuth)
  → Place + Category + Admin CRUD + Nearby/Map (MAP, PLACE-01, ADMIN-01/03/05, DISCOVERY-01)
    → Discovery list + media (DISCOVERY-02..06, PLACE-02)
      → Review + moderation + favorite (REVIEW-01..06, ADMIN-04, PLACE-03/04)
        → i18n đầy đủ + thông báo in-app (LANG, NOTI-01..03)
          → Chatbot (AI-01..06)
AUTH-03 Google song song muộn, sau email.
MAP-07, REVIEW-07, NOTI-04 sau v1.
```

## Việc chưa chốt (không bịa trong SDD)

- Nguồn dữ liệu hàng nghìn quán lúc ra mắt (nhập tay / Google Places / kết hợp).
- Có làm Sign in with Apple cùng Google hay lùi cả OAuth.
- Nền tảng production (Q4 ops cũ).
- Có bật Expo Push ngay khi có NOTI hay chỉ in-app.
