# Ngày 11 · Road Elements & SVM

> Lab cá nhân, AI20K K5. Bạn gán nhãn phần tử đường theo guideline Road v1.0 trên CVAT của lớp: 12 ảnh camera trước BDD100K (A01–A12) và 8 ảnh fisheye WoodScape (B01–B08) là bắt buộc. Sau đó bạn kiểm lô của một bạn khác bằng Phiếu QC 1 và quyết định Release, Hold hoặc Rework. Nếu xong sớm, làm thêm ảnh A13–A16 và B09–B12 để lấy điểm thưởng. Không đưa ảnh lên app.cvat.ai.

| M | Giờ | Việc | Lệnh kiểm tra |
| --- | --- | --- | --- |
| M0 | 13:00 | Điểm danh. Kiểm repo, gói 28 ảnh trong `data/`, đăng nhập CVAT và guideline đã đọc. | `PYTHONPATH=src python3 -m road_lab.road_check selftest` |
| Task 0 | 13:15 | Mở project `d11-road-<tên>`, so sáu label và bốn label Tag với guideline. | `PYTHONPATH=src python3 -m road_lab.taxonomy` (in Raw labels JSON để so) |
| Task 1 | 13:50 | Gán A01–A12, mỗi ảnh theo trọng tâm của nó; xong thì làm A13–A16. | — |
| Task 2 | 15:05 | Gán B01–B08; xong thì làm B09–B12. Dành 5 phút cuối để self-QC. | — |
| Task 3 | 15:40 | Đổi lô với bạn được ghép, điền Phiếu QC 1 ([`templates/nguyenvietanh-qc1-for-luunhatnam.md`](templates/nguyenvietanh-qc1-for-luunhatnam.md)). | — |
| Export | 16:15 | Export CVAT for images 1.1 (không kèm ảnh), mở lại rồi nộp. | `PYTHONPATH=src python3 -m road_lab.road_check lint <tên>-front.zip <tên>-fisheye.zip` |
| Buffer | 16:35 | Sửa lô theo phiếu nhận được. Tùy chọn: hoạt động row-anchor. | `PYTHONPATH=src python3 -m road_lab.rowanchor <tên>-front.zip --image A01` |

## Cài đặt

Chỉ cần Python 3.11 trở lên; code chỉ dùng thư viện chuẩn. Muốn chạy test thì cài thêm pytest.

```bash
git pull
ls data/ | wc -l          # phải là 28: A01…A16, B01…B12
PYTHONPATH=src python3 -m road_lab.road_check selftest   # in SELFTEST PASS
```

Nếu dùng uv:

```bash
uv run --with pytest==8.3.4 pytest -q
```

## Tài liệu

- [Guideline Road v1.0](docs/guideline_road_v1.0.md): đọc trước buổi, ít nhất là bảng label và quy tắc làn.
- [Hướng dẫn bằng hình trên CVAT](docs/huong_dan_cvat_bang_hinh.md): từng bước từ đăng nhập tới export, chụp trên ảnh tổng hợp.
- [Đặc tả bàn giao](docs/delivery_road_v1.0.md): định dạng file nộp, manifest, nộp lại sau Rework.
- [Thang mức lỗi QC 1](docs/qc_severity_road_v1.0.md): tiêu chí nặng / chính / nhẹ và nguồn tham khảo.
- [Mẫu Phiếu QC 1](templates/nguyenvietanh-qc1-for-luunhatnam.md) và [mẫu báo cáo](templates/nguyenvietanh-report.md).
- [Rubric](RUBRIC.md), [checkpoint](CHECKPOINTS.md), [nộp bài](SUBMISSION.md).
- [`lab/AGENTS.md`](lab/AGENTS.md): bản máy đọc của guideline, mô tả input/output của script.
- [Nguồn dữ liệu và giấy phép](ATTRIBUTION.md).

`lint` chỉ kiểm định dạng file. Nó không so bài của bạn với GT và không cho điểm. Coach chấm bằng `road_check check` hoặc `road_check batch` sau buổi.
