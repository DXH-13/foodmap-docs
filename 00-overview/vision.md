# Tầm nhìn sản phẩm

## Vấn đề

Ẩm thực Việt Nam nằm rải rác ở những nơi ứng dụng bản đồ phổ thông làm chưa tốt: gánh
bún riêu chỉ bán buổi sáng ở một góc phố, hàng bánh mì không có biển hiệu, quầy trong
chợ mà người địa phương ai cũng biết nhưng không có mặt trên bản đồ nào.

Ba nhóm khó khăn cụ thể:

1. **Dữ liệu không có hoặc sai.** Hàng rong và quầy chợ hiếm khi có trên Google Maps.
   Có thì cũng thường sai giờ mở cửa, hoặc quán đã đóng cửa từ lâu.
2. **Đánh giá không nói về đồ ăn.** Review trên các nền tảng chung thường nói về không gian,
   chỗ đỗ xe, thái độ phục vụ — ít khi nói món nào ngon và ngon thế nào.
3. **Người nước ngoài bị chặn bởi ngôn ngữ.** Tên món tiếng Việt không dịch được bằng
   trực giác; "bún bò Huế" và "bún bò Nam Bộ" là hai món khác hẳn nhau.

## Sản phẩm

FoodMap là bản đồ đồ ăn Việt Nam do cộng đồng đóng góp, tập trung vào **món ăn** thay vì
**cơ sở kinh doanh**, và có mặt cả tiếng Việt lẫn tiếng Anh.

Ba điểm khác biệt:

- **Nhận cả hàng rong và quầy chợ**, không chỉ nhà hàng có mặt bằng. Mô hình dữ liệu
  hỗ trợ địa điểm không có địa chỉ chính thức và có giờ bán rất hẹp.
- **Cộng đồng sửa dữ liệu.** Người dùng báo sai địa chỉ, sai giờ, quán đã đóng cửa;
  moderator xử lý. Dữ liệu tự lành theo thời gian.
- **Song ngữ ngay từ đầu**, không phải dịch bổ sung sau. Nội dung động có bảng dịch riêng,
  thiếu bản tiếng Anh thì fallback về tiếng Việt.

## Người dùng mục tiêu

Chi tiết ở [`personas.md`](./personas.md). Tóm tắt:

| Nhóm | Nhu cầu chính |
|---|---|
| Người đi ăn hằng ngày | Tìm nhanh chỗ ăn ngon quanh vị trí hiện tại |
| Người đam mê ẩm thực | Ghi lại nơi đã đi, chia sẻ phát hiện, xây dựng danh sách riêng |
| Khách nước ngoài | Hiểu món ăn là gì, tin được đánh giá, đọc bằng tiếng Anh |
| Moderator | Giữ dữ liệu sạch mà không tốn quá nhiều công |

## Phạm vi v1

**Có:** bản đồ và tìm quanh đây · review kèm ảnh/video · feedback sửa dữ liệu ·
song ngữ vi/en · chatbot gợi ý · trang quản trị · xác thực · yêu thích và lịch sử đã đến ·
thông báo.

**Không có ở v1:** đặt bàn, đặt món, thanh toán · mạng xã hội theo dõi bạn bè ·
tài khoản cho chủ quán tự quản lý · gợi ý cá nhân hoá bằng học máy · web app cho
người dùng cuối (chỉ có mobile) · vùng ngoài Việt Nam.

## Thước đo thành công

Đây là các chỉ số đánh giá v1 sau khi ra mắt, không phải yêu cầu kỹ thuật:

| Chỉ số | Mục tiêu 3 tháng đầu |
|---|---|
| Địa điểm đã publish | 5.000 |
| Tỷ lệ địa điểm có ít nhất 1 review | 40% |
| Feedback được xử lý trong 7 ngày | 90% |
| Tỷ lệ người dùng quay lại sau 30 ngày | 25% |
| Thời gian trung bình từ mở app đến thấy kết quả quanh đây | dưới 3 giây |

## Nguyên tắc thiết kế

1. **Bản đồ là màn hình chính**, không phải danh sách. Người dùng nghĩ theo không gian:
   "quanh đây có gì".
2. **Dữ liệu sai còn tệ hơn không có dữ liệu.** Báo sai phải dễ hơn viết review.
3. **Chấp nhận dữ liệu chưa hoàn chỉnh.** Một hàng bún không có số điện thoại, không có
   ảnh, chỉ có toạ độ và tên — vẫn có giá trị. Đừng bắt buộc quá nhiều trường.
4. **Mạng di động Việt Nam không ổn định.** Mọi màn hình phải có trạng thái tải, trạng thái
   lỗi và khả năng thử lại. Ảnh phải lazy-load và cache.
5. **Tiếng Anh là công dân hạng nhất**, không phải bản dịch máy gắn thêm.
