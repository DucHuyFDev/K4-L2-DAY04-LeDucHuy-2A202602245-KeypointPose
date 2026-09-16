# Báo cáo Ngày 4 - Keypoint & Pose

- Họ tên: Lê Đức Huy
- Nhóm: ______
- Ngày: 19/06/2026

## 1. Nhãn của tôi

| Chỉ số                         |                      Giá trị |
| -------------------------------- | -----------------------------: |
| Số ảnh đã gán               |                             20 |
| Số skeleton                     |                             29 |
| v=2 / v=1 / v=0                  |                 331 / 135 / 27 |
| Thời gian trung bình mỗi ảnh | Chưa ghi nhận trong notebook |

Ba keypoint có tỷ lệ `v=1` cao nhất:

1. `left_ear`: 66%
2. `right_ear`: 48%
3. `left_eye`: 38%

Tai có tỷ lệ bị che cao nhất, phù hợp với các tình huống tóc, vật thể hoặc người khác che vùng đầu. Mắt trái đứng thứ ba nhưng tỷ lệ này không chỉ phản ánh độ khó định vị; nó còn phụ thuộc vào hướng mặt và việc một mắt bị che. Vai có tỷ lệ `v=1` thấp hơn nhiều, cho thấy vai thường dễ xác định hơn tai.

Checker đạt định dạng với 20/20 file và không còn lỗi box hoặc tọa độ ngoài ảnh. Còn 7 cảnh báo: `train_04`, `train_10`, `train_13` có các điểm `v=0` trong box nằm giữa ảnh; `train_15` và `train_16` có dấu hiệu đảo trái/phải ở vai/hông.

## 2. Chấm với gold

| Chỉ số                | Trước rework | Sau rework |
| ----------------------- | -------------: | ---------: |
| OKS trung bình         |         0.8572 |     0.9048 |
| OKS@0.50                |         0.9310 |     1.0000 |
| OKS@0.75                |         0.8966 |     0.9655 |
| Lỗi`dao_trai_phai`   |              3 |          1 |
| Lỗi`nham_nguoi`      |              2 |          1 |
| Lỗi`xoa_khop_bi_che` |              0 |          0 |

Sau rework có 29/29 người được ghép, không thiếu và không thừa skeleton. Số lỗi đảo trái/phải giảm từ 3 xuống 1; lỗi nhầm người giảm từ 2 xuống 1; OKS trung bình tăng 0.0476.

### Các thay đổi chính

- `train_03.txt`, `train_06.txt`, `train_09.txt`, `train_11.txt`, `train_13.txt`, `train_14.txt`, `train_15.txt`: cập nhật vị trí hoặc visibility của các keypoint theo kết quả kiểm tra và đối chiếu gold.
- `train_11.txt`: hai ankle ngoài vùng quan sát được đưa về `v=0` với tọa độ `0 0`.
- `train_13.txt`: cập nhật nhãn người 1; vẫn cần kiểm tra trực quan vì còn dấu hiệu đảo trái/phải.
- `train_14.txt` và `train_09.txt`: điểm OKS rất thấp trước rework được cải thiện rõ sau khi sửa nhãn.

Lỗi còn lại sau rework:

- `train_04.jpg`, người gold 1 / nhãn người 2: `left_wrist` bị nhầm sang người khác, OKS `0.8435`.
- `train_13.jpg`, người 1: còn lỗi đảo trái/phải, OKS `0.6964`.

Lỗi đảo trái/phải còn lại ở `train_13.jpg`. Các cảnh báo checker cũng cho thấy cần xem lại vai/hông trong `train_15.jpg` và `train_16.jpg`, dù chúng chưa xuất hiện trong nhóm lỗi gold nghiêm trọng sau rework.

## 3. Kiểm chéo

Chưa có thư mục nhãn của bạn cùng nhóm trong workspace, nên chưa thể tính bảng chênh lệch `%v=1` giữa hai người.

Luật đã thống nhất và áp dụng trong `GUIDELINE_MINI.md`:

- Keypoint bị che nhưng còn trong khung ảnh phải dùng `v=1` và đặt tọa độ ước lượng.
- Keypoint ra ngoài mép ảnh phải dùng `v=0` và tọa độ `0 0`.
- Trái/phải được xác định theo cơ thể người và kiểm tra theo cặp vai, hông, gối, cổ chân.

