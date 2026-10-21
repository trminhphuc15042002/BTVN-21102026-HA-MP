Review bài của Hùng Anh - người chấm: Phúc

Bài được chấm: `E:\SET-2026\n_layer_HA\n_layer` (chạy `PORT=4000 node server.js`, rồi `node kiem-tra-crud.js http://localhost:4000`).

1. PASS 20/20 (bản một file: chưa có bản một file của Hùng Anh để so sánh, nhưng 20/20 là tối đa nên không thể ít hơn)

2. File cần 2 câu:
   - `router.js` cần tới 4 câu: ghép route (`khopPath`, `createRouter`), gửi response (`sendResponse`, dòng 11), đọc và parse body (`readBody`, `readJson`, dòng 26 và 53), và định nghĩa `HttpError` kèm bắt lỗi chung (dòng 3 và 128). Đây là file làm nhiều việc nhất.
   - `server.js` cần 2 câu: dựng bảng route (dòng 12) và khởi động server, kèm cờ `KIEM_TOKEN` (dòng 8, 22).
   - Các file còn lại mỗi file một câu: `middleware.js` (log và kiểm token), `controller.js` (kiểm input rồi chọn status), `repository.js` (tìm, tạo, sửa, xoá), `data-source.js` (load/save/nextId trong bộ nhớ).

3. Controller đụng dữ liệu: không có. `controller.js` chỉ `require('./repository')` (dòng 1) và `require('./router')` (dòng 2), không có `fs`, `.push`, hay mảng dữ liệu riêng. Ba chỗ `.find`/`.push` mà đề bảo tìm đều không xuất hiện ở controller.

4. Repository biết req/res/status: không có. `repository.js` chỉ `require('./data-source')` (dòng 1), nhận và trả object thuần, trả `null` khi không thấy.

5. Dừng: `server.js:28` -> `router.js:91` (`xuLy`) -> `router.js:120` (gọi handler) -> `controller.js:50` (`repo.findById`) -> `repository.js:19` -> `data-source.js:20` (`load`)
   Thứ tự này em đọc từ code, chưa đặt breakpoint thật. Thứ tự đúng: Router trước, Repository sau.

Điểm cần sửa (xếp theo mức quan trọng):

- Phụ thuộc ngược tầng: `middleware.js:1` và `controller.js:2` đều `require('./router')` chỉ để lấy `sendResponse` và `readJson`. Tầng dưới đang phải biết tầng trên. Nên dời `sendResponse`, `readBody`, `readJson`, `HttpError` sang một file `http-utils.js` riêng, router, middleware và controller cùng import từ đó.
- `router.js` làm quá nhiều việc (xem mục 2). Sau khi dời `http-utils.js` như trên thì router chỉ còn ghép route.
- Controller tự gọi `readJson(req)` ở `controller.js:59, 83, 106`. Việc đọc body là việc chung của mọi route ghi, nên thuộc Middleware (đúng nội dung bài 4).
- `file-data-source.js` không file nào `require`, nên đang là code chết: `repository.js:1` đang dùng `data-source.js` lưu trong bộ nhớ, tắt server là mất dữ liệu. Nếu muốn lưu file thì đổi `repository.js:1` sang `file-data-source`; nếu không cần thì xoá file.
- `server.js` ngoài việc khởi động còn khai báo bảng route. Nên tách bảng route sang `routes.js` hoặc đưa vào router.
- File nằm phẳng trong một folder, chưa chia thư mục `controllers/`, `repositories/`, `middlewares/`... như bài mẫu. Không sai chức năng, nhưng khó mở rộng khi có thêm resource.
- `README.md` chỉ có một dòng "SET2026", nên thêm cách chạy và mô tả từng tầng.

Điểm tốt:

- Controller không đụng dữ liệu trực tiếp, đi đúng qua Repository.
- Repository sạch, không biết HTTP, và `locField` lọc field cho phép (`repository.js:3-13`) nên body gửi kèm `id` không ghi đè được id.
- `data-source.js` trả bản sao (`{ ...r }`), nên không lộ tham chiếu nội bộ ra ngoài.
- Có kiểm input đầy đủ (`name`, `price`, `limit`, id không hợp lệ) và xử lý body quá lớn (413).

Nếu là bài của em, em sửa chỗ: chuyển `sendResponse`/`readJson`/`HttpError` sang `http-utils.js`, dời `readJson` ra khỏi controller sang middleware, và nối `file-data-source.js` vào `repository.js` (hoặc xoá).
