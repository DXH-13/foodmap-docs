# Đặc tả yêu cầu phần mềm — FoodMap v1

| | |
|---|---|
| Phiên bản | 1.0 (bản nháp) |
| Phạm vi | FoodMap v1 |
| Đối tượng đọc | Đội phát triển, kiểm thử, AI agent |

Tài liệu liên quan: [tầm nhìn](../00-overview/vision.md) ·
[thuật ngữ](../00-overview/glossary-vi-en.md) · [use case](./use-cases.md) ·
[tiêu chí nghiệm thu](./acceptance-criteria.md) · [ERD](../04-data/erd.md) ·
[hợp đồng API](../03-api/openapi.yaml)

---

## 1. Tổng quan

FoodMap gồm bốn thành phần:

| Thành phần | Công nghệ | Người dùng |
|---|---|---|
| Ứng dụng mobile | React Native + Expo | GUEST, USER |
| API backend | Java 21 + Spring Boot 3 | — |
| Trang quản trị | Next.js 15 | MODERATOR, ADMIN |
| Cơ sở dữ liệu | PostgreSQL 16 + PostGIS | — |

Vai trò và quyền: xem [thuật ngữ](../00-overview/glossary-vi-en.md#vai-trò).

**Quy ước đọc:** "phải" = bắt buộc ở v1. "nên" = mong muốn, có thể lùi sang v1.1.

---

## 2. Yêu cầu chức năng

### 2.1 Xác thực và tài khoản — `AUTH`

| Mã | Yêu cầu |
|---|---|
| **FR-AUTH-01** | Người dùng đăng ký bằng email + mật khẩu. Email phải là duy nhất trong hệ thống. Mật khẩu tối thiểu 8 ký tự, có ít nhất một chữ cái và một chữ số. |
| **FR-AUTH-02** | Sau khi đăng ký, hệ thống gửi email xác minh. Tài khoản chưa xác minh vẫn đăng nhập và xem được, nhưng **không** được viết review, gửi feedback hay ghi visit. |
| **FR-AUTH-03** | Đăng nhập trả về access token (TTL 15 phút) và refresh token (TTL 30 ngày). Refresh token lưu ở Redis để thu hồi được. |
| **FR-AUTH-04** | Refresh token dùng một lần: mỗi lần làm mới sinh token mới và vô hiệu token cũ. Dùng lại token đã tiêu huỷ toàn bộ phiên của user đó và ghi log cảnh báo. |
| **FR-AUTH-05** | Quên mật khẩu: nhập email, nhận link đặt lại có TTL 30 phút, dùng một lần. Email không tồn tại trong hệ thống vẫn trả về **thông báo thành công giống hệt** để không lộ email nào đã đăng ký. |
| **FR-AUTH-06** | Đăng xuất vô hiệu refresh token hiện tại và gỡ push token của thiết bị đó. |
| **FR-AUTH-07** | Mật khẩu băm bằng BCrypt cost 12. Không bao giờ lưu, log hay trả về mật khẩu ở dạng gốc. |
| **FR-AUTH-08** | Giới hạn tốc độ: tối đa 5 lần đăng nhập sai / email / 15 phút; 3 yêu cầu đặt lại mật khẩu / email / giờ. Vượt ngưỡng trả 429. |
| **FR-AUTH-09** | Người dùng xem và sửa được hồ sơ: tên hiển thị, ảnh đại diện, ngôn ngữ ưa thích. |
| **FR-AUTH-10** | Người dùng yêu cầu xoá tài khoản. Xoá mềm ngay, xoá cứng sau 30 ngày. Review đã duyệt được giữ lại nhưng ẩn danh tác giả. |

### 2.2 Địa điểm và bản đồ — `PLACE`

| Mã | Yêu cầu |
|---|---|
| **FR-PLACE-01** | Khách vãng lai (chưa đăng nhập) xem được bản đồ, danh sách địa điểm, chi tiết địa điểm và review đã duyệt. |
| **FR-PLACE-02** | Tìm quanh đây: nhận toạ độ + bán kính, trả về danh sách địa điểm sắp xếp theo khoảng cách tăng dần, kèm `distanceMeters`. Bán kính mặc định 2.000m, tối đa 50.000m. Vượt giới hạn trả 400, **không** âm thầm cắt bớt. |
| **FR-PLACE-03** | Lọc kết quả theo: `placeType`, một hoặc nhiều `category`, điểm trung bình tối thiểu, và "đang mở cửa lúc này". |
| **FR-PLACE-04** | Tìm theo từ khoá trên tên và mô tả địa điểm, không phân biệt hoa thường và không phân biệt dấu (gõ "pho" tìm ra "Phở"). |
| **FR-PLACE-05** | Chi tiết địa điểm trả về: tên, mô tả, loại, danh mục, toạ độ, địa chỉ, giờ mở cửa, ảnh, điểm trung bình, số review, số lượt đến, trạng thái. |
| **FR-PLACE-06** | Hệ thống tính và trả về trạng thái mở cửa **tại thời điểm truy vấn** theo giờ `Asia/Ho_Chi_Minh`, dựa trên giờ mở cửa đã lưu. |
| **FR-PLACE-07** | Giờ mở cửa lưu theo từng ngày trong tuần, cho phép nhiều khoảng trong cùng một ngày (bán sáng và tối, nghỉ trưa) và cho phép đánh dấu nghỉ cả ngày. |
| **FR-PLACE-08** | Địa điểm có nhiều danh mục (quan hệ nhiều-nhiều). |
| **FR-PLACE-09** | API công khai chỉ trả địa điểm ở trạng thái `PUBLISHED` và `TEMPORARILY_CLOSED`. `TEMPORARILY_CLOSED` có gắn nhãn. |
| **FR-PLACE-10** | Người dùng đã xác minh đề xuất địa điểm mới. Đề xuất vào trạng thái `DRAFT`, hiển thị công khai sau khi moderator duyệt. |
| **FR-PLACE-11** | Toạ độ lưu bằng PostGIS `geography(Point, 4326)` với index GiST. API phơi ra `latitude` / `longitude` dạng số. |
| **FR-PLACE-12** | Địa điểm chưa có review nào trả `averageRating = null`, **không** trả `0`. Client hiển thị "Chưa có đánh giá". |

### 2.3 Đánh giá — `REVIEW`

| Mã | Yêu cầu |
|---|---|
| **FR-REVIEW-01** | Người dùng đã xác minh viết review cho một địa điểm: `rating` là số nguyên 1–5 (bắt buộc) và `content` tối đa 2.000 ký tự (tuỳ chọn). |
| **FR-REVIEW-02** | Review đính kèm tối đa **5 ảnh** và **1 video**. Video tối đa 60 giây, 100 MB. |
| **FR-REVIEW-03** | Mỗi người dùng chỉ có **một** review đang hoạt động cho mỗi địa điểm. Gửi lại là cập nhật review cũ, không tạo bản ghi mới. Review đã duyệt bị sửa sẽ quay lại trạng thái `PENDING`. |
| **FR-REVIEW-04** | Review mới luôn vào `PENDING`. Chỉ `APPROVED` mới hiển thị công khai. Tác giả luôn thấy review của chính mình kèm nhãn trạng thái. |
| **FR-REVIEW-05** | Chuyển sang `REJECTED` **bắt buộc** kèm `moderationNote`. Hệ thống gửi thông báo cho tác giả kèm lý do. |
| **FR-REVIEW-06** | `place.averageRating` và `place.reviewCount` tính lại mỗi khi review được duyệt, sửa hoặc gỡ. Chỉ review `APPROVED` được tính. |
| **FR-REVIEW-07** | Danh sách review của một địa điểm có phân trang, sắp xếp theo: mới nhất (mặc định), điểm cao nhất, điểm thấp nhất. |
| **FR-REVIEW-08** | Người dùng xoá được review của chính mình. Xoá mềm; điểm trung bình tính lại ngay. |
| **FR-REVIEW-09** | Media của review bị từ chối giữ trên storage 30 ngày rồi mới xoá, để phục vụ khiếu nại. |

### 2.4 Góp ý sửa dữ liệu — `FEEDBACK`

| Mã | Yêu cầu |
|---|---|
| **FR-FEEDBACK-01** | Người dùng đã xác minh gửi feedback về một địa điểm với `type` thuộc: `WRONG_ADDRESS`, `WRONG_HOURS`, `CLOSED_PERMANENTLY`, `DUPLICATE`, `INAPPROPRIATE`, `OTHER`, kèm mô tả tối đa 1.000 ký tự. |
| **FR-FEEDBACK-02** | Feedback **không** hiển thị công khai. Chỉ tác giả, moderator và admin xem được. |
| **FR-FEEDBACK-03** | Cùng user + cùng place + cùng type đang `OPEN` → không tạo bản ghi mới, trả 409 kèm id bản ghi đang mở. |
| **FR-FEEDBACK-04** | Vòng đời: `OPEN` → `IN_REVIEW` → `RESOLVED` hoặc `DISMISSED`. Chuyển sang `RESOLVED` hoặc `DISMISSED` gửi thông báo cho tác giả. |
| **FR-FEEDBACK-05** | Một địa điểm nhận **3 feedback `CLOSED_PERMANENTLY` từ 3 người khác nhau trong 30 ngày** → tự động đặt `place.needsReview = true` và tạo thông báo cho moderator. |
| **FR-FEEDBACK-06** | Feedback loại `DUPLICATE` cho phép chỉ định id địa điểm bị trùng. |

### 2.5 Yêu thích — `FAVORITE`

| Mã | Yêu cầu |
|---|---|
| **FR-FAVORITE-01** | Người dùng đã đăng nhập thêm/bỏ một địa điểm khỏi danh sách yêu thích. |
| **FR-FAVORITE-02** | Thao tác **idempotent**: yêu thích một địa điểm đã yêu thích trả 200, không trả lỗi. |
| **FR-FAVORITE-03** | Danh sách yêu thích có phân trang, sắp xếp theo thời gian thêm hoặc theo khoảng cách tới vị trí hiện tại. |
| **FR-FAVORITE-04** | Địa điểm đã `PERMANENTLY_CLOSED` vẫn ở lại danh sách yêu thích nhưng gắn nhãn "Đã đóng cửa". |

### 2.6 Lượt đến — `VISIT`

| Mã | Yêu cầu |
|---|---|
| **FR-VISIT-01** | Người dùng đã xác minh ghi nhận một lượt đến tại địa điểm. Yêu cầu gửi kèm toạ độ hiện tại. |
| **FR-VISIT-02** | Chống gian lận: toạ độ phải nằm trong **200m** quanh địa điểm, và tối đa **1 lượt / user / place / ngày** (theo `Asia/Ho_Chi_Minh`). Vi phạm trả 409 kèm mã lỗi rõ ràng. |
| **FR-VISIT-03** | Người dùng xem được lịch sử đã đến của mình, có phân trang, và số lượt của chính mình tại từng địa điểm. |
| **FR-VISIT-04** | `place.visitCount` là **tổng số lượt**. Số người khác nhau lưu riêng ở `distinctVisitorCount`. |
| **FR-VISIT-05** | Người dùng **không** xoá được visit. Chỉ admin xoá được, dùng khi xử lý gian lận. |

### 2.7 Media — `MEDIA`

| Mã | Yêu cầu |
|---|---|
| **FR-MEDIA-01** | Client upload trực tiếp lên object storage qua **presigned URL** do backend cấp. File **không** đi qua backend. |
| **FR-MEDIA-02** | Chấp nhận ảnh `image/jpeg`, `image/png`, `image/webp` tối đa 10 MB; video `video/mp4` tối đa 100 MB và 60 giây. |
| **FR-MEDIA-03** | Backend xác minh loại và kích thước file sau khi client báo upload xong. File không hợp lệ bị đánh dấu và xoá. |
| **FR-MEDIA-04** | Ảnh sinh sẵn hai kích thước: thumbnail (400px) và bản hiển thị (1600px cạnh dài). |
| **FR-MEDIA-05** | Media chưa gắn vào review nào sau 24 giờ sẽ bị dọn tự động. |

### 2.8 Thông báo — `NOTIF`

| Mã | Yêu cầu |
|---|---|
| **FR-NOTIF-01** | Mỗi thông báo tạo một bản ghi in-app. Push chỉ là kênh gửi thêm, không thay thế. Tắt push vẫn thấy thông báo trong app. |
| **FR-NOTIF-02** | Loại thông báo: `REVIEW_APPROVED`, `REVIEW_REJECTED`, `FEEDBACK_RESOLVED`, `NEW_PLACE_NEARBY`, `PLACE_UPDATED`, `SYSTEM_ANNOUNCEMENT`. |
| **FR-NOTIF-03** | Người dùng xem danh sách thông báo có phân trang, đánh dấu đã đọc từng cái hoặc tất cả. |
| **FR-NOTIF-04** | Người dùng bật/tắt push theo từng loại thông báo. |
| **FR-NOTIF-05** | **Không** gửi push trong khoảng 22:00–07:00 giờ Việt Nam, trừ `SYSTEM_ANNOUNCEMENT`. Thông báo bị hoãn được gửi vào 07:00 sáng hôm sau. |
| **FR-NOTIF-06** | Push token đăng ký sau khi đăng nhập, gỡ khi đăng xuất. Token bị Expo báo không hợp lệ sẽ tự xoá. |

### 2.9 Chatbot — `CHAT`

| Mã | Yêu cầu |
|---|---|
| **FR-CHAT-01** | Người dùng đã đăng nhập hỏi chatbot bằng ngôn ngữ tự nhiên để tìm gợi ý địa điểm. |
| **FR-CHAT-02** | Chatbot trả lời bằng **cùng ngôn ngữ với câu hỏi** (tiếng Việt hoặc tiếng Anh). |
| **FR-CHAT-03** | Chatbot truy vấn dữ liệu thật của FoodMap qua tool (`search_places`, `get_place_detail`, `recommend_nearby`), **không** trả lời từ kiến thức chung. Không tìm thấy thì nói rõ là không tìm thấy. |
| **FR-CHAT-04** | Câu trả lời stream về client qua SSE để người dùng thấy phản hồi ngay. |
| **FR-CHAT-05** | Địa điểm được nhắc tới trong câu trả lời phải kèm id, để client render thẻ bấm mở được. |
| **FR-CHAT-06** | Ngữ cảnh hội thoại giữ trong một phiên; người dùng xoá được lịch sử chat. |
| **FR-CHAT-07** | Giới hạn 30 tin nhắn / user / giờ. Vượt trả 429 kèm thời gian chờ. |
| **FR-CHAT-08** | Lỗi hoặc quá tải từ dịch vụ AI phải hiện thông báo thân thiện, **không** để lộ chi tiết kỹ thuật. |

### 2.10 Quản trị — `ADMIN`

| Mã | Yêu cầu |
|---|---|
| **FR-ADMIN-01** | Đăng nhập trang quản trị dùng chung tài khoản với mobile; chỉ `MODERATOR` và `ADMIN` vào được. |
| **FR-ADMIN-02** | Mọi danh sách trong trang quản trị phân trang, lọc và sắp xếp **phía server**. |
| **FR-ADMIN-03** | Hàng chờ duyệt review hiển thị đủ ngữ cảnh ngay trong danh sách: ảnh thu nhỏ, tên địa điểm, tác giả và số review đã bị từ chối trước đó. |
| **FR-ADMIN-04** | Moderator duyệt, từ chối hoặc ẩn review; xử lý feedback; CRUD địa điểm và danh mục. |
| **FR-ADMIN-05** | CRUD địa điểm có công cụ chọn toạ độ trên bản đồ, không bắt nhập tay lat/lng. |
| **FR-ADMIN-06** | Mọi hành động không hoàn tác được (xoá địa điểm, từ chối review, xoá user) phải có bước xác nhận riêng. |
| **FR-ADMIN-07** | Chỉ `ADMIN` quản lý user và gán vai trò. |
| **FR-ADMIN-08** | Trang thống kê: số địa điểm theo trạng thái, review theo trạng thái, feedback đang mở, người dùng mới theo ngày, địa điểm được xem nhiều nhất. |
| **FR-ADMIN-09** | Mọi hành động kiểm duyệt ghi audit log: ai, làm gì, lên đối tượng nào, lúc nào, lý do gì. Audit log **không sửa, không xoá được**. |

### 2.11 Đa ngôn ngữ — `I18N`

| Mã | Yêu cầu |
|---|---|
| **FR-I18N-01** | Toàn bộ chuỗi giao diện của mobile và admin có đủ bản `vi` và `en`. |
| **FR-I18N-02** | Ngôn ngữ mặc định là `vi`. Mobile lấy theo ngôn ngữ hệ điều hành ở lần mở đầu tiên; người dùng đổi được và lựa chọn được lưu lại. |
| **FR-I18N-03** | Nội dung động (tên và mô tả địa điểm, tên danh mục) lưu ở bảng dịch riêng. Thiếu bản `en` → **fallback về `vi`**. Không bao giờ trả chuỗi rỗng hay `null`. |
| **FR-I18N-04** | Review **không** dịch. Giữ nguyên ngôn ngữ tác giả viết, có cột `locale` để gắn nhãn. |
| **FR-I18N-05** | Thông báo lỗi từ API dịch theo header `Accept-Language`. Mã lỗi (`ApiError.code`) **không** dịch. |
| **FR-I18N-06** | Email hệ thống gửi theo ngôn ngữ ưa thích của người nhận. |

---

## 3. Yêu cầu phi chức năng

### Hiệu năng

| Mã | Yêu cầu |
|---|---|
| **NFR-01** | API tìm quanh đây trả về trong **500ms ở phân vị 95** với 10.000 địa điểm trong CSDL và bán kính 2km. |
| **NFR-02** | API chi tiết địa điểm trả về trong **300ms ở phân vị 95**. |
| **NFR-03** | Thời gian từ mở app tới khi bản đồ hiển thị kết quả quanh đây dưới **3 giây** trên mạng 4G và thiết bị tầm trung. |
| **NFR-04** | Token đầu tiên của chatbot xuất hiện trong **2 giây** kể từ khi gửi câu hỏi. |
| **NFR-05** | Hệ thống phục vụ được **500 người dùng đồng thời** mà không vượt các ngưỡng trên. |

### Độ tin cậy và khả năng vận hành

| Mã | Yêu cầu |
|---|---|
| **NFR-06** | Uptime API mục tiêu **99,5%** theo tháng (không tính bảo trì có báo trước). |
| **NFR-07** | Mọi màn hình mobile có đủ ba trạng thái: đang tải, lỗi (có nút thử lại), và rỗng (có hướng dẫn). Không được để màn hình trắng. |
| **NFR-08** | Ảnh và dữ liệu địa điểm đã xem được cache; xem lại chi tiết địa điểm khi mất mạng vẫn hiển thị bản cache kèm nhãn "dữ liệu ngoại tuyến". |
| **NFR-09** | Sao lưu CSDL hằng ngày, giữ 30 ngày. Quy trình khôi phục được kiểm thử ít nhất mỗi quý. |
| **NFR-10** | Log có `traceId` xuyên suốt một request và được trả về trong `ApiError` để hỗ trợ điều tra. |

### Bảo mật

| Mã | Yêu cầu |
|---|---|
| **NFR-11** | Mọi giao tiếp qua HTTPS. Không có endpoint HTTP thuần ở môi trường ngoài local. |
| **NFR-12** | Mật khẩu băm BCrypt cost 12. Không log token, mật khẩu, hay body của endpoint xác thực. |
| **NFR-13** | Phân quyền kiểm tra ở **backend** trên mọi endpoint. Việc ẩn nút ở giao diện không được coi là biện pháp bảo vệ. |
| **NFR-14** | Mọi tham số đầu vào được validate. Truy vấn dùng tham số ràng buộc, không nối chuỗi SQL. |
| **NFR-15** | Bí mật chỉ nạp qua biến môi trường. Không có secret nào trong mã nguồn hay file cấu hình được commit. |
| **NFR-16** | Giới hạn tốc độ ở các endpoint nhạy cảm: đăng nhập, đặt lại mật khẩu, chat, upload media. |

### Khả năng bảo trì

| Mã | Yêu cầu |
|---|---|
| **NFR-17** | `openapi.yaml` là nguồn sự thật của hợp đồng API. Client TypeScript sinh tự động, không viết tay. |
| **NFR-18** | Thay đổi lược đồ CSDL chỉ qua migration Flyway. Migration đã merge không được sửa. |
| **NFR-19** | Độ phủ test của backend tối thiểu **70%** trên tầng service. Mỗi endpoint có ít nhất một test happy path, một test validation và một test phân quyền. |
| **NFR-20** | Integration test dùng Testcontainers với PostGIS thật, không dùng H2. |

### Khả năng tiếp cận và tương thích

| Mã | Yêu cầu |
|---|---|
| **NFR-21** | Hỗ trợ iOS 15+ và Android 8+ (API 26+). |
| **NFR-22** | Giao diện dùng được ở cỡ chữ hệ thống lớn nhất mà không cắt mất nội dung. |
| **NFR-23** | Độ tương phản màu đạt WCAG AA (tối thiểu 4.5:1 cho chữ thường). |
| **NFR-24** | Trang quản trị chạy được trên Chrome, Edge, Firefox và Safari bản mới nhất. |

---

## 4. Ràng buộc và giả định

**Ràng buộc**
- Dữ liệu và ngôn ngữ chỉ phục vụ thị trường Việt Nam ở v1.
- Bản đồ dùng Google Maps; chi phí Places API giới hạn việc seed dữ liệu tự động.
- Không có web app cho người dùng cuối ở v1.

**Giả định**
- Người dùng cho phép truy cập vị trí; nếu từ chối thì app vẫn dùng được ở chế độ tìm theo từ khoá.
- Dữ liệu địa điểm ban đầu do đội nhập tay và seed một phần từ Google Places, sau đó
  cộng đồng bổ sung.
- Có ít nhất một moderator xử lý hàng chờ mỗi ngày.

## 5. Ngoài phạm vi v1

Đặt bàn, đặt món, thanh toán · tài khoản cho chủ quán · theo dõi bạn bè và bảng tin
mạng xã hội · gợi ý cá nhân hoá bằng học máy · web app cho người dùng cuối ·
chế độ ngoại tuyến hoàn toàn · vùng ngoài Việt Nam · quảng cáo trả phí.
