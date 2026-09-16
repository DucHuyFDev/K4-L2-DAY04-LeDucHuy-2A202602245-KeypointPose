# Visibility report

- Thư mục nhãn: `dataset/labels/train`
- 2 ảnh, 3 skeleton, trung bình 15.67 khớp có v > 0 mỗi người
- Tổng: v=2 39 | v=1 8 | v=0 4

So sánh với `../ban_cung_nhom/dataset/labels/train` (0 skeleton).
Cột **lệch** là hiệu số phần trăm v=1 - chỗ nào lệch nhiều nhất là chỗ guideline chưa nói rõ.

| # | Khớp | %v=1 (bạn) | %v=1 (đối chiếu) | lệch |
| ---: | --- | ---: | ---: | ---: |
| 0 | nose | 33% | 0% | 33 |
| 1 | left_eye | 33% | 0% | 33 |
| 2 | right_eye | 33% | 0% | 33 |
| 3 | left_ear | 33% | 0% | 33 |
| 4 | right_ear | 33% | 0% | 33 |
| 7 | left_elbow | 33% | 0% | 33 |
| 9 | left_wrist | 33% | 0% | 33 |
| 10 | right_wrist | 33% | 0% | 33 |
| 5 | left_shoulder | 0% | 0% | 0 |
| 6 | right_shoulder | 0% | 0% | 0 |
| 8 | right_elbow | 0% | 0% | 0 |
| 11 | left_hip | 0% | 0% | 0 |
| 12 | right_hip | 0% | 0% | 0 |
| 13 | left_knee | 0% | 0% | 0 |
| 14 | right_knee | 0% | 0% | 0 |
| 15 | left_ankle | 0% | 0% | 0 |
| 16 | right_ankle | 0% | 0% | 0 |

## Đọc bảng này thế nào

1. Khớp nào có **%v=1 cao**: khớp hay bị che. Cổ tay và hông thường là hai vị trí cần xem lại guideline trước khi kết luận.
2. Khớp nào có **v=0 cao bất thường**: mọi người đang dùng Outside ở chỗ đáng lẽ là Occluded. Đó là lỗi số 3 của slide 46, và nó xoá thẳng khớp đó khỏi bảng điểm OKS.
3. Khi so hai người: **lệch lớn = bất đồng về guideline**, không phải về bức ảnh. Sửa guideline trước, sửa nhãn sau.
