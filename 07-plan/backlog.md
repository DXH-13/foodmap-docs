# Backlog

Ý tưởng và câu hỏi **chưa** thuộc phạm vi v1. Ghi ở đây thay vì bịa chi tiết trong SRS.

Chốt được cái nào thì chuyển sang [`srs.md`](../01-srs/srs.md) với mã yêu cầu đầy đủ,
và xoá khỏi file này.

---

## Câu hỏi mở cần trả lời trước khi ra mắt

| # | Câu hỏi | Vì sao quan trọng |
|---|---|---|
| Q1 | Dữ liệu 5.000 địa điểm ban đầu lấy ở đâu? Nhập tay, seed từ Google Places, hay kết hợp? | Quyết định này ảnh hưởng chi phí Places API và thời gian Phase 6 |
| Q2 | Ai làm moderator, và bao nhiêu người? | Quyết định luồng kiểm duyệt có kịp không, và có cần tự động duyệt một phần không |
| Q3 | Chính sách quyền riêng tư và điều khoản sử dụng ai soạn? | Bắt buộc phải có để submit lên store |
| Q4 | Nền tảng triển khai production? | Xem [`deployment.md`](../06-ops/deployment.md); chưa chốt |
| Q5 | Có cần đăng nhập bằng Google / Apple không? Apple **bắt buộc** Sign in with Apple nếu app có đăng nhập bằng mạng xã hội khác | Ảnh hưởng phạm vi Phase 1 và khả năng được duyệt trên App Store |
| Q6 | Ngưỡng nào thì tự động duyệt review mà không cần moderator? | Nếu hàng chờ vượt khả năng xử lý thì phải có cơ chế này |

---

## Tính năng đã cân nhắc, hoãn sang sau v1

### Tài khoản cho chủ quán
Chủ quán tự quản lý thông tin, trả lời review, cập nhật giờ mở cửa.
**Hoãn vì:** cần cơ chế xác minh quyền sở hữu quán — bài toán riêng, không nhỏ.
Ở v1 chủ quán gửi feedback như người dùng thường.

### Gợi ý cá nhân hoá
Gợi ý dựa trên lịch sử đã đến và đánh giá của từng người.
**Hoãn vì:** cần lượng dữ liệu hành vi đủ lớn mới có ý nghĩa. Chưa có người dùng thì
chưa có dữ liệu.

### Tính năng mạng xã hội
Theo dõi người dùng khác, bảng tin, danh sách chia sẻ được.
**Hoãn vì:** mở rộng phạm vi rất nhiều và kéo theo cả bài toán kiểm duyệt nội dung
giữa người dùng với nhau.

### Web app cho người dùng cuối
**Hoãn vì:** v1 tập trung vào mobile — người dùng tìm quán khi đang di chuyển.
Web hợp lý hơn sau khi có SEO cho trang chi tiết địa điểm.

### Chế độ ngoại tuyến hoàn toàn
Tải trước cả một khu vực để dùng khi không có mạng.
**Hoãn vì:** phức tạp (đồng bộ, giải quyết xung đột). v1 chỉ cache những gì đã xem (NFR-08).

### Đặt bàn, đặt món, thanh toán
**Hoãn vì:** phần lớn địa điểm trong FoodMap là hàng rong và quầy chợ — không có hệ thống
đặt chỗ để tích hợp. Sai đối tượng.

---

## Ý tưởng chưa đánh giá

Ghi lại để không quên, chưa cam kết gì.

- **Danh sách tự tạo** — "Quán ăn sáng quận 1", chia sẻ bằng link
- **Huy hiệu / thành tựu** — thưởng cho người đóng góp nhiều; cần cẩn thận vì có thể
  khuyến khích review kém chất lượng
- **Lọc theo khoảng giá** — cần thêm trường dữ liệu và cách thu thập đáng tin
- **Chỉ đường** — hiện chỉ mở app bản đồ ngoài; tích hợp sâu hơn có thể hữu ích
- **Bộ lọc ăn kiêng** — chay, không gluten, halal; giá trị cao với khách nước ngoài
- **Đóng góp bản dịch** — cho người dùng thêm bản dịch tiếng Anh cho địa điểm,
  moderator duyệt
- **Xem theo góc nhìn quán** — số lượt xem, số lượt lưu; tiền đề cho tài khoản chủ quán
- **Chatbot gợi ý chủ động** — "gần chỗ bạn có quán mới mở"; cần cẩn thận để không phiền
- **Xuất lịch sử đã đến** — dạng CSV hoặc bản đồ tổng kết năm

---

## Nợ kỹ thuật cần theo dõi

Chưa phải vấn đề, nhưng sẽ thành vấn đề nếu bỏ qua.

| Mục | Khi nào cần xử lý |
|---|---|
| `openapi.yaml` là một file duy nhất | Vượt ~2.000 dòng thì tách theo module bằng `$ref` |
| Chưa có cache cho tìm quanh đây | Khi vượt 100.000 địa điểm hoặc p95 chạm ngưỡng NFR-01 |
| Backend là một module Gradle duy nhất | Khi số feature khiến thời gian build khó chịu |
| Chưa có test end-to-end | Trước khi ra mắt production |
| Chưa có giám sát và cảnh báo | Trước khi ra mắt production — cần uptime, tỷ lệ lỗi, p95, dung lượng đĩa |
| Bước `gen-api-client` là thủ công | Nếu xảy ra vài lần quên chạy thì đưa vào CI làm cổng bắt buộc |
| Chưa có staging | Khi có người dùng thật và cần diễn tập phát hành |
