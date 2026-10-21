Review bài của Phúc - người chấm: Hùng Anh

Bài được chấm: clone từ https://github.com/trminhphuc15042002/n_layer (commit 8012e1f), chạy `node server.js` rồi `node kiem-tra-crud.js` ở terminal khác.

1. PASS 20/20 (bản một file: không có bản một file trong repo để so, nhưng 20/20 là tối đa nên không thể ít hơn)

2. File cần 2 câu:
   - `controller.js` cần 3 câu: gửi response (`sendJson`, dòng 5), đọc và parse body (`readBody`, dòng 10), và luật riêng của API products (`list`, `getOne`, `create`, `changeOne`, `removeOne`). Hai việc đầu là dụng cụ HTTP dùng chung, không phải luật riêng của products.
   - Các file còn lại mỗi file một câu: `server.js` (mở port), `router.js` (ghép method + path với controller, bọc try/catch thành 500), `middleware.js` (ghi log), `repository.js` (tìm, tạo, sửa, xoá), `data-source.js` (giữ mảng products trong bộ nhớ).
   - Riêng `repository.js` hơi quá một việc: dòng 6 giữ `nextId` - cấp phát id là việc của nơi lưu dữ liệu, không phải của tầng truy vấn.

3. Controller đụng dữ liệu: không có. `controller.js:3` chỉ `require('./repository')`, trong file không có `fs`, `products.push` hay `products.find`.

4. Repository biết req/res/status: không có. `repository.js` chỉ `require('./data-source')` (dòng 3), nhận và trả object thuần, không thấy thì trả `null`.

5. Dừng: `router.js:9` -> `repository.js:13` -> `data-source.js:9`
   Mình chèn `console.log` vào đúng 3 dòng đó trên một bản copy rồi gửi `GET /products/1`, thứ tự in ra đúng Router trước, Repository sau, DataSource cuối.

Điểm cần sửa (xếp theo mức quan trọng):

- PUT và PATCH không kiểm kiểu, cũng không lọc field (`controller.js:68-82`). `create` kiểm `typeof name === 'string'` và `typeof price === 'number'` (dòng 60) nhưng `changeOne` không kiểm gì, nên API tự mâu thuẫn: cùng một body mà POST trả 400, PATCH trả 200. Chạy thử:
  - `PATCH /products/2 {"price":"abc"}` -> 200 `{"id":2,"name":"Quần jean","price":"abc"}`
  - `PATCH /products/2 {"name":123}` -> 200, name thành số
  - `PUT /products/2 {"name":"X","price":9,"hack":true,"admin":1}` -> 200, cả `hack` và `admin` nằm luôn trong record, vì `repository.js:29` và `:39` spread thẳng `body`. Nên tách một hàm kiểm dùng chung cho cả POST/PUT/PATCH, và ở repository chỉ copy những key cho phép.

- Lấy id bằng `Number()` trần (`controller.js:45`, `:70`, `:85`). `Number` nhận cả hex và ký pháp khoa học nên URL lạ vẫn vào được:
  - `GET /products/0x2` -> 200, trả product id 2
  - `GET /products/1e0` -> 200, trả product id 1
  Chặn trước bằng `/^\d+$/.test(idText)` rồi mới `Number`.

- `data-source.js:9` trả thẳng mảng `products` chứ không trả bản sao, nên trạng thái nội bộ lọt ra ngoài. Chạy thử:
  ```
  const a = repo.findAll();
  a.push({ id: 999, name: 'KHONG QUA REPOSITORY', price: 0 });
  repo.findAll()  ->  đã có phần tử 999
  repo.findAll() === a  ->  true
  ```
  Hệ quả nữa là `save()` đang thành hình thức: `repository.create/replace/update/remove` sửa trực tiếp trên mảng gốc rồi mới gọi `save`, nên bỏ dòng `save` đi thì bộ kiểm tra vẫn 20/20. Hôm nào đổi sang lưu file thì mới lộ ra là thiếu `save`. Sửa: `load()` trả `products.map((p) => ({ ...p }))`.

- `readBody` (`controller.js:10-22`) chỉ nghe `data` và `end`. Client ngắt giữa chừng thì `end` không bao giờ bắn, promise treo vĩnh viễn. Mình mở socket, gửi `Content-Length: 300` rồi cắt dây sau 300ms: server im luôn, 4 giây sau vẫn không trả gì (server không sập, nhưng request đó giữ RAM mãi). Nghe thêm `error` và `aborted`, và nên cắt body ở một mức trần (vd 1 MB) trả 413, không thì một request vài trăm MB là đủ làm sập.

- `limit` âm và `limit` rỗng lọt qua (`controller.js:36-41`). `Number.isNaN` chỉ chặn được `abc`:
  - `?limit=-1` -> 200 nhưng trả thiếu mất phần tử cuối, vì `slice(0, -1)`
  - `?limit=` -> 200 trả mảng rỗng, vì `Number('')` là 0
  - `?limit=2.5` -> 200 trả 2 phần tử
  Nên kiểm số nguyên dương, sai thì 400.

- Phụ thuộc ngược tầng: `sendJson` nằm trong `controller.js:5` nhưng `router.js:27` và `:35` lại gọi `controller.sendJson` để trả 404 và 500. Router phải import Controller chỉ để mượn một hàm HTTP. Nên dời `sendJson` và `readBody` sang `http-utils.js`, cả ba tầng cùng import từ đó.

- `router.js:10` dùng `filter(Boolean)` nên đoạn rỗng bị bỏ qua, URL dị dạng vẫn khớp route: `GET /products//1` và `GET /products/1/` đều trả 200 product 1. Chặn trước nếu path chứa `//` hoặc kết thúc bằng `/`.

- `router.js:35` trả `e.message` thẳng cho client khi 500. Lỗi nội bộ lộ ra ngoài; nên log ở server và trả câu chung chung.

- Router đang là chuỗi `if` lồng nhau (`router.js:13-25`) chứ không phải bảng route. Hiện tại chạy đúng, nhưng bài 4 phải gắn middleware riêng cho POST/PUT/PATCH thì chỗ này sẽ phải viết lại; đổi sang mảng `{ method, path, middlewares, handler }` ngay bây giờ sẽ đỡ hơn.

- Chưa có middleware kiểm token, `middleware.js` mới chỉ có `logRequest`. Phần token của bài 14/10 chưa thấy trong repo.

- Chưa có README, nên người mở repo không biết chạy thế nào và mỗi file làm gì.

Điểm tốt:

- 20/20 PASS, không FAIL bước nào.
- Controller không đụng dữ liệu trực tiếp, đi đúng qua Repository; Repository không biết `req`, `res`, status code.
- `changeOne(action)` (`controller.js:68`) gộp PUT và PATCH bằng một factory, không phải chép hai lần gần giống nhau.
- PUT/PATCH kiểm tồn tại id trước rồi mới đọc body (`controller.js:71`), nên `PUT /products/999` với body hỏng vẫn ra 404 chứ không ra 400. Thứ tự này đúng và dễ làm ngược.
- `nextId` không cấp lại id đã xoá, nên xoá id 3 rồi tạo mới vẫn ra id 4.
- `router.js:30-38` có try/catch, lỗi tầng dưới thành 500 và server vẫn sống.
- Có kèm `postman-collection.json`, tiện cho người chấm gửi thử request.

Nếu là bài của mình, mình sửa chỗ: cho PUT/PATCH dùng chung hàm kiểm với POST và lọc field trước khi ghi, chặn id bằng regex trước khi `Number`, và cho `data-source.load()` trả bản sao thay vì mảng gốc.
