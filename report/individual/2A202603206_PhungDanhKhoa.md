# Báo cáo Day 13 — Phùng Danh Khoa, 2A202603206

## 1. Kết luận trước
- **Thông tin chung:** MSSV `2A202603206`, task `D13-2A202603206`, gồm 10 frame (`s01f17`, `s01f47`, `s02f04`, `s02f27`, `s03f21`, `s03f56`, `s04f13`, `s04f46`, `s05f04`, `s05f32`). Môi trường thực hiện: `macOS` (Apple M1), trình duyệt `Chrome`, CVAT `v2.83.0`; MSSV bạn ngồi cạnh: `(chờ đổi kiểm trực tiếp trên lớp)`.
- **Tổng quan kết quả:** Tổng cộng có `111` cuboid gán thủ công trên 10 frame (`Car`: `90`, `Bus`: `6`, `Cyclist`: `6`, `Pedestrian`: `6`, `Truck`: `3`). Frame có ít cuboid nhất là `s02f04` (`6` cuboid), tiếp theo là `s02f27` và `s04f46` (cùng `8` cuboid), do tại giao lộ `s02f04` nhiều phương tiện cắt ngang nằm ngoài ngưỡng `>= 50.0 m` hoặc bị che trong bóng của xe `#3`, đồng thời các cụm đa phản xạ ma và dải phân cách được loại bỏ nghiêm ngặt. Phiếu `selfcheck` cờ `7/111` cuboid (6 cảnh báo điểm sát mép từ hàng rào/vỉa hè/nhà chờ buýt kế bên và 1 cảnh báo `ground_gap_m = -0.42 m` do mặt đường làn ngược chiều ở `s03f56 #8 Bus` cao hơn dải bụi cây bên cạnh). Tôi tin nhất vào các hộp trong tầm `0–30 m` có 6 mặt bám sát cụm điểm LiDAR và khớp khung chiếu 7 camera; phần yếu nhất cần dè chừng là chiều dài dọc tia nhìn ở một vài xe xa `39–48.3 m` bị che một phần (`14–41` điểm) phải suy luận nửa khuất theo Bảng 4.3.
- **Cách dùng CVAT 3D:**
  - Đổi góc nhìn: xoay phối cảnh 3D bằng chuột trái, dời khung bằng chuột phải, thu phóng bằng con lăn và chuyển qua lại với mặt bằng Top-down.
  - Vẽ và chỉnh cuboid: dùng công cụ `Shape` (không dùng `Track`), chọn đúng class rồi tinh chỉnh từng mặt trên ba khung `Top` / `Side` / `Front`.
  - Kiểm tra hướng và chiếu ảnh: bật mũi tên hướng `+X` (`Cuboid orientation`) rồi chạy `python scripts/lab.py preview` và `selfcheck` để kiểm khớp trên cả 7 camera.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ (`s01f17`, `s01f47`). Ảnh: `t1-s01f17-3d.png`, `t1-s01f17-top.png`, `t1-s01f47-3d.png`, `t1-s01f47-top.png`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f17` | Phía trước (`-y`) ở **cạnh trên**; `+x` sang **trái**, `+y` xuống **dưới** (ra sau) | Dốc dọc nhẹ `~0.009 m/m` đi xuống về phía trước (`-y`): quanh xe `z ≈ -1.80 m`, phía trước `y = -38.0 m` tụt xuống `z ≈ -2.06 m`, phía sau `y = +36.0 m` nâng lên `z ≈ -1.40 m` | `#7 Car` ở `47.08 m` (`67` điểm, thấy rõ trên `cam09` và `cam05`) | `Car` (`8`), `Bus` (`#2` ở `29.11 m`) | Phía sau (`cam09`/`cam08`/`cam05`): `#3`, `#4`, `#5`, `#6`, `#7 Car`; bên trái qua dải cây phân cách (`cam05`/`cam06`/`cam01`): `#8`, `#9 Car` đi ngược chiều (`90.0°`) | Dải cây xanh dọc `x ≈ +10 m` và `x ≈ -5..-8 m`; các cây non dọc dải phân cách tại `x ≈ +10.2 m`; khung biển báo trên cao phía sau xe (`y ≈ +21..+24 m`, `z ≈ +3.2 m`) |
