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

Kiểm chéo với: `SOLO`. Chi tiết ở `reports/review_partner.md`.
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
| bạn vs gold |0.800| 0.783| 0.818| 0.881| 0.945| 0.885| 0.871| 57| 9| 0|
| ByteTrack control vs gold | 0.709| 0.649| 0.776| 0.846| 0.875| 0.749| 0.823| 88| 54| 2|
| BoT-SORT + ReID vs gold | 0.763| 0.711| 0.820| 0.872| 0.900| 0.792| 0.860| 91| 26| 2|
| ReID vs bạn | 0.814| 0.758| 0.874| 0.946| 0.883| 0.768| 0.942| 79| 62| 3|

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`MOTA của tôi thấp hơn IDF1.`

`Nếu MOTA cao mà IDF1 thấp có ý nghĩa là khả năng phát hiện rất tốt nhưng khả năng duy trì ID rất kém `

`MOTA không phạt nặng lỗi ID vì: Lý do nằm ở bản chất công thức tính cục bộ theo frame của MOTA, lỗi ID switch chỉ bị phạt đúng 1 lần tại frame xảy ra cú nhảy. Sự chênh lệch với IDF1: IDF1 đo lường danh tính toàn cục (Global Trajectory) thông qua bài toán ghép cặp tối ưu giữa track dự đoán và track chuẩn`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`BoT-SORT + ReID có cùng 2 lần nhảy ID (IDSW = 2) nhưng đạt IDF1 (0.900 so với 0.875) và AssA (0.820 so với 0.776) cao hơn ByteTrack nhờ duy trì liền mạch quỹ đạo xe buýt xuyên suốt hành trình thay vì bị xé đôi track, tuy nhiên kết quả này phản ánh toàn bộ cải tiến kiến trúc giữa hai tracker chứ không cô lập riêng tác động nhân quả của ReID.`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`DetA tăng từ 0.649 lên 0.711 nhờ FN giảm mạnh từ 54 xuống 26 trong khi FP gần như giữ nguyên (88 lên 91), cho thấy lỗi còn lại chủ yếu thuộc về detector do liên tục phát hiện nhầm các mảng cảnh nền tĩnh thành phương tiện.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`Tại frame 87 với xe con chạy sát xe buýt (track gold 5), bạn đúng vì dùng mắt người bám sát hướng di chuyển để giữ trọn vẹn ID 5, còn ReID sai do đặc trưng ngoại hình bị lẫn vào xe buýt dẫn đến nhảy từ ID 17 sang 18.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`Ở các frame 79–100 với xe con phía sau (track ID 6), bạn cần xem lại nhãn vì đã vẽ đón đầu quá sớm khi xe chưa lộ đủ 30% diện tích thân xe, trong khi cả ground truth lẫn mô hình ReID đều chỉ bắt đầu tính khi xe vào đủ rõ từ khoảng frame 100 trở đi.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Trong GUIDELINE_MINI.md, tôi nên chuẩn hóa cứng ngưỡng vào, ra khung hình (chỉ gán khi xe lộ trên ba mươi phần trăm và bấm Outside ngay khi ra rìa), xử lý rõ quy tắc che khuất nặng cùng danh mục loại trừ vật thể tĩnh; đồng thời trong quy trình làm việc, nên áp dụng mô hình tự động gán nhãn sơ bộ kết hợp quy tắc đặt keyframe tối đa mỗi mười frame và kiểm tra rà soát ba lượt trước khi xuất dữ liệu.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
