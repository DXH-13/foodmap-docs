# Stitch — ID màn đã generate

Không commit secret. Ngôn ngữ thiết kế: **iOS 26 Liquid Glass**, dùng component chuẩn iOS.

**Project Stitch:** `FoodMap` — projectId `12141966794107394607` (PRIVATE, PROJECT_DESIGN)
**Design system hiện dùng:** `assets/ba50408a6def46559ec325336b0e2c11` — "Liquid Glass FoodMap".
Luôn truyền id này vào `designSystem` khi generate màn mới để giữ nhất quán.
Design system cũ `assets/8a1fd477a8104b2bbab0ef28cbe51a21` ("FoodMap Vietnam", phong cách Material ấm) **đã bỏ**.

## Màn đã có

| Màn FoodMap | screenId | Trạng thái |
|---|---|---|
| Bản đồ (MAP-01…06, DISCOVERY-01) | `701e470765a1461496ca6375214f615f` | **Bản dùng** — "FoodMap - Liquid Glass Refined", MOBILE, COMPLETE |
| Bản đồ — glass v1 | `e9267aefcebc448baabcfdbe14550f45` | Kính còn quá đục, giữ để đối chiếu |
| Bản đồ — Material v0 | `1e27b853e19541d8a7d5a985de7b2ba0` | Bản đầu trước khi đổi sang Liquid Glass |

Ảnh và HTML tham chiếu: `stitch/ban-do.png`, `stitch/ban-do.html` (bản dùng),
`stitch/ban-do-v1-material.png` (bản Material cũ).
**HTML chỉ để đọc** — không dán vào `mobile/`.

## Thành phần iOS chuẩn trên màn Bản đồ

| Vùng | Component iOS | Ghi chú RN |
|---|---|---|
| Trên cùng | Status bar (9:41, sóng, wifi, pin) | `expo-status-bar` |
| Tìm kiếm | UISearchBar capsule trên kính, SF Symbol `magnifyingglass`, trailing `line.3.horizontal.decrease.circle.fill` màu terracotta | `TextInput` trong `BlurView` |
| Lọc | Hàng capsule cuộn ngang: Loại · Danh mục · Đang mở · Giá. Chip chọn = nền terracotta chữ trắng | `ScrollView horizontal` |
| Nút bản đồ | 2 nút tròn 44pt góc phải: `location.fill` (về vị trí), `square.3.layers.3d` (lớp bản đồ) | `Pressable` + `BlurView`, tối thiểu 44pt |
| Bottom sheet | Sheet iOS ở detent medium, grabber xám giữa mép trên, header "Khu vực lân cận" + link "Xem thêm" | `@gorhom/bottom-sheet`, `backgroundComponent` = BlurView |
| Dòng địa điểm | List style iOS: thumbnail bo góc 60pt, tiêu đề 17pt semibold, phụ đề 15pt, chip trạng thái, trailing `chevron.right`, separator thụt vào ngang chữ | |
| Tab bar | Tab bar nổi iOS 26: capsule kính tách rời, thấy bản đồ quanh và dưới nó, trên home indicator. `map.fill` · `safari` · `heart` · `person.crop.circle` | Custom `tabBar` cho expo-router + BlurView, **không** dùng tab bar mặc định đục |
| Marker | Teardrop kính, glyph theo loại: `fork.knife` quán ăn · `cart.fill` hàng rong · `basket.fill` chợ · `cup.and.saucer.fill` cà phê. Cluster = vòng tròn kính có số | `react-native-maps` custom marker |
| Vị trí user | Chấm xanh iOS + quầng độ chính xác | `showsUserLocation` |

## Token Liquid Glass (đọc từ HTML Stitch)

| Token | Giá trị | Dùng cho |
|---|---|---|
| Blur nền | `blur(40px)` | Bottom sheet, tab bar |
| Blur nhẹ | `blur(10px)` | Nút tròn, chip, scrim status bar |
| Nền kính | `rgba(255,255,255,0.25 – 0.40)` | Càng nhỏ càng trong |
| Viền specular | 1px trắng mép trên/trên-trái, mờ dần | Bắt buộc để ra chất kính |
| Bo góc | 8px (`ROUND_EIGHT`), concentric cho phần tử lồng nhau | |

Trên RN dùng `expo-blur` (`BlurView` `tint="light"`, `intensity` 40–80). iOS có blur thật;
**Android blur yếu hơn nhiều** — cần nền `rgba` đục hơn để chữ còn đọc được, xem mục "Cần quyết định".

## Theme

| Token | Giá trị |
|---|---|
| Font | Inter (thay cho SF Pro — Stitch không có SF Pro; app thật nên dùng font hệ thống) |
| `primary` | `#9f402d` |
| `primary_container` / brand | `#e2725b` (terracotta) |
| `secondary` | `#006e28`, container `#6ffb85` (chip "Đang mở") |
| `background` / `surface` | `#f9f9f9` |
| `on_surface` | `#1a1c1c` |
| `on_surface_variant` | `#56423e` |
| `outline` | `#89726d` |
| `error` | `#ba1a1a` |
| Bo góc | 8px |

Thang chữ theo iOS Dynamic Type: Title 3 Semibold (header sheet) · Body 17 · Subheadline 15 · Caption 12.

## Cách lấy lại HTML / ảnh

MCP `get_screen` với `name = projects/12141966794107394607/screens/<screenId>`,
rồi tải `htmlCode.downloadUrl` và `screenshot.downloadUrl` (URL có hạn, phải lấy lại mỗi lần).

## Còn thiếu / cần quyết định

- Chip **"Giá"** chưa lộ ra ở mép phải trong ảnh render — vẫn phải có khi code.
- Chưa có state **skeleton / rỗng / lỗi** cho bottom sheet.
- Chưa có **biến thể dark** ("Dark Glass") — kính lên rõ nhất trên nền tối, nên làm khi hỗ trợ dark mode.
- **Liquid Glass là ngôn ngữ của iOS.** FoodMap chạy cả Android. Cần chốt: dùng chung một look kính cho cả hai (blur Android yếu, phải tăng độ đục), hay tách theme Material 3 riêng cho Android.