| `s01f47` | Phía trước (`-y`) ở **cạnh trên**; `+x` sang **trái**, `+y` xuống **dưới** (ra sau) | Mặt đường nhựa chính khá phẳng (`z ≈ -1.76 m` gần xe tới `-1.83 m` ở `y = -35.0 m`); làn xe đạp/vỉa hè bên phải (`x = -8..-12 m`) nâng cao hơn mặt đường `~0.42 m` (`z ≈ -1.35 m`) | `#17 Car` ở `37.87 m` (`31` điểm, trong làn rẽ trái ở `cam03`) | `Car` (`11`), `Truck` (`#3` ở `11.36 m`), `Bus` (`#8` ở `35.42 m`), `Cyclist` (`3`), `Pedestrian` (`1`) | Phía sau (`cam09`): `#11` (`7.10 m`), `#12 Car` (`30.55 m`); trên làn xe đạp/vỉa hè bên phải (`cam08`/`cam07`/`cam03`): `#13`, `#14`, `#15 Cyclist` và `#16 Pedestrian`; bên trái sau cây (`x > +20 m`) bị che khuất | Hàng xe đạp chia sẻ đỗ trống không có người lái (`x ≈ -11 m`, `y = -9..-12 m`), hai người đi bộ bị che nửa dưới sau xe đạp vàng (`x ≈ -11..-13 m`), người áo đỏ lẫn vào cột biển báo (`y ≈ -21.2 m`) |

Từ chính hai frame trên:
- **3 điều chỉ camera cho biết:**
  1. Ở `s01f17`, ảnh `cam01`/`cam03`/`cam06` xác định rõ màu sắc, loại xe (`#2` xe buýt thành phố trắng-đỏ-xanh, `#1` sedan trắng) và hướng đầu xe của các xe bị bụi cây dải phân cách che nửa dưới (`#8`, `#9` nhìn thấy phần kính chắn gió/nóc đi ngược chiều).
  2. Ở `s01f47`, ảnh `cam07` và `cam08` tách bạch hàng xe đạp chia sẻ màu vàng/xanh dựng trống (không gán nhãn) với `#15 Cyclist` đang ngồi trên xe ngay phía sau, đồng thời nhận diện `#16 Pedestrian` đang đi bộ và phân loại `#3 Truck` (xe tải van màu xanh đậm có vạch phản quang đỏ-trắng) cùng `#8 Bus` (xe buýt trung chuyển trắng-đỏ).
  3. Ở `s01f17` và `s01f47`, ảnh `cam03` cho thấy các xe rất xa tại ngã tư phía trước nhưng nằm ngoài phạm vi gán nhãn `>= 50.0 m`.
