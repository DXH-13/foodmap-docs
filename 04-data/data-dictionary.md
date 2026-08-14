# Từ điển dữ liệu

Chi tiết từng cột. Sơ đồ quan hệ: [`erd.md`](./erd.md). Phần địa lý: [`geo-model.md`](./geo-model.md).

**Quy ước chung cho mọi bảng nghiệp vụ:**
- `id` — `UUID PRIMARY KEY DEFAULT gen_random_uuid()`
- `created_at`, `updated_at` — `TIMESTAMPTZ NOT NULL DEFAULT now()`, lưu UTC
- `deleted_at` — `TIMESTAMPTZ NULL`, chỉ có ở bảng xoá mềm
- Enum lưu bằng `VARCHAR` + `CHECK` constraint có tên tường minh

---

## `users`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `email` | `VARCHAR(255)` | ✗ | Duy nhất, lưu chữ thường |
| `password_hash` | `VARCHAR(60)` | ✗ | BCrypt cost 12. **Không bao giờ trả về qua API** |
| `display_name` | `VARCHAR(100)` | ✗ | Tên hiển thị công khai |
| `avatar_url` | `VARCHAR(500)` | ✓ | |
| `role` | `VARCHAR(16)` | ✗ | `USER` \| `MODERATOR` \| `ADMIN`. Mặc định `USER` |
| `preferred_locale` | `VARCHAR(5)` | ✗ | `vi` \| `en`. Mặc định `vi` |
| `email_verified` | `BOOLEAN` | ✗ | Mặc định `false`. `false` thì không được review/feedback/visit |
| `email_verified_at` | `TIMESTAMPTZ` | ✓ | |
| `deleted_at` | `TIMESTAMPTZ` | ✓ | Xoá mềm; xoá cứng sau 30 ngày |

**Index:** `uq_users_email` unique trên `LOWER(email)` với `deleted_at IS NULL`.

---

## `places`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `slug` | `VARCHAR(200)` | ✗ | Duy nhất, dùng cho URL. Sinh từ tên tiếng Việt bỏ dấu |
| `place_type` | `VARCHAR(20)` | ✗ | `RESTAURANT` \| `STREET_FOOD` \| `FOOD_MARKET` \| `CAFE` |
| `location` | `geography(Point,4326)` | ✗ | **Kinh độ trước** khi dựng. Index GiST bắt buộc |
| `address` | `VARCHAR(500)` | ✓ | Hàng rong có thể không có địa chỉ chính thức |
| `phone` | `VARCHAR(20)` | ✓ | |
| `status` | `VARCHAR(24)` | ✗ | `DRAFT` \| `PUBLISHED` \| `TEMPORARILY_CLOSED` \| `PERMANENTLY_CLOSED` |
| `needs_review` | `BOOLEAN` | ✗ | Mặc định `false`. Tự bật khi có 3 báo cáo đóng cửa (FR-FEEDBACK-05) |
| `average_rating` | `NUMERIC(2,1)` | ✓ | **`NULL` khi chưa có review — không phải `0`** |
| `review_count` | `INTEGER` | ✗ | Đếm review `APPROVED`. Mặc định `0` |
| `visit_count` | `BIGINT` | ✗ | **Tổng số lượt**, không phải số người |
| `distinct_visitor_count` | `BIGINT` | ✗ | Số người khác nhau đã đến |
| `created_by` | `UUID` | ✓ | FK → `users`. `NULL` với dữ liệu seed |
| `deleted_at` | `TIMESTAMPTZ` | ✓ | |

**Index:** `idx_places_location` GiST · `idx_places_status` với `deleted_at IS NULL` ·
`uq_places_slug` unique.

---

## `place_translations`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `place_id` | `UUID` | ✗ | FK → `places`, `ON DELETE CASCADE` |
| `locale` | `VARCHAR(5)` | ✗ | `vi` \| `en` |
| `name` | `VARCHAR(255)` | ✗ | |
| `description` | `TEXT` | ✓ | |

**Index:** `uq_place_translations` unique `(place_id, locale)`.
Bản `vi` **bắt buộc**; thiếu `en` thì fallback về `vi` (FR-I18N-03).

Bảng `category_translations` có cấu trúc tương tự, không có `description`.

---

