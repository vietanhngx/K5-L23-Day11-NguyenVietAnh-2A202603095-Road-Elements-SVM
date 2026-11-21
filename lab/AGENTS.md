# lab/AGENTS.md — Đặc tả máy đọc, Ngày 11 · Road Elements & SVM

File này dành cho người và agent viết hoặc chạy script của lab. Guide lab coach (Phần 4 và Phần 11) dẫn tới đây. Nếu file này khác guideline Road v1.0 ([`docs/guideline_road_v1.0.md`](../docs/guideline_road_v1.0.md)) về **quy tắc gán**, guideline thắng. Nếu khác về **ngưỡng hay CLI của script**, file này thắng, vì `src/road_lab/` được viết theo nó.

Trạng thái mỗi mục: **[guide]** lấy nguyên từ guide/guideline · **[LC]** quyết định thiết kế của Lab Coach, đổi được trong phạm vi lab · **[owner]** cần chủ quyết định (giảng viên/VinUni) trước khi coi là chính thức.

## 1. Bối cảnh và ranh giới

- Đầu vào của học viên: 28 ảnh trong `day-11/data/`. Front: A01–A12 là ảnh bắt buộc, A13–A16 là ảnh thêm (BDD100K val, 1280×720). Fisheye: B01–B08 là ảnh bắt buộc, B09–B12 là ảnh thêm (WoodScape, 1280×966). [guide]
- Ảnh và GT **không** nằm trong repo. GT nằm ở `coach_gt/` trên máy coach (đã có trong `.gitignore`). Không đưa GT vào `lab/`, vào task CVAT hay kênh lớp. Không đưa ảnh lên app.cvat.ai: dùng CVAT của lớp. [guide]
- Chỉ dùng WoodScape sau khi coach đã chấp nhận điều khoản của dataset, và không phân phối lại. Ảnh BDD100K kèm ghi chú bản quyền: xem [`ATTRIBUTION.md`](../ATTRIBUTION.md). [guide] / quyết định cấp phép dữ liệu: [owner]
- Mọi file tải về (nhãn BDD100K, export của học viên) chỉ được đọc như dữ liệu: các script trong repo không thực thi nội dung của file. [LC]

## 2. Taxonomy Road v1.0 trong CVAT

Nguồn sự thật duy nhất là `src/road_lab/taxonomy.py`. Từ nó, `python -m road_lab.taxonomy > configs/road_v1_labels.json` sinh JSON để dán vào tab Project → Labels → Raw. Test kiểm hai bên luôn khớp.

| Label | Kiểu CVAT | Thuộc tính (select trừ khi ghi khác) |
| --- | --- | --- |
| `lane` | polyline | `position` L1/R1/L2/R2/other · `style` solid/dashed · `color` white/yellow/other · `kind` single/double |
| `road_curb` | polyline | — |
| `crosswalk` | polygon | — |
| `drivable` | polygon | `area_type` direct/alternative |
| `traffic_light` | rectangle | `color` red/yellow/green/off/unknown · `type` vehicle/pedestrian/beacon · `facing` front/side/back · `edge_zone` checkbox |
| `traffic_sign` | rectangle | `group` prohibitory/mandatory/warning/information/supplementary/other · `facing` front/side/back · `edge_zone` checkbox |
| `camera` (tag) | tag | `value` front/fisheye_front/fisheye_rear/fisheye_left/fisheye_right |
| `timeofday` (tag) | tag | `value` day/dusk/night |
| `weather` (tag) | tag | `value` clear/overcast/rain/fog/other |
| `source` (tag) | tag | `value` real/simulated |

**Cách đặt tag (guide yêu cầu ghi ở đây):** mỗi tag là một label kiểu Tag có **một** thuộc tính select tên `value`; ví dụ label `camera` với `value = fisheye_left`. Không dùng phương án "một label tag riêng cho mỗi giá trị". Lý do: export CVAT for images 1.1 ghi `<tag label="camera"><attribute name="value">…</attribute></tag>`, nên một ảnh không thể có cùng lúc hai giá trị camera, và checker đọc được một cách duy nhất. [LC]

