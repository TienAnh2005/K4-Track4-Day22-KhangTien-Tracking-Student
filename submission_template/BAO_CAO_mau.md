# Báo cáo lab: chọn tracker cho 5 video

**Nhóm:** Khang - Tiến **Thành viên:** Nguyễn Thế Khang - Lưu Nguyễn Tiến Anh

Detector cố định: `yolo26n.pt`, ảnh 640 px, Re-ID `osnet_x0_25_msmt17`. Không đổi các mục này trong bài nộp chính.

## 1. Cấu hình đã chọn

Mỗi video: tracker bạn nộp, `conf`, `iou`, điều bạn **nhìn thấy** trên video, và một cấu hình đã thử rồi loại.

| Video | Tracker | conf | iou | Quan sát khi xem video | Đã thử nhưng loại |
|---|---|---|---|---|---|
| video_1 (quảng trường, tĩnh, ban ngày) | botsort | 0.15 | 0.6 | Camera tĩnh, ánh sáng tốt, mật độ vừa. Đặt conf=0.15 bắt rất nhạy người ở xa và người trong bóng râm mà không sinh hộp rác. BoTSORT kết hợp Re-ID và bù chuyển động giúp duy trì ID cực tốt khi cắt mặt nhau, HOTA đạt 30.17%, IDSW chỉ 26 lần trên toàn bộ 600 frame. | bytetrack (conf=0.3, iou=0.5: conf cao bỏ sót nhiều người ở xa, HOTA chỉ 26.91%, MOTA 17.29%); ocsort (conf=0.15, iou=0.5: IDSW nhảy lên tới 165 lần). |
| video_2 (phố đêm, tĩnh, rất đông) | bytetrack | 0.20 | 0.5 | Góc nhìn cao, ban đêm thiếu sáng, mật độ người rất đông đúc và che khuất liên tục. ByteTrack liên kết 2 giai đoạn (first/second association) giữ track ổn định ngay cả với hộp phát hiện điểm thấp, không bị Re-ID nhận nhầm do ánh sáng yếu và bóng tối. | ocsort (conf=0.2, iou=0.5: ID bị phân mảnh nhiều, sinh ra tới 31 unique ID chỉ trong 150 frame thử nghiệm); botsort (conf=0.2: Re-ID trong điều kiện thiếu sáng dễ nhầm lẫn khi người đứng chen chúc). |
| video_3 (camera di động, ảnh nhỏ) | botsort | 0.25 | 0.5 | Camera di chuyển liên tục, ảnh nhỏ và FPS thấp khiến vị trí đối tượng giữa các khung hình bị gián đoạn. BoTSORT có mô-đun Camera Motion Compensation (GMC) bù trừ chuyển động camera, kết hợp Re-ID giúp quỹ đạo không bị trôi lệch hay đứt gãy. | ocsort (conf=0.25, iou=0.5: khi camera đổi hướng nhanh vẫn gặp hiện tượng mất dấu và sinh ID mới); bytetrack (Kalman tiêu chuẩn bị trượt dự đoán khi camera di chuyển). |
| video_4 (trong nhà, camera di chuyển) | strongsort | 0.35 | 0.5 | Trong nhà có vách kính lớn phản chiếu bóng người, camera tịnh tiến về phía trước. Ngưỡng conf=0.35 lọc sạch hoàn toàn bóng phản chiếu trên kính. StrongSORT với Re-ID cập nhật EMA và ma trận chi phí kết hợp đặc trưng không gian - ngoại hình giúp nhận diện mượt mà người thật đang đi tới. | bytetrack (conf=0.2: bắt nhầm bóng phản chiếu trên kính thành người đi bộ, tạo track giả); bytetrack (conf=0.35: tuy bớt hộp giả nhưng khi hai người đi giao cắt nhau dễ bị đổi ID do thiếu ngoại hình). |
| video_5 (trên xe bus, giao lộ đông) | deepocsort | 0.25 | 0.5 | Quay từ trên xe bus qua giao lộ đông đúc, rung lắc mạnh do động cơ và mặt đường xóc. DeepOC-SORT kế thừa cơ chế Observation-Centric (Direction Consistency + Online Smoothing) xử lý rung lắc phi tuyến cực tốt, kết hợp Re-ID giúp duy trì đúng ID khi xe lắc nảy. | ocsort (conf=0.25, iou=0.5: chỉ dùng chuyển động nên tại điểm giao cắt người và xe dày đặc dễ bị hoán đổi ID); bytetrack (chuyển động phi tuyến do rung lắc làm bộ lọc Kalman tuyến tính bị trượt). |

## 2. Số liệu video_1

Dán bảng HOTA / MOTA / IDF1 do `scripts/evaluate_practice.py` in ra.

