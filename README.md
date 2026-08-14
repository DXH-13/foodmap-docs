# FoodMap — Tài liệu dự án

Repo tài liệu của [FoodMap](https://github.com/DXH-13/foodmap) — ứng dụng bản đồ quán ăn,
hàng ăn và chợ đồ ăn ngon trên đất nước Việt Nam.

Repo này là submodule `docs/` của repo cha.

## Mục lục

| Thư mục | Nội dung |
|---|---|
| [`00-overview/`](./00-overview/) | Tầm nhìn sản phẩm, từ điển thuật ngữ, chân dung người dùng |
| [`01-srs/`](./01-srs/) | Đặc tả yêu cầu phần mềm, use case, tiêu chí nghiệm thu |
| [`02-architecture/`](./02-architecture/) | Sơ đồ C4, các quyết định kiến trúc (ADR) |
| [`03-api/`](./03-api/) | **`openapi.yaml` — hợp đồng API, nguồn sự thật duy nhất** |
| [`04-data/`](./04-data/) | ERD, từ điển dữ liệu, mô hình địa lý |
| [`05-ux/`](./05-ux/) | Luồng người dùng, danh sách màn hình |
| [`06-ops/`](./06-ops/) | Môi trường, triển khai, runbook |
| [`07-plan/`](./07-plan/) | Lộ trình v1, backlog |

## Tài liệu nào quan trọng nhất

1. **[`03-api/openapi.yaml`](./03-api/openapi.yaml)** — hợp đồng API. Backend implement
   theo nó, mobile và admin sinh TypeScript client từ nó. Sửa API luôn bắt đầu từ đây.
2. **[`01-srs/srs.md`](./01-srs/srs.md)** — toàn bộ yêu cầu v1, đánh mã tra cứu được.
3. **[`04-data/erd.md`](./04-data/erd.md)** — lược đồ dữ liệu.

## Quy ước

- Viết bằng **tiếng Việt**; thuật ngữ kỹ thuật giữ nguyên tiếng Anh.
- Dùng đúng từ vựng trong [`00-overview/glossary-vi-en.md`](./00-overview/glossary-vi-en.md).
- Yêu cầu chức năng đánh mã `FR-<module>-<số>`, phi chức năng `NFR-<số>`.
- **ADR đã ghi thì không sửa.** Quyết định sau thay thế bằng ADR mới, trỏ ngược về cái cũ.

Chi tiết: [`CLAUDE.md`](./CLAUDE.md).