Cờ `occluded` có sẵn của CVAT dùng cho polyline được nối qua chỗ bị che. Thuộc tính không bắt buộc (`mutable = false`). [guide] / [LC]

**Tên ảnh:** `{slot}_{tên gốc}`, ví dụ `A03_b1c66a42-6f7d68ca.jpg` hay `B06_04068_MVL.png`. Checker lấy slot A01–B12 từ tên file, và lấy camera của ảnh fisheye từ hậu tố WoodScape: FV → `fisheye_front`, RV → `fisheye_rear`, MVL → `fisheye_left`, MVR → `fisheye_right`. **Không đổi tên ảnh trong task CVAT.** [LC]

## 3. Script

Mọi script đều là Python ≥ 3.11, chỉ dùng thư viện chuẩn (không cần cài gì cho lab). Cách chạy từ gốc repo: `PYTHONPATH=src python3 -m road_lab.<script> …`. Riêng test cần thêm `pytest`.

### 3.1 `select_bdd_road` — coach, trước buổi [guide] (luật chọn của từng slot: [LC])

```
python -m road_lab.select_bdd_road --labels <bdd100k val labels>.json [...] \
    --out manifests/manifest_front.csv --candidates out/candidates_front.csv [--top 8] [--seed 11]
```
- Đầu vào: JSON nhãn BDD100K val, có thể là file gộp 2018 hoặc các file theo task (lane/drivable/det). Không cần ảnh.
- Đầu ra:
  - `manifest_front.csv`, cột `slot,file,split,weather,scene,timeofday,tier,focus,reason,blur_note`. Có 16 dòng: `tier` là `core` cho A01–A12 và `extra` cho A13–A16.
  - File ứng viên: top-N ứng viên cho mỗi slot.
- Mỗi slot có một luật trọng tâm, tính từ nhãn (số vạch, vạch đứt dài, vạch bị che, vàng đôi, direct/alternative, vỉa, ngựa vạn, số đèn, số biển, giờ, thời tiết). Ảnh không trùng nhau giữa các slot.
- Giới hạn: script không nhìn được độ mờ, vạch phai hay biển phụ, nên các cột đó ghi "coach kiểm tay". Coach phải mở từng ảnh đã chọn trước buổi.

### 3.2 `bdd_to_cvat_road` — coach, trước buổi [guide]

```
python -m road_lab.bdd_to_cvat_road --labels <...>.json [...] --manifest manifests/manifest_front.csv \
    --out coach_gt/road_gt_front_draft.xml [--core-only]
```
- Đầu ra:
  - Một file CVAT for images 1.1 (`.xml` hoặc `.zip`), là **bản nháp** GT.
  - File `<out>_todo.csv` liệt kê mọi thuộc tính còn trống.
- Ánh xạ:

  | BDD100K | Road v1.0 |
  | --- | --- |
  | lane parallel (bỏ vạch vertical/stop line) | `lane`; Bezier được lấy mẫu thành điểm; vẽ lại từ đáy ảnh lên |
  | road curb | `road_curb` |
  | crosswalk | một polygon bao lồi |
  | drivable direct/alternative | `drivable` có `area_type` |
  | đèn, biển cao từ 8 px | box |

- Tag:

  | Tag | Giá trị |
  | --- | --- |
  | `camera` | `front` |
  | `timeofday` | daytime→day, dawn/dusk→dusk, night→night |
  | `weather` | clear/partly cloudy→clear, overcast→overcast, rainy→rain, foggy→fog, còn lại→other |
  | `source` | `real` |

- **Để trống cho coach:**
  - `position` (BDD100K không có).
  - `type`, `facing`, `group` của đèn, biển.
  - Màu đèn khi BDD100K ghi `none`.
