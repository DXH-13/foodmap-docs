# Môi trường

| Môi trường | Mục đích | Dữ liệu | Ai truy cập |
|---|---|---|---|
| **local** | Máy lập trình viên | Dữ liệu seed, xoá lại được | Lập trình viên |
| **dev** | Tích hợp, kiểm thử nội bộ | Dữ liệu giả, reset định kỳ | Cả đội |
| **prod** | Người dùng thật | Dữ liệu thật | Người dùng cuối |

Không có staging riêng ở v1. Cân nhắc thêm khi đã có người dùng thật và cần diễn tập
phát hành trước khi lên prod.

---

## local

Toàn bộ hạ tầng chạy bằng Docker qua `infra/docker-compose.yml` ở repo cha.

```bash
./scripts/bootstrap.sh      # lần đầu
./scripts/dev-up.sh         # bật hạ tầng
```

| Dịch vụ | Cổng | Ghi chú |
|---|---|---|
| PostgreSQL + PostGIS | 5433 | db/user/pass đều là `foodmap` |
| Redis | 6380 | |
| MinIO API | 9002 | |
| MinIO Console | 9003 | `minioadmin` / `minioadmin` |
| Mailpit SMTP | 1025 | |
| Mailpit UI | 8025 | Xem email hệ thống gửi |
| Backend | 8080 | |
| Trang quản trị | 3000 | |

**Cổng lệch chuẩn là cố ý.** Máy lập trình viên thường đã có Postgres/Redis/MinIO của
dự án khác chiếm 5432/6379/9000. Tất cả đều là biến trong `infra/.env`, đổi được.

**Email không thoát ra ngoài.** Mọi email đều rơi vào Mailpit — mở http://localhost:8025
để xem link xác minh và đặt lại mật khẩu.

### Kết nối tới CSDL local

```bash
docker exec -it foodmap-db psql -U foodmap -d foodmap
```

### Mobile trên thiết bị thật

`EXPO_PUBLIC_API_BASE_URL` phải là **IP LAN** của máy chạy backend, không phải `localhost` —
thiết bị thật và máy ảo Android không hiểu `localhost` của máy bạn.

```
EXPO_PUBLIC_API_BASE_URL=http://192.168.1.10:8080
```

Tìm IP: `ipconfig` (Windows) hoặc `ifconfig | grep inet` (macOS).

---

## Biến môi trường

Mẫu đầy đủ: `infra/.env.example` ở repo cha. Thêm biến mới **bắt buộc** cập nhật file mẫu
với giá trị giả.

### Bắt buộc điền, không có mặc định dùng được

| Biến | Lấy ở đâu |
|---|---|
| `GOOGLE_MAPS_API_KEY` | Google Cloud Console. Cần bật: Maps SDK for Android, Maps SDK for iOS, Geocoding API, Places API |
| `ANTHROPIC_API_KEY` | https://console.anthropic.com |
| `JWT_SECRET` | Tự sinh, tối thiểu 32 ký tự: `openssl rand -base64 48` |

### Có mặc định, đổi ở prod

`POSTGRES_*` · `REDIS_PORT` · `S3_*` · `MAIL_*` · `SERVER_PORT` · `SPRING_PROFILES_ACTIVE`

### Riêng cho client

| Biến | Dùng ở |
|---|---|
| `EXPO_PUBLIC_API_BASE_URL` | mobile |
| `EXPO_PUBLIC_GOOGLE_MAPS_API_KEY` | mobile |
| `NEXT_PUBLIC_API_BASE_URL` | admin |

⚠️ Biến có tiền tố `EXPO_PUBLIC_` và `NEXT_PUBLIC_` được **nhúng vào bundle client** và
ai cũng đọc được. **Không bao giờ** đặt secret vào đó. Khoá Google Maps ở client phải
được giới hạn theo bundle id / package name và theo API trong Google Cloud Console.

---

## Nguyên tắc quản lý secret

- Secret **chỉ** nạp qua biến môi trường. Không nằm trong mã nguồn, không trong file cấu hình được commit.
- `.env` nằm trong `.gitignore`. Chỉ commit `.env.example`.
- Prod dùng secret manager của nền tảng triển khai, không dùng file `.env`.
- Mỗi môi trường có secret **riêng**. Không dùng chung `JWT_SECRET` giữa dev và prod —
  token dev sẽ hợp lệ ở prod.
- Lộ secret thì xoay vòng ngay, không chờ tới đợt phát hành.

---

## Khác biệt giữa các môi trường

| | local | dev | prod |
|---|---|---|---|
| Lưu media | MinIO | MinIO | S3 |
| Email | Mailpit (không gửi ra ngoài) | Mailpit | SMTP thật |
| TTL access token | 15 phút | 15 phút | 15 phút |
| Mức log | `DEBUG` | `DEBUG` | `INFO` |
| Bật CORS cho localhost | Có | Có | **Không** |
| `ddl-auto` | `validate` | `validate` | `validate` |
| Sao lưu CSDL | Không | Hằng ngày, giữ 7 ngày | Hằng ngày, giữ 30 ngày |
| Rate limit | Nới lỏng | Như prod | Đầy đủ |

**`ddl-auto` luôn là `validate` ở mọi môi trường.** Lược đồ do Flyway quản lý;
để Hibernate tự sửa schema là con đường ngắn nhất tới mất dữ liệu.