## `categories`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `slug` | `VARCHAR(80)` | ✗ | Duy nhất. Ví dụ `pho`, `bun-bo`, `banh-mi` |
| `icon` | `VARCHAR(80)` | ✓ | Tên icon ở client |
| `display_order` | `INTEGER` | ✗ | Thứ tự hiển thị. Mặc định `0` |

`place_categories` là bảng nối: `(place_id, category_id)` làm khoá chính tổ hợp.

---

## `opening_hours`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `place_id` | `UUID` | ✗ | FK → `places`, `ON DELETE CASCADE` |
| `day_of_week` | `SMALLINT` | ✗ | `1` = thứ Hai … `7` = Chủ nhật (ISO-8601) |
| `open_time` | `TIME` | ✓ | `NULL` khi `is_closed_all_day = true` |
| `close_time` | `TIME` | ✓ | Nhỏ hơn `open_time` nghĩa là qua nửa đêm |
| `is_closed_all_day` | `BOOLEAN` | ✗ | Mặc định `false` |

**Cho phép nhiều dòng cùng `day_of_week`** để mô tả quán bán sáng và tối, nghỉ trưa
(FR-PLACE-07). Giờ diễn giải theo `Asia/Ho_Chi_Minh`.

---

## `reviews`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `place_id` | `UUID` | ✗ | FK → `places`, `ON DELETE RESTRICT` |
| `user_id` | `UUID` | ✗ | FK → `users`, `ON DELETE RESTRICT` |
| `rating` | `SMALLINT` | ✗ | `CHECK (rating BETWEEN 1 AND 5)`. Số nguyên, không có nửa sao |
| `content` | `TEXT` | ✓ | Tối đa 2.000 ký tự, ép ở tầng ứng dụng |
| `locale` | `VARCHAR(5)` | ✗ | Ngôn ngữ tác giả viết. **Không dịch** (FR-I18N-04) |
| `status` | `VARCHAR(16)` | ✗ | `PENDING` \| `APPROVED` \| `REJECTED` \| `HIDDEN` |
| `moderation_note` | `TEXT` | ✓ | **Bắt buộc khi `status = REJECTED`** (ép ở tầng ứng dụng) |
| `moderated_by` | `UUID` | ✓ | FK → `users` |
| `moderated_at` | `TIMESTAMPTZ` | ✓ | |
| `deleted_at` | `TIMESTAMPTZ` | ✓ | |

**Index:**
- `uq_reviews_user_place` unique `(place_id, user_id)` **với `deleted_at IS NULL`** —
  ép luật một-review-mỗi-user-mỗi-place ở tầng CSDL (FR-REVIEW-03)
- `idx_reviews_place_status` `(place_id, status)` với `deleted_at IS NULL`

---

## `review_media` / `place_media`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `media_type` | `VARCHAR(10)` | ✗ | `IMAGE` \| `VIDEO` (chỉ `review_media`) |
| `storage_key` | `VARCHAR(500)` | ✗ | Khoá object trên S3/MinIO, **không** phải URL đầy đủ |
| `thumbnail_key` | `VARCHAR(500)` | ✓ | Bản 400px |
| `display_key` | `VARCHAR(500)` | ✓ | Bản 1600px cạnh dài |
| `width`, `height` | `INTEGER` | ✓ | Để client giữ chỗ đúng tỉ lệ trước khi ảnh tải xong |
| `duration_seconds` | `INTEGER` | ✓ | Video, tối đa 60 |
| `size_bytes` | `BIGINT` | ✗ | |
| `display_order` | `INTEGER` | ✗ | Mặc định `0` |

Lưu `storage_key` chứ **không** lưu URL đầy đủ — đổi endpoint hoặc CDN thì không phải
migrate dữ liệu. URL dựng lúc trả API.

Ràng buộc số lượng (5 ảnh + 1 video) ép ở tầng ứng dụng (FR-REVIEW-02).

---

## `feedbacks`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `type` | `VARCHAR(24)` | ✗ | `WRONG_ADDRESS` \| `WRONG_HOURS` \| `CLOSED_PERMANENTLY` \| `DUPLICATE` \| `INAPPROPRIATE` \| `OTHER` |
| `description` | `TEXT` | ✓ | Tối đa 1.000 ký tự |
| `duplicate_of_place_id` | `UUID` | ✓ | FK → `places`. Chỉ dùng khi `type = DUPLICATE` |
| `status` | `VARCHAR(16)` | ✗ | `OPEN` \| `IN_REVIEW` \| `RESOLVED` \| `DISMISSED` |
| `resolution_note` | `TEXT` | ✓ | |
| `resolved_by` | `UUID` | ✓ | FK → `users` |

