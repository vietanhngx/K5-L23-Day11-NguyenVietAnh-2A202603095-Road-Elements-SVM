# Phiếu QC 1 — Road v1.0

Lô được kiểm (học viên): Nguyễn Việt Anh.  
Người kiểm (học viên): Lưu Nhật Nam (kiểm chéo, học viên xác nhận).  
Ngày: 21/11/2026. Phiên bản guideline: Road v1.0.  
File được kiểm (tên + `revision` trong manifest): nguyenvietanh-front.zip, nguyenvietanh-fisheye.zip; revision 3.  
Số ảnh kiểm: 6 (A01, A03, A05, A06, B03, B06) + 2 ảnh thêm (B11, B12).  
Số nhãn đã kiểm (shape + tag): 91 (59 shape + 32 tag; đếm từ bài nộp, không có GT).  

## Tổng hợp theo mã
| Mã | Loại lỗi | Số mục kiểm | Số lỗi | Mức |
| --- | --- | ---: | ---: | --- |
| QC1-01 tag camera | sai thuộc tính / bỏ sót | 32 | 0 | nặng |
| QC1-02 đủ L1, R1 | bỏ sót | 10 | 0 xác nhận* | nặng |
| QC1-03 position | sai thuộc tính / thừa | 14 | 0 xác nhận* | nặng |
| QC1-04 tâm vạch, chiều, điểm dừng | sai phạm vi / bỏ sót | 14 | 0 xác nhận* | chính |
| QC1-05 vạch đứt, che | sai phạm vi / sai thuộc tính | 14 | 0 xác nhận* | chính |
| QC1-06 style, color, kind | sai thuộc tính | 14 | 0 xác nhận* | chính |
| QC1-07 vùng lái | sai phạm vi / bỏ sót / thừa | 26 | 0 xác nhận* | chính |
| QC1-08 đèn, màu | bỏ sót / sai phạm vi / sai thuộc tính | 9 | 0 xác nhận* | nặng / chính |
| QC1-09 facing, type, group, biển phụ | sai thuộc tính | 19 | 0 xác nhận* | chính |
| QC1-10 hình in, đủ biển | bỏ sót / sai phạm vi | 10 | 0 xác nhận* | chính |
| QC1-11 fisheye | sai phạm vi / sai thuộc tính | 21 | 0 xác nhận* | chính |
| QC1-12 file | lỗi file | 91 | 0 | nhẹ / nặng |

*Chỉ đếm lỗi đã xác nhận còn tồn tại ở revision 3; các nghi vấn chưa tính lỗi được ghi riêng bên dưới. Không đồng nghĩa mọi nhãn đều đúng.

## Danh sách lỗi (mỗi lỗi một dòng)
| # | Mã | Ảnh | Nhãn (label, id) | Mức | Mô tả ngắn |
| --- | --- | --- | --- | --- | --- |
| 1 | QC1-09 | A06 | traffic_light, hộp nhỏ bên trái cụm đèn người đi bộ phía phải ảnh | Chính | Đã sửa: revision 2 gán vehicle; revision 3 là pedestrian / unknown / side. Không tính vào lỗi còn tồn tại. |

## Ca mơ hồ
| Ảnh | Câu hỏi cho lead | Lời đáp (nếu có) |
| --- | --- | --- |
| A05 | Không thấy vạch L1/R1; xác nhận miễn hai cảnh báo thiếu vạch? | Đã xác nhận: không vẽ thêm L1/R1 (học viên cung cấp). |
| A06 | Không thấy rõ R1; xác nhận miễn cảnh báo? | Đã đối chiếu và xác nhận không thấy R1; không vẽ thêm vạch. |
| B03 | Có cần kéo L1 tới vệt trắng gần xe? | Giữ nguyên L1. Vệt trắng có thể là túi bóng; rút nghi vấn dừng sớm, không dùng vệt này làm căn cứ nối lane. |
| A03 | Lane ngoài cùng phải và vùng alternative có thuộc làn lưu thông? | Học viên xác nhận alternative chỉ bao phần đường trống phía ngoài hàng xe đỗ; giữ nhãn hiện tại, rút nghi vấn lấn vùng đỗ xe. |
| B06 | Biên direct/alternative quanh dải phân cách nổi có cần chỉnh? | Học viên xác nhận drivable dừng sát mép thân xe và cách dải phân cách một chút; không bao các khối phân cách. Giữ nguyên. |
| A06 | Box đèn nhỏ bên cạnh đèn người đi bộ: xác nhận type/color vì ảnh mờ. | Đã kiểm file revision 3: pedestrian / unknown / side; sửa đúng thông tin đã xác nhận. |
| A09 (ngoài mẫu) | Có vạch sơn R1 không? | Đã đối chiếu và xác nhận chỉ thấy mép đường/bó vỉa; không có R1 để gán. |
| A14 (ngoài mẫu) | Có vạch sơn R1 nhìn thấy trong ảnh đêm không? | Đã đối chiếu và xác nhận không thấy R1; giữ road_curb, không vẽ thêm lane. |

## Quyết định
Lỗi nặng đã xác nhận: 0. Lỗi chính còn tồn tại đã xác nhận: 0 / 91 nhãn = 0%. Lỗi nhẹ đã xác nhận: 0. Ca mơ hồ chưa xử lý sau kiểm chéo: 0. Các cảnh báo thiếu vạch áp dụng hướng dẫn coach: không vẽ thêm vạch không có trên ảnh.  
Quyết định (Release / Hold / Rework): Release (kiểm chéo).  
Lý do (theo luật của phiếu): Không còn lỗi được xác nhận sau kiểm chéo; không vẽ thêm vạch không thấy trên ảnh. Vạch đôi giữ hai tim theo coach; B12 giữ đoạn ngắn. Đây là kết luận kiểm chéo, chưa phải chấm với GT; các ca thiếu vạch áp dụng hướng dẫn coach đã xác nhận với lớp (học viên cung cấp).  

Loại lỗi: bỏ sót = thiếu nhãn phải có; thừa = gán cái không phải đối tượng; sai phạm vi = hình lệch, sai chiều, dừng sai chỗ, tràn vùng. Ba loại này theo cách chia lỗi gán nhãn của ISO/IEC 5259-4 (mục 8.4.2); "sai thuộc tính" và "lỗi file" là khóa học thêm. Cách xếp mức: [thang mức lỗi](../qc_severity_road_v1.0.md).
