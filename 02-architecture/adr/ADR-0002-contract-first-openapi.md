# ADR-0002: OpenAPI là nguồn sự thật của hợp đồng API

| | |
|---|---|
| Trạng thái | Chấp nhận |
| Ngày | 2026-08-14 |

## Bối cảnh

[ADR-0001](./ADR-0001-tech-stack.md) chọn Java cho backend và TypeScript cho mobile
lẫn admin. Hệ quả trực tiếp: **hai bên không share type với nhau được**.

Ba client-server pair phải khớp nhau về hình dạng dữ liệu:
`backend ↔ mobile`, `backend ↔ admin`. Không có cơ chế nào ép điều đó, DTO ở ba nơi
sẽ trôi khỏi nhau âm thầm, và lỗi chỉ lộ ra ở runtime — thường là trên máy người dùng.

## Quyết định

`docs/03-api/openapi.yaml` là **nguồn sự thật duy nhất** của hợp đồng API.

Quy trình bắt buộc khi thêm hoặc sửa endpoint:

```
1. Sửa openapi.yaml
2. Sinh lại TypeScript client cho mobile và admin
3. Implement backend theo spec
4. Dùng client đã sinh ở client
5. Test
```

Hai ràng buộc kèm theo:

- **Không sửa tay file trong thư mục `generated/`.** Chúng bị ghi đè.
- **Không thêm endpoint ở backend mà không cập nhật spec.**

Backend chạy springdoc để sinh spec **từ code** tại `/v3/api-docs`, dùng làm công cụ
đối chiếu — phát hiện khi code đã trôi khỏi hợp đồng.

## Phương án đã cân nhắc

**Code-first: sinh spec từ annotation Java, client sinh từ spec đó.**
Ít bước hơn, không phải viết YAML tay. Loại vì spec trở thành *hệ quả* của code backend
thay vì *hợp đồng* giữa các bên: mobile không thể bắt đầu làm trước khi backend xong,
và mọi thay đổi backend tự động thành thay đổi hợp đồng mà không ai xem xét.

**Không sinh client, viết type TypeScript tay ở mỗi bên.**
Đơn giản lúc đầu. Loại vì đó chính là vấn đề cần tránh — ba bản DTO viết tay sẽ lệch nhau,
và không có gì phát hiện ra.

**Đổi backend sang TypeScript để share type trực tiếp.**
Giải quyết triệt để vấn đề. Loại ở ADR-0001 vì đội quen Java, và cái giá của việc đổi
ngôn ngữ backend lớn hơn cái giá của một bước sinh code.

**gRPC + Protobuf.** Hợp đồng chặt hơn, sinh code cho cả hai ngôn ngữ. Loại vì hỗ trợ
trên React Native còn phiền, khó debug bằng công cụ HTTP thông thường, và lợi ích chưa
đủ bù chi phí ở quy mô v1.

## Hệ quả

### Tích cực
- Một chỗ duy nhất định nghĩa hợp đồng; ba bên không thể lệch nhau âm thầm.
- Mobile và admin bắt đầu được **trước** khi backend implement xong — chỉ cần spec.
- Đổi hợp đồng làm TypeScript báo lỗi biên dịch ngay ở đúng chỗ bị ảnh hưởng.
  Đây là tính năng, không phải phiền toái.
- Spec là tài liệu API luôn cập nhật, không phải wiki viết tay rồi bỏ đó.

### Tiêu cực
- **Thêm một bước thủ công phải nhớ:** chạy generator sau khi sửa spec. Quên thì client
  vẫn dùng type cũ. Đây là rủi ro chính của quyết định này.
- Viết YAML OpenAPI tay dài dòng hơn viết annotation Java.
- Có thể trôi hai chiều: backend implement lệch spec, hoặc spec mô tả thứ backend chưa có.
  Cần công cụ đối chiếu (subagent `api-contract-guard` + springdoc).
- Thay đổi phá vỡ tốn công hơn: mobile không cập nhật tức thì được, phải giữ field cũ
  qua vài phiên bản app.

### Cần theo dõi
- Số lần phát hiện client và spec lệch nhau. Xảy ra nhiều thì đưa bước sinh client
  và kiểm tra `git diff` sạch vào CI như một cổng bắt buộc.
- Độ dài của `openapi.yaml`. Vượt khoảng 2.000 dòng thì tách theo module bằng `$ref`
  sang nhiều file.
