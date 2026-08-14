# ADR-0001: Chọn công nghệ nền tảng cho v1

| | |
|---|---|
| Trạng thái | Chấp nhận |
| Ngày | 2026-08-14 |

## Bối cảnh

FoodMap cần một ứng dụng chạy được trên cả iOS và Android, một API backend, một trang
quản trị, và một cơ sở dữ liệu hỗ trợ truy vấn địa lý. Dự án bắt đầu từ con số không,
đội nhỏ, và cần ra được v1 dùng thật.

Ràng buộc thực tế:
- Đội đã quen **Java**; học một ngôn ngữ backend mới sẽ làm chậm v1 đáng kể.
- Chỉ có một đội, không đủ người để duy trì hai codebase mobile riêng biệt.
- Tính năng cốt lõi là "tìm quán quanh đây" — truy vấn địa lý phải nhanh và đúng.
- Dữ liệu POI quán ăn ở Việt Nam mỏng trên OpenStreetMap.

## Quyết định

| Lớp | Chọn |
|---|---|
| Mobile | React Native + Expo, TypeScript |
| Backend | Java 21 + Spring Boot 3, Gradle Kotlin DSL |
| CSDL | PostgreSQL 16 + PostGIS |
| Cache / session | Redis 7 |
| Lưu media | MinIO (dev) → S3 (prod) |
| Trang quản trị | Next.js 15 App Router, TypeScript, shadcn/ui |
| Bản đồ | Google Maps Platform |
| Chatbot | Claude API (`claude-opus-5`) qua Anthropic Java SDK |
| Migration | Flyway |

## Phương án đã cân nhắc

**Mobile: Flutter.** UI mượt và hiệu năng tốt hơn ở một số kịch bản. Loại vì phải học
Dart từ đầu, không dùng chung được gì với trang quản trị, và hệ sinh thái thư viện
bản đồ / media / social feed cho React Native trưởng thành hơn ở đúng những thứ v1 cần.

**Mobile: Kotlin Multiplatform.** Gần native nhất và tận dụng được kiến thức JVM sẵn có.
Loại vì phần chia sẻ UI còn non, setup nặng, và rủi ro tiến độ cho v1 quá cao.

**Mobile: hai app native riêng.** Chất lượng tốt nhất nhưng nhân đôi khối lượng công việc.
Không khả thi với quy mô đội hiện tại.

**Backend: Go.** Hiệu năng cao, binary nhẹ, deploy rẻ. Loại vì đội chưa quen, và lợi thế
hiệu năng chưa thành vấn đề ở quy mô v1 (500 người dùng đồng thời).

**Backend: Python + FastAPI.** Thuận cho phần AI. Loại vì tầng ORM và hỗ trợ dữ liệu
địa lý yếu hơn Hibernate Spatial, và việc gọi Claude API từ Java cũng đủ đơn giản —
không đáng đổi cả backend chỉ vì một module.

**Backend: NestJS.** Cho phép share type trực tiếp với mobile và admin. Loại vì đội quen
Java hơn, và vấn đề share type đã được giải bằng OpenAPI (xem [ADR-0002](./ADR-0002-contract-first-openapi.md)).

**Bản đồ: Mapbox.** Tuỳ biến giao diện đẹp hơn và rẻ hơn khi scale. Loại vì dữ liệu POI
quán ăn ở Việt Nam mỏng hơn Google, và Places API của Google là nguồn seed dữ liệu ban đầu.

**Bản đồ: MapLibre + OpenStreetMap.** Miễn phí hoàn toàn. Loại vì gần như phải tự xây
toàn bộ dataset quán ăn — quá tốn cho v1.

## Hệ quả

### Tích cực
- Một codebase mobile cho hai nền tảng; Expo lo phần build và cập nhật OTA.
- Đội viết backend bằng ngôn ngữ đã quen → ít rủi ro tiến độ.
- Mobile và admin cùng TypeScript → dùng chung được TS client sinh tự động, chung quy ước.
- PostGIS xử lý truy vấn địa lý ở tầng CSDL, có index — đúng chỗ nhất
  (xem [ADR-0003](./ADR-0003-postgis-cho-truy-van-dia-ly.md)).
- MinIO cùng API với S3 → môi trường dev giống prod, đổi chỉ cần đổi endpoint.

### Tiêu cực
- **Java và TypeScript không share type trực tiếp.** Phải có cơ chế đồng bộ hợp đồng —
  giải bằng OpenAPI, nhưng đó là một bước thủ công phải nhớ chạy.
- Spring Boot khởi động chậm hơn Go hay Node, và tốn RAM hơn → chi phí hạ tầng cao hơn.
- Expo có giới hạn: thư viện native chưa hỗ trợ thì phải dùng development build,
  mất một phần tiện lợi.
- Google Maps tính phí theo lượt gọi; chi phí tăng theo lượng người dùng và giới hạn
  việc seed dữ liệu tự động.
- Ba ngôn ngữ trong một dự án (Java, TypeScript, SQL) → chi phí ngữ cảnh khi chuyển qua lại.

### Cần theo dõi
- Chi phí Google Maps mỗi tháng. Vượt ngưỡng chịu được thì xem lại Mapbox — lớp
  `MapProvider` nên được giữ đủ mỏng để đổi được.
- Thời gian khởi động và mức RAM của backend khi số module tăng.
- Tần suất quên chạy `gen-api-client` sau khi sửa spec. Xảy ra thường xuyên thì phải
  đưa vào CI như một bước bắt buộc.
