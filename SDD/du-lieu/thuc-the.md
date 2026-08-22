# Mô hình thực thể v1 (theo Feature Breakdown)

Không thay thế migration Flyway. Schema đã có ở `backend` (`V1__init_schema.sql`) gần khớp; dưới đây là **phần cần có / cần thêm** để đủ PDF.

Toạ độ: `geography(Point, 4326)` + index GiST. API JSON: `latitude`, `longitude`.

## Đã có hướng (giữ)

`users`, token reset/verify, `places`, `place_translations`, `categories`, `category_translations`, `place_categories`, giờ mở cửa, `reviews` + media, `favorites`, thông báo, phiên chat — bám luật: một review/user/place; rating 1–5; `average_rating` null khi chưa có review duyệt.

## Thêm hoặc làm rõ so với schema hiện tại

| Hạng mục | Lý do PDF | Thiết kế |
|---|---|---|
| `users.locked_at` | ADMIN-02 khoá user | NULL = hoạt động; khác NULL thì login trả 403 mã `USER_LOCKED` |
| `places.price_level` | MAP-06 khoảng giá | SMALLINT 1–4 NULL = chưa biết; không bịa giá |
| `place_media` | PLACE-02 media do admin | Tách khỏi media review |
| `place_view_stats` hoặc bảng sự kiện xem | DISCOVERY-02 trending | Cộng dồn theo ngày + place; không lưu mọi tap nếu chưa cần |
| `oauth_accounts` | AUTH-03 | `(provider, subject)` unique; `user_id` FK. Tạo khi implement Google, không chặn email |
| `audit_logs` | ADMIN-02, ADMIN-04 | Append-only: actor, action, entity, metadata, thời điểm |

## Cố ý không có ở v1

| Thực thể | Function ID |
|---|---|
| `review_likes` | REVIEW-07 Could |
| `review_reports` / `content_reports` | REVIEW-08, ADMIN-06 Won't |
| `notification_preferences` | NOTI-05 Won't |
| `visits` | Không có trong PDF |

Recent (DISCOVERY-04): **không bắt buộc bảng** — lưu `placeId[]` trên máy.

## i18n dữ liệu

Khoá `(entity_id, locale)` với `locale IN ('vi','en')`. Đọc: locale user → fallback `vi`. Review giữ `reviews.locale`, không dịch.
