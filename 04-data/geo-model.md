# Mô hình dữ liệu địa lý

Quyết định nền tảng và lý do: [ADR-0003](../02-architecture/adr/ADR-0003-postgis-cho-truy-van-dia-ly.md).
Quy ước viết migration: skill `db-migration` ở repo cha.

---

## Cách lưu toạ độ

**Một cột `geography`, không phải hai cột lat/lng:**

```sql
location geography(Point, 4326) NOT NULL
```

- `geography` — tính khoảng cách bằng **mét trên mặt cầu**. `geometry` tính trên mặt phẳng
  và cần chọn hệ chiếu phù hợp cho từng vùng; không đáng phiền cho bài toán này.
- `4326` — hệ toạ độ WGS-84, chính là thứ GPS và mọi API bản đồ trả về.
- `Point` — mỗi địa điểm là một điểm, không phải vùng.

**Index GiST là bắt buộc**, không phải tối ưu hoá tuỳ chọn:

```sql
CREATE INDEX idx_places_location ON places USING GIST (location);
```

Thiếu index này, truy vấn tìm quanh đây quét toàn bảng. Kết quả vẫn đúng nhưng chậm gấp
hàng chục lần — đây là loại lỗi rất khó phát hiện vì không có gì báo sai.

---

## ⚠️ Hai cái bẫy

### 1. Thứ tự toạ độ: `ST_MakePoint(lng, lat)`

PostGIS nhận **kinh độ trước, vĩ độ sau** — ngược với cách người ta thường đọc và viết.

```sql
-- ĐÚNG: kinh độ trước
ST_SetSRID(ST_MakePoint(106.6297, 10.8231), 4326)::geography   -- TP.HCM

-- SAI: đảo thứ tự → điểm rơi vào Ấn Độ Dương
ST_SetSRID(ST_MakePoint(10.8231, 106.6297), 4326)::geography
```

API JSON của FoodMap thì phơi ra `latitude` rồi `longitude` (thứ tự quen thuộc với người dùng).
**Chỗ chuyển đổi giữa hai quy ước là nơi lỗi hay xảy ra nhất.** Viết test cho nó.

Việt Nam nằm trong khoảng: kinh độ `102.1` → `109.5`, vĩ độ `8.2` → `23.4`.
Kiểm tra đầu vào bằng khoảng này bắt được phần lớn trường hợp đảo nhầm.

### 2. `ST_DWithin` dùng được index, `ST_Distance` thì không

```sql
-- ĐÚNG: dùng index GiST
WHERE ST_DWithin(p.location, :origin, :radius_meters)

-- SAI: quét toàn bảng, kết quả vẫn đúng nhưng rất chậm
WHERE ST_Distance(p.location, :origin) < :radius_meters
```

Dùng `ST_Distance` trong `SELECT` để **lấy** khoảng cách thì không sao — chỉ đừng dùng nó
để **lọc** trong `WHERE`.

---

## Truy vấn tìm quanh đây

```sql
SELECT
    p.id,
    p.slug,
    p.place_type,
    p.average_rating,
    ST_Distance(p.location, :origin)      AS distance_meters,
    ST_Y(p.location::geometry)            AS latitude,
    ST_X(p.location::geometry)            AS longitude
FROM places p
WHERE p.deleted_at IS NULL
  AND p.status IN ('PUBLISHED', 'TEMPORARILY_CLOSED')
  AND ST_DWithin(p.location, :origin, :radius_meters)
ORDER BY p.location <-> :origin
LIMIT :limit OFFSET :offset;
```

Trong đó `:origin` được dựng từ toạ độ người dùng:

```sql
ST_SetSRID(ST_MakePoint(:longitude, :latitude), 4326)::geography
```

- `<->` là toán tử KNN, sắp xếp theo khoảng cách và **dùng được index**.
  Dùng `ORDER BY ST_Distance(...)` cũng ra kết quả đúng nhưng không tận dụng index tốt bằng.
- `ST_Y` trả vĩ độ, `ST_X` trả kinh độ — lại là chỗ dễ nhầm, cần ép về `geometry` trước.

## Kiểm tra vùng cho lượt đến

Luật chống spam (FR-VISIT-02) yêu cầu người dùng ở trong bán kính 200m quanh địa điểm:

```sql
SELECT ST_DWithin(p.location, :user_location, 200) AS is_near,
       ST_Distance(p.location, :user_location)     AS distance_meters
FROM places p
WHERE p.id = :place_id;
```

Trả về cả khoảng cách để thông báo lỗi nói được người dùng đang cách bao xa.

---

## Ràng buộc bán kính

| | Giá trị |
|---|---|
| Mặc định | 2.000 m |
| Tối thiểu | 100 m |
| Tối đa | 50.000 m |

Vượt giới hạn → trả **400** mã `RADIUS_OUT_OF_RANGE`. **Không** âm thầm cắt về 50.000:
người gọi cần biết yêu cầu của họ không được thực hiện như đã yêu cầu (FR-PLACE-02).

---

## Phía Java

Hibernate Spatial với kiểu `org.locationtech.jts.geom.Point`:

```java
@Column(columnDefinition = "geography(Point,4326)", nullable = false)
private Point location;
```

Cấu hình dialect: `org.hibernate.spatial.dialect.postgis.PostgisPGDialect`.

Dựng `Point` từ lat/lng — **lại là chỗ đảo thứ tự**:

```java
private static final GeometryFactory GEO =
        new GeometryFactory(new PrecisionModel(), 4326);

static Point toPoint(double latitude, double longitude) {
    // Coordinate(x, y) = (kinh độ, vĩ độ)
    return GEO.createPoint(new Coordinate(longitude, latitude));
}
```

Bọc phép chuyển đổi này trong **một** hàm tiện ích duy nhất và viết test cho nó.
Rải `new Coordinate(...)` khắp codebase là bảo đảm sẽ có chỗ viết nhầm thứ tự.

---

## Kiểm tra hiệu năng

Định kỳ chạy `EXPLAIN ANALYZE` cho truy vấn tìm quanh đây và xác nhận kế hoạch thực thi
có `Index Scan using idx_places_location`, **không** có `Seq Scan on places` (AC-NFR-02).

```sql
EXPLAIN ANALYZE
SELECT p.id, ST_Distance(p.location, ST_SetSRID(ST_MakePoint(106.6297, 10.8231), 4326)::geography)
FROM places p
WHERE ST_DWithin(p.location, ST_SetSRID(ST_MakePoint(106.6297, 10.8231), 4326)::geography, 2000)
ORDER BY p.location <-> ST_SetSRID(ST_MakePoint(106.6297, 10.8231), 4326)::geography
LIMIT 20;
```

Xuất hiện `Seq Scan` nghĩa là index không được dùng — kiểm tra lại xem có ai đổi
`ST_DWithin` thành `ST_Distance` trong `WHERE` không, hoặc index có bị xoá không.