- **3 điều chỉ LiDAR cho biết:**
  1. Ở `s01f17`, LiDAR đo chính xác độ dốc dọc mặt đường (`z = -1.404 m` ở `y = +36.0 m` xuống `-2.055 m` ở `y = -38.0 m`), điều không thể đọc trực tiếp từ ảnh 2D và quyết định cao độ `z_bottom` đúng cho `#9 Car` (`-2.140 m`) so với `#5 Car` (`-1.525 m`).
  2. Ở `s01f47`, LiDAR phát hiện `#7 Car` (`20.80 m`, `57` điểm trên nắp cốp và nóc ở `z = -0.55..-0.25 m`) nằm ngay phía trước `#2 Car` dù camera gắn trên kính lái (`cam03`/`cam01`) bị `#2` che khuất hoàn toàn, nhờ cảm biến LiDAR trên nóc xe nhìn qua được nóc `#2`.
  3. Ở `s01f47`, LiDAR đo chính xác bậc nâng cao `+0.42 m` của làn xe đạp bên phải (`z ≈ -1.35 m` so với `-1.77 m` mặt đường) và tách rõ khoảng cách dọc thực của hàng xe nối đuôi trên làn 2 (`#1` ở `9.35 m`, `#4` ở `16.27 m`, `#6` ở `22.75 m`, `#8` ở `35.42 m`).

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f04` | Cạnh trên (`-y` lên, `+x` trái, `+y` xuống) | Ngã tư rộng có đường hơi dốc xuống về phía nhánh trái (`z ≈ -2.02 m` ở `#4`); ngay trong nón bóng che sau `#3 Car` xuất hiện cụm đa phản xạ ma `81` điểm tại `(-1.2, -27.2)` trên mặt đường hoàn toàn trống |
| `s02f27` | Cạnh trên (`-y` lên, `+x` trái, `+y` xuống) | Mặt đường dốc xuống phía trước (`-1.69 m` sau xe xuống `-1.90..-2.26 m` phía trước); dải phân cách trắng dài ở `x ≈ +1.9 m` và vách nhà chờ buýt ở `x ≈ +10 m` dễ nhầm thành cạnh xe, kèm các điểm rơi sâu tới `z = -13.72 m` |
| `s03f21` | Cạnh trên (`-y` lên, `+x` trái, `+y` xuống) | Ngã tư đông (`15` cuboid); mặt đường bên nhánh phải/sau-phải dốc cao lên `z ≈ -1.25..-1.55 m` so với `-2.05 m` phía trước; `#5` SUV đen tạo nón bóng che rộng xóa sạch vòng mặt đường phía sau tới `x > 30 m` |
| `s03f56` | Cạnh trên (`-y` lên, `+x` trái, `+y` xuống) | Làn ngược chiều bên trái (`+x`) cao hơn làn xe (`z ≈ -1.48 m` so với `-1.75 m`), đường gom bên phải leo dốc cao tới `-0.3 m`; phía trước `#1 Car` có vệt phản xạ ma lơ lửng ở `(-0.15, -17.0)` trên mặt đường trống |
| `s04f13` | Cạnh trên (`-y` lên, `+x` trái, `+y` xuống) | Đường lên dốc rõ về phía trước (`z ≈ -2.08 m` ở sau lên `-1.72 m` quanh xe và `-1.29 m` ở `y ≈ -42 m`), khiến các vòng mặt đường phía trước `y < -24 m` vượt ngưỡng `-1.55 m` trên thang màu |
| `s04f46` | Cạnh trên (`-y` lên, `+x` trái, `+y` xuống) | Toàn bộ tuyến đường uốn cong nhẹ sang phải (`-x`) về phía trước nên góc `yaw_deg` phải bám tiếp tuyến làn (`264°–269°` phía trước, `272°` phía sau); có vài điểm dị thường dưới mặt đất tới `z = -16.64 m` ở góc `(-9.9, -34.3)` |
| `s05f04` | Cạnh trên (`-y` lên, `+x` trái, `+y` xuống) | Thùng xe tải nặng `#3 Truck` ngay sau xe che kín phần sau thân xe và luồng xe phía sau; dải phân cách lá đỏ (`x ≈ +5.5 m`) che nửa dưới của 4 xe ngược chiều `#9`, `#10`, `#11`, `#13` |
| `s05f32` | Cạnh trên (`-y` lên, `+x` trái, `+y` xuống) | Nhà chờ xe buýt bên phải (`x ≈ -14 m`) có hai áp phích quảng cáo hình người lớn (loại bỏ) đứng ngay cạnh hai người thật ngồi ghế (`#3`, `#4 Pedestrian`) và dãy xe đạp đỗ trống |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f17`, `#1` | `Car` | Đúng (`271.0°`, dọc làn `-y`, khớp `cam01`) | Đúng (`z_bottom = -1.805 m`, `ground_gap_m = 0.09 m`) | Có (Top/Side/Front + `cam01`) | Đúng (`4.400 × 1.780 × 1.470 m`, trong khoảng sedan `3.8–5.0 × 1.70–1.90 × 1.40–1.60 m`) |
| (b) xe buýt hoặc xe tải | `s02f04`, `#2` | `Bus` | Đúng (`270.5°`, hướng `-y`, khớp đầu xe trên `cam09`) | Đúng (`z_bottom = -1.770 m`, `ground_gap_m = 0.00 m`) | Có (Top/Side/Front + `cam09`) | Đúng (`11.800 × 2.600 × 3.260 m`, trong khoảng xe buýt thường `10.0–12.5 × 2.50–2.60 × 3.00–3.50 m`) |
| (c) người đi bộ hoặc xe hai bánh | `s01f47`, `#13` | `Cyclist` | Đúng (`270.0°`, tiến `-y` trên làn xe đạp, khớp `cam08`) | Đúng (`z_bottom = -1.370 m`, chạm mặt làn xe đạp nâng cao `z ≈ -1.35 m`, `ground_gap_m = 0.16 m`) | Có (Top/Side/Front + `cam08`) | Đúng (`1.800 × 0.750 × 1.620 m`, trong khoảng `Cyclist` `1.6–2.3 × 0.50–0.95 × 1.40–1.90 m`) |

