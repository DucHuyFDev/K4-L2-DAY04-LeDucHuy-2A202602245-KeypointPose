# Mini guideline - nhóm: ______  |  người gán: ______  |  ngày: ______

> Báo cáo kết quả tương ứng: `reports/REPORT.md`.
# Guideline Gán Nhãn Pose

## 1. Phạm vi

- Bộ nhãn sử dụng 17 keypoint COCO theo đúng tên và thứ tự trong `tools/poselib.py`.
- Mỗi người trong ảnh có một skeleton đầy đủ 17 điểm.
- Không xóa keypoint khó nhìn; dùng visibility để mô tả trạng thái.
- Trái/phải được xác định theo cơ thể người, không theo phía trái/phải của ảnh.
- Không dùng cờ `Hidden` (`h`) vì định dạng YOLO cần `v=0`, `v=1` hoặc `v=2`.

## 2. Quy tắc visibility

| Tình huống | Gán | Cách đặt điểm |
| --- | --- | --- |
| Nhìn thấy rõ keypoint | `v=2` | Đặt đúng tâm giải phẫu nhìn thấy |
| Bị vật/người/trang phục che nhưng vị trí còn nằm trong ảnh | `v=1` | Ước lượng vị trí dựa trên các đoạn xương liền kề |
| Keypoint nằm ngoài mép ảnh, không thể quan sát trong khung | `v=0` | Đặt tọa độ `0 0` |

Không dùng `v=0` chỉ vì keypoint khó nhìn. Nếu phần cơ thể còn trong khung và có thể suy ra vị trí, phải dùng `v=1`.

## 3. Luật cụ thể của nhóm

| Tình huống | Luật áp dụng | Bằng chứng cần dùng |
| --- | --- | --- |
| Hông dưới quần áo dài | Gán `v=1` nếu hông bị vải che nhưng còn trong box; đặt theo trục vai-gối và hình dáng thân | Vị trí vai, eo, đầu gối và hướng thân |
| Tai bị tóc hoặc mũ che | Gán `v=1` nếu vùng tai còn trong ảnh và có thể suy ra từ mắt/mặt; chỉ `v=0` khi tai ra ngoài mép ảnh | Vị trí mắt, đường mặt và mép ảnh |
| Người bị cắt ở mép ảnh | Các điểm đã ra ngoài ảnh dùng `v=0` và `0 0`; điểm còn trong ảnh vẫn phải gán `v=1` hoặc `v=2` | Mép ảnh và phần xương liền kề |
| Cổ tay sau tay lái hoặc thân người | Gán `v=1`, đặt theo hướng cẳng tay-bàn tay và khớp khuỷu | Trục khuỷu-cổ tay và vật che |
| Hai người chồng lên nhau | Giữ skeleton theo từng người; không lấy keypoint của người bên cạnh | Box, vai-hông và chuỗi xương của chính người đó |
| Người quá nhỏ | Vẫn gán nếu nhận diện được người và box; dùng `v=1` cho điểm bị che, không tự ý bỏ skeleton | Box người và các keypoint còn nhìn thấy |

## 4. Các ca mơ hồ đã gặp

### Ca 1: `train_11.jpg`, người 1, hai ankle

- Hai ankle bị che bởi vật thể ở phần dưới ảnh và tọa độ gốc nằm ngoài ảnh.
- Gán `left_ankle` và `right_ankle` là `v=0`, tọa độ `0 0`.
- Không giữ tọa độ y lớn hơn 1 khi visibility là `v=1`.
- Nếu người khác giữ hai điểm này là `v=1`, file sẽ vi phạm chuẩn hóa và model học vị trí ngoài ảnh.

### Ca 2: `train_13.jpg`, người 1, các keypoint chân

- Checker cảnh báo có bốn điểm `v=0` trong khi box người nằm gọn trong ảnh.
- Theo guideline, cần kiểm tra ảnh: nếu chân bị vật/người che nhưng vẫn trong khung thì đổi sang `v=1` và đặt điểm ước lượng.
- Chỉ giữ `v=0` nếu điểm thật sự nằm ngoài mép ảnh.
- Đây là khác biệt giữa “bị che” và “ra ngoài khung”, không được quyết định chỉ vì không thấy bề mặt keypoint.

### Ca 3: `train_15.jpg` và `train_16.jpg`, vai/hông trái phải

- Checker phát hiện quan hệ trái/phải của vai và hông không cùng hướng với hai mắt.
- Cần xác định trái/phải theo cơ thể người, sau đó kiểm tra lại cả cặp vai và cặp hông cùng lúc.
- Không sửa riêng một điểm vì có thể làm skeleton bị chéo.
- Đây là lỗi nguy hiểm hơn lệch vài pixel vì model sẽ học sai nhãn trái/phải.

## 5. Quy trình kiểm tra trước khi train

```bash
python3 tools/check_pose_labels.py \
  --images dataset/images/train \
  --labels dataset/labels/train

python3 tools/visibility_report.py \
  --labels dataset/labels/train \
  --out outputs/visibility_report.json \
  --markdown reports/visibility_report.md

python3 tools/evaluate_pose_annotations.py \
  --pred dataset/labels/train \
  --gold gold/labels/train \
  --images dataset/images/train \
  --out outputs/eval_vs_gold.json
```

Kết quả checker phải kết thúc bằng `ĐẠT định dạng`. Cảnh báo vẫn cần xem lại, đặc biệt là đảo trái/phải và các điểm `v=0` trong box nằm giữa ảnh.

## 6. Quy tắc bằng chứng

Khi phân vân giữa `v=1` và `v=0`, trả lời hai câu hỏi:

1. Vị trí keypoint có còn nằm trong biên ảnh không?
2. Có thể ước lượng vị trí từ xương liền kề, box hoặc tư thế cơ thể không?

Nếu còn trong ảnh và ước lượng được, dùng `v=1` và đặt điểm. Chỉ dùng `v=0` khi keypoint đã ra ngoài ảnh hoặc không tồn tại trong vùng quan sát.