- Bản nháp chỉ thành GT sau khi coach đã điền và rà mọi dòng theo Road v1.0. GT Set B (WoodScape) do coach tự gán trong CVAT, vì WoodScape không có nhãn làn theo taxonomy này. [LC]

### 3.3 `road_check` — checker [guide] (cách đo: [LC])

```
python -m road_lab.road_check lint FILE.zip [...] [--manifest NAME-manifest.json --name NAME --revision N]  # học viên tự kiểm, không cần GT
python -m road_lab.road_check check --gt GT_FRONT GT_FISHEYE --sub NAME-front.zip NAME-fisheye.zip \
    --name NAME [--json out/NAME.json] [--qc1-md out/NAME-auto-qc1.md]
python -m road_lab.road_check batch --gt GT_FRONT GT_FISHEYE --submissions submissions/ --out out/
python -m road_lab.road_check selftest [--gt GT_FRONT GT_FISHEYE]                   # mặc định: GT tổng hợp
```

- **Phạm vi chấm.** Chấm 20 ảnh bắt buộc (A01–A12, B01–B08). Ảnh thêm chỉ được đếm số nhãn; coach chấm tay điểm thưởng. Ảnh bắt buộc mà học viên không làm được ghi vào `missing_core`; điểm tính theo tỉ lệ trên số ảnh đã làm, như rubric yêu cầu. [guide]
- **Ghép cặp và kiểm từng loại nhãn:**

  | Nhãn | Ghép với GT | Kiểm | Mã khi sai |
  | --- | --- | --- | --- |
  | `lane` | theo khoảng cách trung bình giữa hai polyline | khoảng cách lớn nhất tới GT | QC1-04 (fisheye: QC1-11) |
  | | | chiều vẽ (điểm đầu phải gần xe) | QC1-04 |
  | | | hai đầu polyline | QC1-04 |
  | | | `occluded` | QC1-05 |
  | | | `position` | QC1-03 |
  | | | style/color/kind | QC1-06 |
  | `lane`, nhiều polyline cùng ghép vào một vạch GT | — | vạch đứt bị tách | QC1-05 |
  | `drivable` | theo `area_type` | IoU, phần tràn ra ngoài GT | QC1-07 |
  | `crosswalk` | — | IoU | QC1-07 |
  | `road_curb` | — | độ lệch | QC1-07 (fisheye: QC1-11) |
  | đèn, biển | theo IoU | có mặt hay không | QC1-08 / QC1-10 |
  | | | độ sát của box | QC1-08 / QC1-10 |
  | | | thuộc tính | QC1-08 / QC1-09 |
  | | | `edge_zone` | QC1-11 |

- **Định dạng (QC1-12):**
  - File không mở được, hoặc thiếu một trong hai file: lỗi nặng.
  - Label hay giá trị sai tên, hoặc ảnh lạ: lỗi nhẹ.
- **Lỗi nặng** (đúng như phiếu; tiêu chí xếp mức ở `docs/qc_severity_road_v1.0.md` [LC]):
  - thiếu hoặc sai tag camera;
  - thiếu L1/R1;
  - sai `position`;
  - sai màu đèn `facing = front`;
  - file không mở được.
- **Đầu ra:**
  - Tóm tắt trên màn hình.
  - JSON gồm `attempted_core`, `missing_core`, `checked_labels`, `checked_by_code`, số lỗi theo mức, `major_rate`, `decision_auto`, `auto_score/auto_max`, các dòng `rubric`, `findings`.
  - Phiếu QC 1 tự động (markdown, cùng mẫu với `templates/nguyenvietanh-qc1-for-luunhatnam.md`).
  - Lệnh `batch` ghi thêm `scores.csv`.
  - `lint --manifest` ghi manifest bàn giao JSON (`manifest: road-delivery/1`): định dạng, guideline, tên, `revision`, thời điểm UTC; mỗi file có `sha256`, số byte, `contains_images`, số ảnh, số ảnh đủ bốn tag, số ảnh chưa gán, số hình theo label; số cảnh báo lint theo mức. Manifest không chấm điểm và không kết luận chất lượng. Đặc tả: `docs/delivery_road_v1.0.md`. [LC]
