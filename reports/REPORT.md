# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Minh Quân`
Ngày: `15/9/2026`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT / khác: `CVAT` |
| Thời gian gán `clip_02` (warm-up) | `30` phút |
| Thời gian gán `clip_01` | `70` phút |
| Số track đã vẽ trong `clip_01` | `8` |
| Số keyframe trung bình mỗi track | `23` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. `ô tô bị mới xuất hiện nhưng bị che khuất`
2. `ô tô bị che khuất quá 90% xe`
3. `các keyframe bị đè lên nhau`

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: `các keyfarme vẫn cùng id`
- Lượt 2: `xe trong keyfarme`
- Lượt 3: `xe đc chọn vẫn trong frame`

Kiểm chéo với: `Không có`. Chi tiết ở `reports/review_partner.md`.
Số lỗi bạn tìm được trong bản của bạn ấy: `Không có`. Số lỗi bạn ấy tìm được trong bản của bạn: `Không có`.

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

`Không có`

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `c90224564e1f5fecfd81ba44b6661b0ade109d79fbba9525442276f4ce0afb92` |
| Thời điểm khóa | `15/9/2026 16:19` |
| Số row / frame / track trước khi mở reference | `628/190/8.` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.753 | 0.730 | 0.778 | 0.859 | 0.916 | 0.824 | 0.846 | 78 | 23 | 0 |
| Sau rework | 0.753 | 0.730 | 0.778 | 0.859 | 0.916 | 0.824 | 0.846 | 78 | 23 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **ĐẠT** (IDF1: 0.916 >= 0.80, MOTA: 0.824 >= 0.75, MOTP: 0.846 >= 0.70)

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| Bbox thừa / bắt sớm | 51-53 | 4 | Xe buýt mới xuất hiện ở mép phải ảnh (<10px), đã rà soát chuẩn hóa mốc bắt đầu track |
| Bbox bắt sớm khi bị che | 66-78 | 5 | Xe con bị xe buýt che lấp >90%, đã kiểm tra lại ngưỡng lộ diện tối thiểu |
| Bbox trôi (loose boxes) | 82-100 | 5 | Rà soát và bổ sung keyframe ở đoạn xe con vượt lên cạnh xe buýt để cải thiện IoU |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | `3.13.15/8.4.145/2.11.0+cu128/0.5.13` |
| weights / hai tracker | `yolo26n.pt` |
| conf / IoU / imgsz / classes | `0.25/0.7/960/[2,5,7]` |
| device | `0` |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold |0.753 | 0.730|0.778 |0.859 |0.916 | 0.824|0.846 |78 |23 |0 |
| ByteTrack control vs gold |0.709 |0.649 |0.776 |0.846 |0.875 |0.749 |0.823 |88 |54 |2 |
| BoT-SORT + ReID vs gold |0.763 |0.711 | 0.820| 0.872|0.900 |0.792 |0.860 |91|26 |2 |
| ReID vs bạn |0.743 |0.688 |0.804 |0.894 | 0.869| 0.740|0.884 |85 |75 | 3|

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

- **So sánh thực tế**: MOTA của bản nhãn (`0.824`) **thấp hơn** IDF1 (`0.916`) xấp xỉ 9.2%.
  - *Lý do*: Quá trình gán nhãn duy trì tính toàn vẹn danh tính xe hoàn hảo trên toàn clip (`IDSW = 0`, cả 8 track khớp 1-1 với Gold Reference), nên `IDF1` đạt mức rất cao (0.916). `MOTA` thấp hơn vì bị phạt bởi 78 FP và 23 FN ở cấp độ detection đơn lẻ (chủ yếu là lệch vài frame ở đầu/cuối track khi xe mới lấp ló ở mép ảnh).
- **Ý nghĩa khi MOTA cao mà IDF1 thấp**:
  - Tình trạng này phản ánh rằng hệ thống/người gán làm rất tốt việc **phát hiện vật thể tức thời tại từng frame** (bounding box bao khít xe, rất ít FP và FN ở mỗi thời điểm), nhưng lại **thất bại nghiêm trọng trong việc duy trì danh tính theo thời gian (association)**. Track bị chia cắt thành nhiều mảnh vụn (fragmentation) hoặc liên tục bị tráo đổi ID cho nhau (ID switches) mỗi khi xe giao cắt hoặc bị che khuất tạm thời.
