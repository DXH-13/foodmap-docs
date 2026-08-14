# Quy ước viết tài liệu FoodMap

Repo này chứa **tài liệu**, không chứa code. Ngoại lệ duy nhất là
`03-api/openapi.yaml` — đó là hợp đồng kỹ thuật, sửa nó kéo theo cả backend, mobile và admin.

## Ngôn ngữ và thuật ngữ

- Viết bằng **tiếng Việt**. Thuật ngữ kỹ thuật giữ nguyên tiếng Anh, giải thích ở lần dùng đầu.
- Dùng đúng từ vựng trong `00-overview/glossary-vi-en.md`:
  - `place` — dùng chung cho quán ăn, hàng ăn, chợ. **Không** viết "nhà hàng" khi ý là place.
  - `review` ≠ `feedback`. Review là đánh giá công khai có sao; feedback là báo dữ liệu sai,
    không công khai.
- Định danh kỹ thuật (tên bảng, cột, field API, enum) viết tiếng Anh, đúng như trong code.

## Đánh mã yêu cầu

```
FR-<module>-<số>     yêu cầu chức năng    ví dụ: FR-PLACE-03
NFR-<số>             yêu cầu phi chức năng ví dụ: NFR-05
UC-<module>-<số>     use case              ví dụ: UC-REVIEW-01
```

Module: `AUTH` `PLACE` `REVIEW` `FEEDBACK` `FAVORITE` `VISIT` `NOTIF` `CHAT` `ADMIN` `I18N`.

Mã đã cấp thì **không tái sử dụng**. Yêu cầu bị bỏ thì đánh dấu ~~gạch ngang~~ kèm lý do,
không xoá dòng — người khác có thể đang tham chiếu tới mã đó.

## Yêu cầu phải kiểm chứng được

Sai: *"Hệ thống phải phản hồi nhanh."*
Đúng: *"API tìm quanh đây trả về trong 500ms ở phân vị 95 với 10.000 địa điểm trong CSDL."*

Sai: *"Giao diện phải thân thiện."*
Đúng: *"Người dùng mới hoàn tất đăng ký trong tối đa 3 màn hình, không quá 6 trường nhập."*

Không viết được tiêu chí đo được thì đó chưa phải yêu cầu — đưa vào `07-plan/backlog.md`
kèm câu hỏi mở.

## Use case

Mỗi use case có đủ: **Tác nhân · Tiền điều kiện · Luồng chính · Luồng thay thế · Hậu điều kiện**.
Luồng chính đánh số bước. Luồng thay thế trỏ về bước tương ứng (`3a.`, `3b.`).

## ADR

Mẫu: **Bối cảnh · Quyết định · Phương án đã cân nhắc · Hệ quả** (ghi cả hệ quả xấu, không chỉ tốt).

**ADR đã merge thì không sửa nội dung.** Đổi ý thì viết ADR mới, đặt trạng thái ADR cũ
thành `Bị thay thế bởi ADR-XXXX` và thêm liên kết hai chiều.

## openapi.yaml

Sửa file này là thay đổi hợp đồng, ảnh hưởng cả ba phần code. Quy ước đặt tên, cấu trúc
schema và cách xử lý breaking change: xem skill `api-contract` ở repo cha.

Sau khi sửa, luôn chạy `./scripts/gen-api-client.sh` ở repo cha và kiểm tra kiểu.

## Không viết cái chưa chốt

Tính năng chưa được quyết thì ghi vào `07-plan/backlog.md` kèm câu hỏi mở, đừng bịa
chi tiết trong SRS. Tài liệu mô tả thứ không tồn tại còn tệ hơn không có tài liệu.
