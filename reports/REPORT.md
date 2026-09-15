# Báo cáo Ngày 3 — Tracking Annotation

Họ tên: `Nguyễn Văn Tuyển`
Ngày: `16/09/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `25` phút |
| Thời gian gán `clip_01` | `45` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `4` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `Xe bị che khuất một phần bởi xe khác: Giữ nguyên track ID và tiếp tục vẽ bbox nếu thấy một phần xe.`
2. `Xe ở rìa khung hình chỉ xuất hiện một góc nhỏ: Chỉ bắt đầu tạo track khi thấy rõ một phần diện tích xe theo đúng guideline.`
3. `Xe di chuyển nhanh làm bbox bị trôi giữa các frame: Thêm keyframe trung gian ở các khung hình xe chuyển hướng hoặc thay đổi tốc độ.`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `Bắt được lỗi trùng ID khi hai xe đi sát nhau.`
- Lượt 2: `Phát hiện bbox bị dư vài frame sau khi xe đã ra khỏi khung hình.`
- Lượt 3: `Phát hiện bbox bị trôi (lệch khỏi vật thể) ở các frame giữa hai keyframe.`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `9e85cd00b6ab451ee2287a4a8c111384c1ca0b895fb675d29978c1836711d95c` |
| Thời điểm khóa | `2026-09-15 17:40` |
| Số row / frame / track trước khi mở reference | `618 / 190 / 8` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | `0.802` | `0.783` | `0.823` | `0.883` | `0.949` | `0.893` | `0.870` | `5` | `38` | `0` |
| Sau rework | `0.802` | `0.783` | `0.823` | `0.883` | `0.949` | `0.893` | `0.870` | `5` | `38` | `0` |

Qua cổng (`IDF1 >= 0.80, MOTA >= 0.75, MOTP >= 0.70`): **có**

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa | 80-100 | 6 | Bấm outside đúng frame xe rời khung hình |
| Bbox thừa | 73-78 | 5 | Bấm outside đúng frame xe rời khung hình |
| Bbox trôi | 108 | 6 | Thêm keyframe để cố định lại bbox |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13` |
| weights / hai tracker | `yolo26n.pt / bytetrack.yaml & botsort-reid.yaml` |
| conf / IoU / imgsz / classes | `0.25 / 0.7 / 960 / [2, 5, 7]` |
| device | `0 (GPU)` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | `0.802` | `0.783` | `0.823` | `0.883` | `0.949` | `0.893` | `0.870` | `53` | `8` | `0` |
| ByteTrack control vs gold | `0.709` | `0.649` | `0.776` | `0.846` | `0.875` | `0.749` | `0.823` | `88` | `54` | `2` |
| BoT-SORT + ReID vs gold | `0.763` | `0.711` | `0.820` | `0.872` | `0.900` | `0.792` | `0.860` | `91` | `26` | `2` |
| ReID vs bạn | `0.775` | `0.722` | `0.833` | `0.910` | `0.884` | `0.770` | `0.901` | `79` | `59` | `4` |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

`IDF1 (0.949) cao hơn MOTA (0.893)[cite: 1]. Nếu MOTA cao mà IDF1 thấp, điều đó cho thấy detector tìm vật thể tốt nhưng tracker liên tục làm nhảy ID. MOTA không phạt nặng lỗi ID vì chỉ tính mỗi lần đổi ID (IDSW) là một lỗi đơn lẻ, trong khi IDF1 đo lường sự nhất quán danh tính của toàn bộ track từ đầu đến cuối.`

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

`ReID giúp tăng IDF1 (0.900 so với 0.875) và AssA (0.820 so với 0.776), IDSW giữ nguyên là 2[cite: 1]. Tại chuỗi frame 105-115, ReID nối lại ID chính xác nhờ đặc trưng ngoại hình sau khi xe bị che khuất, giúp giảm FN đáng kể[cite: 1]. Lưu ý đây là so sánh giữa hai hệ thống tracker, không cô lập riêng tác động causal của ReID vì hai thuật toán sử dụng bộ mã nguồn khác nhau[cite: 1].`

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

`ReID giúp DetA tăng từ 0.649 lên 0.711, FN giảm mạnh từ 54 xuống 26[cite: 1]. Tuy nhiên FP vẫn cao (91)[cite: 1]. Lỗi còn lại chủ yếu nằm ở Detector (YOLO) khi nhận nhầm vật thể tĩnh hoặc bỏ sót xe nhỏ, chứ không phải do lỗi Association.`

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

`At frame 106, ReID bị dư 2 bbox (FP)[cite: 1] do nhận nhầm vật thể tĩnh ở mép đường thành xe. Nhãn gán tay của tôi đúng vì đã bỏ qua vật thể này.`

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

`At frame 91, ReID gợi ý 1 bbox mà nhãn tay của tôi bỏ sót[cite: 1]. Khi kiểm tra lại, đó là xe nhỏ đi vào mép khung hình mà tôi đã bỏ qua ở lượt gán đầu.`

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

`Sửa GUIDELINE_MINI.md: Quy định rõ diện tích hiển thị tối thiểu (chẳng hạn >30% xe) mới bắt đầu tạo track khi vào khung hình. Đổi quy trình: Bổ sung 1 lượt rà soát riêng chuyên kiểm tra vùng viền mép ảnh và các đoạn xe bị occlusion ngắn.`

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md`
- [x] `reports/REPORT.md`