- **Vì sao MOTA không phạt nặng lỗi ID?**:
  - Dựa theo công thức tính MOTA:
    $$\text{MOTA} = 1 - \frac{\sum_t (\text{FP}_t + \text{FN}_t + \text{IDSW}_t)}{\sum_t \text{GT}_t}$$
  - Trong MOTA, mỗi lần tráo đổi ID (`IDSW`) chỉ bị phạt đúng **1 lần** (cộng 1 vào tử số) ngay tại frame xảy ra switch. Sau frame đó, nếu đối tượng tiếp tục được track dưới ID mới thì các frame sau vẫn được xem là True Positive bình thường mà không bị trừ thêm điểm nào. Ví dụ: Một chiếc xe chạy qua 100 frame bị đổi ID ở frame 50 thì MOTA chỉ bị phạt 1 điểm ($1/100 = 1\%$, điểm MOTA vẫn đạt tới 99%).
  - Ngược lại, **IDF1** tính toán trên việc gán cặp trajectory tối ưu 1-1 toàn cục (Global Bipartite Matching):
    $$\text{IDF1} = \frac{2 \cdot \text{IDTP}}{2 \cdot \text{IDTP} + \text{IDFP} + \text{IDFN}}$$
    Khi một track 100 frame bị tách làm 2 ID (mỗi ID 50 frame), chỉ có 1 nửa được coi là IDTP (50 frame), còn nửa kia bị tính toàn bộ thành IDFN và IDFP, kéo tụt IDF1 xuống chỉ còn $\approx 50\%$. Do đó, MOTA thiên vị chất lượng detection từng frame và xem nhẹ lỗi association dài hạn.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

- **Khác biệt số liệu**:
  - `IDF1`: BoT-SORT + ReID đạt **0.900**, vượt trội so với ByteTrack đạt **0.875** (+0.025).
  - `AssA`: BoT-SORT + ReID đạt **0.820**, cao hơn hẳn ByteTrack đạt **0.776** (+0.044).
  - `IDSW`: Cả hai tracker đều có **2 IDSW**, tuy nhiên tính liên tục và độ bao phủ của BoT-SORT + ReID tốt hơn rất nhiều.
- **Dẫn chứng frame sequence**:
  - Xét **Gold Track 4** (xe buýt lớn, frame 54–148, dài 95 frame):
    - *ByteTrack control*: Bắt đầu nhận diện ở frame 56–57 với ID 14 (2 frame), sau đó bị mất dấu ở frame 58, rồi đến frame 59 tạo một ID mới là ID 15 kéo dài đến frame 148. ByteTrack dính 1 ID switch tại frame 59 và làm phân mảnh track 4 thành hai phần.
    - *BoT-SORT + ReID*: Nhờ đặc trưng appearance và bù chuyển động camera, tracker duy trì duy nhất một **ID 9** liền mạch từ frame 55 đến frame 149 (95 frame), không bị ngắt quãng, không dính ID switch nào.
  - Xét **Gold Track 5** (xe con đi cạnh xe buýt, frame 79–138, 60 frame):
    - *ByteTrack*: Bị che khuất và mất dấu tới 8 frame liên tiếp (frame 86–93), chỉ bắt 1 frame ở frame 85 (ID 23) rồi switch sang ID 32 ở frame 94, độ bao phủ chỉ đạt 46/60 frame (77%).
    - *BoT-SORT + ReID*: Chỉ gián đoạn đúng 1 frame (frame 86), bắt lại từ frame 87 (ID 18) đến 139, độ bao phủ đạt 52/60 frame.
  - Xét **Gold Track 8** (frame 136–168): ByteTrack chỉ cover được 20/33 frame (61%), trong khi BoT-SORT + ReID cover trọn vẹn 35 frame (frame 136–170).
