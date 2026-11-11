# Nhật ký thay đổi hợp đồng API

Ghi rõ mọi thay đổi phá vỡ (breaking change). Xem skill `api-contract`
ở repo cha: xoá field, đổi kiểu field, thêm field bắt buộc vào request,
xoá giá trị enum, đổi mã lỗi, đổi path đều là phá vỡ.

## 2026-11-11 — Sửa spec: bỏ `bearerAuth` thừa ở login/refresh, bổ sung ràng buộc đã mất

- **Bỏ `security: bearerAuth` ở `POST /app/v1/auth/login` và `POST /app/v1/auth/refresh`.**
  Hai operation này phải gọi được khi chưa có (hoặc đã hết hạn) access token;
  lúc khôi phục spec ở bước 1, `bearerAuth` đã được gán máy móc cho mọi
  operation có trả 401 — sai với 2 operation này. Runtime (`SecurityConfig`)
  vốn đã để cả hai là public, giờ spec khớp lại với runtime. Các operation
  khác giữ nguyên `bearerAuth` (logout, resend-verification, `/me`, …).
- **Bổ sung ràng buộc đã mất trong lúc khôi phục** (đối chiếu từng cái với
  code backend, không đoán mò):
  - `GET /app/v1/places/nearby`: `latitude` minimum −90 / maximum 90,
    `longitude` minimum −180 / maximum 180, `radiusMeters` thành integer,
    default 2000, minimum 100, maximum 50000 (theo `PlaceController` +
    `PlaceGeoProperties` / `application.yml`).
  - `latitude`/`longitude` của `GET /app/v1/me/favorites`: cùng giới hạn
    −90..90 / −180..180.
  - `PageParam`/`SizeParam` (dùng chung cho mọi endpoint phân trang):
    thành integer; `page` default 0, minimum 0; `size` default 20,
    minimum 1, maximum 100 (theo `PlaceController`/`AdminUserController`).
  - `RegisterRequest.password` và `ResetPasswordRequest.newPassword`:
    minLength 8, maxLength 128, pattern `^(?=.*[A-Za-z])(?=.*\d).{8,}$`
    (theo `AuthService.PASSWORD_PATTERN` + `@Size`); `displayName`
    maxLength 100.
- Không thêm/bớt endpoint, không đổi operationId hay field bắt buộc.
  Kiểm chứng: chạy `openapi-typescript` 7.13.0 trên spec mới, diff với
  `schema.ts` đã commit của mobile và admin đều **bằng 0** — client sinh
  ra không đổi, không cần sinh lại.

## 2026-11-11 — Bước 1: đổi tiền tố hợp đồng (BREAKING CHANGE)

Kiến trúc backend thống nhất, bước 1: tách hợp đồng thành hai nhóm
`/app/v1` (mobile) và `/admin/v1` (admin), thay cho tiền tố chung `/api/v1`.

- `/api/v1/*` → `/app/v1/*` (30 đường dẫn: auth, me, places, categories,
  reviews, feedbacks, favorites, visits, media, notifications, chat).
- `/api/v1/admin/*` → `/admin/v1/*` (9 đường dẫn đã có sẵn: reviews,
  feedbacks, places, users, stats — chỉ đổi tiền tố, không thêm endpoint mới).
- Không thêm, không bớt endpoint, không đổi operationId, schema, enum
  hay field bắt buộc nào.
- Backend đổi theo: `PlaceController` sang `/app/v1/places`,
  `SecurityConfig` sang `/app/v1/...` và `/admin/v1/**`, CORS sang
  `/app/**` và `/admin/**`.
- Client TypeScript của mobile và admin được sinh lại từ spec mới
  (`scripts/gen-api-client.sh`); các chỗ gọi đổi sang đường dẫn mới.

### Khôi phục `openapi.yaml` từ client đã sinh

File `openapi.yaml` đã bị mất khỏi repo (SDD/README.md, script sinh client
và CI đều đọc nó nhưng file không còn trong repo). Bản này được khôi phục
lại từ `mobile/src/api/generated/schema.ts` bằng trình phân tích tự động
(`workspace/work/foodmap/reconstruct/parse_schema.py`).

Kiểm chứng round-trip: chạy `openapi-typescript` 7.13.0 trên spec mới và
so với client cũ sau khi đổi máy móc `/api/v1` → `/app/v1`,
`/api/v1/admin/` → `/admin/v1/` — **diff bằng 0**. Mọi path, operation,
parameter, request/response, schema, enum và field bắt buộc đều khớp.

### Những gì client đã sinh không giữ được (ghi rõ để bổ sung sau)

Client sinh ra là ảnh chiếu mất mát của spec gốc; những thứ dưới đây
không khôi phục được từ `schema.ts` và đã được điền lại theo quy ước
của dự án (skill `api-contract`, `OpenApiConfig`, `SecurityConfig`),
hoặc còn thiếu:

- `info` (title "FoodMap API", version "1.0.0"), `servers`
  (`http://localhost:8080`), `tags` theo tên module, `securityScheme`
  `bearerAuth` (HTTP bearer, JWT — cùng tên với backend) — điền lại theo
  quy ước; `security: bearerAuth` được gán cho mọi operation có trả 401 —
  cách gán này sau đó được phát hiện là sai với `login` và `refreshToken`
  (xem mục sửa spec 2026-11-11 ở trên), vì hai operation đó phải gọi được
  khi chưa có token.
- `minimum`/`maximum` và `default` của parameter: client sinh ra không
  giữ các giá trị này. Ví dụ `PlaceController.searchNearbyPlaces` hiện có
  `@Min(-90) @Max(90)` cho latitude, `@Min(-180) @Max(180)` cho
  longitude, `page` mặc định 0, `size` mặc định 20 (tối đa 100) — spec
  khôi phục chưa có các ràng buộc này. Bổ sung ở bước sau, kèm kiểm tra
  từng operation, không đoán mò cho cả spec.
- Mô tả operation nào spec gốc có mà client không giữ thì đã mất theo;
  chỉ những mô tả còn trong `schema.ts` mới được khôi phục.
- Spec gốc được xác định là OpenAPI **3.1.0** (không phải 3.0.x) nhờ hai
  dấu vết trong client: schema có 3 dòng `@example` (mảng `examples`
  của 3.1) và `| null` trên thuộc tính `$ref` (`oneOf` với
  `{type: "null"}` của 3.1). Trường nullable dùng `type: [T, "null"]`,
  không dùng `nullable: true`.
