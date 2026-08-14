# Use case

Mỗi use case có: **Tác nhân · Tiền điều kiện · Luồng chính · Luồng thay thế · Hậu điều kiện**.
Luồng thay thế đánh số theo bước tương ứng ở luồng chính (`3a`, `3b`…).

---

## UC-PLACE-01 — Tìm quán ăn quanh vị trí hiện tại

**Tác nhân:** GUEST, USER
**Tiền điều kiện:** Ứng dụng đã mở. Không bắt buộc đăng nhập.
**Yêu cầu liên quan:** FR-PLACE-01, FR-PLACE-02, FR-PLACE-03, FR-PLACE-06, NFR-01, NFR-03

**Luồng chính**
1. Người dùng mở tab Bản đồ.
2. Hệ thống xin quyền truy cập vị trí (nếu chưa có), kèm giải thích ngắn vì sao cần.
3. Hệ thống lấy toạ độ hiện tại và căn bản đồ vào đó.
4. Hệ thống gọi API tìm quanh đây với bán kính mặc định 2.000m.
5. Hệ thống hiển thị marker trên bản đồ và danh sách rút gọn bên dưới, sắp xếp theo
   khoảng cách tăng dần, mỗi mục kèm tên, khoảng cách, điểm trung bình và trạng thái
   đang mở / đã đóng.
6. Người dùng bấm một marker hoặc một mục trong danh sách → chuyển sang UC-PLACE-02.

**Luồng thay thế**

- **2a. Người dùng từ chối quyền vị trí.**
  Hệ thống hiển thị trạng thái rỗng có ý nghĩa kèm ô tìm theo tên thành phố/quận và
  nút mở Cài đặt. Không để màn hình trắng, không crash.
- **4a. Không có địa điểm nào trong bán kính.**
  Hệ thống báo "Không có địa điểm nào trong bán kính 2km" và gợi ý mở rộng bán kính.
- **4b. Gọi API thất bại (mất mạng, lỗi server).**
  Hệ thống hiển thị thông báo lỗi kèm nút Thử lại. Có dữ liệu cache thì hiển thị kèm
  nhãn "dữ liệu ngoại tuyến" (NFR-08).
- **5a. Người dùng di chuyển bản đồ.**
  Hệ thống chỉ gọi lại API sau khi bản đồ **ngừng** di chuyển, có debounce 500ms.
  Bán kính suy ra từ vùng hiển thị, giới hạn trong 2–50km.
- **5b. Người dùng đặt bộ lọc** (loại địa điểm, danh mục, điểm tối thiểu, đang mở cửa).
  Hệ thống gọi lại API với bộ lọc và cập nhật cả marker lẫn danh sách.

**Hậu điều kiện:** Người dùng thấy danh sách địa điểm phù hợp quanh vị trí hiện tại.
Không có dữ liệu nào bị thay đổi.

---

## UC-PLACE-02 — Xem chi tiết địa điểm

**Tác nhân:** GUEST, USER
**Tiền điều kiện:** Đã chọn một địa điểm.
**Yêu cầu liên quan:** FR-PLACE-05, FR-PLACE-06, FR-PLACE-12, FR-REVIEW-07, FR-I18N-03

**Luồng chính**
1. Hệ thống tải chi tiết địa điểm và trang đầu tiên của review đã duyệt.
2. Hệ thống hiển thị: ảnh, tên, loại, danh mục, điểm trung bình, số review, số lượt đến,
   địa chỉ, giờ mở cửa, trạng thái mở cửa lúc này, và danh sách review.
3. Người dùng cuộn xuống → hệ thống tải thêm review (phân trang vô hạn).

**Luồng thay thế**

- **2a. Địa điểm chưa có review nào.**
  Hiển thị "Chưa có đánh giá" — **không** hiển thị 0 sao (FR-PLACE-12).
- **2b. Ngôn ngữ hiện tại là `en` nhưng địa điểm chưa có bản dịch tiếng Anh.**
  Hiển thị nội dung tiếng Việt, không để trống (FR-I18N-03).
