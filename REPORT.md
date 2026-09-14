# Báo cáo — Ngày 2: phát hiện vật thể
Họ và tên: Trần Quốc Trọng
MSSV: 2A202602123
Hình thức: cá nhân
Mã cặp: SOLO

## 1. Bài độc lập và nguồn dữ liệu
- Mã SHA-256 của ZIP ảnh được cấp: `f7d99888f21440fb0374d84962b93213bd8c14e665d093cc8d37f4c61b71ed33`
- Bốn mã ảnh: drive_022, drive_033, drive_038, drive_008
- Số vật thể thực tế tôi đã gán: 101
- Mã SHA-256 của gói YOLO của tôi: `194c28f06903cbf56f3ba7bc41a9f69bf1bfe9f137123f6ed3583e16a297da4d`
- Mã SHA-256 của gói CVAT gốc của tôi: `2724e6516c546fcecb7562fa34ac2089d20c7dbefbc078acd03eded05c5c4ad1`
- Nguồn đối chiếu: bộ tham chiếu do người hướng dẫn thực hành cấp
- Mã SHA-256 của gói đối chiếu: `c8bbc767d8bb9a29f4ca5abf0c3516e5c2af94c58143a980b0148cfe0b500d2b`
- Mã lần phát và thời điểm tôi nhận bộ tham chiếu: Ca thực hành Ngày 2 ngày 14/09/2026
- Giải thích vì sao bài của tôi vẫn độc lập trước khi đối chiếu:
Tôi tự cài CVAT trên máy bằng Docker và tự gán nhãn xong cả 4 ảnh. Sau khi xuất đủ 2 file zip và nộp vào các ô kiểm tra ban đầu trên Colab xong xuôi thì tôi mới tải file của thầy về để so sánh, nên bài làm hoàn toàn độc lập từ trước.

## 2. Quyết định phân lớp
| Ảnh/vật thể | Lớp | Dấu hiệu nhìn thấy | Quy tắc áp dụng |
| :--- | :--- | :--- | :--- |
| drive_022_obj1 | car | Xe sedan 4 chỗ đi giữa đường, thấy rõ đầu và đuôi xe | Theo quy tắc lớp car |
| drive_022_obj2 | bus | Xe buýt màu vàng, thân dài và có nhiều cửa sổ dọc xe | Theo quy tắc lớp bus |
| drive_038_obj1 | truck | Xe tải có thùng chở hàng phía sau tách biệt với đầu xe | Theo quy tắc lớp truck |
| drive_008_obj1 | van | Xe dạng hình hộp kín, lớn hơn xe 4 chỗ và không có thùng hở | Theo quy tắc lớp van |

Nêu một ví dụ cho thấy lớp và thuộc tính là hai loại thông tin khác nhau:
Lớp là loại xe, ví dụ xe đó là ô tô con thì bản chất nó là ô tô con. Còn thuộc tính là trạng thái lúc chụp ảnh, ví dụ cái xe đó đi qua mép ảnh bị cắt mất nửa thân thì thuộc tính là truncated, hoặc bị xe khác đi trước che mất bánh thì thuộc tính là occluded.

## 3. Tự kiểm tra và sửa nhãn
| Trước khi sửa | Loại lỗi | Cách phát hiện | Sau khi sửa và quy tắc |
| :--- | :--- | :--- | :--- |
| Gán nhầm van thành bus | lớp | So sánh chiều dài xe với xe buýt thật bên cạnh thấy ngắn hơn nhiều | Sửa lại thành van |
| Box bị rộng thừa nhiều mặt đường | hình học | Zoom to lên thấy mép box cách xa lốp xe | Kéo lại các cạnh cho sát mép xe |
| Xe bị che đuôi nhưng chọn clear | thuộc tính | Nhìn kỹ thấy đuôi xe bị đầu xe sau đè lên | Đổi thuộc tính sang occluded |

- Số hộp needs_review trước và sau khi kiểm: Trước khi kiểm tôi có 7 hộp, sau khi kiểm tra lại thì tôi xử lý xong hết còn 0 hộp.
- Một quyết định chưa đủ bằng chứng và cách tôi xử lý: Có một xe ở góc xa ngã tư bị bóng râm che tối và hơi mờ, chưa rõ là xe hatchback hay xe van nhỏ. Tôi đánh dấu needs_review, sau đó zoom to lên so kích thước với xe đi cùng làn thấy ngang cỡ xe 4 chỗ nên tôi gán nhãn car.

