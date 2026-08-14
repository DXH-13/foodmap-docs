# Chân dung người dùng

Bốn nhóm dưới đây định hướng các quyết định thiết kế. Khi phân vân giữa hai phương án,
hỏi: phương án nào phục vụ nhóm nào tốt hơn, và nhóm đó có quan trọng ở v1 không.

---

## 1. Minh — người đi ăn hằng ngày

**28 tuổi, nhân viên văn phòng, TP.HCM.** Ăn ngoài 10–12 bữa mỗi tuần.

**Bối cảnh sử dụng:** 11h45, đứng dậy khỏi bàn làm việc, mở app trên đường ra thang máy.
Có 30 giây để quyết định. Sóng 4G trong toà nhà chập chờn.

**Cần:**
- Mở app là thấy ngay quán quanh mình, không phải gõ gì
- Lọc nhanh theo món (bún, cơm, phở) và theo khoảng cách đi bộ
- Biết quán còn mở cửa hay không **ngay lúc này**

**Bực mình khi:** phải đăng nhập mới xem được · bản đồ tải chậm · quán hiện trên bản đồ
nhưng thực tế đã đóng cửa · phải bấm 4 lần mới thấy giờ mở cửa.

**Kéo theo yêu cầu:** khách vãng lai xem được bản đồ và chi tiết quán mà không cần đăng nhập
(FR-PLACE-01) · hiển thị trạng thái đang mở/đã đóng theo giờ hiện tại (FR-PLACE-06) ·
tìm quanh đây phải trả về dưới 500ms (NFR-01).

---

## 2. Hương — người đam mê ẩm thực

**34 tuổi, làm marketing, Hà Nội.** Đi ăn là sở thích, hay dẫn bạn bè đi khám phá.

**Bối cảnh sử dụng:** buổi tối, ngồi nhà, dành 15–20 phút xem và ghi lại các quán đã đi
trong tuần. Có ảnh chụp món ăn muốn đăng.

**Cần:**
- Lưu danh sách quán yêu thích, xem lại được lịch sử đã đi
- Viết review có ảnh và video, đủ dài để nói được điều muốn nói
- Đóng góp quán mới mà mình phát hiện

**Bực mình khi:** giới hạn ảnh quá ít · review bị từ chối mà không nói lý do ·
mất công viết rồi app crash · không biết mình đã đến quán này bao nhiêu lần.

**Kéo theo yêu cầu:** review đính kèm tối đa 5 ảnh + 1 video (FR-REVIEW-02) ·
từ chối review **bắt buộc** kèm lý do và gửi thông báo cho tác giả (FR-REVIEW-05) ·
đếm và hiển thị số lượt đến của chính mình tại từng quán (FR-VISIT-03).

---

## 3. Sarah — khách nước ngoài

**31 tuổi, du lịch Việt Nam 2 tuần.** Không nói tiếng Việt.

**Bối cảnh sử dụng:** đứng ở phố cổ, muốn ăn thứ gì đó "đúng chất", nhưng không biết
biển hiệu ghi gì và món trên menu là món gì. Dùng eSIM, mạng chậm.

**Cần:**
- Toàn bộ giao diện bằng tiếng Anh
- Hiểu **món đó là gì** trước khi gọi, không chỉ biết tên phiên âm
- Tin được đánh giá — biết quán này được người địa phương đánh giá thế nào

**Bực mình khi:** app tiếng Anh nhưng nội dung vẫn tiếng Việt · tên món dịch máy vô nghĩa ·
tất cả review đều tiếng Việt mà không có cách nào hiểu.

**Kéo theo yêu cầu:** toàn bộ chuỗi giao diện có bản `en` (FR-I18N-01) ·
nội dung động có bảng dịch riêng, thiếu `en` thì fallback về `vi`, không bao giờ để trống
(FR-I18N-03) · review giữ nguyên ngôn ngữ gốc và có gắn nhãn ngôn ngữ (FR-I18N-04) ·
chatbot trả lời được bằng tiếng Anh (FR-CHAT-02).

---

## 4. Tuấn — kiểm duyệt viên

**26 tuổi, cộng tác viên bán thời gian.** Dành khoảng 1 giờ mỗi ngày xử lý hàng chờ.

**Bối cảnh sử dụng:** trên máy tính, trang quản trị, xử lý liên tục nhiều mục một lúc.

**Cần:**
- Danh sách chờ có thứ tự ưu tiên rõ ràng, không phải tự tìm việc
- Xem đủ ngữ cảnh để quyết trong vài giây, không phải mở nhiều tab
- Thao tác nhanh cho các trường hợp phổ biến

**Bực mình khi:** phải mở từng review ở tab mới · không thấy được lịch sử của người dùng
đang bị xét · vô tình xoá nhầm mà không hoàn tác được.

**Kéo theo yêu cầu:** hàng chờ có phân trang, lọc và sắp xếp phía server (FR-ADMIN-02) ·
hiển thị ngữ cảnh ngay trong danh sách: ảnh thu nhỏ, thông tin place, lịch sử tác giả
(FR-ADMIN-03) · mọi hành động không hoàn tác được phải có bước xác nhận (FR-ADMIN-06) ·
tự động gắn cờ place khi có 3 báo cáo `CLOSED_PERMANENTLY` trong 30 ngày (FR-FEEDBACK-05).

---

## Ai không phải người dùng của v1

Ghi rõ để tránh bị kéo phạm vi:

- **Chủ quán.** Không có tài khoản riêng để tự quản lý thông tin quán ở v1.
  Muốn sửa thì gửi feedback như người dùng thường.
- **Nhà quảng cáo.** Không có vị trí trả phí, không có xếp hạng ưu tiên.
- **Người dùng web.** Không có web app cho người dùng cuối; trang admin chỉ dành cho
  moderator và admin.
- **Người dùng ngoài Việt Nam.** Dữ liệu và ngôn ngữ đều tập trung vào thị trường Việt Nam.
