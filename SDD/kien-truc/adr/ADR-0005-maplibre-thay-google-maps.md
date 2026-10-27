# ADR-0005: Vẽ bản đồ trên app bằng MapLibre + OpenFreeMap thay cho Google Maps

**Trạng thái:** Đã chấp nhận · **Ngày:** 2026-10-26

Thay thế một phần ADR-0001 (tech stack): chỉ phần **thư viện vẽ bản đồ trên app di động**.
Quyết định dùng Google Places API để seed dữ liệu địa điểm ban đầu (ở backend) giữ nguyên.

## Bối cảnh

App di động vẽ bản đồ bằng `react-native-maps` với `PROVIDER_GOOGLE`. Trên Android,
thư viện này **luôn** chạy bằng Google Maps SDK, nên:

- Phải có khoá Google Maps nhúng vào APK. Thiếu khoá thì app crash ngay khi mở tab Bản đồ —
  tab mặc định.
- Project Google Cloud bắt buộc gắn tài khoản thanh toán, kể cả khi dùng trong hạn mức miễn phí.
- Mọi lập trình viên muốn build app trên máy mình đều cần khoá — chặn việc chạy thử nhanh.

Bản đồ ở FoodMap **chỉ để vẽ**: dữ liệu địa điểm và truy vấn "quanh đây" đến từ PostGIS
của backend, không đến từ nhà cung cấp bản đồ.

## Quyết định

Dùng **`@maplibre/maplibre-react-native`** (MapLibre Native) với bản đồ nền
**OpenFreeMap** (`https://tiles.openfreemap.org/styles/liberty`), dữ liệu OpenStreetMap.

- Không khoá API, không thanh toán, không giới hạn lượt tải tile.
- Một thư viện, một giao diện bản đồ cho cả iOS lẫn Android.

## Phương án đã cân nhắc

| Phương án | Lý do không chọn |
|---|---|
| Giữ Google Maps | Cần khoá và thẻ thanh toán để build được app; đó chính là vấn đề cần gỡ |
| `react-native-maps` + tile OSM (`UrlTile`) | Trên Android vẫn chạy trên Google Maps SDK — vẫn cần khoá |
| Leaflet trong WebView | Chậm, giật khi kéo, khó tương tác giữa marker và phần native |
| MapLibre + tile thương mại (MapTiler, Stadia) | Lại cần khoá; OpenFreeMap đủ cho v1 |

## Hệ quả

**Tốt**
- Build app không cần bất kỳ khoá nào. Bỏ `EXPO_PUBLIC_GOOGLE_MAPS_API_KEY` khỏi mobile.
- Không phát sinh chi phí bản đồ theo lượng người dùng.
- Đổi kiểu bản đồ chỉ là đổi URL style (`liberty`, `positron`, `bright`), hoặc tự host tile sau này.

**Xấu**
- **Không chạy trong Expo Go** — MapLibre là native module. Chạy thử phải dùng development
  build (`npx expo run:android`) hoặc APK.
- Dữ liệu nền OSM ở Việt Nam thưa hơn Google ở vùng nông thôn và kém chi tiết về POI.
  Ảnh hưởng nhỏ: marker quán ăn vẽ từ dữ liệu của chính FoodMap.
- OpenFreeMap là dịch vụ cộng đồng, **không có SLA**. Nếu cần cam kết sẵn sàng, chuyển sang
  tự host tile (OpenFreeMap phát hành bộ tile để tự chạy) hoặc nhà cung cấp trả phí — chỉ
  đổi `MAP_STYLE_URL`.
- Bắt buộc hiện ghi công OpenStreetMap trên bản đồ (giấy phép ODbL). Không được tắt `attribution`.
- Bản web vẫn không chạy màn hình Bản đồ (như trước).
