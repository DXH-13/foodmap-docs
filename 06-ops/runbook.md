# Runbook

Xử lý sự cố thường gặp. Mục tiêu: người trực đọc là làm được, không phải đi hỏi người khác.

---

## Môi trường dev

### Container không lên được

```bash
docker compose -f infra/docker-compose.yml ps
docker compose -f infra/docker-compose.yml logs <service> --tail 50
```

| Triệu chứng | Nguyên nhân thường gặp | Cách xử lý |
|---|---|---|
| `port is already allocated` | Dịch vụ khác đang giữ cổng | Đổi cổng trong `infra/.env` (mọi cổng đều là biến), hoặc tắt dịch vụ kia |
| `db` mãi không `healthy` | Volume hỏng do tắt máy đột ngột | `docker compose down -v` rồi `up -d` — **xoá sạch dữ liệu local** |
| `minio` không `healthy` | Hết dung lượng đĩa | Kiểm tra đĩa; `docker system prune` để dọn image cũ |

### Backend không khởi động

**Lỗi Flyway `FlywayValidateException` — checksum không khớp**

Có người sửa một file migration đã chạy. Đây là điều **không được phép** làm.

```bash
# Chỉ ở môi trường local, dữ liệu bỏ được:
cd backend && ./gradlew flywayClean flywayMigrate
```

Ở dev hoặc prod thì **không** dùng `flywayClean` — nó xoá sạch schema. Phải khôi phục
file migration về nội dung gốc và viết migration mới để sửa.

**Lỗi `relation "places" does not exist`**

Migration chưa chạy. Kiểm tra log khởi động xem Flyway có bị lỗi trước đó không.

**Lỗi kết nối CSDL**

```bash
docker exec foodmap-db pg_isready -U foodmap
```

Kiểm tra cổng trong `infra/.env` khớp với cấu hình datasource của backend (mặc định **5433**,
không phải 5432).

### Mobile không gọi được API

| Triệu chứng | Nguyên nhân | Cách xử lý |
|---|---|---|
| `Network request failed` trên thiết bị thật | `EXPO_PUBLIC_API_BASE_URL` đang là `localhost` | Đổi thành IP LAN của máy chạy backend |
| Máy ảo Android không gọi được | Cùng lý do | Dùng IP LAN, hoặc `10.0.2.2` cho emulator Android |
| Gọi được nhưng 401 mọi request | Token hết hạn hoặc chưa gắn header | Kiểm tra interceptor có gắn `Authorization: Bearer` không |
| Bản đồ trắng | Khoá Google Maps sai hoặc chưa bật API | Kiểm tra `EXPO_PUBLIC_GOOGLE_MAPS_API_KEY` và các API đã bật trong Google Cloud Console |

### TypeScript báo lỗi ở `src/api/generated/`

**Đừng sửa file trong thư mục đó.** Chạy lại generator:

```bash
./scripts/gen-api-client.sh
```

Vẫn lỗi thì `openapi.yaml` và code đang lệch nhau — chạy subagent `api-contract-guard`
để xác định lệch ở đâu.

---

## Sự cố production

### API trả 5xx hàng loạt

1. Kiểm tra healthcheck: `/actuator/health/readiness`.
2. Xem log, lọc theo `traceId` của một request lỗi mẫu.
3. Khoanh vùng phụ thuộc:

| Phụ thuộc | Kiểm tra nhanh | Ảnh hưởng nếu chết |
|---|---|---|
| PostgreSQL | `pg_isready` | Toàn bộ API chết |
| Redis | `redis-cli ping` | Đăng nhập và rate limit lỗi; phần đọc vẫn chạy |
| S3 | Thử `HEAD` một object | Upload và xem ảnh mới lỗi |
| Claude API | Kiểm tra trang trạng thái | **Chỉ** chat lỗi |

4. Nếu do một lần phát hành gần đây → rollback về image trước.

### Truy vấn tìm quanh đây chậm

Nghi ngờ đầu tiên: **index GiST không được dùng**.

```sql
EXPLAIN ANALYZE
SELECT p.id
FROM places p
WHERE ST_DWithin(p.location, ST_SetSRID(ST_MakePoint(106.6297, 10.8231), 4326)::geography, 2000);
```