- **Vòng hiệu chuẩn & thống nhất guideline:** Áp dụng thống nhất quy chuẩn Bảng 4.3 và các nguyên tắc gán nhãn trên cả 10 frame: (1) tuyệt đối không gán `Cyclist` cho xe đạp/xe máy đỗ trống không có người điều khiển (`s01f47`, `s02f27`, `s03f21`, `s05f32`); (2) phân biệt rõ xe tải hàng hóa (`s01f47 #3`, `s05f04 #3`, `s05f32 #10 Truck`) với xe khách đa dụng cửa kính đầy đủ (`s04f13 #3 Car` dài `5.100 m`, cao `2.180 m`) và xe buýt cỡ nhỏ (`s03f21 #7`, `s01f47 #8 Bus` dài `6.200–6.400 m`); (3) đặt `z_bottom` theo bề mặt cục bộ thực tế (kể cả độ dốc đường `s01f17`/`s04f13` hay làn xe đạp nâng cao `+0.42 m` ở `s01f47`), và trong tầm `30–48 m` chỉ giữ lại các xe hiện rõ đồng thời trên cả LiDAR và camera.
- **Các vật bỏ qua vì không đủ cơ sở:**
  - `UNCERTAIN_SCOPE` (`>= 50.0 m` hoặc xe đỗ trống): các xe xa tại ngã tư phía trước/sau trên `cam03`/`cam09` (`> 50 m`), xe buýt xanh đang đi tới phía sau `s04f13 #7` (mặt trước `y ≈ +43.8 m` nên tâm `~52 m >= 50 m`), hai xe buýt ở `s02f04` (`y < -47.7 m`, tâm `> 52 m`), và các dãy xe đạp đỗ trống ở `s01f47`, `s02f27`, `s03f21`, `s05f32`.
  - `LOW_VISIBILITY` (bị che hoặc quá thưa không đủ bằng chứng đa cảm biến): dải nóc `7` điểm sau `#5` ở `s04f46` (`y ≈ -45.65 m`, khuất hoàn toàn sau `#5` trong `cam03`); xe ẩn sau bụi cây đối diện ở `s05f32` (`x ≈ +13.8 m, y ≈ +38.5 m`, thấy trên BEV nhưng khuất sau tán cây trong `cam09`); cụm `~20` điểm sau `#4` ở `s03f56`; SUV trắng sau `#3` ở `s02f04`; và người đi bộ bị hàng rào/nhà chờ/cây che khuất lẫn cụm nền (`s01f17`, `s01f47`, `s02f27`, `s03f21`, `s05f32`).
  - `UNCERTAIN_CLASS`: cụm đứng lẫn cạnh trụ/biển báo ở `s03f21` (`-13.7, 16.9`) và `s01f47` (`-10.0, -21.2`) không tách được giữa người và vật thể tĩnh trên cả ảnh lẫn LiDAR.

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s01f47` / `t4-1-bong-che.png` / `D13-2A202603206` | **Giả thuyết:** thân xe `#2 Car` (`10.25 m`) chắn các tia LiDAR tầm thấp hướng `-y`, chỉ để các tia cao lướt qua nóc `#2` chạm tới phần trên của `#7 Car` (`20.80 m`) | Trên BEV thấy nón bóng che mất sạch vòng quét mặt đường phía sau `#2` tới `y ≈ -25 m`; trên mặt cắt Side, `#7` chỉ có `57` điểm từ độ cao `z > -0.55 m` trở lên (mất toàn bộ cản và bánh xe), dù trên `cam03` `#7` bị `#2` che khuất hoàn toàn | Dùng LiDAR trên cao xác định vị trí `#7`, bật `occluded = 1`, neo mặt nóc/cốp nhìn thấy và hạ `z_bottom = -1.770 m` chạm mặt đường theo Bảng 4.3 (`4.450 × 1.800 × 1.520 m`) |
| 2. Thưa ở xa | `s04f13` / `t4-2-thua-o-xa.png` / `D13-2A202603206` | **Giả thuyết:** chùm tia dọc của LiDAR phân kỳ theo khoảng cách khiến tại cự ly `> 45 m` chỉ còn vài vòng tia quét trúng mặt sau của xe xa (kèm một phần thân bị xe trước che) | Trong cùng frame, `#2 Car` ở gần (`7.20 m`) thu được `2059` điểm dày đặc đủ sườn, nóc và cản, trong khi `#10 Car` ở xa (`46.07 m`) chỉ còn `29` điểm rời rạc trên cản sau và mép kính hậu | Không bó hẹp hộp vào `29` điểm mà neo chặt mặt sau nhìn thấy trên `cam03` và kéo đủ chiều dài sedan `4.500 × 1.780 × 1.400 m` theo Bảng 4.3 (`occluded = 1`); bỏ qua nếu không xác nhận được trên camera |
| 3. Phản xạ thấp / ít điểm bất thường | `s03f21` / `t4-3-phan-xa-thap.png` / `D13-2A202603206` | **Giả thuyết:** lớp sơn đen bóng của `#2` SUV hấp thụ và tán xạ gương chùm laser hồng ngoại mạnh hơn sơn trắng mờ của `#6` SUV, cộng thêm bụi cây thấp dải phân cách che phần gầm | Ở cự ly tương đương `~20 m`, bên trong hộp `#2` SUV tối màu (`19.36 m`) chỉ trả về `70` điểm (trung vị intensity `0.080`, trung bình `0.108`), còn `#6` SUV trắng (`21.98 m`) trả về `551` điểm (gấp `~7.9` lần; trung vị intensity `0.150`, trung bình `1.275`) | Không dựa vào số điểm ít để đoán nhầm kích thước; đối chiếu `cam01` xác định đúng dáng SUV và hoàn thiện phần thân sau bị che theo Bảng 4.3 (`4.600 × 1.880 × 1.600 m`) |
| 4. Điểm lạ | `s02f35` / `t4-4-diem-la.png` / `task chung D13-HC485087` | **Giả thuyết:** phản xạ đa đường (multipath) qua bề mặt kính hoặc kim loại phẳng gần xe kéo dài giả tạo thời gian bay của xung laser, làm điểm 3D rơi sâu xuống dưới mặt đất | Góc 3D và mặt cắt Side lộ rõ cụm `50` điểm đỏ cô lập tại `(x ≈ 3.0..7.5 m, y ≈ 28.0..29.6 m)` với cao độ `z = -13.49..-10.24 m` (sâu hơn mặt đường thực `-1.9 m` tới `~11 m`), dù `cam09` xác nhận mặt đường hoàn toàn phẳng | Tuyệt đối loại trừ khỏi quá trình gán nhãn và ước lượng mặt đất (`ground_z`). Hiện tượng tương tự cũng gặp ở cụm đa phản xạ ma `81` điểm trên mặt đường trống tại `s02f04` `(-1.2, -27.2)` và `s03f56` `(-0.15, -17.0)` |

