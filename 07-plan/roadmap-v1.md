# Lộ trình v1

Chia theo **phase phụ thuộc nhau**, không theo tuần cố định — ước lượng thời gian ở giai đoạn
này sẽ sai, còn thứ tự phụ thuộc thì đúng.

Mỗi phase có tiêu chí "xong" kiểm chứng được. Chưa đạt thì không sang phase sau.

---

## Phase 0 — Nền tảng ✅ Đã xong

Dựng workspace và bộ khung để mọi việc sau đó có chỗ đứng.

- [x] Repo cha + 4 submodule
- [x] Hạ tầng dev bằng Docker (PostGIS, Redis, MinIO, Mailpit)
- [x] Script bootstrap / dev-up / gen-api-client
- [x] Hướng dẫn cho AI: `AGENTS.md`, skills, subagents, slash commands
- [x] Tài liệu: SRS, ERD, ADR, `openapi.yaml` v1
- [x] Skeleton chạy được cho backend, mobile, admin
- [x] CI cơ bản cho từng repo con + kiểm tra hợp đồng API ở repo cha

**Xong khi:** clone repo cha vào máy sạch, chạy `bootstrap` rồi `dev-up`, cả ba phần
khởi động được và mobile gọi được một endpoint thật của backend.

### Đã kiểm chứng

Chạy trên bản clone hoàn toàn mới, không dùng lại gì từ máy phát triển:

| Hạng mục | Kết quả |
|---|---|
| `bootstrap` từ clone sạch | exit 0, 4 submodule đầy đủ, 3 file `.env` được tạo |
| Hạ tầng Docker | 4/4 service `healthy` |
| PostGIS | 3.4, extension đã bật |
| Backend `./gradlew test` | 5/5 pass — Flyway chạy sạch từ CSDL rỗng, index GiST được dùng |
| `GET /places/nearby` | Trả đúng 4 địa điểm, sắp theo khoảng cách, ẩn bản ghi `DRAFT` |
| Đa ngôn ngữ ở API | `Accept-Language: en` đổi `message`, giữ nguyên `code` |
| 401 vs 403 | Chưa đăng nhập → 401; sai quyền → 403, thông báo đúng ngôn ngữ |
| Mobile | typecheck + lint sạch, `expo export` bundle 3.8MB thành công |
| Admin | typecheck + lint + build sạch, trang chủ hiển thị dữ liệu thật từ backend |
| Sinh lại TS client | Không sinh ra khác biệt so với bản đã commit |

### Ba lỗi thật đã phát hiện và sửa nhờ bước kiểm chứng này

1. `npm install` trên cây sạch cài **thiếu** — gói có mặt nhưng mất toàn bộ file `.d.ts`,
   khiến `tsc` báo "Cannot find module" cho gói đang nằm ngay trong `node_modules`.
   `bootstrap` đã đổi sang `npm ci`.
2. Script `.ps1` chết vì hai lý do riêng biệt: PowerShell nuốt dấu nháy khi truyền cho
   lệnh native, và `$ErrorActionPreference = 'Stop'` biến cảnh báo trên stderr của `npm`
   thành lỗi dừng.
3. `.gitignore` của `create-next-app` dùng mẫu `.env*` nên nuốt luôn cả `.env.example` —
   người mới clone về không biết cần biến môi trường nào.

---

## Phase 1 — Xác thực và nền tảng dữ liệu

Không có phase này thì mọi tính năng cần đăng nhập đều bị chặn.

**Backend**
- Lược đồ CSDL đầy đủ qua Flyway (users, places, categories, translations, opening_hours)
- Đăng ký, đăng nhập, refresh token dùng một lần, đăng xuất
- Xác minh email, quên/đặt lại mật khẩu (email vào Mailpit ở dev)
- Phân quyền theo vai trò, rate limit ở endpoint nhạy cảm
- Dữ liệu seed: danh mục món ăn + khoảng 50 địa điểm mẫu ở TP.HCM và Hà Nội

**Mobile:** màn hình đăng nhập, đăng ký, quên mật khẩu; lưu token bằng `expo-secure-store`

**Admin:** đăng nhập, layout dashboard, chặn theo vai trò

**Xong khi:** đăng ký được tài khoản, nhận email xác minh trong Mailpit, đăng nhập từ cả
mobile lẫn admin, và token hết hạn thì tự làm mới.

---

## Phase 2 — Bản đồ và địa điểm

Trái tim của sản phẩm.

**Backend**
- Tìm quanh đây bằng PostGIS (`ST_DWithin` + index GiST), có lọc
- Tìm theo từ khoá không dấu
- Chi tiết địa điểm kèm giờ mở cửa và trạng thái mở cửa lúc này
- Trả nội dung đã dịch, fallback `en` → `vi`

**Mobile**
- Tab Bản đồ: Google Maps, marker có clustering, danh sách rút gọn
- Thanh lọc: loại, danh mục, điểm tối thiểu, đang mở cửa
- Màn chi tiết địa điểm
- Xử lý quyền vị trí đầy đủ, gồm cả khi bị từ chối

