# Báo cáo Day 13 — Tạ Đức Thắng, 2A202603163

## 1. Kết luận trước

- Mã số: 2A202603163 · task `D13-2A202603163`
- 10 frame: s01f13, s01f31, s02f21, s02f37, s03f22, s03f45, s04f25, s04f52, s05f15, s05f37
- Windows 11 · trình duyệt: COCCOC · CVAT 2.82.1 · bạn ngồi cạnh: 2A202603043

Tôi vẽ **68 cuboid** dạng Shape: s01f13 = 8, s02f21 = 9, s02f37 = 9, s03f22 = 8, s03f45 = 7, s04f25 = 7, còn s01f31, s04f52, s05f15, s05f37 mỗi frame 5. Các frame chỉ có 5 hộp là frame ít vật gần và rõ: ví dụ ở s01f31 có 2/5 hộp nằm xa hơn 35 m, ở s05f37 cả 5 hộp đều xa hơn 22 m. `selfcheck` còn cờ **8/68** cuboid, giảm từ 60/68 trước khi sửa lỗi xoay 90°. Phần tôi tin nhất là **đáy**: trong 64 hộp có ước được mặt đường, 61 hộp có `ground_gap_m` trong khoảng ±0,3 m. Phần còn yếu nhất là **hướng**: phần lớn hộp được sửa từ yaw 0° sang 270° bằng phương pháp hàng loạt, nên hướng đầu xe phải kiểm lại trên ảnh camera. Ngoài ra chưa bật occluded cho hộp nào (0/68), và người đi bộ đang để yaw = 0°.

Cách dùng CVAT:

- Đổi góc nhìn: kéo chuột trái để xoay, kéo chuột phải để dời, lăn để zoom. Xoay khung 3D tới khi nhìn thẳng xuống để có ảnh nhìn từ trên xuống.
- Vẽ cuboid: chọn label → **Shape** → nháy đúp lên cụm điểm. Chỉnh tâm, dài, rộng, yaw ở **Top**; chỉnh đáy và chiều cao ở **Side/Front**.
- Bật **Appearance → Cuboid orientation** để thấy vạch đỏ `+X`. Xem cuboid trên 7 ảnh camera bằng `python scripts/lab.py preview` (ảnh trong `work/xem/`).

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ. Ảnh: `t1-s01f13-3d`, `t1-s01f13-top`, `t1-s01f31-3d`, `t1-s01f31-top`.

| Frame  | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu                                     | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m)                                                                                 | Vật xa nhất đọc được (~ m)                           | Loại vật thấy        | Vật phía sau / hai bên                                                                                             | Cụm chưa đọc được                                                                                                                                                                                            |
| ------ | ---------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------- | -------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| s01f13 | Phía trước (−y) ở **cạnh trên** của ảnh; +x chỉ sang trái ảnh (bên trái xe), +y chỉ xuống dưới (ra sau xe) | Gần phẳng; đáy các hộp từ −1,75 m (gần) tới −1,96 m (khoảng 36 m phía sau), tức mặt đường thấp hơn LiDAR khoảng 1,7–1,9 m | ≥ 36 m (hộp xa nhất đã vẽ: Car khoảng 36 m phía sau) | Car, Bus, Pedestrian | Phía sau: 3 xe con ở khoảng 21 m, 30 m, 36 m; sau-trái: 1 xe con khoảng 36 m; sau-phải: 1 người đi bộ khoảng 9,5 m | Cụm điểm dày dạng khối chữ nhật, không có hộp, sát bên phải xe (trong khoảng 10 m, hướng 3 giờ). Trên point cloud không đọc được là gì; đối chiếu cam07/cam08 thì có thể là nhà chờ xe buýt và trụ quảng cáo |
| s01f31 | Phía trước (−y) ở **cạnh trên** của ảnh; +x chỉ sang trái ảnh, +y chỉ xuống dưới                           | Gần phẳng; đáy các hộp gần khoảng −1,71 m đến −1,81 m                                                                     | ≥ 44 m (Car khoảng 44 m phía sau)                    | Car                  | Phía sau: xe con khoảng 8,7 m và 44 m; sau-trái: xe con khoảng 17 m và 38 m                                        | Dải điểm rất dày và rối, không có hộp, kéo dài dọc bên phải xe (khoảng 5–15 m). Đối chiếu cam07/cam08 thì có thể là dải cây bụi kèm hàng xe đạp, xe máy điện đỗ kín                                          |

