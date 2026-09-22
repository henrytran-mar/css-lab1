# Mô hình đe dọa — Trang bán hàng trực tuyến (Lab S1)

## Hệ thống được chọn

Một trang bán hàng trực tuyến gồm **đúng ba thành phần**:

1. **Trình duyệt** — giao diện người dùng, gửi yêu cầu HTTP tới máy chủ.
2. **Máy chủ ứng dụng** — xử lý logic nghiệp vụ, xác thực phiên, truy vấn và ghi dữ liệu.
3. **Cơ sở dữ liệu** — lưu tài khoản khách hàng, sản phẩm và đơn hàng.

**Luồng dữ liệu:** khách duyệt sản phẩm và đặt hàng trên trình duyệt → yêu cầu được gửi tới máy chủ ứng dụng → máy chủ đọc/ghi cơ sở dữ liệu rồi trả kết quả về trình duyệt.

## Tám mối đe dọa

| Mã | Mối đe dọa (tóm tắt) | Nguyên lý | ATT&CK | Tác động × Khả năng |
|---|---|---|---|---|
| M02 | Đọc đơn hàng khác bằng cách sửa mã đơn hàng (IDOR) | 2 | T1530 | 4 × 4 = **16** |
| M01 | Đọc toàn bộ bảng tài khoản qua SQL injection | 5 | T1190 | 5 × 3 = **15** |
| M05 | Mật khẩu / thẻ thanh toán lưu dạng chữ rõ | 2 | T1552 | 5 × 3 = **15** |
| M03 | Kịch bản độc hại trong đánh giá sản phẩm (XSS) | 3 | T1059.007 | 4 × 3 = 12 |
| M07 | Trang lỗi lộ thông tin cấu hình nội bộ | 1 | T1592 | 3 × 4 = 12 |
| M08 | Cookie phiên thiếu cờ an toàn, dễ bị chiếm | 3 | T1539 | 4 × 3 = 12 |
| M04 | Đăng nhập bằng mật khẩu mặc định tài khoản quản trị | 4 | T1110 | 5 × 2 = 10 |
| M06 | Thư viện bên thứ ba bị nhiễm độc (chuỗi cung ứng) | 6 | T1195 | 3 × 3 = 9 |

## Ba mối được chọn để xử lý

**M02 (IDOR, 16), M01 (SQL injection, 15), M05 (dữ liệu nhạy cảm dạng chữ rõ, 15).**

**Vì sao chọn ba mối này chứ không phải ba mối khác:** cả ba đều nằm ở nửa trên bảng xếp hạng theo tích tác động × khả năng, và mỗi mối đại diện cho một nhóm lỗi riêng mà không trùng nhau — một lỗi *kiểm soát truy cập* (M02), một lỗi *xử lý đầu vào* (M01), và một lỗi *lưu trữ dữ liệu* (M05). Sửa được cả ba là vá được ba 'miền' khác nhau của hệ thống, nên tiền bỏ ra có phạm vi phủ rộng nhất. Ngược lại, M04 có tác động cao (5) nhưng khả năng thấp (2) vì giả định tài khoản quản trị mặc định hiếm còn, và M03/M08 tuy chạm trình duyệt người dùng nhưng điểm thấp hơn nên bị bỏ lại.

Điểm bỏ lại chấp nhận được: M03 (XSS) và M08 (cookie) một phần được giảm rủi ro sẵn như hệ qủa của việc mã hóa đầu ra và bật cờ Secure/HttpOnly khi xử lý M01/M02; M06 (chuỗi cung ứng) và M07 (lộ thông tin) để lại cho một lượt sau vì tác động thấp hơn.

## Ước lượng chi phí xử lý (nguyên lý thứ bảy)

| Mối | Biện pháp | Chi phí |
|---|---|---|
| M02 | Kiểm tra quyền sở hữu ở mọi truy vấn và thêm bộ kiểm thử | ~8 ngày công |
| M01 | Chuyển toàn bộ truy vấn sang tham số hóa + kiểm thử hồi quy | ~5 ngày công |
| M05 | Băm mật khẩu, mã hóa dữ liệu thanh toán, di trú dữ liệu cũ | ~10 ngày công |

Tổng ~23 ngày công, nhỏ hơn nhiều so với giá trị rò rỉ toàn bộ dữ liệu khách hàng trong một lần vỡ, nên khoản đầu tư này đáng bỏ.

## Kiểm tra

- `docs/threat-model.json` — mô hình theo lược đồ `schema/threat-model.schema.json`.
- `evidence/S1/preflight.txt` — môi trường máy (CPU, Python, Docker, daemon `chay`).
- Chạy `make verify` (hoặc `python -m pytest tests/`) cho 14/14 phép kiểm xanh.