## 5. Số từ cuboid của tôi (Bước 5)
Số liệu trích xuất trực tiếp từ `submission/D13-2A202603206-bangA.csv` và `submission/D13-2A202603206-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `(s01f17, #1, Car)` | `-1.070 m` | `1.470 m` | `-1.805 m` | `8.22 m` | `8.29 m` | `271.0°` | Chạm đường (`ground_gap_m = 0.09 m`) |
| `(s02f04, #2, Bus)` | `-0.140 m` | `3.260 m` | `-1.770 m` | `13.42 m` | `13.43 m` | `270.5°` | Chạm đường (`ground_gap_m = 0.00 m`) |
| `(s01f47, #13, Cyclist)` | `-0.560 m` | `1.620 m` | `-1.370 m` | `11.39 m` | `11.41 m` | `270.0°` | Chạm mặt làn xe đạp nâng cao (`z ≈ -1.35 m`, `ground_gap_m = 0.16 m`) |

- **(a) Cuboid gần nhất và xa nhất:** Gần nhất là `s02f04 #1 Car` (`dist_m = 3.00 m`, `r = 3.18 m`); xa nhất là `s05f32 #13 Car` (`dist_m = 48.34 m`, `r = 48.35 m`).
- **(b) Kích thước l, w, h so với Bảng 4.3:** `s01f47 #16 Pedestrian` có `l × w × h = 0.750 × 0.750 × 1.540 m`, nằm gọn trong khoảng người lớn (`0.4–0.9 × 0.5–0.9 × 1.50–1.95 m`); `s01f17 #1 Car` có `l × w × h = 4.400 × 1.780 × 1.470 m`, nằm gọn trong khoảng sedan (`3.8–5.0 × 1.70–1.90 × 1.40–1.60 m`) — không có chiều nào vượt khoảng Bảng 4.3.
- **(c) Hướng `yaw_deg`:** Xe đi cùng chiều phía trước `s01f17 #1 Car` có `yaw_deg = 271.0°` (xấp xỉ `270°`); xe đi ngược chiều `s01f17 #8 Car` có `yaw_deg = 90.0°` (đúng `90°`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `(s05f04, #1, Car)` | `3.37 m` | `5523` |
| `(s04f13, #1, Car)` | `11.70 m` | `1299` |
| `(s03f21, #6, Car)` | `21.98 m` | `551` |
| `(s04f13, #6, Car)` | `30.73 m` | `137` |

- **Quy luật mật độ:** Khi khoảng cách tăng gần `1.9` lần từ `11.70 m` lên `21.98 m`, số điểm giảm còn `0.42` lần (`1299 → 551`), và khi tăng `9.1` lần từ `3.37 m` lên `30.73 m`, số điểm giảm `~40` lần (`5523 → 137`), bám sát đường cong suy giảm mật độ theo `1/r²` trong bài giảng (`~1968` điểm ở `0–10 m`, `718` ở `10–20 m`, `214` ở `20–30 m`, `72` ở `30–40 m`, `19` ở `60–80 m`).
- **Tự kiểm (`preview` / `selfcheck`):**
  - Ba sửa lỗi tiêu biểu qua kiểm tra trực quan 2 vòng: (1) ở `s04f46 #1 Car`, Pass 1 đặt tâm `y = -0.25 m` lệch ra sau khiến khung chiếu đè vào tường xanh trên `cam07`/`cam08` (`820` điểm, `357` điểm sát mép); Pass 2 dời về `y = -4.350 m` ôm trọn `2765` điểm và `gap = 0.00 m`; (2) ở `s01f47`, kiểm tra ảnh cắt phát hiện `#15` và `#16` bị tráo nhãn (`Pedestrian ↔ Cyclist`), đồng thời `#17 Car` bị ngược góc `180°` (`90.0° → 270.0°`) sau khi đối chiếu dải phân cách tại `x = +11.2 m`; (3) ở `s01f17 #9 Car`, hạ `z` từ `-1.10 m` xuống `-1.400 m` để khớp độ dốc `-2.14 m` ở phía trước trên `cam01`/`cam06`.
  - Các cảnh báo giữ nguyên (`7/111`): `s01f17 #4 Car` (`229` điểm từ hàng rào cây sát mép phải `x = -4.8 m`), `s03f21 #3` & `#8 Car` (`108` và `44` điểm từ dải phân cách/cây cảnh sát cạnh), `s03f56 #8 Bus` (`ground_gap_m = -0.42 m` do mặt đường làn ngược chiều tại `z ≈ -1.48 m` cao hơn dải bụi cây xung quanh dù đáy hộp `-1.440 m` khớp mặt đường), `s05f32 #3` & `#4 Pedestrian` (`121` và `132` điểm từ vách/ghế nhà chờ buýt ngay sau lưng hai người đang ngồi), và `s05f32 #8 Car` (`89` điểm từ bụi cây dải phân cách sát cạnh xe) — đều đã kiểm tra trên ảnh cắt và giữ nguyên để không làm phình hộp vào vật nền.
- **Đổi kiểm:** Chưa đổi kiểm trực tiếp trên lớp; ghi chú chờ đổi kiểm 2 con số khi lên lớp với bạn ngồi cạnh.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
- **(a) Góc nhìn mô hình học:** Nhãn 3D phải đo bằng mét trong không gian thực thay vì pixel vì kích thước vật lý và khoảng cách phanh/va chạm là bất biến theo vị trí camera, giúp mô hình dự báo quỹ đạo trong hệ tọa độ quanh xe tự lái. Các cụm quá thưa cần ngưỡng tối thiểu (thấy đủ phần thân đặc trưng và xác nhận được trên camera) để tránh nạp nhãn đoán mò sai kích thước vào tập huấn luyện.
- **(b) Cách dùng AI:**
  - Công cụ: `Antigravity` hỗ trợ dựng trình xem trực quan đa góc (BEV có lưới tọa độ 1 m/5 m, mặt cắt đứng Side/Front, khung chiếu 7 camera và ảnh cắt cận cảnh từng vật).
  - Câu hỏi chính: đọc nhanh các mặt cắt `z` mặt đường cục bộ theo từng vị trí `(x, y)`, phát hiện các hộp còn lệch mép, lơ lửng hoặc ngược hướng mũi tên `+X` khi đối chiếu `preview`/`selfcheck`, và kiểm tra tính nhất quán số liệu báo cáo với `bangA.csv`/`tukiem.csv`.
  - Cách dùng kết quả: mọi cuboid đều do tôi trực tiếp đọc bằng mắt trên point cloud đơn frame + 7 camera và tinh chỉnh thủ công qua 2 vòng trước khi chốt `bangA.csv` và báo cáo.
- **(c) Điều còn chưa chắc:** Các xe ở rìa xa `40–48.3 m` hoặc bị che một phần (`s04f13 #10` có `29` điểm, `s05f04 #15` có `18` điểm, `s01f47 #10` có `14` điểm, `s05f32 #13` ở `48.34 m` có `41` điểm) chỉ lộ mặt cản sau và mép nóc nên chiều dài dọc tia nhìn có thể còn sai số ước lượng theo Bảng 4.3 khoảng `±0.3–0.5 m`.
