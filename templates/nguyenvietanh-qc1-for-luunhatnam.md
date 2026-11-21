# Phiếu QC 1 — Road v1.0

Lô được kiểm (học viên): Lưu Nhật Nam.  
Người kiểm (học viên): Nguyễn Việt Anh.  
Ngày: 21/11/2026. Phiên bản guideline: Road v1.0.  
File được kiểm (tên + `revision` trong manifest): luunhatnam-front.zip, luunhatnam-fisheye.zip; revision 1, theo phiếu Nam gửi.  
Số ảnh kiểm: 6 (A01, A02, A05, A08, B01, B06), không có ảnh thêm.  
Số nhãn đã kiểm (shape + tag): 91 (67 shape + 24 tag), theo phiếu kiểm chéo được cung cấp.  

## Tổng hợp theo mã

| Mã | Loại lỗi | Số mục kiểm | Số lỗi | Mức |
| --- | --- | ---: | ---: | --- |
| QC1-01 tag camera | sai thuộc tính / bỏ sót | 24 tag / 6 camera | 0 | nặng |
| QC1-02 đủ L1, R1 | bỏ sót | 4 ảnh front | Chờ xác nhận A05 | nặng |
| QC1-03 position | sai thuộc tính / thừa | 14 lane | 0 | nặng |
| QC1-04 tâm vạch, chiều, điểm dừng | sai phạm vi / bỏ sót | 27 polyline | 1 | chính |
| QC1-05 vạch đứt, che | sai phạm vi / sai thuộc tính | 14 lane | 0 | chính |
| QC1-06 style, color, kind | sai thuộc tính | 14 lane | 0 | chính |
| QC1-07 vùng lái | sai phạm vi / bỏ sót / thừa | 17 drivable | 0 | chính |
| QC1-08 đèn, màu | bỏ sót / sai phạm vi / sai thuộc tính | 11 đèn | 0 | nặng / chính |
| QC1-09 facing, type, group, biển phụ | sai thuộc tính | 20 box đèn/biển | 0 | chính |
| QC1-10 hình in, đủ biển | bỏ sót / sai phạm vi | 9 biển | 0 | chính |
| QC1-11 fisheye | sai phạm vi / sai thuộc tính | 2 ảnh fisheye | Chưa chốt | chính |
| QC1-12 file | lỗi file | 2 ZIP | Đọc được theo phiếu | nhẹ / nặng |

## Danh sách lỗi (mỗi lỗi một dòng)

| # | Mã | Ảnh | Nhãn (label, id) | Mức | Mô tả ngắn |
| --- | --- | --- | --- | --- | --- |
| 1 | QC1-04 | A08 | lane, ID 2144 trong bản chụp QC trước | Chính | Vẽ từ xa về gần. Phiếu ghi revision 1 vẫn còn lỗi; cần đảo thứ tự điểm và export lại. |

## Ca mơ hồ

| Ảnh | Câu hỏi cho lead | Lời đáp (nếu có) |
| --- | --- | --- |
| A05, B01 | Không thấy vạch làn rõ; có được miễn cảnh báo thiếu L1/R1? | Phiếu Nam gửi ghi chờ coach xác nhận. |
| A08 | Lỗi đảo chiều lane đã sửa trong ZIP mới chưa? | Phiếu ghi đã sửa trên CVAT nhưng revision 1 vẫn còn lỗi; chờ file mới. |
| Ảnh có xe che mặt đường | Cách vẽ drivable theo phần nhìn thấy có được chấp nhận? | Theo phiếu Nam gửi, giảng viên đã chấp nhận cách vẽ này. |
| B01, B06 | Polyline fisheye có lệch quá 5 px không? | Phiếu chưa có kết quả đo trên toàn đường. |

## Quyết định

Lỗi nặng: chưa chốt. Lỗi chính đã ghi nhận: 1 / 91 nhãn = 1,10%. Lỗi nhẹ: chưa chốt. Ca chờ lead: A05, B01; A08 chờ đối chiếu bản sửa.  
Quyết định (Release / Hold / Rework): Hold (kết quả kiểm chéo đã xác nhận).  
Lý do (theo luật của phiếu): Đã kiểm chéo và xác nhận lỗi đảo chiều lane A08; còn ca cần xác nhận và chờ đối chiếu bản sửa. Số liệu theo phiếu được cung cấp và xác nhận của người kiểm; AI chưa đọc ZIP bài Nam. Phiếu gốc ghi gộp lỗi nặng/nhẹ là 1, chưa tách rõ nên chưa chốt hai số này.  

Loại lỗi: bỏ sót = thiếu nhãn phải có; thừa = gán cái không phải đối tượng; sai phạm vi = hình lệch, sai chiều, dừng sai chỗ, tràn vùng. Ba loại này theo cách chia lỗi gán nhãn của ISO/IEC 5259-4 (mục 8.4.2); "sai thuộc tính" và "lỗi file" là khóa học thêm. Cách xếp mức: [thang mức lỗi](../docs/qc_severity_road_v1.0.md).