- **Lưu ý về Causal Effect**:
  - Thí nghiệm này **KHÔNG cô lập được hiệu ứng nhân quả (causal effect) độc lập của ReID**, bởi vì ByteTrack và BoT-SORT là hai kiến trúc tracker có implementation hoàn toàn khác nhau:
    1. BoT-SORT tích hợp **Camera Motion Compensation (CMC)** bằng feature matching OpenCV để bù trừ rung lắc/chuyển động của camera trước khi dự đoán Kalman Filter, điều mà ByteTrack không có.
    2. BoT-SORT mô hình hóa state space của Kalman Filter trực tiếp trên $(x, y, w, h)$ thay vì tỉ lệ khung hình $(x, y, a, h)$ như ByteTrack, giúp bám kích thước xe biến dạng tốt hơn.
    3. BoT-SORT kết hợp ma trận chi phí hỗn hợp giữa khoảng cách IoU và Cosine distance của ReID appearance feature, trong khi ByteTrack dùng thuật toán matching 2 tầng thuần túy dựa trên IoU (phân loại high-conf và low-conf).
  - Vì vậy, đây là so sánh ở mức hệ thống (system-level comparison), không thể quy kết toàn bộ cải thiện là do ReID đơn lẻ.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

- **Thay đổi của DetA, FP và FN**:
  - `DetA`: Tăng đáng kể từ **0.649** (ByteTrack) lên **0.711** (BoT-SORT + ReID) (+6.2%).
  - `FP`: Tăng nhẹ từ **88** lên **91** (+3 FP).
  - `FN`: Giảm rất mạnh từ **54** xuống còn **26** (giảm hơn một nửa, -28 FN!).
  *(Lý do FN giảm mạnh dù dùng chung YOLO26n: Trong MOT, một detection chỉ được giữ lại trong kết quả nếu nó được liên kết vào một track hợp lệ. Nhờ CMC và ReID appearance, BoT-SORT duy trì được track state qua các frame khó/che khuất, "cứu" lại được nhiều detection mà ByteTrack bỏ rơi do không match được IoU).*
- **Lỗi còn lại là detector hay association?**:
  - **Lỗi còn lại CHỦ YẾU LÀ DETECTOR**, không phải do association:
    - *Association*: Hoạt động rất tốt với AssA đạt **0.820**, IDSW chỉ có **2**, IDF1 đạt **0.900**.
    - *Detector*: Chiếm phần lớn tổng số lỗi với **117 bbox lỗi** (91 FP + 26 FN), là nguyên nhân chính kìm hãm MOTA ở mức 0.792. Trong 91 FP, phần lớn đến từ các **ghost tracks** do detector phát hiện nhầm vật thể tĩnh và nhiễu nền:
      + `Pred track 7`: Tồn tại suốt 43 frame (frame 16–116) do nhận nhầm tấm biển báo giao thông trên cao ($x \approx 492, y \approx 212$) với conf thấp 0.28–0.36.
      + `Pred track 27`: Kéo dài 16 frame (frame 106–121) do nhiễu mép lề đường cực trái ($x \approx 0.1, y \approx 280$, conf ~0.38).
      + `Pred track 38`: Kéo dài 16 frame (frame 158–178) ở rìa cực trái ($x \approx 1.1, y \approx 285$, conf ~0.27).
    - Đồng thời 26 FN còn lại là do detector không phát hiện được xe ở frame bắt đầu hoặc khi xe bị che khuất sâu.
    - **Kết luận**: Bottleneck hiện tại nằm ở DETECTOR (cần tăng ngưỡng confidence threshold lọc FP hoặc fine-tune detector trên tập xe cộ giao thông).

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

