# Danh sách màn hình v1

Ánh xạ Function ID trong Feature Breakdown. Luồng: [`luong-nguoi-dung.md`](./luong-nguoi-dung.md).

Mọi màn: skeleton khi tải, lỗi + Thử lại, trạng thái rỗng có hướng dẫn, cache + nhãn ngoại tuyến nếu đã xem.

## Mobile (expo-router)

### `app/(auth)/`

| Màn | Route | Function ID |
|---|---|---|
| Đăng nhập | `login` | AUTH-02, AUTH-03 (nút Google nếu bật) |
| Đăng ký | `register` | AUTH-01 |
| Quên mật khẩu | `forgot-password` | AUTH-05 |
| Đặt lại mật khẩu | `reset-password` | AUTH-06 |

### `app/(tabs)/` — tab mặc định là Bản đồ

| Màn | Route | Function ID |
|---|---|---|
| Bản đồ | `index` hoặc `map` | MAP-01…06, DISCOVERY-01 |
| Khám phá | `explore` | MAP-05, DISCOVERY-02…06 |
| Yêu thích | `favorites` | PLACE-04 |
| Cá nhân | `profile` | AUTH-04, LANG-01, vào thông báo |

Khách vào được Bản đồ và Khám phá. Tab Yêu thích trống + CTA đăng nhập.

### Stack

| Màn | Route | Function ID |
|---|---|---|
| Chi tiết địa điểm | `place/[id]` | PLACE-01…03, REVIEW-01, MAP-07 (nút mở app bản đồ — Could) |
| Viết / sửa đánh giá | `place/[id]/review` | REVIEW-02…06 |
| Chatbot | `chat` | AI-01…06 |
| Thông báo | `notifications` | NOTI-01…03 |
| Cài đặt | `settings` | LANG-01, AUTH-04 |

Không làm màn “báo cáo review” (REVIEW-08 Won't) và không làm màn cấu hình loại thông báo (NOTI-05 Won't).

## Admin (App Router)

Chỉ `MODERATOR` / `ADMIN` sau `/login`. `USER` thấy trang không đủ quyền.

| Màn | Route | Function ID | Role |
|---|---|---|---|
| Đăng nhập | `/login` | AUTH-02, AUTH-07 | — |
| Dashboard | `/` | ADMIN-01 | MODERATOR, ADMIN |
| Địa điểm | `/places`, `/places/new`, `/places/[id]` | ADMIN-03 | MODERATOR, ADMIN |
| Danh mục | `/categories` | ADMIN-05 | MODERATOR, ADMIN |
| Duyệt review | `/reviews` | ADMIN-04 | MODERATOR, ADMIN |
| Người dùng | `/users` | ADMIN-02 | ADMIN |

Không có `/reports` (ADMIN-06 Won't).
