# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Tên: `Nguyễn Văn Tuyển`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung (nếu có): `Không`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Tránh làm tăng IDSW (ID Switch) vô lý khi xe chỉ bị che tạm thời trong thời gian ngắn. |
| Xe bị che lâu hơn ngưỡng trên | Bấm outside khi bị che hoàn toàn và tạo track mới khi xe xuất hiện lại | Sau 25 frame bị che hoàn toàn, tracker không đủ căn cứ ngoại hình để chắc chắn đó là cùng một xe. |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới** (tạo ID mới) | Đảm bảo đúng chuẩn MOT Challenge; không nối track nếu xe đã đi ra ngoài viền ảnh. |
| Hai xe cắt nhau / chồng lên nhau | Xe ở phía trước giữ nguyên bbox và ID; xe phía sau gán bbox phần nhìn thấy được và giữ nguyên ID | Duy trì track ID liên tục cho cả hai xe, tránh bị nhảy ID sang xe bị che khuất. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `xe xuất hiện > 30% diện tích thiết bị / kích thước nhận diện.` |
| Xe đang đỗ, không di chuyển | `Gán nhãn như xe đang chạy, đặt keyframe cố định bbox ở frame đầu và frame cuối của chuỗi đỗ.` |
| Keyframe đặt dày ở đâu | `Đặt dày tại các frame xe chuyển hướng (cua), thay đổi tốc độ nhanh, hoặc khi bắt đầu/kết thúc bị che khuất.` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / Frame 105–115 / ID 6`
- Tình huống: `Xe bị che khuất một phần bởi xe buýt ở frame 105–110, sau đó xuất hiện lại ở frame 115.`
- Quyết định: `Giữ nguyên ID 6 cho xe, không tạo track mới khi xe xuất hiện lại.`
- Lý do: `Xe chỉ bị che khuất một phần dưới ngưỡng 25 frame, tuân thủ guideline về duy trì ID cho xe bị che tạm thời.`
 
### Ca 2
- Clip / frame / ID: `clip_01 / Frame 165-170 / ID 5`
- Tình huống: `Xe con màu đỏ đi sát ra mép khung hình bên phải, chỉ còn thấy một phần nhỏ đuôi xe trước khi mất hút hoàn toàn.`
- Quyết định: `Vẽ bbox chạm sát mép ảnh đến frame 77, sang frame 78 bấm outside để kết thúc track.`
- Lý do: `Tránh lỗi bbox thừa (FP) kéo dài sau khi xe đã đi hoàn toàn ra khỏi khung hình.`

### Ca 3
- Clip / frame / ID: `clip_02 / Frame 7-10 / ID 1`
- Tình huống: `Xe con màu trắng đi sát ra mép khung hình bên trái, chỉ còn thấy một phần nhỏ đuôi xe trước khi mất hút hoàn toàn.`
- Quyết định: `Vẽ bbox chạm sát mép ảnh đến frame 7, sang frame 10 bấm outside để kết thúc track.`
- Lý do: `Theo guideline, xe ở rìa khung hình chỉ xuất hiện một góc nhỏ: Chỉ bắt đầu tạo track khi thấy rõ một phần diện tích xe theo đúng guideline.`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- `Xử lý occlusion ngắn (< 5 frame): Quy định rõ ràng việc giữ nguyên ID và bấm outside trong các frame bị che 100%, thay vì tạo ID mới hay cố suy đoán vẽ bbox đè lên vật che khuất.`
- `Ngưỡng xuất hiện ở mép ảnh: Bổ sung quy tắc chỉ bắt đầu tạo Track ID mới khi xe đã vào khung hình và nhìn rõ được trên 30% diện tích xe, tránh gán nhãn cho các điểm ảnh lập lòe khó xác định ở góc viền.`
