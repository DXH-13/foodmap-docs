# Tiêu chí nghiệm thu

Viết theo dạng **Given / When / Then**. Mỗi tiêu chí phải kiểm chứng được bằng một test
tự động hoặc một bước kiểm thử thủ công cụ thể.

---

## AUTH

**AC-AUTH-01** — Đăng ký thành công
> **Given** email `an@example.com` chưa tồn tại
> **When** đăng ký với mật khẩu `MatKhau123`
> **Then** trả 201, tài khoản ở trạng thái chưa xác minh, và một email xác minh được gửi

**AC-AUTH-02** — Email trùng
> **Given** `an@example.com` đã đăng ký
> **When** đăng ký lại với email đó
> **Then** trả 409 mã `EMAIL_ALREADY_EXISTS`

**AC-AUTH-03** — Chưa xác minh không viết được review
> **Given** đã đăng nhập nhưng email chưa xác minh
> **When** gửi review
> **Then** trả 403 mã `EMAIL_NOT_VERIFIED`

**AC-AUTH-04** — Refresh token dùng một lần
> **Given** đã làm mới token bằng refresh token `R1`, nhận được `R2`
> **When** gọi làm mới lần nữa bằng `R1`
> **Then** trả 401 và **toàn bộ** refresh token của user đó bị vô hiệu

**AC-AUTH-05** — Quên mật khẩu không lộ email
> **Given** `khongton@example.com` chưa từng đăng ký
> **When** yêu cầu đặt lại mật khẩu cho email đó
> **Then** trả 200 với **thông báo giống hệt** trường hợp email có thật, và không email nào được gửi

**AC-AUTH-06** — Giới hạn đăng nhập sai
> **Given** đã đăng nhập sai 5 lần cho `an@example.com` trong 15 phút
> **When** thử lần thứ 6 (kể cả đúng mật khẩu)
> **Then** trả 429 kèm header `Retry-After`

---

## PLACE

**AC-PLACE-01** — Khách vãng lai xem được bản đồ
> **Given** không gửi token
> **When** gọi `GET /places/nearby`
> **Then** trả 200 kèm danh sách địa điểm

**AC-PLACE-02** — Sắp xếp theo khoảng cách
> **Given** có 3 địa điểm cách vị trí truy vấn lần lượt 100m, 500m, 1.500m
> **When** tìm quanh đây bán kính 2.000m
> **Then** trả đủ 3, đúng thứ tự tăng dần, mỗi mục có `distanceMeters` sai số dưới 10m

**AC-PLACE-03** — Bán kính vượt giới hạn
> **Given** bất kỳ
> **When** tìm quanh đây với `radiusMeters = 60000`
> **Then** trả 400 mã `RADIUS_OUT_OF_RANGE` — **không** âm thầm cắt về 50.000

**AC-PLACE-04** — Tìm không dấu
> **Given** có địa điểm tên `Phở Thìn`
> **When** tìm từ khoá `pho thin`
> **Then** địa điểm đó nằm trong kết quả

**AC-PLACE-05** — Trạng thái mở cửa theo giờ hiện tại
> **Given** địa điểm mở 06:00–10:00 và 17:00–21:00 thứ Hai
> **When** truy vấn lúc 11:30 thứ Hai giờ Việt Nam
> **Then** `isOpenNow = false` và `nextOpenAt` là 17:00 cùng ngày

**AC-PLACE-06** — Không có review
> **Given** địa điểm chưa có review `APPROVED` nào
> **When** xem chi tiết
> **Then** `averageRating` là `null`, **không** phải `0`

**AC-PLACE-07** — Địa điểm nháp không lộ ra công khai
> **Given** địa điểm ở trạng thái `DRAFT`
> **When** gọi API công khai bất kỳ
> **Then** địa điểm đó không xuất hiện trong kết quả

---

## REVIEW

**AC-REVIEW-01** — Review mới ở trạng thái chờ
> **Given** user đã xác minh
> **When** gửi review rating 5
> **Then** trả 201 với `status = PENDING`, và `place.averageRating` **không đổi**

**AC-REVIEW-02** — Một review mỗi user mỗi place
> **Given** user đã có review cho place X
> **When** gửi review mới cho place X
> **Then** review cũ được **cập nhật**, tổng số review của place không tăng

**AC-REVIEW-03** — Sửa review đã duyệt thì quay lại chờ duyệt
> **Given** user có review `APPROVED` cho place X
> **When** sửa nội dung review đó
> **Then** `status` trở thành `PENDING` và `place.averageRating` được tính lại (bỏ review này ra)

**AC-REVIEW-04** — Từ chối bắt buộc có lý do
> **Given** moderator đang xử lý một review `PENDING`
> **When** từ chối mà không nhập `moderationNote`
> **Then** trả 400 mã `MODERATION_NOTE_REQUIRED`

**AC-REVIEW-05** — Duyệt thì cập nhật điểm trung bình
> **Given** place X có 2 review `APPROVED` điểm 4 và 5 (trung bình 4.5)
> **When** duyệt thêm một review điểm 3
> **Then** `averageRating = 4.0` và `reviewCount = 3`

**AC-REVIEW-06** — Giới hạn media
> **Given** user đang tạo review
> **When** đính kèm 6 ảnh
> **Then** trả 400 mã `TOO_MANY_IMAGES`

---

## FEEDBACK

**AC-FEEDBACK-01** — Không tạo trùng feedback đang mở
> **Given** user đã có feedback `WRONG_HOURS` đang `OPEN` cho place X
> **When** gửi tiếp một feedback `WRONG_HOURS` cho place X
> **Then** trả 409 mã `FEEDBACK_ALREADY_OPEN` kèm `existingFeedbackId`