## 4. Model

Các số dưới đây lấy từ output notebook trên tập test gồm 10 ảnh và 13 người.

| Chỉ số           | YOLO26n-pose gốc | Sau fine-tune |  Chênh |
| ------------------ | ----------------: | ------------: | ------: |
| `pose_mAP50`     |            0.8450 |        0.8450 | +0.0000 |
| `pose_mAP50-95`  |            0.6853 |        0.6908 | +0.0055 |
| `pose_precision` |            0.9734 |        0.9792 | +0.0058 |
| `pose_recall`    |            0.8462 |        0.8462 | +0.0000 |
| `box_mAP50-95`   |            0.8119 |        0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook

1. `pose_mAP50-95` tăng `0.0055`, từ `0.6853` lên `0.6908`. Đây là cải thiện nhỏ, cho thấy nhãn mới giúp model khớp pose tốt hơn một chút nhưng chưa tạo ra thay đổi lớn trên tập test. Dataset chỉ có 20 ảnh train và 29 skeleton nên không đủ để kết luận model đã tổng quát hóa mạnh hơn.
2. Ở baseline, `box_mAP50-95` cao hơn `pose_mAP50-95` là `0.1266`; sau fine-tune chênh lệch là `0.1133`. Model tìm người dễ hơn tìm chính xác 17 keypoint, vì box chỉ cần bao phủ người còn pose yêu cầu vị trí và thứ tự từng khớp.
3. Hai trường hợp lệch số người đáng chú ý trong output là `train_10` (model 2 người, nhãn 1 người) và `train_03` (model 4 người, nhãn 2 người). Đây là dấu hiệu cần kiểm tra lỗi thiếu/thừa người và có thể liên quan đến nhầm người hoặc dự đoán dư. Với nhãn gold, lỗi còn lại được phân loại cụ thể là nhầm `left_wrist` ở `train_04` và đảo trái/phải ở `train_13`.
4. OKS thấp nhất giữa model và nhãn sau lần chạy mới là `train_06` với `0.620`. Con số này chỉ cho biết model và nhãn chưa đồng thuận; chưa thể kết luận bên nào đúng nếu không xem ảnh và skeleton overlay. Căn cứ cần dùng là vị trí khớp trên ảnh, hướng trái/phải theo cơ thể và so sánh với các keypoint liền kề.
5. Ảnh có nhãn bị đánh giá kém nhất theo gold không hoàn toàn trùng ảnh model bất đồng thấp nhất. `train_13` còn lỗi đảo trái/phải với gold, trong khi `train_06` có OKS model-vs-label thấp nhất. Điều này cho thấy lỗi nhãn và lỗi model không phải lúc nào cũng trùng nhau; cần dùng cả gold, OKS và kiểm tra trực quan.

Training dừng sớm ở epoch 39; checkpoint tốt nhất là epoch 9. Sau epoch 13, các chỉ số validation giảm mạnh, vì vậy nên dùng `best.pt`, không dùng `last.pt`.

## 5. Một rule evidence đã dùng

Trong `train_11.jpg`, người 1, hai keypoint `left_ankle` và `right_ankle` nằm ngoài vùng quan sát ở phía dưới ảnh. Tọa độ cũ lần lượt có y lớn hơn 1 sau chuẩn hóa, nên không thể giữ visibility `v=1`. Hai điểm được chuyển thành `v=0` với tọa độ `0 0`, vì chúng đã ra ngoài khung ảnh chứ không chỉ bị che trong khung. Đây là bằng chứng định dạng và hình ảnh phù hợp với quy tắc: bị che trong ảnh dùng `v=1`, ra ngoài ảnh dùng `v=0`.

## Kết luận

Bản nhãn sau rework đã cải thiện rõ: OKS trung bình tăng từ `0.8572` lên `0.9048`, OKS@0.50 đạt `1.0`, và lỗi đảo trái/phải giảm từ 3 xuống 1. Tuy nhiên vẫn cần xem lại `train_13`, `train_15`, `train_16` và các cảnh báo `v=0` ở `train_04`, `train_10`, `train_13`. Fine-tune chỉ tăng pose mAP50-95 thêm `0.0055`, nên kết quả hiện tại phù hợp để báo cáo vòng lặp dữ liệu, chưa đủ để khẳng định model đã tốt hơn đáng kể trong thực tế.