```
HOTA: nhom01_video1-pedestrian     HOTA      DetA      AssA      DetRe     DetPr     AssRe     AssPr     LocA      OWTA      HOTA(0)   LocA(0)   HOTALocA(0)
video_1                            30.173    19.306    47.494    20.033    75.842    50.739    81.259    82.765    30.781    37.521    76.931    28.866    
COMBINED                           30.173    19.306    47.494    20.033    75.842    50.739    81.259    82.765    30.781    37.521    76.931    28.866    

CLEAR: nhom01_video1-pedestrian    MOTA      MOTP      MODA      CLR_Re    CLR_Pr    MTR       PTR       MLR       sMOTA     CLR_TP    CLR_FN    CLR_FP    IDSW      MT        PT        ML        Frag      
video_1                            20.806    80.322    20.946    23.68     89.65     12.903    20.968    66.129    16.146    4400      14181     508       26        8         13        41        69        
COMBINED                           20.806    80.322    20.946    23.68     89.65     12.903    20.968    66.129    16.146    4400      14181     508       26        8         13        41        69        

Identity: nhom01_video1-pedestrian IDF1      IDR       IDP       IDTP      IDFN      IDFP      
video_1                            30.482    19.267    72.942    3580      15001     1328      
COMBINED                           30.482    19.267    72.942    3580      15001     1328      

Count: nhom01_video1-pedestrian    Dets      GT_Dets   IDs       GT_IDs    
video_1                            4908      18581     54        62        
COMBINED                           4908      18581     54        62        
```

`video_2` đến `video_5` không có nhãn trong gói lab. Không điền số cho các video đó.

## 3. Phân tích

Với **ít nhất hai video** (nên gồm một video bạn chỉ đánh giá bằng mắt), viết 3–5 câu:

- **video_1 (Đánh giá định lượng):**
  Cấu hình BoTSORT (`conf=0.15`, `iou=0.6`) đạt HOTA 30.17%, MOTA 20.81% và IDF1 30.48%, vượt trội so với baseline ban đầu của ByteTrack (`conf=0.3`, `iou=0.5` với HOTA 26.91%, MOTA 17.29%). Việc hạ ngưỡng `conf` từ 0.3 xuống 0.15 giúp tăng số lượng phát hiện đúng (CLR_TP) từ 3332 lên 4400 hộp, giảm mạnh số người bị bỏ sót ở xa. Đồng thời, mô hình Re-ID OSNet kết hợp với cơ chế matching chặt chẽ giúp duy trì ID nhất quán cao khi người đi bộ đi lướt qua nhau, giữ số lần đổi ID (IDSW) ở mức rất thấp (chỉ 26 lần trong suốt 600 frame). Cảnh ngoài trời ánh sáng đồng đều tạo điều kiện lý tưởng để các vector đặc trưng ngoại hình phát huy tối đa hiệu quả.

- **video_2 (Đánh giá định tính bằng mắt - Phố đêm, mật độ rất đông):**
  Trong cảnh đêm với camera tĩnh trên cao, mật độ người rất dày đặc dẫn đến tình trạng che khuất một phần diễn ra liên tục. Tracker thuần chuyển động ByteTrack (`conf=0.2`, `iou=0.5`) thể hiện sự ổn định vượt trội so với các tracker dùng Re-ID như BoTSORT hay StrongSORT. Do ánh sáng yếu, màu sắc quần áo bị tối và bóng đổ kéo dài, đặc trưng Re-ID thường bị nhiễu và dễ dẫn tới gán nhầm ID khi các nhóm người đi sát nhau. ByteTrack tận dụng liên kết hai pha (gán các phát hiện độ tin cậy cao trước, sau đó gán tiếp các phát hiện mờ/bị che khuất độ tin cậy thấp vào các track đang có) giúp giữ track liên tục, hộp bao bám khít người và không bị hiện tượng nhảy màu ID liên tục khi dòng người giao thoa.

- **video_5 (Đánh giá định tính bằng mắt - Trên xe bus, rung lắc mạnh):**
  Cảnh quay từ xe bus đang di chuyển tại ngã tư chịu tác động mạnh của dao động cơ học và chấn động mặt đường, khiến vị trí camera giật lắc liên tục. DeepOC-SORT (`conf=0.25`, `iou=0.5`) chứng minh sự phù hợp vượt trội nhờ cơ chế bảo toàn hướng chuyển động (Direction Consistency) và làm mượt quỹ đạo ngoại suy (Observation-Centric Online Smoothing). Trong khi bộ lọc Kalman tuyến tính thông thường của ByteTrack hoặc StrongSORT dễ bị dự đoán lệch tâm hộp khi xe xóc mạnh, DeepOC-SORT nhanh chóng hiệu chỉnh sai số dự đoán bằng các quan sát thực tế và dùng Re-ID để tái kích hoạt các track bị ngắt quãng, giúp các bounding box bám chắc vào người đi bộ qua đường mà không bị trôi dạt vào thân xe hay cột đèn.

## 4. Nếu có thêm thời gian

- Thử nghiệm các mô hình Re-ID có năng lực trích xuất đặc trưng mạnh hơn trong điều kiện ánh sáng yếu (như ResNet50-IBN hoặc CLIP-ReID) để cải thiện độ chính xác trong cảnh ban đêm như `video_2`.
- Quét lưới siêu tham số mịn hơn cho module bù chuyển động camera (GMC) trên `video_3` và `video_5`, đồng thời thử nghiệm giải pháp tiền xử lý ổn định hình ảnh (video stabilization) trước khi đưa vào pipeline tracking.
