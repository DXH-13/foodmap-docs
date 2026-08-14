# Danh sách màn hình

Màn hình cần có ở v1, kèm route và yêu cầu liên quan.
Luồng điều hướng: [`user-flows.md`](./user-flows.md).

---

## Ứng dụng mobile

Route theo quy ước [expo-router](https://docs.expo.dev/router/introduction/): file = route.

### Nhóm chưa đăng nhập — `app/(auth)/`

| Màn hình | Route | Nội dung chính | Yêu cầu |
|---|---|---|---|
| Đăng nhập | `(auth)/login` | Email, mật khẩu, link quên mật khẩu, link đăng ký | FR-AUTH-03 |
| Đăng ký | `(auth)/register` | Email, mật khẩu, tên hiển thị | FR-AUTH-01 |
| Quên mật khẩu | `(auth)/forgot-password` | Nhập email; **thông báo giống nhau** dù email có tồn tại hay không | FR-AUTH-05 |
| Đặt lại mật khẩu | `(auth)/reset-password` | Mở từ link trong email, nhập mật khẩu mới hai lần | FR-AUTH-05 |

### Nhóm tab chính — `app/(tabs)/`

| Màn hình | Route | Nội dung chính | Yêu cầu |
|---|---|---|---|
| **Bản đồ** | `(tabs)/map` | Bản đồ Google + marker có clustering, danh sách rút gọn phía dưới, thanh lọc (loại, danh mục, điểm tối thiểu, đang mở cửa), nút về vị trí hiện tại | FR-PLACE-01→04, FR-PLACE-06 |
| **Khám phá** | `(tabs)/explore` | Tìm theo từ khoá (không dấu), duyệt theo danh mục | FR-PLACE-04 |
| **Yêu thích** | `(tabs)/favorites` | Danh sách yêu thích, sắp xếp theo thời gian thêm hoặc khoảng cách; địa điểm đã đóng cửa gắn nhãn | FR-FAVORITE-03, 04 |
| **Cá nhân** | `(tabs)/profile` | Hồ sơ, lịch sử đã đến, đánh giá của tôi, feedback của tôi, cài đặt | FR-AUTH-09, FR-VISIT-03 |

Tab Bản đồ là **màn hình mặc định** — người dùng nghĩ theo không gian, không theo danh sách.

### Màn hình chi tiết

| Màn hình | Route | Nội dung chính | Yêu cầu |
|---|---|---|---|
| Chi tiết địa điểm | `place/[id]` | Ảnh, tên, loại, danh mục, điểm trung bình, giờ mở cửa + trạng thái hiện tại, địa chỉ, số lượt đến, danh sách đánh giá (cuộn vô hạn), các nút hành động | FR-PLACE-05, 06, 12 |
| Viết đánh giá | `place/[id]/review` | Chọn sao, nội dung, thêm ảnh/video có tiến trình từng file | FR-REVIEW-01→03, FR-MEDIA-01 |
| Báo thông tin sai | `place/[id]/feedback` | Chọn loại, mô tả; loại "Trùng lặp" cho chọn địa điểm bị trùng | FR-FEEDBACK-01, 06 |
| Đề xuất địa điểm | `place/suggest` | Tên (vi bắt buộc, en tuỳ chọn), loại, chọn toạ độ trên bản đồ, danh mục | FR-PLACE-10 |
| Chatbot | `chat/` | Khung hội thoại, câu trả lời stream, thẻ địa điểm bấm mở được | FR-CHAT-01→05 |
| Thông báo | `notifications/` | Danh sách, đánh dấu đã đọc, bấm để mở đối tượng liên quan | FR-NOTIF-03 |
| Cài đặt | `settings/` | Ngôn ngữ (vi/en), cấu hình push theo loại, đăng xuất, xoá tài khoản | FR-I18N-02, FR-NOTIF-04, FR-AUTH-10 |

### Trạng thái bắt buộc của mọi màn hình

Không màn hình nào được để trắng. Mỗi màn hình phải xử lý đủ (NFR-07):

| Trạng thái | Yêu cầu |
|---|---|
| Đang tải | Skeleton, **không** dùng spinner toàn màn hình |
| Lỗi | Thông báo thân thiện + nút Thử lại. Không hiện chi tiết kỹ thuật |
| Rỗng | Giải thích vì sao rỗng + gợi ý hành động cụ thể |
| Ngoại tuyến | Hiện dữ liệu cache kèm nhãn "dữ liệu ngoại tuyến" |
| Thiếu quyền | Giải thích vì sao cần quyền + nút mở Cài đặt |

---

## Trang quản trị

Next.js App Router. Toàn bộ nằm sau kiểm tra vai trò `MODERATOR` hoặc `ADMIN`.

| Màn hình | Route | Nội dung chính | Yêu cầu |
|---|---|---|---|
| Đăng nhập | `/login` | Dùng chung tài khoản với mobile; `USER` thường bị từ chối | FR-ADMIN-01 |
| Tổng quan | `/` | Số liệu thống kê, số mục đang chờ xử lý | FR-ADMIN-08 |
| Duyệt đánh giá | `/reviews` | Hàng chờ có ngữ cảnh: ảnh thu nhỏ, tên quán, số lần tác giả bị từ chối trước đó | FR-ADMIN-03, FR-REVIEW-04, 05 |
| Xử lý feedback | `/feedbacks` | Lọc theo loại và trạng thái; xử lý kèm ghi chú | FR-FEEDBACK-04 |
| Quản lý địa điểm | `/places` | Bảng có phân trang/lọc/sắp xếp phía server | FR-ADMIN-02 |
| Thêm / sửa địa điểm | `/places/new`, `/places/[id]` | Form đa ngôn ngữ (vi bắt buộc, en tuỳ chọn), **chọn toạ độ trên bản đồ**, giờ mở cửa nhiều khoảng, ảnh | FR-ADMIN-05, FR-PLACE-07 |
| Quản lý danh mục | `/categories` | CRUD danh mục và bản dịch | FR-ADMIN-04 |
| Quản lý người dùng | `/users` | Chỉ `ADMIN`. Tìm kiếm, gán vai trò | FR-ADMIN-07 |
| Nhật ký kiểm toán | `/audit-logs` | Chỉ đọc, lọc theo người thực hiện, hành động, đối tượng | FR-ADMIN-09 |

### Quy tắc giao diện quản trị

- **Mọi bảng phân trang phía server.** Dữ liệu sẽ lớn; phân trang phía client sẽ vỡ.
- **Hành động không hoàn tác được phải có bước xác nhận riêng** (FR-ADMIN-06).
- **Từ chối đánh giá bắt buộc nhập lý do** — không phải trường tuỳ chọn (FR-REVIEW-05).
- Ẩn nút theo vai trò cho gọn giao diện, nhưng **backend mới là nơi chặn thật** (NFR-13).
- Trang chọn toạ độ phải cho kéo thả marker và tìm theo địa chỉ, không bắt gõ số.