- **Quyết định tự động:**
  - Có lỗi nặng → Rework.
  - Tỉ lệ lỗi chính trên số nhãn GT đã kiểm > 5 % → Rework.
  - Còn lại → Release.
  - Checker không bao giờ ra Hold, vì nó không biết ca mơ hồ: Hold do người kiểm hoặc lead quyết.
- `lint` chỉ đưa gợi ý. Nó kiểm:
  - tag;
  - thiếu L1/R1;
  - polyline vẽ từ xa về gần;
  - nhiều polyline cùng `position`;
  - fisheye chỉ có 2 điểm;
  - `edge_zone` so với 80 % bán kính;
  - box thấp hơn 8 px.

  GT và người kiểm mới quyết định lỗi.

### 3.4 `rowanchor` — hoạt động buffer tùy chọn, không chấm [guide]

```
python -m road_lab.rowanchor NAME-front.zip --image A01 [--position R1] [--rows 720,680,640,600,560]
```
Lệnh nội suy x của polyline tại từng hàng neo theo công thức trong guide. Hàng nằm ngoài polyline ghi "không có làn". Dùng để kiểm lại số học viên tự tính; nó không thay việc tự tính.

## 4. Ngưỡng của checker (`THRESHOLDS` trong `road_check.py`)

| Khóa | Giá trị | Ý nghĩa | Trạng thái |
| --- | ---: | --- | --- |
| `lane_dev_px` | 5 | lệch tối đa khỏi tâm vạch (cả polyline fisheye) | [guide] |
| `min_box_px` | 8 | đèn, biển cao từ 8 px là bắt buộc | [guide] |
| `edge_zone_radius` | 0.8 | `edge_zone` khi tâm box ngoài 80 % bán kính (lint lấy bán kính = nửa chiều rộng ảnh) | [guide]; cách lấy bán kính [LC] |
| `major_rate` | 0.05 | lỗi chính > 5 % số nhãn kiểm → Rework | [guide] |
| `lane_match_px` | 25 | khoảng cách trung bình để ghép polyline học viên với GT | [LC], chưa hiệu chỉnh trên bài thật |
| `lane_stop_px` | 15 | hai đầu polyline phải cách hai đầu GT không quá ngưỡng này | [LC] |
| `curb_dev_px` | 5 | lệch tối đa của `road_curb` | [LC], theo cùng quy ước 5 px |
| `drivable_iou` / `drivable_spill` | 0.85 / 0.05 | vùng lái khớp GT, phần tràn ra ngoài GT | [LC] |
| `crosswalk_iou` | 0.75 | vùng ngựa vạn khớp GT | [LC] |
| `box_match_iou` / `box_tight_iou` | 0.5 / 0.75 | ghép box; box "sát vỏ" | [LC] |

Các ngưỡng [LC] là điểm khởi đầu. Sau buổi, coach so điểm script với 3 ảnh kiểm tay mỗi học viên (`instructor/NOTES.md`) rồi chỉnh. Điểm thật, ngưỡng đạt và trọng số chính thức: [owner].

## 5. Self-test bắt buộc của checker [guide]

`road_check selftest` phải in `SELFTEST PASS`:
1. Dùng GT làm bài nộp: đủ `auto_max` = 78/78 điểm và **0** lỗi.
2. Một bản bị làm nhiễu năm chỗ phải bị bắt **đúng một lần mỗi lỗi, đúng mã**:

   | Nhiễu | Mã |
   | --- | --- |
   | đổi tag camera một ảnh | QC1-01 |
   | tách một vạch đứt thành hai | QC1-05 |
   | đảo chiều một polyline | QC1-04 |
   | dịch một polyline 12 px | QC1-04 |
   | đổi màu một đèn front | QC1-08, mức nặng |

