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
