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
| 2 | | | | | |
| 5 | | | | | |
| 7 | | | | | |
| 10 | | | | | |
| 11 | | | | | |

Kết quả cuối: PASS .../12

## Bài 4 - Dời đọc body sang middleware

Vì sao kiểm token trước rồi mới đọc body?

> ...

Breakpoint khi gửi `node gui-request.js json-hong`:

- Dừng ở middleware mới: `file:dòng`
- Không dừng ở controller: `file:dòng`

Kết quả: kiem-tra-5-bug.js PASS .../12, kiem-tra-bai-4.js PASS .../9