**Admin:** CRUD địa điểm với công cụ chọn toạ độ trên bản đồ, form đa ngôn ngữ, giờ mở cửa

**Xong khi:** người dùng chưa đăng nhập mở app thấy quán quanh mình trong dưới 3 giây,
lọc được, và mở được chi tiết. Đạt NFR-01 với 10.000 địa điểm.

---

## Phase 3 — Nội dung do người dùng tạo

**Backend**
- Upload media qua presigned URL, xác minh file, sinh thumbnail
- Gửi và cập nhật review; vòng đời kiểm duyệt; tính lại điểm trung bình
- Feedback với luật chống trùng và tự gắn cờ sau 3 báo cáo đóng cửa
- Yêu thích (idempotent)
- Ghi lượt đến với kiểm tra bán kính 200m và giới hạn 1 lượt/ngày

**Mobile:** viết review kèm ảnh/video, báo sai thông tin, yêu thích, check-in,
lịch sử đã đến

**Admin:** hàng chờ duyệt review có ngữ cảnh, xử lý feedback, audit log

**Xong khi:** viết được review có ảnh, moderator duyệt được, điểm trung bình cập nhật đúng,
và luật chống spam visit hoạt động (kiểm bằng AC-VISIT-02, 03, 04).

---

## Phase 4 — Đa ngôn ngữ và thông báo

**Backend:** thông báo in-app + Expo Push, hoãn qua đêm, cấu hình theo loại;
thông báo lỗi dịch theo `Accept-Language`; email theo ngôn ngữ người nhận

**Mobile:** i18next đầy đủ cho mọi màn hình, đổi ngôn ngữ trong Cài đặt,
màn hình thông báo, đăng ký/gỡ push token

**Admin:** next-intl đầy đủ

**Xong khi:** đổi sang tiếng Anh thì **không còn chuỗi tiếng Việt nào** trên giao diện,
nội dung động fallback đúng, và push tới thiết bị thật.

---

## Phase 5 — Chatbot AI

**Backend**
- Tích hợp Anthropic Java SDK, model `claude-opus-5`
- Đăng ký tool: `search_places`, `get_place_detail`, `recommend_nearby`
- Stream SSE, lưu lịch sử phiên, rate limit 30 tin/giờ

**Mobile:** giao diện chat, hiển thị stream, thẻ địa điểm bấm mở được

**Xong khi:** hỏi "quanh đây có quán bún bò nào ngon không" thì nhận được gợi ý từ
**dữ liệu thật** của FoodMap, có thẻ bấm mở được, và hỏi về thứ không có trong CSDL thì
chatbot nói thẳng là không tìm thấy (AC-CHAT-01).

---

## Phase 6 — Hoàn thiện và phát hành

- Thống kê cho trang quản trị
- Tối ưu hiệu năng, đo và đối chiếu các NFR
- Rà soát bảo mật: phân quyền ở mọi endpoint, không secret nào bị commit
- Trạng thái rỗng / lỗi / ngoại tuyến cho **mọi** màn hình
- Kiểm tra khả năng tiếp cận: cỡ chữ lớn, độ tương phản
- Chuẩn bị store: ảnh chụp màn hình, mô tả, chính sách quyền riêng tư
- Nhập dữ liệu thật: mục tiêu 5.000 địa điểm
- Diễn tập khôi phục từ bản sao lưu

**Xong khi:** toàn bộ tiêu chí trong
[`acceptance-criteria.md`](../01-srs/acceptance-criteria.md) pass, và bản build production
chạy trên thiết bị thật của cả hai nền tảng.

---

## Đường tới hạn

```
Phase 0 → Phase 1 → Phase 2 → Phase 3 → Phase 6
                        ↘ Phase 4 ↗
                        ↘ Phase 5 ↗
```

Phase 4 và 5 làm song song được sau khi Phase 2 xong. Phase 3 phụ thuộc Phase 2
(review cần có địa điểm). Phase 6 cần tất cả.

## Rủi ro

| Rủi ro | Ảnh hưởng | Giảm nhẹ |
|---|---|---|
| Không đủ dữ liệu địa điểm lúc ra mắt | App rỗng thì không ai dùng | Bắt đầu nhập dữ liệu **song song với Phase 2**, không đợi tới Phase 6 |
| Chi phí Google Maps vượt dự tính | Phải đổi nhà cung cấp giữa chừng | Theo dõi chi phí từ Phase 2; giữ lớp bọc bản đồ đủ mỏng để đổi được |
| Duyệt app store bị từ chối | Chậm ra mắt | Đọc kỹ hướng dẫn của Apple từ Phase 4, nhất là phần quyền riêng tư và quyền vị trí |
| Không đủ moderator xử lý hàng chờ | Dữ liệu bẩn tích tụ | Đo tốc độ hàng chờ ngay từ Phase 3; cân nhắc tự động duyệt review của người dùng có uy tín |
| Quên chạy `gen-api-client` sau khi sửa spec | Client dùng type cũ, lỗi runtime | Đưa vào CI như cổng bắt buộc nếu xảy ra vài lần |
