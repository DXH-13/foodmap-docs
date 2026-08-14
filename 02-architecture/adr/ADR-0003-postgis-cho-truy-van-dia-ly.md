# ADR-0003: Dùng PostGIS thay vì tính khoảng cách ở tầng ứng dụng

| | |
|---|---|
| Trạng thái | Chấp nhận |
| Ngày | 2026-08-14 |

## Bối cảnh

Tính năng cốt lõi của FoodMap là "tìm quán quanh đây": cho một toạ độ và một bán kính,
trả về các địa điểm trong vùng đó, sắp xếp theo khoảng cách.

Yêu cầu [NFR-01](../../01-srs/srs.md#hiệu-năng) đặt mục tiêu **500ms ở phân vị 95 với
10.000 địa điểm**. Đây là truy vấn được gọi nhiều nhất trong toàn hệ thống — mỗi lần
người dùng mở app hoặc di chuyển bản đồ.

## Quyết định

Lưu toạ độ bằng một cột PostGIS **`geography(Point, 4326)`** với **index GiST**, và
thực hiện lọc theo bán kính ngay trong CSDL bằng `ST_DWithin`.

```sql
location geography(Point, 4326) NOT NULL
```

```sql
CREATE INDEX idx_places_location ON places USING GIST (location);
```

```sql
SELECT p.*, ST_Distance(p.location, :origin) AS distance_meters
FROM places p
WHERE p.status = 'PUBLISHED'
  AND p.deleted_at IS NULL
  AND ST_DWithin(p.location, :origin, :radius_meters)
ORDER BY p.location <-> :origin
LIMIT :limit;
```

Phía Java dùng Hibernate Spatial với kiểu `org.locationtech.jts.geom.Point`.

Chọn `geography` (không phải `geometry`) vì nó tính khoảng cách bằng **mét trên mặt cầu** —
đúng cho bài toán "quanh đây" mà không phải chọn hệ chiếu phù hợp cho từng vùng.

## Phương án đã cân nhắc

**Hai cột `latitude` / `longitude` + công thức Haversine ở tầng Java.**
Không cần extension, dễ hiểu. Loại vì phải nạp toàn bộ địa điểm về ứng dụng rồi mới lọc —
với 10.000 bản ghi là 10.000 dòng qua mạng và qua bộ nhớ cho **mỗi** lần tìm.
Không đạt được NFR-01, và không mở rộng được.

**Hai cột lat/lng + lọc thô bằng bounding box trong SQL, rồi tính chính xác ở Java.**
Nhanh hơn phương án trên đáng kể. Loại vì đây là tự dựng lại một phần PostGIS bằng tay,
kém chính xác ở gần cực và ở đường đổi ngày, và vẫn phải sắp xếp ở tầng ứng dụng.
PostGIS làm đúng việc này tốt hơn và đã được kiểm chứng.

**Elasticsearch với `geo_point`.** Truy vấn địa lý mạnh, tìm toàn văn tốt. Loại vì
thêm một hệ thống lưu trữ nữa phải vận hành và đồng bộ, trong khi dữ liệu đã nằm ở
PostgreSQL. Chi phí vận hành không đáng ở quy mô v1.

**Redis GEO commands.** Rất nhanh cho truy vấn bán kính. Loại vì Redis là bộ nhớ đệm,
không phải nguồn sự thật; phải đồng bộ hai chiều và xử lý trường hợp lệch dữ liệu.
Vẫn có thể dùng làm cache cho kết quả tìm quanh đây sau này.

**Gọi Google Places API để tìm.** Loại vì dữ liệu của FoodMap là dữ liệu riêng (do cộng đồng
đóng góp), không nằm trên Google; và chi phí theo lượt gọi sẽ tăng theo lượng truy cập.

## Hệ quả

### Tích cực
- Lọc và sắp xếp diễn ra trong CSDL, có index — chỉ những bản ghi cần thiết đi qua mạng.
- `ST_Distance` trả khoảng cách chính xác theo mét, dùng thẳng cho giao diện.
- Toán tử KNN `<->` sắp xếp theo khoảng cách và cũng dùng được index.
- Lọc kết hợp (loại địa điểm, danh mục, điểm tối thiểu) làm luôn trong cùng một truy vấn.
- PostGIS là extension trưởng thành, tài liệu tốt, không phải hệ thống mới phải vận hành.

### Tiêu cực
- Ràng buộc chặt vào PostgreSQL. Đổi sang CSDL khác sẽ phải viết lại toàn bộ tầng truy vấn địa lý.
- **Không test bằng H2 được** — H2 không có PostGIS. Mọi integration test phải dùng
  Testcontainers với image PostGIS thật, làm test chậm hơn.
- Hibernate Spatial thêm một lớp phụ thuộc và cần cấu hình dialect đúng.
- Có hai bẫy dễ mắc:
  - `ST_MakePoint(lng, lat)` — **kinh độ trước**, ngược với cách người ta hay đọc.
    Nhầm là kết quả ra giữa đại dương.
  - `ST_Distance(...) < x` trong `WHERE` **không dùng được index**; phải viết `ST_DWithin`.
    Sai chỗ này thì truy vấn vẫn đúng kết quả nhưng chậm gấp hàng chục lần — rất khó phát hiện.

### Cần theo dõi
- Chạy `EXPLAIN ANALYZE` định kỳ cho truy vấn tìm quanh đây và xác nhận kế hoạch thực thi
  vẫn dùng index GiST, không rơi về `Seq Scan` (xem AC-NFR-02).
- Thời gian phản hồi khi số địa điểm vượt 100.000. Lúc đó cân nhắc thêm cache Redis cho
  các vùng truy vấn phổ biến.