- **2c. Địa điểm ở trạng thái `TEMPORARILY_CLOSED`.**
  Hiển thị nhãn nổi bật "Tạm đóng cửa" ở đầu màn hình.

**Hậu điều kiện:** Chi tiết địa điểm được hiển thị. Không thay đổi dữ liệu.

---

## UC-REVIEW-01 — Viết đánh giá kèm ảnh

**Tác nhân:** USER (đã xác minh email)
**Tiền điều kiện:** Đã đăng nhập, email đã xác minh, đang ở màn hình chi tiết địa điểm.
**Yêu cầu liên quan:** FR-REVIEW-01 → 06, FR-MEDIA-01 → 04, FR-AUTH-02

**Luồng chính**
1. Người dùng bấm "Viết đánh giá".
2. Hệ thống hiển thị form: chọn sao (1–5, bắt buộc), nội dung (tối đa 2.000 ký tự),
   nút thêm ảnh/video.
3. Người dùng chọn tối đa 5 ảnh và 1 video từ thư viện hoặc chụp mới.
4. Hệ thống nén media phía client, xin presigned URL từ backend, upload thẳng lên
   object storage, hiển thị tiến trình từng file.
5. Người dùng bấm Gửi.
6. Hệ thống tạo review ở trạng thái `PENDING` và gắn media vừa upload.
7. Hệ thống hiển thị xác nhận: "Đánh giá của bạn đang chờ duyệt".

**Luồng thay thế**

- **1a. Chưa đăng nhập.** Chuyển sang màn hình đăng nhập, quay lại đúng chỗ sau khi xong.
- **1b. Đã đăng nhập nhưng chưa xác minh email.**
  Hiển thị thông báo yêu cầu xác minh kèm nút gửi lại email (FR-AUTH-02).
- **1c. Người dùng đã có review cho địa điểm này.**
  Form nạp sẵn nội dung cũ; khi gửi là **cập nhật**, không tạo mới. Review đang `APPROVED`
  bị sửa sẽ quay về `PENDING` (FR-REVIEW-03).
- **3a. Vượt quá 5 ảnh hoặc 1 video.** Chặn ngay ở giao diện kèm giải thích giới hạn.
- **4a. Upload một file thất bại.**
  Cho thử lại riêng file đó, không bắt làm lại toàn bộ. Người dùng cũng bỏ được file lỗi.
- **5a. Chưa chọn sao.** Chặn gửi và làm nổi bật trường bắt buộc.
- **6a. Gọi API thất bại.** Giữ nguyên nội dung đã nhập, hiển thị lỗi và nút Thử lại.
  **Không** để mất bài viết của người dùng.

**Hậu điều kiện:** Review tồn tại ở trạng thái `PENDING`. Điểm trung bình của địa điểm
**chưa** thay đổi (chỉ đổi sau khi duyệt — FR-REVIEW-06).

---

## UC-REVIEW-02 — Kiểm duyệt đánh giá

**Tác nhân:** MODERATOR, ADMIN
**Tiền điều kiện:** Đã đăng nhập trang quản trị với vai trò phù hợp.
**Yêu cầu liên quan:** FR-REVIEW-04, FR-REVIEW-05, FR-REVIEW-06, FR-ADMIN-03, FR-ADMIN-06, FR-ADMIN-09

**Luồng chính**
1. Moderator mở hàng chờ duyệt review, mặc định lọc `PENDING`, sắp xếp cũ nhất trước.
2. Hệ thống hiển thị danh sách kèm ngữ cảnh: ảnh thu nhỏ, sao, trích nội dung, tên địa điểm,
   tác giả và số review đã bị từ chối trước đó của tác giả (FR-ADMIN-03).
3. Moderator xem một mục và bấm **Duyệt**.
4. Hệ thống chuyển review sang `APPROVED`, tính lại `averageRating` và `reviewCount`
   của địa điểm, gửi thông báo `REVIEW_APPROVED` cho tác giả, và ghi audit log.

**Luồng thay thế**

