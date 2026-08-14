# Từ điển thuật ngữ

Cột **Code** là định danh dùng trong mã nguồn, tên bảng, tên field API và enum.
Luôn dùng đúng chuỗi ở cột này — đừng tự đặt tên đồng nghĩa.

## Thực thể chính

| Tiếng Việt | English | Code | Ghi chú |
|---|---|---|---|
| Địa điểm | Place | `place` | Danh từ chung cho quán ăn, hàng ăn, chợ, quán cà phê |
| Quán ăn | Restaurant | `place_type = RESTAURANT` | Có mặt bằng cố định, bàn ghế |
| Hàng ăn / xe đẩy | Street food stall | `place_type = STREET_FOOD` | Vỉa hè, xe đẩy, gánh hàng rong |
| Chợ đồ ăn | Food market | `place_type = FOOD_MARKET` | Khu chợ, food court, nhiều quầy |
| Quán cà phê | Cafe | `place_type = CAFE` | |
| Danh mục | Category | `category` | Phở, bún, bánh mì, hải sản… — quan hệ nhiều-nhiều với place |
| Đánh giá | Review | `review` | Có `rating` 1–5 + nội dung + media, hiển thị công khai |
| Góp ý / báo sai | Feedback | `feedback` | Báo dữ liệu sai, **không** công khai |
| Lượt đến | Visit | `visit` | Một lần check-in của user tại place |
| Yêu thích | Favorite | `favorite` | Bookmark, không có trạng thái trung gian |
| Tệp media | Media | `media` | Ảnh hoặc video đính kèm review |
| Thông báo | Notification | `notification` | Bản ghi in-app, push là kênh gửi thêm |
| Giờ mở cửa | Opening hours | `opening_hours` | Theo từng ngày trong tuần, cho phép nhiều khoảng/ngày |

## Vai trò

| Tiếng Việt | Code | Quyền |
|---|---|---|
| Khách vãng lai | `GUEST` | Chỉ xem nội dung công khai, chưa đăng nhập |
| Người dùng | `USER` | Review, feedback, yêu thích, ghi visit |
| Kiểm duyệt viên | `MODERATOR` | Duyệt review và feedback, CRUD place và category |
| Quản trị viên | `ADMIN` | Toàn quyền, gồm quản lý user và gán vai trò |

## Trạng thái

**Địa điểm** — `place.status`

| Tiếng Việt | Code | Hiển thị ở API công khai |
|---|---|---|
| Nháp | `DRAFT` | Không |
| Đã đăng | `PUBLISHED` | Có |
| Tạm đóng cửa | `TEMPORARILY_CLOSED` | Có, kèm nhãn |
| Đóng cửa vĩnh viễn | `PERMANENTLY_CLOSED` | Không |

**Đánh giá** — `review.status`

| Tiếng Việt | Code | Tính vào điểm trung bình |
|---|---|---|
| Chờ duyệt | `PENDING` | Không |
| Đã duyệt | `APPROVED` | Có |
| Bị từ chối | `REJECTED` | Không |
| Bị ẩn | `HIDDEN` | Không |

**Góp ý** — `feedback.status`: `OPEN` → `IN_REVIEW` → `RESOLVED` hoặc `DISMISSED`

**Loại góp ý** — `feedback.type`

| Tiếng Việt | Code |
|---|---|
| Sai địa chỉ | `WRONG_ADDRESS` |
| Sai giờ mở cửa | `WRONG_HOURS` |
| Đã đóng cửa vĩnh viễn | `CLOSED_PERMANENTLY` |
| Trùng lặp | `DUPLICATE` |
| Nội dung không phù hợp | `INAPPROPRIATE` |
| Khác | `OTHER` |

**Loại thông báo** — `notification.type`

`REVIEW_APPROVED` · `REVIEW_REJECTED` · `FEEDBACK_RESOLVED` · `NEW_PLACE_NEARBY` ·
`PLACE_UPDATED` · `SYSTEM_ANNOUNCEMENT`

## Thuật ngữ kỹ thuật

| Thuật ngữ | Giải thích |
|---|---|
| PostGIS | Phần mở rộng của PostgreSQL cho dữ liệu địa lý |
| `geography(Point, 4326)` | Kiểu cột lưu toạ độ; 4326 là hệ toạ độ WGS-84 (chuẩn GPS) |
| Index GiST | Chỉ mục không gian, bắt buộc để truy vấn "quanh đây" chạy nhanh |
| `ST_DWithin` | Hàm PostGIS lọc theo bán kính (mét); dùng được index |
| Flyway | Công cụ quản lý migration cơ sở dữ liệu, chạy tuần tự theo version |
| OpenAPI | Đặc tả mô tả API; ở đây là `docs/03-api/openapi.yaml` |
| Contract-first | Sửa hợp đồng API trước, rồi mới sửa code hai bên |
| Presigned URL | Đường dẫn tạm có chữ ký, cho phép upload thẳng lên S3/MinIO |
| Expo Push | Dịch vụ đẩy thông báo của Expo, gói cả APNs và FCM |

## Những nhầm lẫn cần tránh

- **`review` vs `feedback`.** Review = cảm nhận về đồ ăn, công khai, có sao.
  Feedback = báo cáo dữ liệu sai, riêng tư, không có sao.
- **`place` vs "nhà hàng".** Phần lớn địa điểm trong FoodMap không phải nhà hàng.
  Viết "nhà hàng" trong tài liệu là sai nghĩa.
- **`visit_count` vs `distinct_visitor_count`.** Cái đầu là tổng số lượt, cái sau là
  số người khác nhau. Hai con số khác nhau, đừng dùng lẫn.
- **Thứ tự toạ độ.** PostGIS dùng `ST_MakePoint(lng, lat)` — **kinh độ trước**.
  API JSON thì phơi ra `latitude` rồi `longitude`. Nhầm là ra giữa biển.