**AC-FEEDBACK-02** — Feedback không công khai
> **Given** user A đã gửi feedback cho place X
> **When** user B (vai trò `USER`) gọi API xem feedback của place X
> **Then** trả 403

**AC-FEEDBACK-03** — Tự gắn cờ sau 3 báo cáo đóng cửa
> **Given** place X đã có 2 feedback `CLOSED_PERMANENTLY` từ 2 user khác nhau trong 30 ngày
> **When** user thứ ba gửi feedback `CLOSED_PERMANENTLY` cho place X
> **Then** `place.needsReview = true` và có thông báo gửi tới moderator

---

## VISIT

**AC-VISIT-01** — Ghi visit hợp lệ
> **Given** user đang ở cách place X 50m
> **When** ghi visit
> **Then** trả 201, `place.visitCount` tăng 1

**AC-VISIT-02** — Quá xa
> **Given** user đang ở cách place X 500m
> **When** ghi visit
> **Then** trả 409 mã `VISIT_TOO_FAR`, `visitCount` **không đổi**

**AC-VISIT-03** — Trùng trong ngày
> **Given** user đã ghi visit cho place X lúc 08:00 hôm nay (giờ Việt Nam)
> **When** ghi visit lần nữa lúc 19:00 cùng ngày
> **Then** trả 409 mã `VISIT_ALREADY_TODAY`, `visitCount` **không đổi**

**AC-VISIT-04** — Ngày mới thì ghi được
> **Given** user đã ghi visit cho place X lúc 23:00 hôm qua
> **When** ghi visit lúc 01:00 hôm nay (giờ Việt Nam)
> **Then** trả 201 — ranh giới ngày tính theo `Asia/Ho_Chi_Minh`, không phải UTC

---

## FAVORITE

**AC-FAVORITE-01** — Idempotent
> **Given** place X đã nằm trong danh sách yêu thích của user
> **When** gọi thêm yêu thích lần nữa
> **Then** trả 200, danh sách không có bản ghi trùng

---

## I18N

**AC-I18N-01** — Fallback về tiếng Việt
> **Given** place X chỉ có bản dịch `vi`, không có `en`
> **When** gọi API với `Accept-Language: en`
> **Then** trả về tên và mô tả **tiếng Việt** — không phải chuỗi rỗng, không phải `null`

**AC-I18N-02** — Lỗi được dịch, mã lỗi thì không
> **Given** bất kỳ
> **When** gây lỗi 404 với `Accept-Language: en`
> **Then** `message` bằng tiếng Anh, `code` vẫn là `PLACE_NOT_FOUND` (không dịch)

**AC-I18N-03** — Không thiếu key dịch
> **Given** bộ file i18n của mobile và admin
> **When** so sánh tập key giữa `vi.json` và `en.json`
> **Then** hai tập key giống hệt nhau — không key nào chỉ có ở một bên

---

## NOTIF

**AC-NOTIF-01** — Không push ban đêm
> **Given** thông báo loại `REVIEW_APPROVED` được tạo lúc 23:30 giờ Việt Nam
> **Then** bản ghi in-app được tạo **ngay**, nhưng push **không** gửi lúc đó mà hoãn tới 07:00

**AC-NOTIF-02** — Thông báo hệ thống không bị hoãn
> **Given** thông báo `SYSTEM_ANNOUNCEMENT` được tạo lúc 23:30
> **Then** push được gửi ngay

---

## CHAT

**AC-CHAT-01** — Không bịa địa điểm
> **Given** trong CSDL không có địa điểm nào khớp "quán sushi ở Cà Mau"
> **When** hỏi chatbot câu đó
> **Then** chatbot trả lời là không tìm thấy — **không** nêu tên địa điểm nào không có trong CSDL

**AC-CHAT-02** — Trả lời đúng ngôn ngữ
> **Given** user hỏi bằng tiếng Anh
> **Then** chatbot trả lời bằng tiếng Anh

**AC-CHAT-03** — Có id để mở chi tiết
> **Given** chatbot nhắc tới 3 địa điểm trong câu trả lời
> **Then** phản hồi kèm `placeIds` gồm đúng 3 id, client render được 3 thẻ bấm mở

**AC-CHAT-04** — Giới hạn tốc độ
> **Given** user đã gửi 30 tin nhắn trong giờ vừa rồi
> **When** gửi tin thứ 31
> **Then** trả 429 kèm `Retry-After`

---

## Phi chức năng

**AC-NFR-01** — Hiệu năng tìm quanh đây
> **Given** CSDL có 10.000 địa điểm
> **When** chạy 100 lượt tìm quanh đây bán kính 2km
> **Then** phân vị 95 của thời gian phản hồi dưới 500ms

**AC-NFR-02** — Truy vấn địa lý dùng index
> **When** chạy `EXPLAIN ANALYZE` cho truy vấn tìm quanh đây
> **Then** kế hoạch thực thi có dùng index GiST trên `places.location`,
> **không** có `Seq Scan` trên bảng `places`

**AC-NFR-03** — Phân quyền chặn ở backend
> **Given** user vai trò `USER` (không phải moderator)
> **When** gọi thẳng API duyệt review bằng curl
> **Then** trả 403 — không phụ thuộc việc giao diện có ẩn nút hay không

**AC-NFR-04** — Không secret trong repo
> **When** quét toàn bộ mã nguồn đã commit
> **Then** không tìm thấy API key, mật khẩu hay token nào

**AC-NFR-05** — Migration chạy sạch từ đầu
> **Given** một database rỗng
> **When** chạy toàn bộ migration Flyway
> **Then** hoàn tất không lỗi và lược đồ khớp với ERD