- **3a. Moderator bấm Từ chối.**
  Hệ thống **bắt buộc** nhập lý do trước khi cho xác nhận (FR-REVIEW-05).
  Sau khi xác nhận: chuyển `REJECTED`, gửi thông báo `REVIEW_REJECTED` kèm lý do,
  ghi audit log. Điểm trung bình không đổi.
- **3b. Moderator bấm Ẩn** với một review đang `APPROVED`.
  Chuyển sang `HIDDEN`, tính lại điểm trung bình, ghi audit log. Có thể hiện lại sau.
- **3c. Review đã bị người khác xử lý trong lúc đang xem.**
  Hệ thống báo xung đột và tải lại trạng thái mới, không ghi đè.

**Hậu điều kiện:** Review ở trạng thái cuối. Điểm trung bình của địa điểm phản ánh đúng
tập review `APPROVED`. Có bản ghi audit log không sửa được.

---

## UC-FEEDBACK-01 — Báo thông tin địa điểm sai

**Tác nhân:** USER (đã xác minh email)
**Tiền điều kiện:** Đã đăng nhập, đang ở màn hình chi tiết địa điểm.
**Yêu cầu liên quan:** FR-FEEDBACK-01 → 06

**Luồng chính**
1. Người dùng bấm "Báo thông tin sai".
2. Hệ thống hiển thị danh sách loại: sai địa chỉ, sai giờ mở cửa, đã đóng cửa vĩnh viễn,
   trùng lặp, nội dung không phù hợp, khác.
3. Người dùng chọn loại và nhập mô tả (tối đa 1.000 ký tự).
4. Người dùng gửi.
5. Hệ thống tạo feedback ở trạng thái `OPEN` và xác nhận đã nhận.

**Luồng thay thế**

- **2a. Người dùng chọn "Trùng lặp".**
  Hệ thống cho phép tìm và chọn địa điểm bị trùng (FR-FEEDBACK-06).
- **4a. Người dùng đã có feedback cùng loại đang `OPEN` cho địa điểm này.**
  Hệ thống trả 409, hiển thị "Bạn đã báo vấn đề này rồi, đang được xử lý"
  kèm liên kết tới báo cáo đang mở (FR-FEEDBACK-03).
- **5a. Đây là feedback `CLOSED_PERMANENTLY` thứ ba từ ba người khác nhau trong 30 ngày.**
  Hệ thống đặt `place.needsReview = true` và tạo thông báo cho moderator (FR-FEEDBACK-05).

**Hậu điều kiện:** Feedback tồn tại ở trạng thái `OPEN`, chỉ tác giả và moderator xem được.
Địa điểm **không** bị thay đổi tự động.

---

## UC-VISIT-01 — Ghi nhận đã đến quán

**Tác nhân:** USER (đã xác minh email)
**Tiền điều kiện:** Đã đăng nhập, đã cấp quyền vị trí, đang ở màn hình chi tiết địa điểm.
**Yêu cầu liên quan:** FR-VISIT-01 → 04

**Luồng chính**
1. Người dùng bấm "Đã đến đây".
2. Hệ thống lấy toạ độ hiện tại.
3. Hệ thống gửi yêu cầu ghi visit kèm toạ độ.
4. Backend kiểm tra: toạ độ nằm trong 200m quanh địa điểm, và user chưa ghi visit cho
   địa điểm này trong ngày hôm nay (giờ Việt Nam).
5. Hệ thống ghi visit, tăng `visitCount` của địa điểm, và hiển thị số lượt của
   người dùng tại địa điểm này.

**Luồng thay thế**

- **2a. Không lấy được vị trí.**
  Báo lỗi kèm hướng dẫn bật GPS. Không cho ghi visit khi thiếu toạ độ.
- **4a. Toạ độ nằm ngoài bán kính 200m.**
  Trả 409 mã `VISIT_TOO_FAR`, hiển thị "Bạn cần ở gần quán để ghi nhận đã đến"
  kèm khoảng cách hiện tại.