Chỉ LiDAR cho biết (từ hai frame trên): (1) khoảng cách theo mét, ví dụ xe buýt phía trước ở s01f13 cách khoảng 30,5 m; (2) kích thước thật theo mét, ví dụ xe buýt đó dài khoảng 14,3 m, cao khoảng 3,1 m; (3) vật ở phía sau và hai bên mà cam01 không thấy, ví dụ 3 xe phía sau ở s01f13 nằm trong vùng 360° của LiDAR. Chỉ camera cho biết: (1) màu sắc, ví dụ xe con màu xanh sát phía trước-trái ở s01f13, dải cây bụi lá đỏ ở s01f31; (2) biển báo, đèn giao thông và vạch kẻ đường (mũi tên làn, vạch vàng trạm buýt, vạch người đi bộ); (3) bản chất của các cụm điểm không có hộp: nhà chờ xe buýt ở s01f13, hàng xe đạp đỗ ở s01f31, rào chắn; và điều kiện cảnh: ban ngày, trời nắng.

Tám frame lướt:

| Frame  | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có)                                                                                                    |
| ------ | ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------- |
| s02f21 | Cạnh trên                                        | Không                                                                                                               |
| s02f37 | Cạnh trên                                        | Không                                                                                                               |
| s03f22 | Cạnh trên                                        | Không                                                                                                               |
| s03f45 | Cạnh trên                                        | Không                                                                                                               |
| s04f25 | Cạnh trên                                        | Không                                                                                                               |
| s04f52 | Cạnh trên                                        | Không                                                                                                               |
| s05f15 | Cạnh trên                                        | Một xe ở làn đối diện bị bụi cây che (thấy ở cam05, lộ một phần ở cam09); hộp của xe này chưa bật occluded          |
| s05f37 | Cạnh trên                                        | Xe buýt xanh ở làn giữa phía trước (khoảng 35 m) bị gán nhầm class Car (s05f37#4), nằm cạnh xe buýt vàng đã gán Bus |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật                              | Frame, idx | Class      | Mũi tên yaw đúng hướng?                                                                                                                                                        | Đáy chạm đường?                                    | Đã kiểm Top/Side/Front + ảnh?                                                               | Kích thước trong bảng 4.3?                                                                             |
| -------------------------------- | ---------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| (a) xe con gần                   | s01f13, 1  | Car        | yaw = 269,9° (cùng chiều xe thu dữ liệu). Chưa xác nhận riêng đầu xe trên ảnh                                                                                                  | Có: z đáy = −1,745 m, `ground_gap_m` = 0,01 m      | Đã đối chiếu ảnh camera (`preview`): xe con màu xanh sát phía trước-trái, trong khoảng 10 m | Có: 4,97 × 1,84 × 1,48 m, nằm trong khoảng sedan                                                       |
| (b) xe buýt hoặc xe tải          | s01f13, 3  | Bus        | Ban đầu **sai**: yaw = 106,2° (ngược đầu) trong khi ảnh camera cho thấy xe buýt chạy phía trước, cùng chiều. Đã xoay 180° thành 286,2° (còn lệch khoảng 16° so với trục đường) | Có: z đáy = −1,620 m, `ground_gap_m` = 0,13 m      | Đã đối chiếu ảnh camera (`preview`): xe buýt ở phía trước, khoảng 30–35 m                   | Một phần: cao 3,06 m trong khoảng; dài 14,33 m vượt xe buýt thường (10–12,5 m); rộng 2,93 m vượt 2,6 m |
| (c) người đi bộ hoặc xe hai bánh | s01f13, 5  | Pedestrian | Chưa: yaw = 0,0° (mặc định, chưa đặt theo hướng đi)                                                                                                                            | Hơi cao: z đáy = −1,426 m, `ground_gap_m` = 0,29 m | Đã đối chiếu ảnh camera (`preview`)                                                         | Có: 0,53 × 0,65 × 1,72 m, nằm trong khoảng người lớn                                                   |

Vòng hiệu chuẩn: tôi không ghi lại chi tiết độ lệch so với các bạn trong lớp.

Vật bỏ qua:

- Xe đạp, xe máy **đỗ không có người** trên vỉa hè hoặc sau dải cây (s01f31, s02f21, s02f37). Theo guideline, loại này không vẽ.
- Xe ở làn đối diện bị che khuất hoặc ở xa ngoài khoảng 35 m: xe van trắng bên trái ở s05f15, xe đen làn ngược chiều ở s04f52, xe buýt xa ở s03f45, xe xa ở ngã tư ở s03f22 và s04f25 (LOW_VISIBILITY / UNCERTAIN_SCOPE).
- Người đi bộ ở vỉa hè xa ở s02f21 (LOW_VISIBILITY).
- Vật hạ tầng không thuộc 5 class (nhà chờ xe buýt, cọc tiêu, rào chắn, cây bụi).

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng                           | Frame / tên ảnh / nguồn                                                  | Nguyên nhân (giả thuyết)                                                                                                                                                 | Thấy gì trên point cloud                                                                                                                                                                                                 | Hệ quả khi gán (theo guideline)                                                                                                                      |
| ------------------------------------ | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1. Bóng che                          | s05f15, task của tôi · `t4-1-bong-che`                                   | Giả thuyết: tia LiDAR bị vật lớn phía trước chặn lại nên không tới được vùng phía sau vật đó                                                                             | Xe SUV sát bên phải (s05f15#2, cách 4,64 m, 5537 điểm) để lại một vùng trống lớn phía sau nó. Một xe ở làn đối diện bị bụi cây che, chỉ lộ một phần (cam05, cam09)                                                       | Vùng trống sau vật che là **chưa quan sát được**, không phải đường trống. Vật bị che một phần thì bật **occluded**, cuboid bao cả phần suy luận được |
| 2. Thưa ở xa                         | s05f37, task của tôi · `t4-2-thua-o-xa`                                  | Giả thuyết: các tia LiDAR toả ra theo góc nên càng xa khoảng cách giữa các tia càng lớn, ít tia trúng vật                                                                | Hai xe buýt phía trước nằm sát hoặc ngoài 35 m. Xe buýt vàng (s05f37#2) cách 62,27 m dù dài khoảng 14 m chỉ còn 111 điểm; xe con s05f37#5 cách 48,55 m chỉ còn 45 điểm. Hộp Bus ở 62,27 m nằm ngoài phạm vi 50 m của lab | Thấy khoảng **15–20% thể tích** hoặc suy luận được thì vẽ, kích thước đối chiếu bảng 4.3; ít hơn thì bỏ qua, ghi **LOW_VISIBILITY**                  |
| 3. Phản xạ thấp / ít điểm bất thường | s01f31, task của tôi · `t4-3-phan-xa-thap`                               | Giả thuyết: bề mặt tối màu hoặc kính phản xạ ít năng lượng về cảm biến (hoặc phản xạ gương đi hướng khác), nên ít điểm và intensity thấp; cũng có thể do bị che một phần | Xe con s01f31#5 cách 37,70 m chỉ có 49 điểm, ít hơn xe s01f31#4 ở xa hơn (43,99 m, 53 điểm). Trên camera, bên phải có một xe tối màu nằm sau hàng cây                                                                    | Chỉ ghi giả thuyết, **không khẳng định** nguyên nhân. Kích thước dựa vào bảng 4.3 và ảnh camera, không thu hộp theo số điểm ít                       |
| 4. Điểm lạ                           | s02f35, mượn từ task chung `D13-HC485087` (frame gợi ý) · `t4-4-diem-la` | Giả thuyết: phản xạ nhiều lần (multipath) qua bề mặt phản quang, hoặc nhiễu của cảm biến, tạo điểm ở vị trí không có vật thật                                            | Cụm khoảng 46 điểm phía sau bên trái, cách khoảng 29 m, ở z ≈ −13,5 m, tức thấp hơn mặt đường quanh đó khoảng 11 m. Phải xoay nghiêng khung 3D mới thấy rõ                                                               | **Không gán thành vật**, không vẽ cuboid                                                                                                             |

## 5. Số từ cuboid của tôi (Bước 5)

Số lấy từ `submission/D13-2A202603163-bangA.csv`; số điểm và khoảng hở lấy từ `-tukiem.csv`.

| Cuboid (frame, idx, class) | z        | h       | z đáy = z − h/2 | dist_m  | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường)                                                                                              |
| -------------------------- | -------- | ------- | --------------- | ------- | --------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| s01f13, 1, Car             | −1,004 m | 1,482 m | −1,745 m        | 5,25 m  | 5,34 m          | 269,9°  | Chạm đường: lệch −0,05 m so với −1,7 m; `ground_gap_m` = 0,01 m                                                                     |
| s03f22, 4, Car             | −0,945 m | 1,384 m | −1,637 m        | 17,28 m | 17,31 m         | 270,0°  | Gần −1,7 m nhưng **chìm** so với mặt đường quanh hộp: `ground_gap_m` = −0,34 m (mặt đường ở đó cao hơn −1,7 m)                      |
| s05f37, 4, Car             | −0,846 m | 2,393 m | −2,043 m        | 35,28 m | 35,29 m         | 270,0°  | Lệch 0,34 m so với −1,7 m nhưng **chạm đường** theo mặt đường quanh hộp (`ground_gap_m` = 0,03 m); mặt đường ở khoảng 35 m thấp hơn |

Hai hộp sau cho thấy mốc −1,7 m chỉ đúng gần xe: đường dốc thì nên so với mặt đường quanh hộp.

(a) Cuboid **gần nhất**: s05f15#2 Car, `dist_m` = 4,64 m, r = 4,70 m. Cuboid **xa nhất được chấm**: s05f37#5 Car, `dist_m` = 48,55 m, r = 48,56 m. Hộp xa nhất trong bảng A là s05f37#2 Bus, `dist_m` = 62,27 m, r = 62,28 m: hộp này vượt 50 m nên không được chấm. Ở khoảng cách này r gần bằng `dist_m`, vì |z| (dưới 1,2 m) rất nhỏ so với x, y.

(b) Kích thước. Người đi bộ s01f13#5: 0,53 × 0,65 × 1,72 m, cả ba chiều nằm trong khoảng người lớn (0,4–0,9 × 0,5–0,9 × 1,50–1,95 m). Xe con s01f13#1: 4,97 × 1,84 × 1,48 m, nằm trong khoảng sedan (3,8–5,0 × 1,70–1,90 × 1,40–1,60 m). Hộp ngoài khoảng: s01f13#7 Car 6,40 × 2,68 × 2,31 m, cả ba chiều vượt khoảng xe con. Hộp này chỉ có 65 điểm ở khoảng 36 m, nên có thể đã vẽ rộng hơn vật thật, hoặc đó là xe cỡ lớn hơn (sai class). Cần kiểm lại trên ảnh camera.

(c) Yaw. Xe cùng chiều phía trước s02f21#1 (`az_deg` = −3,4°, cách 13,40 m): yaw = 270,0°, đúng mốc khoảng 270°. Xe ngược chiều s01f13#2 (cách 18,98 m, phía trước bên trái): yaw = 88,1°, gần mốc 90°.

Mật độ theo khoảng cách (Car):

| Cuboid (frame, idx, class) | dist_m  | Số điểm (`points_in`) |
| -------------------------- | ------- | --------------------- |
| s04f52, 1, Car             | 6,52 m  | 3972                  |
| s02f21, 1, Car             | 13,40 m | 665                   |
| s02f37, 1, Car             | 29,51 m | 158                   |
| s01f31, 4, Car             | 43,99 m | 53                    |

Quy luật: xa gấp khoảng 2 lần (6,5 → 13,4 m) thì số điểm còn khoảng 1/6; xa gấp khoảng 4,5 lần (→ 29,5 m) thì còn khoảng 1/25; xa gấp khoảng 6,7 lần (→ 44 m) thì còn khoảng 1/75. Số điểm giảm nhanh hơn một chút so với tỉ lệ 1/d². So với đường cong ở slide: 665 điểm ở 13,4 m gần mức 718 điểm (10–20 m); 158 điểm ở 29,5 m nằm giữa 214 (20–30 m) và 72 (30–40 m); 53 điểm ở 44 m nằm giữa 72 (30–40 m) và 19 (60–80 m). Riêng 3972 điểm ở 6,5 m cao hơn mức trung vị 1968 của nhóm 0–10 m, vì xe này rất gần, ở cạnh bên.

Tự kiểm. Các cuboid đã sửa sau `selfcheck` (sửa cùng AI, xem §6b):

- 57 hộp có cờ `length < width`: đổi chỗ dài/rộng và xoay yaw −90°. Ví dụ s03f45#4 Cyclist trước là 0,76 × 1,84 m, sau là 1,84 × 0,76 m.
- s03f22#2 Car: đáy chìm 0,26 m, đã nâng lên 0,26 m. Hiện `ground_gap_m` = 0,00 m.
- s02f21#9 Car: đáy lơ lửng 0,32 m, đã hạ xuống 0,32 m. Hiện `ground_gap_m` = 0,00 m.
- s05f15#4 Truck: rộng 3,79 m, đã thu về 2,50 m.

Các cờ còn lại (8 hộp): s01f13#5 (quá chật), s01f13#6 (rộng 2,57 m), s01f13#7 (6,40 × 2,68 × 2,31 m), s02f37#6 (rộng 2,41 m), s03f22#3 (cao 1,10 m), s03f22#4 (chìm 0,34 m), s04f52#3 (6,30 × 2,35 m, có thể là bán tải → Truck), s05f37#4 (cao 2,39 m; ảnh camera cho thấy đây là xe buýt xanh bị gán nhầm Car). Ngoài ra s01f13#3 Bus đã được xoay 180°, vì ảnh camera cho thấy xe buýt chạy cùng chiều mà hộp ban đầu ngược đầu. Tám hộp này hiện giữ nguyên: không chỉnh chỉ để tắt cờ. Kích thước và class của chúng cần đối chiếu thêm với ảnh camera; riêng s03f22#4 là ứng viên sửa đáy tiếp theo (ghi ở §6c).

đổi kiểm: 2A202603043 người đã kiểm chéo với bạn, hai con số nào được đối chiếu với bảng A, khớp.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

**(a)** Model 3D dự đoán vị trí, kích thước và hướng của vật trong không gian thật để xe lập kế hoạch đi. Nhãn đo bằng mét thì model học được khoảng cách và kích thước không phụ thuộc vào camera hay độ phân giải ảnh: một xe con luôn dài khoảng 4–5 m dù ở gần hay xa, còn số pixel thì đổi theo khoảng cách. Vật ở xa chỉ còn vài chục điểm (ví dụ 53 điểm ở 44 m so với 3972 điểm ở 6,5 m). Nếu không có quy tắc riêng (ngưỡng 15–20% thể tích, LOW_VISIBILITY), mỗi người sẽ vẽ hoặc đoán theo một kiểu: model học phải nhãn không nhất quán hoặc nhãn "bịa" ở vùng không đủ dữ liệu, dễ báo sai vật hoặc sai kích thước ở xa.

**(b) Dùng AI:**

- Công cụ: Claude Code (Claude Opus 5.5), chạy trên máy cá nhân. Không đưa cho AI ảnh camera, ảnh chụp màn hình, `work/xem/` hay file trong `data-VF`.
- Câu hỏi chính: hướng dẫn các bước làm lab, cài môi trường và CVAT, giải thích cờ `length < width`; viết script PowerShell tự kiểm bảng A (`kiem_frame.ps1`, ngoài repo).

**(c) Điều còn chưa chắc:**

- Hướng đầu xe của các hộp sửa hàng loạt, nhất là xe bên trái (x > 2,5 m) có thể là xe ngược chiều.
- Chưa bật occluded cho hộp nào, dù có xe ở xa phía sau có thể bị xe khác che.
- Người đi bộ và xe hai bánh có hướng chưa đặt theo hướng đi.
- Đáy s03f22#4 so với mặt đường quanh hộp.