**Index:** `idx_feedbacks_open` bộ phận `(user_id, place_id, type)` với `status = 'OPEN'` —
ép luật không tạo trùng (FR-FEEDBACK-03).

---

## `visits`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `user_id` | `UUID` | ✗ | FK → `users` |
| `place_id` | `UUID` | ✗ | FK → `places` |
| `visit_date` | `DATE` | ✗ | Ngày theo **`Asia/Ho_Chi_Minh`**, tính khi ghi. Không phải UTC |
| `recorded_location` | `geography(Point,4326)` | ✗ | Vị trí người dùng lúc check-in |
| `distance_meters` | `INTEGER` | ✗ | Khoảng cách tới địa điểm lúc check-in, để đối chiếu sau |

**Index:** `uq_visits_user_place_day` unique `(user_id, place_id, visit_date)` —
ép luật 1 lượt/ngày (FR-VISIT-02) ngay ở CSDL.

Không xoá mềm. Chỉ admin xoá cứng khi xử lý gian lận (FR-VISIT-05).

---

## `favorites`

`(user_id, place_id)` + `created_at`. Index `uq_favorites_user_place` unique `(user_id, place_id)` —
khiến thao tác thêm yêu thích idempotent một cách tự nhiên (FR-FAVORITE-02).

---

## `notifications`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `type` | `VARCHAR(32)` | ✗ | Xem [thuật ngữ](../00-overview/glossary-vi-en.md#trạng-thái) |
| `title_key` | `VARCHAR(100)` | ✗ | **Key i18n**, không phải chuỗi đã dịch |
| `payload` | `JSONB` | ✗ | Tham số dựng nội dung: `placeId`, `placeName`, `reason`… |
| `is_read` | `BOOLEAN` | ✗ | Mặc định `false` |
| `push_sent_at` | `TIMESTAMPTZ` | ✓ | `NULL` = chưa gửi push (có thể đang hoãn qua đêm) |
| `push_scheduled_at` | `TIMESTAMPTZ` | ✓ | Thời điểm dự kiến gửi khi bị hoãn (FR-NOTIF-05) |

Lưu `title_key` + `payload` thay vì chuỗi đã dịch, vì người dùng đổi ngôn ngữ được —
thông báo cũ phải hiển thị theo ngôn ngữ hiện tại.

---

## `push_tokens`

`token` (unique), `platform` (`IOS` \| `ANDROID`), `last_used_at`.
Token bị Expo báo không hợp lệ sẽ tự xoá (FR-NOTIF-06).

## `refresh_tokens`

`token_hash` (unique — **lưu hash, không lưu token gốc**), `expires_at`, `revoked_at`.
Cơ chế dùng-một-lần: mỗi lần làm mới thu hồi bản cũ; dùng lại bản đã thu hồi
vô hiệu toàn bộ phiên của user (FR-AUTH-04).

## `chat_sessions` / `chat_messages`

`chat_messages.role` là `USER` \| `ASSISTANT`.
`referenced_place_ids` là `JSONB` mảng UUID — id các địa điểm được nhắc tới, để client
render thẻ bấm mở (FR-CHAT-05).

---

## `audit_logs`

| Cột | Kiểu | Null | Mô tả |
|---|---|---|---|
| `actor_id` | `UUID` | ✗ | Ai thực hiện |
| `action` | `VARCHAR(64)` | ✗ | `REVIEW_APPROVED`, `PLACE_DELETED`, `ROLE_CHANGED`… |
| `entity_type` | `VARCHAR(32)` | ✗ | `PLACE`, `REVIEW`, `USER`… |
| `entity_id` | `UUID` | ✗ | |
| `before_state` | `JSONB` | ✓ | |
| `after_state` | `JSONB` | ✓ | |
| `note` | `TEXT` | ✓ | Lý do, ví dụ `moderation_note` khi từ chối review |

**Chỉ ghi thêm.** Không `UPDATE`, không `DELETE`, không có `deleted_at` (FR-ADMIN-09).
Quyền của user ứng dụng trên bảng này nên giới hạn ở `INSERT` và `SELECT`.