- **4b. Đã ghi visit cho địa điểm này hôm nay.**
  Trả 409 mã `VISIT_ALREADY_TODAY`, hiển thị "Bạn đã ghi nhận đến quán này hôm nay".
  Đây **không** phải lỗi hệ thống, thông báo phải nhẹ nhàng.

**Hậu điều kiện:** Có thêm một bản ghi visit. `place.visitCount` tăng 1.
`distinctVisitorCount` chỉ tăng nếu đây là lần đầu người dùng này đến.

---

## UC-CHAT-01 — Hỏi chatbot gợi ý quán ăn

**Tác nhân:** USER
**Tiền điều kiện:** Đã đăng nhập.
**Yêu cầu liên quan:** FR-CHAT-01 → 08, NFR-04

**Luồng chính**
1. Người dùng mở tab Chat và nhập câu hỏi, ví dụ "quanh đây có quán bún bò nào ngon không".
2. Hệ thống gửi câu hỏi kèm toạ độ hiện tại (nếu có quyền) lên backend.
3. Backend gọi mô hình AI, mô hình gọi ngược các tool `search_places` / `recommend_nearby`
   để truy vấn dữ liệu thật của FoodMap.
4. Backend stream câu trả lời về client qua SSE; token đầu tiên xuất hiện trong 2 giây.
5. Client hiển thị câu trả lời dần, và render thẻ bấm được cho từng địa điểm được nhắc tới.
6. Người dùng bấm một thẻ → chuyển sang UC-PLACE-02.

**Luồng thay thế**

- **1a. Câu hỏi bằng tiếng Anh.** Chatbot trả lời bằng tiếng Anh (FR-CHAT-02).
- **3a. Không tìm thấy địa điểm nào phù hợp.**
  Chatbot nói rõ là không tìm thấy và gợi ý mở rộng tiêu chí. **Không** bịa ra địa điểm
  không có trong CSDL (FR-CHAT-03).
- **4a. Dịch vụ AI lỗi hoặc quá tải.**
  Hiển thị thông báo thân thiện kèm nút Thử lại. Không hiện chi tiết kỹ thuật (FR-CHAT-08).
- **2a. Người dùng đã gửi quá 30 tin nhắn trong giờ vừa rồi.**
  Trả 429 kèm thời gian chờ, hiển thị rõ khi nào dùng lại được (FR-CHAT-07).

**Hậu điều kiện:** Câu hỏi và câu trả lời được lưu vào lịch sử phiên chat.
Không có dữ liệu địa điểm nào bị thay đổi.

---

## UC-AUTH-01 — Đặt lại mật khẩu đã quên

**Tác nhân:** GUEST
**Tiền điều kiện:** Đang ở màn hình đăng nhập.
**Yêu cầu liên quan:** FR-AUTH-05, FR-AUTH-08, NFR-12

**Luồng chính**
1. Người dùng bấm "Quên mật khẩu".
2. Người dùng nhập email và gửi.
3. Hệ thống hiển thị: "Nếu email này đã đăng ký, chúng tôi đã gửi hướng dẫn đặt lại mật khẩu."
4. Hệ thống gửi email chứa link đặt lại (TTL 30 phút, dùng một lần).
5. Người dùng mở link, nhập mật khẩu mới hai lần.
6. Hệ thống đổi mật khẩu, vô hiệu toàn bộ refresh token đang có, và chuyển về màn hình
   đăng nhập.

**Luồng thay thế**

- **3a. Email không tồn tại trong hệ thống.**
  Hệ thống hiển thị **thông báo giống hệt** bước 3 và không gửi email nào.
  Đây là chủ ý — để không lộ email nào đã đăng ký (FR-AUTH-05).
- **5a. Link đã hết hạn hoặc đã dùng.**
  Báo lỗi rõ ràng kèm nút yêu cầu link mới.
- **2a. Người dùng đã yêu cầu 3 lần trong giờ vừa rồi.**
  Trả 429 (FR-AUTH-08). Thông báo vẫn giữ nguyên dạng chung chung để không lộ thông tin.

**Hậu điều kiện:** Mật khẩu đã đổi và mọi phiên cũ bị vô hiệu. Token đặt lại không dùng lại được.