## 4. Một dòng nhãn YOLO
- Dòng class x_center y_center width height:
`0 0.263047 0.570781 0.123906 0.07875`
- Tên lớp và tọa độ điểm ảnh xyxy:
Lớp car, tọa độ điểm ảnh lần lượt là x1 128.7, y1 340.1, x2 208.0, y2 390.5
- Vì sao dòng đúng định dạng vẫn có thể sai lớp, phạm vi hoặc hình học?
Vì file text chỉ lưu các con số từ 0 đến 1. Code chỉ kiểm tra xem số có đúng định dạng không chứ không biết nội dung bên trong. Do đó mình gán nhầm xe tải thành xe con thì file vẫn đúng định dạng, hoặc vẽ khung lệch khỏi xe hay vẽ nhầm vào bóng râm thì hệ thống vẫn nhận là đúng cú pháp.

## 5. Huấn luyện và dự đoán thử
- Ba mã ảnh huấn luyện: drive_022, drive_033, drive_038
- Mã ảnh thẩm định: drive_008
- Mô tả một dự đoán trong detect_result.jpg: Mô hình tìm được đúng các xe buýt và xe ô tô đi ở giữa đường với độ tin cậy trên 75%.
- Dự đoán đó gợi ý tôi cần kiểm lại quy tắc hoặc dữ liệu nào? Mô hình hay bỏ sót mấy xe ở xa và xe bị che khuất, cho thấy tôi cần chú ý gán kỹ hơn ở các trường hợp bị che và các xe nhỏ ở mép ngoài.
- Minh chứng nào có thể bác bỏ nhận định của tôi? Cần test thêm trên nhiều ảnh khác chụp ở góc máy khác xem mô hình có thực sự bị yếu phần đó không hay do ảnh này ở góc xa bị mờ quá.
- Vì sao kết quả trên bốn ảnh không phải phép đánh giá mô hình dùng thực tế? Vì tập dữ liệu có 4 ảnh là quá ít, mô hình chỉ đang học thuộc vị trí các xe trong ảnh chứ chưa tổng quát hóa được. Đi làm thực tế phải test trên hàng nghìn ảnh ở nhiều điều kiện trời mưa nắng khác nhau mới đánh giá chuẩn được.

## 6. Đối chiếu nhãn
- Số hộp ghép được: 48
- IoU trung bình và trung vị: IoU trung bình là 0.88, trung vị là 0.89
- Mức đồng thuận lớp: 72.9%
- Số hộp phía tôi không ghép được: 53
- Số hộp phía đối chiếu không ghép được: 2
- Một điểm khác biệt cụ thể: Tôi gán hết tất cả 101 xe nhìn thấy trong ảnh kể cả các xe ở tít xa và xe đỗ bên lề đường, còn bộ nhãn của thầy chỉ gán 50 xe chính đang chạy ở gần. Ngoài ra có vài xe 7 chỗ tôi gán là car nhưng bên thầy gán là van.
- Quy tắc hoặc hành động sửa phát sinh: Cần thống nhất lại xem các xe ở quá xa hoặc đỗ ngoài đường chính có phải gán không, và quy định rõ xe 7 chỗ xếp vào car hay van.
- Vì sao mức đồng thuận cao không chứng minh mọi nhãn đều đúng? Vì nếu cả hai bên cùng hiểu sai quy tắc hoặc cùng nhìn nhầm một loại xe giống nhau thì kết quả so khớp vẫn ra điểm cao, dù thực tế nhãn đó bị gán sai.

## 7. Kiểm tra kho GitHub cá nhân
- [x] Có phiếu quy tắc với ba tình huống mơ hồ.
- [x] Có kết quả kiểm hai gói xuất.
- [x] Có thông tin lần huấn luyện và ảnh dự đoán.
- [x] Có tóm tắt, bảng và ảnh phủ của bước đối chiếu.
- [x] Không có gói xuất thô, bộ nhãn tham chiếu hoặc trọng số mô hình.
- [x] Không có dữ liệu VinFast/khách hàng/ảnh cá nhân/mật khẩu/mã truy cập.
