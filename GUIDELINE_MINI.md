# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `SOLO / Trương Công Hoài Nam`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `...`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới ... frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `...` |
| Xe bị che lâu hơn ngưỡng trên | `...` | `...` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `...` |
| Hai xe cắt nhau / chồng lên nhau | `...` | `...` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `10%` |
| Xe đang đỗ, không di chuyển | `vẽ Bbox 1 lần rồi chạy theo frame` |
| Keyframe đặt dày ở đâu | `ở các ngã rẽ` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `Clip1 / frame 80 / vehicle 5`
- Tình huống: `Xe con bị khuất một phần lớn thân sau cản đuôi của xe buýt`
- Quyết định: `Tiếp tục giữ nguyên ID và vẽ bounding box ôm sát phần thân xe còn nhìn thấy được, không ngắt track và không bọc lấn sang đuôi xe buýt.`
- Lý do: `Mặc dù bị che khuất đáng kể, phần đầu và nóc xe vẫn hiển thị rõ ràng trên 20% diện tích với quỹ đạo di chuyển tịnh tiến liên tục`

### Ca 2
- Clip / frame / ID: `Clip1 / frame 70 / vehicle 8`
- Tình huống: `Xe con ở làn xa phía sau đầu xe buýt mới chỉ nhú một góc nhỏ mờ ở mép khung hình`
- Quyết định: `Không tạo track mới tại frame này`
- Lý do: `vật thể mới lộ dưới 10% diện tích và chưa rõ hình dạng phương tiện`

### Ca 3
- Clip / frame / ID: `Clip1 / frame 1 / không id`
- Tình huống: `Khối kiến trúc dạng ki-ốt có mái che hình hộp cố định ven đường dễ nhầm với xe tải nhỏ, xe buýt nhỏ đang đỗ`
- Quyết định: `Bỏ qua hoàn toàn, không gán bounding box hay tạo track ID 8 cho vật thể này.`
- Lý do: `Đây là kiến trúc cảnh nền tĩnh không tham gia giao thông`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `...`
- `...`
