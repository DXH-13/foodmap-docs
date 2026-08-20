# FoodMap — Tài liệu dự án

Repo tài liệu của [FoodMap](https://github.com/DXH-13/foodmap) — ứng dụng bản đồ quán ăn,
hàng ăn và chợ đồ ăn ngon trên đất nước Việt Nam.

Repo này là submodule `docs/` của repo cha.

## Mục lục

| Thư mục | Nội dung |
|---|---|
| [`Introduction/`](./Introduction/) | Tổng quan dự án, tầm nhìn, chân dung người dùng, từ điển thuật ngữ |
| [`Management-Plan/`](./Management-Plan/) | Kế hoạch dự án, lộ trình, backlog, phân công, quản lý rủi ro |
| [`SRS/`](./SRS/) | Đặc tả yêu cầu phần mềm, use case, tiêu chí nghiệm thu |
| [`SDD/`](./SDD/) | **Thiết kế hệ thống — chứa `api/openapi.yaml`, hợp đồng API** |
| [`Test-Document/`](./Test-Document/) | Kế hoạch kiểm thử, test case, ma trận phủ, báo cáo kết quả |
| [`User-Guides/`](./User-Guides/) | Hướng dẫn sử dụng cho người dùng cuối và quản trị viên |

## Tài liệu nào quan trọng nhất

1. **[`SDD/api/openapi.yaml`](./SDD/api/openapi.yaml)** — hợp đồng API. Backend implement
   theo nó, mobile và admin sinh TypeScript client từ nó. Sửa API luôn bắt đầu từ đây.
2. **[`SRS/`](./SRS/)** — toàn bộ yêu cầu v1, đánh mã tra cứu được.
3. **[`SDD/du-lieu/`](./SDD/)** — lược đồ dữ liệu, ERD, mô hình địa lý.

## Quy ước

- Viết bằng **tiếng Việt**; thuật ngữ kỹ thuật giữ nguyên tiếng Anh.
- Dùng đúng từ vựng trong [`Introduction/`](./Introduction/) — từ điển thuật ngữ vi–en.
- Yêu cầu chức năng đánh mã `FR-<module>-<số>`, phi chức năng `NFR-<số>`.
- **ADR đã ghi thì không sửa.** Quyết định sau thay thế bằng ADR mới, trỏ ngược về cái cũ.
- Tên thư mục cấp một viết theo tiêu đề tài liệu bàn giao, **không dùng khoảng trắng**
  (`Management-Plan`, không phải `Management Plan`) — đường dẫn còn được script và CI đọc.

Chi tiết: [`CLAUDE.md`](./CLAUDE.md).
