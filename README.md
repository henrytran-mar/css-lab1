# Mô hình đe dọa — Trang bán hàng trực tuyến

## Hệ thống tôi chọn

Em chọn một trang bán hàng trực tuyến gồm đúng ba thành phần, vì nó quen thuộc và đủ rõ để vẽ luồng dữ liệu:

1. **Trình duyệt** — nơi khách xem sản phẩm và bấm đặt hàng, chỉ giao tiếp qua HTTP với máy chủ.
2. **Máy chủ ứng dụng** — làm logic nghiệp vụ, giữ phiên đăng nhập, và hỏi cơ sở dữ liệu khi cần.
3. **Cơ sở dữ liệu** — cất tài khoản khách hàng, danh mục sản phẩm và đơn hàng.

Luồng đơn giản: trình duyệt gửi yêu cầu → máy chủ ứng dụng xử lý rồi đọc/ghi cơ sở dữ liệu → trả kết quả về trình duyệt.

## Danh sách 8 mối đe dọa

Xếp theo tích tác động × khả năng, từ cao xuống thấp:

| Mã | Mối đe dọa tóm tắt | Nguyên lý | ATT&CK | Tác động × Khả năng |
|---|---|---|---|---|
| M02 | Sửa mã đơn hàng trên URL để đọc đơn của người khác | 2 | T1530 | 4 × 4 = 16 |
| M01 | Nhập chuỗi lạ vào ô tìm kiếm để đọc bảng tài khoản (SQLi) | 5 | T1190 | 5 × 3 = 15 |
| M05 | Mật khẩu / thẻ thanh toán lưu dạng chữ rõ trong DB | 2 | T1552 | 5 × 3 = 15 |
| M03 | Chèn kịch bản vào đánh giá, chạy trong trình duyệt người khác | 3 | T1059.007 | 4 × 3 = 12 |
| M07 | Trang lỗi lộ thông tin cấu hình bên trong | 1 | T1592 | 3 × 4 = 12 |
| M08 | Cookie phiên thiếu cờ an toàn, dễ bị chiếm | 3 | T1539 | 4 × 3 = 12 |
| M04 | Vẫn nhận mật khẩu mặc định của tài khoản quản trị | 4 | T1110 | 5 × 2 = 10 |
| M06 | Thư viện bên thứ ba bị nhúng mã độc (chuỗi cung ứng) | 6 | T1195 | 3 × 3 = 9 |

## Ba mối tôi quyết định xử lý

Chọn **M02 (16), M01 (15) và M05 (15)**.

Lý do: cả ba đều nằm ở nửa trên của bảng, và quan trọng hơn, mỗi con thuộc một kiểu lỗi khác nhau — M02 là lỗi *kiểm soát truy cập*, M01 là lỗi *tin đầu vào từ người dùng*, M05 là lỗi *lưu trữ dữ liệu*. Vá được ba chỗ này là chạm vào ba miền khác nhau của hệ thống, nên số tiền bỏ ra có tầm phủ rộng nhất thay vì chỉ lo một góc. Tôi cũng so sánh với các mối bỏ lại: M04 có tác động tới 5 nhưng khả năng chỉ 2, vì tôi giả định tài khoản admin mặc định hiếm khi còn sót; M03 và M08 chạm trình duyệt người dùng nhưng điểm thấp hơn nên để sau.

Phần bỏ lại: M03 và M08 coi như được giảm bớt rủi ro sẵn vì khi xử lý M01, M02 tôi sẽ mã hóa đầu ra và bật cờ Secure/HttpOnly cho cookie. M06 (chuỗi cung ứng) và M07 (lộ thông tin) tác động thấp hơn hẳn nên tôi để dành cho đợt cải tiến sau, không đưa vào đợt này.

## Ước lượng chi phí (nguyên lý thứ bảy)

| Mối | Biện pháp | Chi phí dự kiến |
|---|---|---|
| M02 | Viết bộ kiểm tra quyền sở hữu cho mọi truy vấn đơn hàng + kiểm thử | khoảng 8 ngày công |
| M01 | Đổi toàn bộ truy vấn sang tham số hóa + chạy lại kiểm thử hồi quy | khoảng 5 ngày công |
| M05 | Băm mật khẩu, mã hóa dữ liệu thanh toán, di trú dữ liệu cũ | khoảng 10 ngày công |

Tổng cỡ 23 ngày công — rẻ hơn nhiều so với thiệt hại khi một lần rò rỉ toàn bộ dữ liệu khách hàng, nên em thấy đáng đầu tư.

## Kiểm tra

- `docs/threat-model.json` — mô hình theo lược đồ `schema/threat-model.schema.json`.
- `evidence/S1/preflight.txt` — môi trường máy, Docker daemon đang `chay`.
- Chạy `make verify` (hoặc `python -m pytest tests/`) được 14/14 phép kiểm xanh.
