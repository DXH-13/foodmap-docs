# Architecture Decision Records

Mỗi ADR ghi lại **một** quyết định kiến trúc: bối cảnh lúc đó, quyết định đã chọn,
các phương án đã cân nhắc, và hệ quả — cả tốt lẫn xấu.

## Danh sách

| Mã | Tiêu đề | Trạng thái | Ngày |
|---|---|---|---|
| [ADR-0001](./ADR-0001-tech-stack.md) | Chọn công nghệ nền tảng cho v1 | Chấp nhận | 2026-08-14 |
| [ADR-0002](./ADR-0002-contract-first-openapi.md) | OpenAPI là nguồn sự thật của hợp đồng API | Chấp nhận | 2026-08-14 |
| [ADR-0003](./ADR-0003-postgis-cho-truy-van-dia-ly.md) | Dùng PostGIS thay vì tính khoảng cách ở tầng ứng dụng | Chấp nhận | 2026-08-14 |
| [ADR-0004](./ADR-0004-git-submodules.md) | Repo cha + git submodules thay vì monorepo | Chấp nhận | 2026-08-14 |

## Quy tắc

**ADR đã merge thì không sửa nội dung.** Đổi ý thì viết ADR mới, đặt trạng thái ADR cũ
thành `Bị thay thế bởi ADR-XXXX`, và thêm liên kết hai chiều. Lịch sử quyết định có giá trị
riêng của nó — biết *tại sao* trước đây chọn khác giúp tránh lặp lại sai lầm cũ.

**Trạng thái:** `Đề xuất` · `Chấp nhận` · `Bị thay thế bởi ADR-XXXX` · `Bị huỷ`

## Mẫu

```markdown
# ADR-XXXX: <tiêu đề>

| | |
|---|---|
| Trạng thái | Đề xuất / Chấp nhận / Bị thay thế bởi ADR-YYYY |
| Ngày | YYYY-MM-DD |

## Bối cảnh
Vấn đề là gì, ràng buộc nào đang có, tại sao phải quyết ngay bây giờ.

## Quyết định
Chọn gì. Viết ở thể chủ động: "Chúng tôi dùng X."

## Phương án đã cân nhắc
Từng phương án kèm lý do loại bỏ. Không liệt kê phương án hình thức —
chỉ ghi những cái thật sự đã cân nhắc.

## Hệ quả
### Tích cực
### Tiêu cực
### Cần theo dõi
Dấu hiệu nào cho thấy quyết định này cần xem lại.
```
