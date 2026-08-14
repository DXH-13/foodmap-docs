# Triển khai

> **Trạng thái:** v1 chưa chốt nền tảng triển khai production. Tài liệu này ghi lại
> những gì đã quyết và những gì còn để mở. Chốt xong thì viết ADR và cập nhật file này.

---

## Backend

### Đóng gói

Docker image nhiều tầng, chạy bằng JRE 21 (không phải JDK), user không phải root.

```dockerfile
FROM eclipse-temurin:21-jdk-alpine AS build
WORKDIR /app
COPY . .
RUN ./gradlew bootJar --no-daemon

FROM eclipse-temurin:21-jre-alpine
RUN addgroup -S app && adduser -S app -G app
USER app
COPY --from=build /app/build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "/app/app.jar"]
```

### Migration khi triển khai

Flyway chạy **tự động lúc khởi động ứng dụng**. Hệ quả cần biết:

- Nhiều instance khởi động cùng lúc: Flyway dùng khoá cấp CSDL, chỉ một instance chạy
  migration, các instance còn lại chờ. An toàn nhưng làm khởi động chậm hơn.
- Migration lỗi = ứng dụng không lên được. Đây là **chủ ý** — chạy với schema sai còn tệ hơn.
- **Migration phải tương thích ngược một phiên bản.** Trong lúc rolling update, code cũ
  và code mới cùng chạy trên một schema. Xoá cột phải làm hai bước qua hai lần phát hành:
  lần một ngừng dùng cột, lần hai mới xoá.

### Healthcheck

| Endpoint | Dùng cho |
|---|---|
| `/actuator/health/liveness` | Container còn sống không |
| `/actuator/health/readiness` | Sẵn sàng nhận traffic chưa (đã kết nối được DB và Redis) |

Readiness phải kiểm tra được cả PostgreSQL và Redis.

### Cấu hình

Toàn bộ qua biến môi trường (xem [`environments.md`](./environments.md)).
`SPRING_PROFILES_ACTIVE=prod`. Secret lấy từ secret manager của nền tảng, **không** từ file `.env`.

---

## Mobile

### Build

EAS Build của Expo. Ba profile:

| Profile | Dùng cho | Phân phối |
|---|---|---|
| `development` | Có dev client, debug được | Nội bộ |
| `preview` | Bản build thật cho tester | Nội bộ / TestFlight |
| `production` | Bản lên store | App Store / Play Store |

```bash
eas build --profile production --platform all
eas submit --platform all
```

### Cập nhật OTA

EAS Update cho phép đẩy sửa lỗi JavaScript mà không cần duyệt lại ở store.

**Chỉ dùng OTA cho thay đổi thuần JavaScript.** Thay đổi native (thêm thư viện native,
đổi quyền, đổi cấu hình app) **bắt buộc** build và submit lại. Đẩy OTA một thay đổi
cần native sẽ làm app crash trên máy người dùng.

### Khoá API ở client

`EXPO_PUBLIC_GOOGLE_MAPS_API_KEY` nằm trong bundle và ai cũng đọc được.
Bắt buộc giới hạn khoá này trong Google Cloud Console theo:
- Bundle ID (iOS) và package name + SHA-1 (Android)
- Chỉ những API thật sự cần

Khoá không giới hạn bị lấy dùng sẽ khiến bạn trả tiền cho lưu lượng của người khác.

---

## Trang quản trị

Next.js standalone output, đóng gói Docker hoặc deploy lên nền tảng hỗ trợ Next.js.

**Trang quản trị không nên mở công khai ra internet.** Ít nhất phải có một lớp
hạn chế truy cập (VPN, IP allowlist, hoặc reverse proxy có xác thực) bên cạnh
đăng nhập của ứng dụng.

---

## Thứ tự triển khai

1. **Backend trước.** API mới phải sẵn sàng trước khi client dùng tới.
2. **Trang quản trị.** Cập nhật tức thì, không cần duyệt.
3. **Mobile sau cùng.** Qua store nên chậm nhất, và **luôn có người dùng ở bản cũ** —
   đây là lý do API phải giữ tương thích ngược (xem [CHANGELOG](../03-api/CHANGELOG.md)).

---

## Còn để mở

| Câu hỏi | Ghi chú |
|---|---|
| Nền tảng chạy backend | Cân nhắc: VPS + Docker Compose (đơn giản, rẻ) · Kubernetes (linh hoạt, phức tạp) · PaaS như Railway/Render (nhanh nhất để bắt đầu) |
| Nhà cung cấp CSDL | Postgres tự quản hay dịch vụ managed. Managed đắt hơn nhưng có sẵn sao lưu và failover |
| CDN cho media | Cần khi lượng ảnh tăng; chưa gấp ở v1 |
| Giám sát và cảnh báo | Cần ít nhất: uptime, tỷ lệ lỗi, độ trễ p95, dung lượng đĩa |
| Pipeline CI/CD | Mức tối thiểu ở v1: chạy test trên mọi PR |

Chốt từng mục thì viết ADR trong [`02-architecture/adr/`](../02-architecture/adr/).
