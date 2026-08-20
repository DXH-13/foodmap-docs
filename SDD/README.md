# SDD — Tài liệu thiết kế phần mềm

Thiết kế hệ thống FoodMap: kiến trúc, hợp đồng API, mô hình dữ liệu, thiết kế giao diện
và vận hành. Trả lời câu hỏi *xây dựng như thế nào*, tiếp nối phần *cần gì* của `../SRS/`.

## Nội dung thuộc thư mục này

| Đường dẫn | Mô tả |
|---|---|
| **[`api/openapi.yaml`](./api/openapi.yaml)** | **Hợp đồng API — nguồn sự thật duy nhất** |
| `api/CHANGELOG.md` | Nhật ký thay đổi hợp đồng API, ghi rõ breaking change |
| `kien-truc/` | Sơ đồ C4 (context, container), các quyết định kiến trúc (ADR) |
| `du-lieu/` | ERD, từ điển dữ liệu, mô hình địa lý PostGIS |
| `giao-dien/` | Luồng người dùng, danh sách màn hình |
| `van-hanh/` | Môi trường, quy trình triển khai, runbook |

## `api/openapi.yaml` — đọc kỹ trước khi sửa

File này là **ngoại lệ duy nhất** trong repo tài liệu: nó không phải văn bản mà là
đầu vào của build. Backend implement theo nó; mobile và admin sinh TypeScript client
từ nó; CI của repo cha đọc trực tiếp đường dẫn này.

Sửa nó là thay đổi hợp đồng, ảnh hưởng cả ba phần code. Sau khi sửa, luôn chạy
`./scripts/gen-api-client.sh` ở repo cha và kiểm tra kiểu.

Quy ước đặt tên, cấu trúc schema và cách xử lý breaking change: xem skill `api-contract`
ở repo cha.

## ADR

Mẫu: **Bối cảnh · Quyết định · Phương án đã cân nhắc · Hệ quả** (ghi cả hệ quả xấu,
không chỉ tốt).

**ADR đã merge thì không sửa nội dung.** Đổi ý thì viết ADR mới, đặt trạng thái ADR cũ
thành `Bị thay thế bởi ADR-XXXX` và thêm liên kết hai chiều.
