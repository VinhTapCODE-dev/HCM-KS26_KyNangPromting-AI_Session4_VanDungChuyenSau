## Thiết kế INPUT/OUTPUT

**INPUT:**

Danh sách đơn gồm: mã đơn, tên khách, số điện thoại, sản phẩm, số lượng, đơn giá, mã giảm giá.


**Output:**

| Trạm | Việc làm | Kết quả |
|------|----------|---------|
| 1 | Kiểm tra dữ liệu đơn | JSON, mỗi đơn là `HOP_LE` hoặc `LOI` kèm lý do |
| 2 | Tính tiền từng bước | Các bước tính và tổng tiền trong khối `[RESULT: ...]` |
| 3 | Viết tin nhắn xác nhận | Tin nhắn ngắn cho từng khách |

**Quy tắc của cửa hàng**

- Mã `GIAM10` giảm 10% tiền hàng.
- Phí ship là 30.000đ, miễn phí nếu tiền hàng sau giảm từ 500.000đ trở lên.

## Viết Promp và chạy thử

**Dữ liệu thử**

```
DH01 | Nguyễn An | 0901234567 | Tai nghe | SL 2 | 150000 | GIAM10
DH02 | Trần Bình | (trống) | Bàn phím | SL 1 | 400000 | không
DH03 | Lê Chi | 0912345678 | Chuột | SL -1 | 200000 | không
DH04 | Phạm Dũng | 0987654321 | Màn hình | SL 1 | 1200000 | GIAM10
DH05 | Hoàng Em | 0933333333 | Bàn phím | SL 1 | 400000 | SALE50
```

### Trạm 1: Kiểm tra dữ liệu

**Prompt:**

```
Vai trò: nhân viên kiểm tra đơn hàng của cửa hàng đồ công nghệ.
Nhiệm vụ: kiểm tra từng đơn trong danh sách bên dưới.
Điều kiện:
- Nếu số điện thoại trống thì trạng thái là LOI, lý do "thiếu SĐT".
- Nếu số lượng nhỏ hơn hoặc bằng 0 thì trạng thái là LOI, lý do "số lượng không hợp lệ".
- Nếu mã giảm giá không phải GIAM10 hoặc "không" thì ghi chú "mã không hợp lệ".
- Không tự bịa thông tin còn thiếu.
Yêu cầu: chỉ trả về JSON, không viết lời chào hay giải thích.
Mẫu: [{"ma_don":"...","trang_thai":"HOP_LE hoặc LOI","ly_do":"..."}]
Danh sách đơn: """[dán 5 đơn]"""
```

### Trạm 2: Tính tiền 

**Prompt:**

```
Vai trò: nhân viên tính tiền của cửa hàng.
Nhiệm vụ: tính tổng thanh toán cho từng đơn bên dưới.
Hãy suy nghĩ từng bước cho mỗi đơn:
- Bước 1: tiền hàng = số lượng x đơn giá.
- Bước 2: giảm giá. Chỉ mã GIAM10 được giảm 10%, mã khác thì không giảm.
- Bước 3: phí ship 30.000đ, miễn phí nếu tiền hàng sau giảm từ 500.000đ trở lên.
- Bước 4: tổng thanh toán = tiền hàng sau giảm + phí ship.
Kiểm tra lại từng bước. Cuối mỗi đơn ghi kết quả trong khối [RESULT: mã đơn - số tiền] ở dòng cuối.
Đơn cần tính: """[dán DH01, DH04, DH05]"""
```

### Trạm 3: Viết tin nhắn xác nhận

**Prompt:**

```
Vai trò: nhân viên chăm sóc khách hàng của cửa hàng.
Nhiệm vụ: viết tin nhắn xác nhận đơn cho từng khách dựa trên kết quả bên dưới.
Yêu cầu: lịch sự, dưới 40 từ mỗi tin, nêu rõ mã đơn và số tiền, không hứa thêm điều gì ngoài thông tin đã có.
Với đơn có ghi chú "mã không hợp lệ", báo nhẹ nhàng là mã giảm giá chưa áp dụng được.
Dữ liệu: """[dán kết quả Trạm 2 và ghi chú Trạm 1]"""
```