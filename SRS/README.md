# SRS — Đặc tả yêu cầu phần mềm

Yêu cầu chức năng và phi chức năng của FoodMap v1, đánh mã để tra cứu chéo từ
SDD, test case và bug report.

## Nội dung thuộc thư mục này

| Tài liệu | Mô tả |
|---|---|
| `srs.md` | Toàn bộ yêu cầu v1, đánh mã tra cứu được |
| `use-cases.md` | Use case chi tiết theo module |
| `acceptance-criteria.md` | Tiêu chí nghiệm thu cho từng yêu cầu |

## Đánh mã yêu cầu

```
FR-<module>-<số>      yêu cầu chức năng      ví dụ: FR-PLACE-03
NFR-<số>              yêu cầu phi chức năng  ví dụ: NFR-05
UC-<module>-<số>      use case               ví dụ: UC-REVIEW-01
```

Module: `AUTH` `PLACE` `REVIEW` `FEEDBACK` `FAVORITE` `VISIT` `NOTIF` `CHAT` `ADMIN` `I18N`.

Mã đã cấp thì **không tái sử dụng**. Yêu cầu bị bỏ thì đánh dấu ~~gạch ngang~~ kèm lý do,
không xoá dòng — người khác có thể đang tham chiếu tới mã đó.

## Yêu cầu phải kiểm chứng được

Sai: *"Hệ thống phải phản hồi nhanh."*
Đúng: *"API tìm quanh đây trả về trong 500ms ở phân vị 95 với 10.000 địa điểm trong CSDL."*

Sai: *"Giao diện phải thân thiện."*
Đúng: *"Người dùng mới hoàn tất đăng ký trong tối đa 3 màn hình, không quá 6 trường nhập."*

Không viết được tiêu chí đo được thì đó chưa phải yêu cầu — đưa vào
`../Management-Plan/backlog.md` kèm câu hỏi mở.

## Use case

Mỗi use case có đủ: **Tác nhân · Tiền điều kiện · Luồng chính · Luồng thay thế · Hậu điều kiện**.
Luồng chính đánh số bước. Luồng thay thế trỏ về bước tương ứng (`3a.`, `3b.`).
