# Ghi chép bài tập tối 21/10 - Tách tầng

# Bài 2 - Đoạn code này ở tầng nào

| Đoạn | Tầng | Vì sao (1 câu) | Cặp bạn chọn | Đáp án |
|---|---|---|---|---|
| 1 | Helper/utils | Hàm nhỏ chỉ định dạng giá tiền, không đụng req, res. | Helper/utils | Helper/utils |
| 2 | Middleware | Đụng req, res để chặn request không phải JSON, dùng chung cho nhiều route. | Middleware | Middleware |
| 3 | Middleware | Đọc IP từ req, trả 429 qua res, giới hạn request cho mọi route. | Middleware | Middleware |
| 4 | Controller | Kiểm tra tồn kho theo req.body rồi trả lỗi, là luật riêng của API đặt hàng. | Controller | Controller |
| 5 | Service | Gọi ra hệ thống bên ngoài (API tỷ giá) bằng fetch. | Service | Service |
| 6 | Repository | Tìm sản phẩm theo giá, không đụng req, res, lấy dữ liệu qua dataSource. | Repository | Repository |
| 7 | DataSource | Biết dữ liệu nằm ở file nào (ORDERS_FILE) và đọc file đó. | DataSource | DataSource |
| 8 | Router | Ghép method GET + URL /orders/:id với handler (và middleware). | Router | Router |
| 9 | Helper/utils | Hàm nhỏ kiểm tra số nguyên dương, không đụng req, res. | Helper/utils | Helper/utils |
| 10 | Service | Gọi dịch vụ gửi email bên ngoài bằng fetch. | Service | Service |
| 11 | Controller | Tính tổng tiền và giảm 10% khi trên 500000 là luật riêng của API đơn hàng. | Controller | Controller hoặc Helper/utils |
| 12 | Middleware | Đụng req để ghi log, dùng chung cho mọi route, trả true cho đi tiếp. | Middleware | Middleware |
| 13 | Middleware | Đụng req để đọc và parse body, dùng chung cho nhiều route. | Middleware | Middleware |

## Sau khi đối chiếu (điền sau bước 2 và 3)
Đoạn xếp sai và lý do lúc đầu chọn nhầm:

Tất cả 13 đoạn em xếp đúng ngay từ đầu, không có đoạn nào sai.

- Đối chiếu với cặp Thịnh - Thắng: 13/13 đoạn hai bên chọn giống nhau, không có đoạn nào khác để đọc lý do.
- Đoạn 11: em chọn Controller, đáp án nhận cả Controller và Helper/utils (nếu tách thành hàm calcTotal(order) không đụng req, res thì là helper), nên em vẫn tính là đúng.
- Các đoạn dễ nhầm theo đáp án là 2, 3, 5, 10, 12, 13: em xếp đúng vì hàm đụng req, res và dùng chung nhiều route thì là Middleware, còn hàm gọi API bên ngoài thì là Service.

## Bài 3 - Săn 5 bug

Cột "Em đoán tầng" ghi trước khi mở code.

| Bước FAIL | Em đoán tầng | Tầng thật | file:dòng | Dòng cũ -> dòng mới | Breakpoint thấy gì |
|---|---|---|---|---|---|
| 2 | Repository | Controller | `server/controllers/product-controller.js:16` | `findById(params.id)` -> `findById(Number(params.id))` | `params.id` là string `"1"` (lấy từ URL), còn `p.id` trong file là number `1`, nên `p.id === id` ở `product-repository.js:10` luôn false. Repository không có lỗi, lỗi ở chỗ lấy id từ URL, việc của Controller |
| 5 | Middleware | Middleware | `server/middlewares/require-token.js:6` | `if (!req.headers.authorization)` -> `if (req.headers.authorization !== \`Bearer ${DEMO_TOKEN}\`)` | Điều kiện chỉ kiểm có header hay không, token sai vẫn có header nên đi tiếp vào controller và trả 201. `DEMO_TOKEN` khai báo ở dòng 3 nhưng không được dùng |
| 7 | DataSource | DataSource | `server/data-sources/file-data-source.js:11` | `JSON.stringify(data.products, null, 2)` -> `JSON.stringify(data, null, 2)` | `load()` trả `{ products, nextId }` nhưng `save()` chỉ ghi mảng `products` xuống file, nên lần đọc sau `const { products } = ...` ra `undefined` và list trả body rỗng |
| 10 | Middleware | Router | `server/router.js:12` | `middlewares: []` -> `middlewares: [requireToken]` | Route DELETE khai báo danh sách middleware rỗng, trong khi POST, PUT, PATCH đều có `[requireToken]`. Vòng lặp middleware ở `router.js:41` không có gì để chạy nên request đi thẳng vào `remove`. Không dừng ở `require-token.js:6` |
| 11 | Repository | Repository | `server/repositories/product-repository.js:43` | `splice(index)` -> `splice(index, 1)` | `splice(index)` xoá từ vị trí `index` tới hết mảng. Xoá product 2 (index 1) làm `data.products` còn 1 phần tử thay vì 2, mất cả product 3 |

Kết quả cuối: PASS 12/12 (`node kiem-tra-5-bug.js` in `PASS 12, FAIL 0`)

Ghi chú trung thực:

- Cột "Em đoán tầng" dựa trên status và body của script, ghi trước khi mở code. Em có dùng AI hỗ trợ bài này.
- Cột "Breakpoint thấy gì" được đối chiếu từ code và output của script. Em đã đọc và hiểu từng dòng sửa.
- Mỗi bug sửa đúng 1 dòng, ở 5 file khác nhau, mỗi bug một commit (xem `git log`).

## Bài 4 - Dời đọc body sang middleware

Vì sao kiểm token trước rồi mới đọc body?

> ...

Breakpoint khi gửi `node gui-request.js json-hong`:

- Dừng ở middleware mới: `file:dòng`
- Không dừng ở controller: `file:dòng`

Kết quả: kiem-tra-5-bug.js PASS .../12, kiem-tra-bai-4.js PASS .../9
