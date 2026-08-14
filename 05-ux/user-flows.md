# Luồng người dùng

Các luồng chính, ở mức điều hướng. Chi tiết hành vi và luồng thay thế:
[use case](../01-srs/use-cases.md). Danh sách màn hình: [`screens.md`](./screens.md).

---

## Lần mở app đầu tiên

```mermaid
flowchart TD
    A[Mở app] --> B{Đã đăng nhập?}
    B -->|Chưa| C[Tab Bản đồ ở chế độ khách]
    B -->|Rồi| D[Tab Bản đồ đã đăng nhập]
    C --> E{Xin quyền vị trí}
    D --> E
    E -->|Cho phép| F[Căn bản đồ vào vị trí hiện tại<br/>tải danh sách quanh đây]
    E -->|Từ chối| G[Trạng thái rỗng có ý nghĩa<br/>+ ô tìm theo khu vực<br/>+ nút mở Cài đặt]
    F --> H[Xem bản đồ và danh sách]
    G --> H
```

**Chủ ý:** khách vãng lai dùng được ngay, không có màn hình đăng nhập chắn ở đầu.
Chỉ chặn khi người dùng thật sự cần tài khoản (viết review, yêu thích, check-in).

---

## Từ khám phá tới đóng góp

```mermaid
flowchart LR
    A[Bản đồ] --> B[Chi tiết địa điểm]
    B --> C{Muốn làm gì}
    C -->|Viết đánh giá| D{Đã đăng nhập?}
    C -->|Yêu thích| D
    C -->|Đã đến đây| D
    C -->|Báo sai| D
    C -->|Chỉ xem| B
    D -->|Chưa| E[Đăng nhập / Đăng ký]
    E --> F{Email đã xác minh?}
    D -->|Rồi| F
    F -->|Chưa| G[Nhắc xác minh<br/>+ nút gửi lại email]
    F -->|Rồi| H[Thực hiện hành động]
    H --> B
```

**Chủ ý:** sau khi đăng nhập, quay lại **đúng chỗ đang dở**, không đẩy về màn hình chính.
Mất ngữ cảnh là lý do phổ biến khiến người dùng bỏ giữa chừng.

---

## Viết đánh giá

```mermaid
flowchart TD
    A[Chi tiết địa điểm] --> B[Bấm Viết đánh giá]
    B --> C{Đã có đánh giá<br/>cho quán này?}
    C -->|Rồi| D[Form nạp sẵn nội dung cũ<br/>chế độ Cập nhật]
    C -->|Chưa| E[Form trống]
    D --> F[Chọn sao + nhập nội dung]
    E --> F
    F --> G[Thêm ảnh / video]
    G --> H[Nén phía client<br/>xin presigned URL<br/>upload thẳng lên storage]
    H --> I{Upload lỗi?}
    I -->|Có| J[Thử lại riêng file đó<br/>hoặc bỏ file]
    J --> H
    I -->|Không| K[Bấm Gửi]
    K --> L[Đánh giá vào trạng thái PENDING]
    L --> M[Thông báo: đang chờ duyệt]
```

**Chủ ý:** upload lỗi chỉ phải làm lại **file đó**, không phải toàn bộ. Gửi thất bại
thì **giữ nguyên nội dung đã nhập** — không được để mất bài viết của người dùng.

---

## Kiểm duyệt (trang quản trị)

```mermaid
flowchart TD
    A[Hàng chờ duyệt] --> B[Xem một mục<br/>kèm ngữ cảnh ngay trong danh sách]
    B --> C{Quyết định}
    C -->|Duyệt| D[APPROVED<br/>tính lại điểm trung bình<br/>gửi thông báo]
    C -->|Từ chối| E[Bắt buộc nhập lý do]
    E --> F[REJECTED<br/>gửi thông báo kèm lý do]
    C -->|Ẩn| G[HIDDEN<br/>tính lại điểm trung bình]
    D --> H[Ghi audit log]
    F --> H
    G --> H
    H --> A
```

**Chủ ý:** ngữ cảnh (ảnh thu nhỏ, tên quán, lịch sử tác giả) hiển thị **ngay trong
danh sách**, không phải mở tab mới cho từng mục. Moderator xử lý hàng chục mục mỗi phiên.

---

## Chatbot

```mermaid
flowchart TD
    A[Tab Chat] --> B[Nhập câu hỏi]
    B --> C[Gửi kèm toạ độ hiện tại nếu có]
    C --> D[Backend gọi mô hình AI]
    D --> E[Mô hình gọi tool<br/>truy vấn dữ liệu thật của FoodMap]
    E --> F[Stream câu trả lời qua SSE]
    F --> G[Hiển thị dần + thẻ địa điểm bấm được]
    G --> H{Bấm một thẻ?}
    H -->|Có| I[Chi tiết địa điểm]
    H -->|Không| B
```

**Chủ ý:** stream để token đầu tiên xuất hiện trong 2 giây — chờ 10 giây nhìn màn hình
trống thì người dùng bỏ đi. Không tìm thấy thì nói thẳng, **không bịa địa điểm**.

---

## Quyết định điều hướng chung

| Tình huống | Cách xử lý |
|---|---|
| Chưa đăng nhập, chạm hành động cần tài khoản | Mở màn hình đăng nhập, **quay lại đúng chỗ** sau khi xong |
| Đã đăng nhập nhưng chưa xác minh email | Hiện nhắc nhở inline + nút gửi lại, **không** chặn cả màn hình |
| Từ chối quyền vị trí | Trạng thái rỗng có ý nghĩa + tìm theo khu vực + nút mở Cài đặt |
| Mất mạng | Hiện dữ liệu cache kèm nhãn "ngoại tuyến" + nút Thử lại |
| Không có kết quả | Nói rõ vì sao và gợi ý hành động (mở rộng bán kính, bỏ bớt bộ lọc) |
| Lỗi server | Thông báo thân thiện + nút Thử lại. **Không** hiện chi tiết kỹ thuật |
