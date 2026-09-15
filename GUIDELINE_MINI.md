# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Minh Quân`
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

Bổ sung của nhóm (nếu có): `Không có`

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 30 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | `có thể xe bị che trong khoảng thời gian ngắn nhưng xe chưa đi qua khỏi ảnh` |
| Xe bị che lâu hơn ngưỡng trên | `xe bị che lâu hơn 30 frame sẽ đc găn id mới` | `xe bị che quá lâu đồng nghĩa với có thể lúc khuất xe đã đi ra ngoài khung ảnh` |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | `id của xe sẽ là id mới vì coi đó là xe mới` |
| Hai xe cắt nhau / chồng lên nhau | `coi là 2 id khác nhau` | `tránh việc 2 xe gần nhau bị lẫn id vào nhau` |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: `hiển thị pixel của xe` |
| Xe đang đỗ, không di chuyển | `luôn track xe dù xe có đỗ hay di chuyển` |
| Keyframe đặt dày ở đâu | `khi 2 xe đè lên nhau` |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `64`
- Tình huống: `xe con rất nhỏ chỉ nhìn đc góc đèn của xe`
- Quyết định: `ko gắn id`
- Lý do: `chưa đủ chứng cứ để gắn id`

### Ca 2
- Clip / frame / ID: `89`
- Tình huống: `xe bị che còn lại rất nhỏ`
- Quyết định: `vẫn giữ id`
- Lý do: `do vẫn nhận ra xe`

### Ca 3
- Clip / frame / ID: `49`
- Tình huống: `xe chỉ lộ ra 1 chút pixel`
- Quyết định: `ko nhận diện xe`
- Lý do: `chưa đủ chứng cứ để gắn id`

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

1. **Ngưỡng bắt đầu track (Entry threshold)**:
   - *Cũ*: "Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; hiển thị pixel của xe" — quá định tính, dẫn đến việc gán sớm 3 frame ở Track 4 khi xe chỉ mới là vệt mỏng < 10px ở rìa ảnh.
   - *Sửa lại*: Chỉ bắt đầu tạo track khi phần nhìn thấy của xe đạt tối thiểu chiều rộng $\ge 15\text{ px}$ hoặc diện tích $\ge 200\text{ px}^2$, đồng thời phải nhận diện được tối thiểu 2 đặc trưng kết cấu của xe (đèn, bánh xe, kính xe).

2. **Luật che khuất nặng khi xe mới xuất hiện (Occlusion on entry)**:
   - *Cũ*: Chưa có quy định cho xe mới xuất hiện từ phía sau vật thể lớn khác.
   - *Sửa lại*: Nếu xe mới xuất hiện nhưng bị xe khác phía trước che lấp $> 85\% - 90\%$ (như trường hợp Track 5 ở frame 66–78 bị xe buýt che), không vội gán nhãn cho đến khi xe vượt lên lộ diện $> 15-20\%$ thân xe.

3. **Luật kết thúc track ở biên ảnh (Exit threshold)**:
   - *Cũ*: "bbox chạm đúng rìa, không đoán phần ngoài ảnh".
   - *Sửa lại*: Phải bấm `outside` dứt khoát ngay tại frame đầu tiên mà phần nhìn thấy của xe chạm mép cắt và diện tích còn lại trong khung hình $< 15\text{ px}$. Tuyệt đối không kéo dài bbox thêm 1-2 frame sau khi xe đã thoát khỏi tầm nhìn.

4. **Danh mục loại trừ vật thể tĩnh dạng khối hộp**:
   - *Bổ sung*: Biển báo chỉ dẫn giao thông trên cao treo trên giá long môn, quầy ki-ốt / trạm gác ven đường dù có dạng khối hộp chữ nhật đối xứng cũng tuyệt đối không gán nhãn (tránh nhầm lẫn như Ghost Track 7 của model).
