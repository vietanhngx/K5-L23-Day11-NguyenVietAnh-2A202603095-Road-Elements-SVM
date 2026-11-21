# Đặc tả bàn giao nhãn · Road v1.0

Đặc tả này do khóa học xây dựng. Doanh nghiệp không công bố định dạng file họ nhận, nên lab dùng định dạng gốc của công cụ (CVAT) và thêm một manifest mô tả lô gửi đi. Đây không phải đặc tả của VinFast. Lab không tuyên bố tuân thủ ASAM OpenLABEL hay chuẩn định dạng nào.

## 1. Gửi gì

| File | Định dạng | Chấm điểm? |
| --- | --- | --- |
| `<tên>-front.zip` | CVAT for images 1.1, không kèm ảnh; task front (A01–A12) và front-extra nếu có | có |
| `<tên>-fisheye.zip` | CVAT for images 1.1, không kèm ảnh; task fisheye (B01–B08) và fisheye-extra nếu có | có |
| `<tên>-manifest.json` | Manifest bàn giao do `road_check lint --manifest` tạo (mục 3) | không |
| `<tên>-qc1-for-<tên bạn>.md` | Phiếu QC 1 theo [`templates/nguyenvietanh-qc1-for-luunhatnam.md`](../templates/nguyenvietanh-qc1-for-luunhatnam.md) | có |
| `<tên>-report.md` | Báo cáo theo [`templates/nguyenvietanh-report.md`](../templates/nguyenvietanh-report.md) | có |

`<tên>` viết liền, không dấu, không khoảng trắng (ví dụ `nguyenvanan`). Script chấm của coach tìm đúng tên `<tên>-front.zip` và `<tên>-fisheye.zip`, nên không thêm hậu tố.

## 2. Quy tắc cho file nhãn

- Export từ CVAT bằng **CVAT for images 1.1**, bỏ chọn *Save images*. Ảnh thuộc bộ dữ liệu có giấy phép riêng; người nhận đã có ảnh, chỉ cần nhãn. Manifest ghi `contains_images` để bắt lỗi này.
- Tên label, thuộc tính và giá trị đúng như Road v1.0 (bảng ở guideline mục 1). Label lạ là lỗi QC1-12.
- Tên ảnh giữ nguyên như khi coach tạo task; đổi tên làm ảnh rơi khỏi lô bắt buộc.
- Mỗi ảnh có đủ bốn tag. Mọi job ở trạng thái completed trước khi export.
- File phải mở lại được bằng `road_check lint`; không còn cảnh báo mức nặng. Ngoại lệ duy nhất: QC1-02 (thiếu L1/R1) ở ảnh mà coach đã nhìn và xác nhận không thấy vạch thì được miễn; ghi mã ảnh và lý do vào phiếu QC 1, vì lint không có GT nên không biết vạch có thấy được hay không. Manifest vẫn đếm cảnh báo này trong `lint.nặng`.

## 3. Manifest bàn giao

Lệnh, chạy sau khi export xong:

```bash
PYTHONPATH=src python3 -m road_lab.road_check lint <tên>-front.zip <tên>-fisheye.zip --manifest <tên>-manifest.json --name <tên> --revision 1
```

`--revision` là lần nộp: 1 cho lần đầu, 2 cho lần nộp lại sau Rework, cứ thế tăng. Manifest gồm:

| Trường | Nghĩa |
| --- | --- |
| `format`, `guideline` | Định dạng file và phiên bản guideline dùng khi gán |
| `name`, `revision`, `created_at` | Ai nộp, lần nộp thứ mấy, lúc nào (UTC) |
| `files[].sha256`, `files[].bytes` | Dấu vân tay của từng file: người nhận biết chắc đang kiểm đúng bản nào |
| `files[].images`, `images_all_tags`, `images_unlabelled` | Số ảnh, số ảnh đủ bốn tag, số ảnh chưa có nhãn |
| `files[].shapes_by_label` | Số hình theo từng label |
| `files[].contains_images` | Zip có lỡ kèm ảnh không (phải là `false`) |
| `lint` | Số cảnh báo nặng / chính / nhẹ của `lint` lúc tạo manifest |

Manifest chỉ mô tả lô gửi đi. Nó không kết luận chất lượng; kết luận do người kiểm và GT đưa ra.

## 4. Nộp lại sau Rework

1. **Trước khi export lại**, cất bản cũ: chép hai zip và manifest hiện tại vào thư mục `r1/` trên máy, đổi tên manifest cũ thành `<tên>-manifest-r1.json`. Lệnh tạo manifest mới ghi đè file cùng tên, nên phải cất trước.
2. Sửa trên CVAT, export lại, **giữ nguyên tên zip**.
3. Tạo manifest mới với `--revision 2`.
4. Ghi vào báo cáo (mục 4): đã sửa lỗi nào, theo phiếu QC nào, và lần nộp mới.

Nhờ `sha256` và `revision`, người kiểm phân biệt được bản gốc với bản sửa mà không cần đổi tên file chấm. `road_check` từ chối ghi manifest đè lên chính file nhãn, và không tạo manifest khi còn file không mở được. Cách giữ cả bản gốc lẫn bản sửa, kèm thời điểm và người sửa, đi theo mô tả ở ISO/IEC 5259-4 mục 8.4.9.

## 5. Đặc tả này dựa vào đâu

| Nguồn | Lấy gì |
| --- | --- |
| Tài liệu CVAT, định dạng CVAT for images 1.1 | Cấu trúc `annotations.xml`: label có kiểu (polyline, polygon, rectangle, tag), thuộc tính khai báo trong `meta`, giá trị nằm trong từng hình |
| ASAM OpenLABEL 1.0 | Định dạng nhãn trung lập với công cụ, viết bằng JSON có JSON Schema, có phần metadata (người gán, phiên bản file, phiên bản schema). Lab mượn ý "file nhãn đi kèm metadata"; lab **không** xuất OpenLABEL |
| ISO/PAS 8800 Điều 11 | Đặc tả gán nhãn là một phần metadata của bộ dữ liệu; vì vậy manifest ghi phiên bản guideline |
| ISO/IEC 5259-4 mục 8.4.7–8.4.9 | Tự kiểm, kiểm, trả sửa; giữ bản gốc và bản sửa |
| ISO/IEC 5259-2 (Tra-ML, Aud-ML) | Các phép đo truy vết và khả năng kiểm toán dữ liệu: biết nhãn từ đâu, ai làm, bản nào |

Ở dự án thật, bên nhận thường quy định định dạng riêng (có thể là OpenLABEL, COCO hay định dạng nội bộ) và một bộ kiểm tự động trước khi nhận. Lab tập thói quen đó với định dạng của CVAT.

---
AI20K · AI in Action · VinUni.
