# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Trương Công Hoài Nam`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `60` phút |
| Thời gian gán `clip_01` | `60` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `5` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Tại frame 91, xe con xám di chuyển song song ngay sát cạnh đầu xe buýt , sau đó xe buýt lớn dần che mất tầm nhìn của xe con. Rất dễ bị lỗi nhảy ID, gộp nhầm box vào xe buýt hoặc mất dấu khi xe bị che.`

`Giữ nguyên ID của cả hai xe, khi xe con bắt đầu bị che một phần thân thì bật thuộc tính Occluded. Dựa vào vận tốc và quỹ đạo thẳng trước đó của xe con để dự đoán và đặt hộp bao nội suy bám sát vị trí thực tế.`
2. `Góc máy từ trên cao nhìn chéo khiến đầu xe con phía sau đè lên đuôi xe buýt, dễ bị nhầm xe sau là một phần của xe buýt.`

`Phóng to khu vực tiếp giáp, căn chỉnh cạnh đuôi của bounding box xe buýt khít với đèn hậu; tạo một track riêng biệt cho xe con phía sau, bật thuộc tính Occluded`
3. `Gian hàng ven đường có dạng khối hộp, kích thước, cửa kính và mái che rất giống một xe buýt mini hoặc xe bán hàng lưu động`

`Bỏ qua hoàn toàn, không tạo Bounding Box hay Track ID cho gian hàng này, phóng to chi tiết để quan sát phần móng tiếp đất để phân biệt với xe cơ giới`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Cả bài không bị nhảy ID lần nào`
- Lượt 2: `Bắt đúng bệnh quên ngắt box khi xe chạy mất. Vẽ box quá vội khi xe mới xén vào`
- Lượt 3: `Phát hiện box bị trôi và lệch khỏi thân xe do lười đặt keyframe, thả cho máy tự nội suy quá dài. Khi xe hơi đổi hướng hoặc đổi tốc độ là box lệch ngay`

Kiểm chéo với: `...`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `...`. Số lỗi bạn ấy tìm được trong bản của bạn: `...`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`...`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `...` |
| Thời điểm khóa | `...` |
| Số row / frame / track trước khi mở reference | `...` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | | | | | | | | | | |
| Sau rework | | | | | | | | | | |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có / chưa**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| | | | |
| | | | |
| | | | |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `  "python": "3.13.15", "ultralytics": "8.4.145", "torch": "2.11.0+cpu", "lap": "0.5.13"` |
| weights / hai tracker | ` "weights": "yolo26n.pt", "tracker": "bytetrack.yaml" và tracker": "botsort-reid.yaml"` |
| conf / IoU / imgsz / classes | ` "conf": 0.25, "iou": 0.7, "imgsz": 960, "classes": [2,5,7],` |
| device | `gpu` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | | | | | | | | | | |
| ByteTrack control vs gold | | | | | | | | | | |
| BoT-SORT + ReID vs gold | | | | | | | | | | |
| ReID vs bạn | | | | | | | | | | |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`...`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`...`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`...`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`...`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`...`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`...`

## 7. Tệp đã nộp

- [ ] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [ ] `outputs/eval_vs_gold.json`
- [ ] `outputs/model_bytetrack_clip_01.txt`
- [ ] `outputs/model_reid_clip_01.txt`
- [ ] `outputs/model_run_config.json`
- [ ] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [ ] `reports/REPORT.md` (file này)