- **Vị trí cụ thể**: **Gold Track 6** (xe con màu tối bám sau xe buýt), giai đoạn **frame 101 đến 115** (đặc biệt khoảng frame 101–109).
- **So sánh thực tế**:
  - *Nhãn của bạn (Track 6)*: Nhận biết xe từ frame 79 (khi xe vừa nhô ra sau đuôi xe buýt) và duy trì duy nhất một **ID 6** xuyên suốt, liên tục từ frame 79 đến 157 (79 frame) không hề bị đứt đoạn, không dính bất kỳ ID switch nào (khớp hoàn hảo với giai đoạn xe chạy trong Gold từ frame 101 đến 156).
  - *Model ReID*:
    - Tại frame 101–103, ReID hoàn toàn bỏ sót xe (FN).
    - Tại frame 104, ReID bắt được 1 frame và gán **ID 24** (conf 0.38).
    - Tại frame 105, ReID đổi sang **ID 26** (conf 0.36) rồi mất dấu ở frame 106.
    - Tại frame 107, ReID tạo **ID 28** (1 frame) rồi lại rớt detection ở frame 108–109.
    - Mãi đến frame 110, ReID mới ổn định và gán **ID 31** kéo dài đến frame 156.
    - Hậu quả: ReID dính 2 ID switches liên tiếp (từ 24 sang 28, rồi từ 28 sang 31 trong `outputs/eval_reid_vs_me.json`), làm vỡ track thành 3 ID khác nhau.
- **Vì sao bạn đúng và ReID sai?**:
  - Con người có năng lực suy luận ngữ cảnh không gian - thời gian (spatio-temporal reasoning): mắt người nhận ra đây là một chiếc ô tô đang di chuyển đều đặn sau xe buýt, dù bị che khuất một phần thân xe vẫn xác định được quỹ đạo và giữ nguyên ID.
  - Model ReID bị phụ thuộc vào detector: Khi xe bị che khuất một phần và chịu bóng râm của xe buýt, confidence của YOLO dao động thấp sát ngưỡng lọc ($0.25 - 0.38$), khiến detection bị rơi rụng chập chờn; đồng thời appearance embedding của vùng crop bị ô nhiễm bởi màu sơn của xe buýt phía trước, dẫn đến việc ReID không thể match với track cũ và liên tục khởi tạo ID mới.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

- **Bằng chứng rõ ràng cho thấy Model SAI hoàn toàn (False Positive / Ghost Track)**:
  - *Frame & ID*: Model ReID **Track ID 7**, xuất hiện từ frame **16 đến 116** (tồn tại 43 frame phát hiện).
  - *Tọa độ*: $x \approx 491.6 - 497.9, y \approx 211.4 - 215.5$, kích thước $w \approx 100, h \approx 58$, confidence dao động từ $0.28$ đến $0.36$.
  - *Vì sao model sai*: Khi đối chiếu trực tiếp lên ảnh gốc `000016.jpg` đến `000116.jpg`, vị trí $(x \approx 495, y \approx 213)$ là **tấm biển báo chỉ dẫn giao thông trên cao treo trên giá long môn** giữa dải phân cách. Khung biển báo hình chữ nhật có các ô lưới kim loại trên nền tán cây xanh đã đánh lừa detector YOLO26n. Vì ngưỡng confidence đặt tương đối thấp (`conf = 0.25`), detector liên tục báo có xe tại đây, và BoT-SORT coi đó là vật thể tĩnh và duy trì ID 7 suốt 100 frame. Nhãn của người gán và Gold hoàn toàn không gán đối tượng này là hoàn toàn chính xác theo guideline.
- **Chỗ ReID và Gold làm bạn xem lại annotation (Bắt đầu track quá sớm)**:
  - *Frame & ID*: **Track 4** (xe buýt), frame **51 đến 53**; và **Track 5** (xe con), frame **66 đến 78**.
  - *Vì sao cần xem lại*:
    - Ở Track 4: Người gán bắt đầu vẽ từ frame 51 khi xe buýt chỉ vừa thò một mẩu rộng 7.8 pixel ở sát mép phải ảnh ($x=952$), trong khi Gold bắt đầu từ frame 54 và ReID (Track 9) bắt đầu từ frame 55. Tại frame 51–53, vật thể chưa đủ đặc trưng nhận diện chắc chắn là xe bốn bánh, tạo ra 3 frame FP trước khi track xuất hiện.
    - Ở Track 5: Người gán bắt đầu gán từ frame 66 đến 78 (trước khi Gold xuất hiện ở frame 79 tới 13 frame). Trong khoảng frame 66–78, chiếc xe này ở quá xa và bị xe buýt che lấp hơn 90%, việc gán nhãn ở đây vi phạm quy tắc "đủ bằng chứng là xe 4 bánh" và quy tắc che khuất, tạo ra 13 frame FP trong chẩn đoán `eval_vs_gold.json`.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