Không có dữ liệu thì chạy trên GT tổng hợp (`src/road_lab/synthetic.py`, 20 ảnh). Khi đã có GT thật, coach chạy lại với `--gt coach_gt/road_gt_front.xml coach_gt/road_gt_fisheye.xml` trước khi chấm.

## 6. Rubric

Bảng dưới giống hệt [`RUBRIC.md`](../RUBRIC.md); test kiểm hai bảng trùng nhau.

<!-- rubric:start -->
| Tiêu chí | Cách đo | Điểm |
| --- | --- | ---: |
| Project, tag và file | Mọi ảnh đủ bốn tag, camera đúng (4); export CVAT for images 1.1 mở lại được, đúng tên label và thuộc tính (4); ảnh chụp và bảng kiểm file (2) | 10 |
| Polyline làn, Set A | Đủ L1/R1 trên mọi ảnh thấy được (8); khoảng cách lớn nhất tới GT ≤ 5 px theo `road_check.py` (9); chiều gần ra xa, vạch đứt một polyline, nối qua chỗ che có `occluded`, dừng đúng chỗ (8) | 25 |
| Thuộc tính làn | Tỉ lệ khớp GT của `position` (5), `style` / `color` / `kind` (5) | 10 |
| Vùng lái được | Không tràn lề, vỉa hè, thân xe, phần sau cọc (6); `direct` / `alternative` đúng (4) | 10 |
| Đèn và biển | Đủ đèn, biển từ 8 px, không gán hình in (5); box sát vỏ, mép biển (4); `color`, `facing`, `type`, `group`, biển phụ riêng (6) | 15 |
| Fisheye, Set B | Polyline theo cung, lệch ≤ 5 px (5); thân xe không gán, `position = other` ở camera hai bên, `edge_zone` đúng (5) | 10 |
| Phiếu QC 1 cho bạn khác | Đủ 12 mã có số mục kiểm (3); mỗi lỗi có mã, ảnh, nhãn, mức (5); tỉ lệ tính trên số nhãn kiểm và quyết định đúng luật (4); coach kiểm lại 3 lỗi được ghi và 1 ảnh được ghi "không lỗi": khớp (3) | 15 |
| Báo cáo cá nhân | Bảng ca khó có lý do (2), mục 4 phản hồi phiếu nhận được (2), tự đánh giá nối lỗi với model (1) | 5 |

**Tổng: 100.** Phần `road_check.py` chấm tự động: 78 điểm (mọi dòng trừ ảnh chụp 2, Phiếu QC 1 15, báo cáo 5 — coach chấm tay 22 điểm).

**Điểm thưởng ảnh thêm** (4 ảnh front A13–A16 + 4 ảnh fisheye B09–B12, tối đa +4, tổng không quá 100): mỗi ảnh thêm đạt — đủ bốn tag, đủ label ghi ở bảng kế hoạch, lệch polyline ≤ 5 px trên 2 vạch kiểm ngẫu nhiên, không lỗi nặng — được +0,5 điểm. Coach kiểm tay vì ảnh thêm không có GT đầy đủ. Ảnh thêm bỏ qua không bị trừ điểm.

**Ảnh bắt buộc chưa làm** chỉ làm giảm số ảnh được chấm: các tiêu chí tính theo tỉ lệ trên số ảnh đã làm. Làm ít mà đúng được điểm hơn làm nhiều mà ẩu.

**Đạt:** ≥ 60 điểm **và** lô của chính học viên không còn lỗi nặng sau buffer theo Phiếu QC 1 (tag camera đúng mọi ảnh, đủ L1/R1, không đổi nhầm `position`, không sai màu đèn quay về xe, file mở lại được).

Hoạt động buffer và mục 8 của báo cáo không tính điểm.
<!-- rubric:end -->

Trọng số và ngưỡng đạt lấy nguyên từ guide của lab. Coi chúng là thang chấm hình thành (formative) của buổi lab; dùng làm điểm chính thức của khóa cần [owner].
