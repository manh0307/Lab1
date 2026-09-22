# Lab S1. Mô hình đe dọa — Hệ thống web bán hàng

**Môi trường để tái lập:** `sudo apt install make python3 python3-pip -y && pip3 install pytest jsonschema --break-system-packages`, sau đó chạy `make preflight` rồi `make verify`.

## Hệ thống được chọn

Website bán hàng trực tuyến quy mô nhỏ, ba thành phần: **trình duyệt khách hàng** (duyệt sản phẩm, đặt hàng, thanh toán) → **máy chủ ứng dụng web** (xử lý logic, xác thực người dùng) → **cơ sở dữ liệu** (lưu sản phẩm, đơn hàng, tài khoản).
Browser ──HTTPS──▶ App server ──SQL──▶ Database
Browser ◀───────── App server ◀─────── Database

Ranh giới tin cậy: App server không tin dữ liệu do Browser gửi lên; Database chỉ tin truy vấn đã qua xác thực từ App server.

## Ba mối đe dọa được chọn xử lý

(Chi tiết đầy đủ 8 mối đe dọa nằm trong `docs/threat-model.json`)

| Mã | Mối đe dọa | Điểm (Tác động × Khả năng) | Chi phí xử lý |
|----|-----------|:---:|---|
| M03 | Đọc chéo đơn hàng giữa khách hàng do thiếu kiểm mã người dùng (IDOR) | 16 | 8 giờ công lập trình viên |
| M01 | Chèn SQL qua ô tìm kiếm sản phẩm | 15 | 12 giờ công lập trình viên |
| M07 | Lộ tệp sao lưu cơ sở dữ liệu trong thư mục công khai | 15 | 4 giờ công vận hành |

Ba mối này có điểm tác động × khả năng cao nhất (16, 15, 15) trong khi chi phí xử lý thấp (4 đến 12 giờ công) so với thiệt hại nếu bị khai thác — toàn bộ dữ liệu khách hàng và đơn hàng bị lộ. Đây là lựa chọn theo tỉ lệ lợi ích trên chi phí cao nhất, không chỉ dựa thuần vào thứ hạng điểm số.

**Rủi ro bị bỏ lại:** năm mối còn lại (M02, M04, M05, M06, M08, điểm 10-12) tạm chưa xử lý trong đợt này vì khả năng khai thác thấp hơn hoặc cần đầu tư hạ tầng lớn hơn (dựng HTTPS, thay thư viện, thêm giới hạn đăng nhập). Đây là đánh đổi chi phí và tỉ lệ có chủ đích, không phải bị bỏ sót, và cần được xử lý ở đợt tiếp theo.

## Bằng chứng đã kiểm

`evidence/S1/preflight.txt` xác nhận môi trường Docker, Python chạy đúng. `make verify` chạy qua 14 phép kiểm tự động (schema, số lượng mối đe dọa, mã ATT&CK, ba mối được chọn, ước lượng chi phí) — đạt **14/14**.