Thấy `Seq Scan on places` là có vấn đề. Hai nguyên nhân phổ biến:

1. Có người đổi `ST_DWithin` thành `ST_Distance(...) < x` trong `WHERE` — cách viết đó
   **không dùng được index**. Kết quả vẫn đúng nên rất khó phát hiện.
2. Index bị xoá hoặc chưa được tạo:

```sql
SELECT indexname FROM pg_indexes WHERE tablename = 'places';
-- phải có idx_places_location
```

Chi tiết: [`04-data/geo-model.md`](../04-data/geo-model.md).

### Điểm trung bình sai

`average_rating` là cột dẫn xuất, có thể lệch nếu một lần cập nhật thất bại giữa chừng.
Kiểm tra:

```sql
SELECT p.id, p.average_rating, p.review_count,
       ROUND(AVG(r.rating)::numeric, 1) AS thuc_te,
       COUNT(r.id) AS dem_thuc_te
FROM places p
LEFT JOIN reviews r ON r.place_id = p.id
                   AND r.status = 'APPROVED'
                   AND r.deleted_at IS NULL
WHERE p.deleted_at IS NULL
GROUP BY p.id, p.average_rating, p.review_count
HAVING p.review_count <> COUNT(r.id)
    OR COALESCE(p.average_rating, -1) <> COALESCE(ROUND(AVG(r.rating)::numeric, 1), -1);
```

Có kết quả nghĩa là đang lệch. Chạy job tính lại, và **tìm nguyên nhân** —
lệch dữ liệu dẫn xuất luôn có lý do ở tầng ứng dụng.

### Chatbot không trả lời

1. Kiểm tra `ANTHROPIC_API_KEY` còn hợp lệ.
2. Xem có bị rate limit từ phía Anthropic không (429 trong log).
3. Xác nhận người dùng thấy thông báo thân thiện, **không** thấy chi tiết kỹ thuật (FR-CHAT-08).
4. Chat chết **không được** làm ảnh hưởng phần còn lại của app — nếu có thì đó là lỗi
   cần sửa, không phải sự cố vận hành.

### Push không tới

| Kiểm tra | Ý nghĩa |
|---|---|
| Bản ghi in-app có được tạo không? | Không có → lỗi ở tầng tạo thông báo, không phải push |
| `push_sent_at` có giá trị chưa? | `NULL` + có `push_scheduled_at` → **đang bị hoãn qua đêm**, đúng thiết kế (FR-NOTIF-05) |
| Push token còn hợp lệ? | Expo báo `DeviceNotRegistered` thì token phải bị xoá tự động |

Nhớ: người dùng tắt push vẫn **phải** thấy thông báo trong app (FR-NOTIF-01).

---

## Khôi phục dữ liệu

```bash
# Sao lưu
docker exec foodmap-db pg_dump -U foodmap -Fc foodmap > backup_$(date +%F).dump

# Khôi phục vào CSDL rỗng
docker exec -i foodmap-db pg_restore -U foodmap -d foodmap --clean --if-exists < backup.dump
```

Sau khi khôi phục, **luôn** kiểm tra PostGIS còn hoạt động:

```sql
SELECT PostGIS_Version();
SELECT count(*) FROM places WHERE location IS NOT NULL;
```

Quy trình khôi phục phải được diễn tập ít nhất mỗi quý (NFR-09). Bản sao lưu chưa từng
khôi phục thử không phải là bản sao lưu.

---

## Liên hệ nhanh

| Vấn đề | Xem trước |
|---|---|
| Lược đồ CSDL | [`04-data/erd.md`](../04-data/erd.md), [`data-dictionary.md`](../04-data/data-dictionary.md) |
| Truy vấn địa lý | [`04-data/geo-model.md`](../04-data/geo-model.md) |
| Hợp đồng API | [`03-api/openapi.yaml`](../03-api/openapi.yaml) |
| Yêu cầu nghiệp vụ | [`01-srs/srs.md`](../01-srs/srs.md) |
| Biến môi trường | [`environments.md`](./environments.md) |
