# Nhật ký thay đổi hợp đồng API

Ghi lại mọi thay đổi của [`openapi.yaml`](./openapi.yaml). Bắt buộc ghi với
**thay đổi phá vỡ** — mobile không cập nhật tức thì được, người dùng phải tải bản app mới.

## Thế nào là thay đổi phá vỡ

Xoá field · đổi kiểu field · thêm field **bắt buộc** vào request · xoá giá trị enum ·
đổi mã lỗi · đổi path hoặc method.

**Không** phá vỡ: thêm field tuỳ chọn vào request · thêm field vào response ·
thêm giá trị enum mới vào *response* (client phải xử lý giá trị lạ) · thêm endpoint mới.

## Cách xử lý khi buộc phải phá vỡ

1. Thêm field mới song song, đánh dấu field cũ `deprecated: true`.
2. Giữ field cũ ít nhất **2 phiên bản app**.
3. Buộc phải đổi ngay thì tạo path `/api/v2/...`, giữ `/api/v1/...` chạy song song.
4. Ghi vào file này, nêu rõ ảnh hưởng tới bên nào và cần làm gì.

---

## [1.2.0] — 2026-10-27

Không có thay đổi phá vỡ — chỉ thêm.

| Thay đổi | Lý do |
|---|---|
| Thêm `PlaceDetail.externalLinks` (schema `PlaceExternalLink`, enum `ExternalPlatform`) | Trang chi tiết quán dẫn sang bài review trên TikTok, Facebook, Instagram, YouTube |
| Thêm field tuỳ chọn `UpsertPlaceRequest.externalLinks` | Admin nhập/sửa danh sách liên kết khi tạo hoặc sửa địa điểm |

---

## [1.1.0] — 2026-10-26

Không có thay đổi phá vỡ — chỉ thêm.

| Thay đổi | Lý do |
|---|---|
| Thêm `GET /api/v1/reviews/feed` (`listReviewFeed`) | Lưới bài viết ở tab Khám phá |
| Thêm field tuỳ chọn `Review.place` (schema mới `ReviewPlace`) | Thẻ bài viết cần tên quán khi danh sách trộn nhiều địa điểm (bảng tin, `GET /me/reviews`) |
| Thêm tham số `sort` (`RELEVANCE` \| `POPULAR`) cho `GET /api/v1/places` | Mục "Phổ biến" ở Khám phá (DISCOVERY-03) — đặc tả đã có, hợp đồng còn thiếu |
| `GET /api/v1/places/{placeId}` và `.../reviews` trả cả quán `PERMANENTLY_CLOSED` (trước đây 404) | Quán đã đóng hẳn vẫn nằm trong Yêu thích; bấm vào phải xem được, kèm nhãn |

---

## [1.0.0] — 2026-08-14

Phiên bản đầu tiên. Chưa có gì để phá vỡ.

**Nhóm endpoint**

| Nhóm | Nội dung |
|---|---|
| `auth` | Đăng ký, đăng nhập, làm mới token (dùng một lần), đăng xuất, quên/đặt lại mật khẩu, xác minh email |
| `user` | Xem, sửa, yêu cầu xoá hồ sơ |
| `place` | Tìm quanh đây, tìm từ khoá, chi tiết, đề xuất địa điểm mới |
| `category` | Danh sách danh mục món ăn |
| `review` | Gửi/cập nhật, xem theo địa điểm, xem của mình, xoá của mình |
| `feedback` | Gửi báo sai, xem của mình |
| `favorite` | Thêm (idempotent), bỏ, xem danh sách |
| `visit` | Ghi lượt đến, xem lịch sử của mình |
| `media` | Xin presigned URL, xác nhận upload |
| `notification` | Danh sách, đánh dấu đã đọc, cấu hình push, đăng ký push token |
| `chat` | Gửi tin nhắn (SSE), xem và xoá lịch sử phiên |
| `admin` | Kiểm duyệt review, xử lý feedback, CRUD địa điểm, quản lý user, thống kê |

**Quy ước áp dụng từ đầu**
- Mọi danh sách phân trang qua schema `PageMeta` (`content` + `page` + `size` +
  `totalElements` + `totalPages`).
- Mọi lỗi dùng schema `ApiError` (`code` + `message` + `details?` + `traceId`).
- `code` **không dịch** (client so sánh bằng nó); `message` **có dịch** theo `Accept-Language`.
- Thời gian là `date-time` ISO-8601 UTC. Ngày lịch (`visitDate`) là `date` theo giờ
  `Asia/Ho_Chi_Minh`.
- ID là `uuid`.
- `averageRating` **nullable** — `null` khi chưa có đánh giá, không phải `0`.