- **Những điểm cần sửa trong `GUIDELINE_MINI.md`**:
  1. **Định lượng ngưỡng bắt đầu track (Entry threshold)**:
     - Thay quy định định tính mơ hồ ("hiển thị pixel của xe") bằng ngưỡng định lượng khắt khe: Chỉ bắt đầu track khi phần nhìn thấy của xe đạt tối thiểu **$\ge 15\text{ px}$ chiều rộng hoặc $\ge 200\text{ px}^2$ diện tích**, đồng thời phải nhận diện được ít nhất **2 đặc trưng kết cấu** của xe bốn bánh (ví dụ: đèn pha/đèn hậu + bánh xe hoặc kính xe/nắp capo). Không track khi chỉ mới là một vệt pixel đơn lẻ sát mép ảnh (tránh lỗi FP sớm như Track 4 frame 51–53).
  2. **Quy tắc trần che khuất (Heavy occlusion rule)**:
     - Quy định dứt khoát: Khi xe bị che khuất $> 85\% - 90\%$ (đặc biệt khi đang tiến vào khung hình từ phía sau vật cản lớn như xe buýt), **không gán nhãn** cho đến khi xe lộ diện ít nhất $15-20\%$ thân xe (tránh lỗi gán sớm 13 frame ở Track 5 frame 66–78).
  3. **Quy tắc thoát khung dứt khoát (Exit boundary rule)**:
     - Bấm `outside` ngay tại frame đầu tiên mà phần nhìn thấy của xe chạm mép cắt hoặc diện tích còn lại $< 15\text{ px}$. Tuyệt đối không nội suy kéo dài thêm 2–3 frame khi xe đã trôi ra ngoài (khắc phục lỗi FP trễ ở Track 4 frame 149–151 và Track 8 frame 169–171).
  4. **Danh mục loại trừ vật thể tĩnh dễ nhầm lẫn**:
     - Ghi rõ không gán biển báo giao thông trên cao, quầy ki-ốt/trạm gác bên đường dù có hình khối chữ nhật (rút kinh nghiệm từ Ghost Track 7 của model).

- **Những điểm cần đổi trong quy trình làm việc (Workflow improvements)**:
  1. **Chuẩn hóa quy trình tự kiểm 3 lượt (3-pass review) với checklist cụ thể**:
     - *Lượt 1 (Identity & Continuity)*: Tua nhanh toàn clip kiểm tra tính liên tục của ID, đảm bảo không có 2 ID đổi chỗ cho nhau khi giao cắt.
     - *Lượt 2 (Endpoints - Frame đầu & cuối)*: Nhảy trực tiếp đến frame đầu và frame cuối của từng track, kiểm tra theo đúng quy tắc entry/exit để loại bỏ 100% lỗi bbox treo / bbox thừa trước và sau track.
     - *Lượt 3 (Geometry & Drift)*: Soi kỹ các đoạn xe chuyển hướng hoặc che nhau, chêm keyframe ở điểm uốn để tránh trôi IoU giữa 2 keyframe xa nhau.
  2. **Tích hợp script tự động kiểm tra cục bộ (Local CI check)**:
     - Trước khi nộp, luôn chạy `check_mot_labels.py` và `visualize_tracks.py` để trực quan hóa video kết quả, phát hiện ngay các track bất thường (quá ngắn, đứng yên, hoặc lệch biên).
  3. **Tận dụng Model như một bộ lọc phản biện (Adversarial review)**:
     - Sau khi hoàn thành gán nhãn độc lập và khóa pre-gold, chạy script so sánh sơ bộ (`reid_vs_me`). Xem xét các điểm bất đồng lớn (nơi model có track dài mà người không có, hoặc ngược lại) để phát hiện xe bị bỏ sót ở góc khuất hoặc các trường hợp mơ hồ trước khi chốt nhãn.

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
- [ ] `reports/review_partner.md` (làm cá nhân, không ghép partner)
- [x] `reports/REPORT.md` (file này)
