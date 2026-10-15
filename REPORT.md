MSSV: 202602594

# Lab 11 · Scenario Forge — báo cáo

Mỗi câu: trả lời ngắn, **trích số từ `evidence/`** (ghi tên file và trường, ví dụ `mc.json: crashes = 4`). Grader chạy lại với MSSV của bạn; số không khớp `evidence/` thì mất điểm phần D (RUBRIC), và câu dựa trên số sai khó được điểm đầy đủ. Không cần văn hay: một bảng nhỏ và hai, ba câu giải thích là đủ.

Thị trấn của bạn (`meta.json: _run.town`): **D**

## 1. Phát lại log có trung thực không? (bước A)

**Trả lời (điền từ evidence):**
- `replay.json: dropped_human_crash_logs = 0` (n = 400, n_used = 400): không kịch bản nào bị bỏ vì người lái gây va chạm.
- `replay.json: replay.crashes = 106`, theo loại tiếp xúc `rear = 100`, `side = 6` (không có `front`).
- `replay.json: reactive.crashes = 0`, `crash_in_both = 0`.
- 100/106 va chạm của phát lại là `rear`: trong log phát lại, xe phía sau là bản ghi cố định, không phản ứng với việc planner phanh, nên nó đâm vào ego. Trong thế giới phản ứng, xe đó biết giữ khoảng cách nên không va chạm. Đây là va chạm **giả** do phát lại.
- `replay_only` có 20 kịch bản, ví dụ `scenario = 5` (`contact = rear`, `min_ttc_reactive = 2.41`): phát lại báo va chạm, thế giới phản ứng thì không.

Trong `replay.json`: bao nhiêu kịch bản bị bỏ vì người lái gây va chạm, bao nhiêu va chạm khi planner chạy trên log phát lại, bao nhiêu trong thế giới phản ứng, theo từng loại tiếp xúc. Giải thích vì sao phần lớn va chạm của "phát lại" là `rear`, và vì sao đó là va chạm **giả**. Chọn một kịch bản trong `replay_only` và nói chuyện gì đã xảy ra.

## 2. Kịch bản bất khả thi (bước B)

**Trả lời (một phần, điền từ evidence):**
- Các luật trong `is_feasible` (`lab11/feasible.py`): (1) gia tốc ngang của lần chuyển làn `(pi/cutDur)^2 * 3.5/2` không vượt `mu*g` (giới hạn bám đường); (2) phanh sau chuyển làn `cutBrake <= mu*g` (giới hạn ma sát); (3) `pedV <= 2.5` m/s (người đi bộ chạy nhanh nhất); (4) `0 < cutV <= 22` m/s (tốc độ hợp lý của xe).
- `hunt.json`: số kịch bản bất khả thi: `bus_bypass` 1, `cut_in` 1, `crosswalk_block` 9, `rear_ended` 7. Cuộc săn `crosswalk_block` nhiều nhất (9).
- CEM chọn các mẫu có `level` cao nhất (gần va chạm nhất). Các tham số cực đoan (đổi làn quá gấp, phanh quá mạnh so với `mu`, người đi bộ chạy quá nhanh) làm xe khó tránh nhất, nên điểm số cao nhất và proposal bị kéo về phía đó; `crosswalk_block` và `rear_ended` có nhiều mẫu bất khả thi nhất (9 và 7) vì chúng nhạy với `pedV`, `cutBrake`.
- Nếu **đếm** chúng là va chạm, ước lượng `p` bị thổi phồng bởi các va chạm không thể xảy ra ngoài đời. Vì thế mẫu bất khả thi bị loại trước khi tính và được báo riêng ở trường `infeasible`.

Liệt kê các luật trong `is_feasible` và căn cứ vật lý của từng luật (ma sát, gia tốc, tốc độ người). Trong `hunt.json`, cuộc săn nào sinh nhiều kịch bản bất khả thi nhất, và vì sao CEM lại bị kéo về phía đó? Điều gì xảy ra với ước lượng nếu bạn **đếm** chúng là va chạm?

## 3. Monte Carlo cho bạn biết gì (bước C1)

**Trả lời (một phần, điền từ evidence):**
- `mc.json: n = 3000, crashes = 0, p = 0.0`, Wilson 95 %: `lo = 1.08e-19`, `hi = 1.279e-03`. Cả 5 kiểu (`per_mode`) đều `k = 0`.
- Với k = 0 không thể thu hẹp khoảng quanh p̂ = 0; chỉ có cận trên 0,13 % (khoảng 3/n). Muốn ±20 % của một p thật nhỏ như vài 1e-4 thì cần hàng triệu lần chạy (n ~ 100/p).
- Mọi kiểu đều chưa từng xuất hiện trong 3000 lần chạy; con số đúng cần báo là cận trên `hi = 1.279e-03` chứ không phải 0.
- Số lần chạy cho ±20 %: nửa độ rộng Wilson xấp xỉ `1.96 * sqrt(p/n)`, đặt bằng `0.2 p` thì `n ≈ 96/p`. Với `p ≈ 6.3e-04` (trung bình `bus_bypass` của 3 seed trong `estimate.json`) cần `n ≈ 1.5e5` lần chạy; nếu lấy `p` bằng cận trên `hi = 1.279e-03` thì `n ≈ 7.5e4`. Đều lớn hơn nhiều so với `n = 3000`.

`mc.json`: k, n, p̂ và khoảng Wilson 95 %. Với p̂ đó, cần bao nhiêu lần chạy để khoảng tin cậy hẹp lại còn ±20 % của p? Có kiểu thất bại nào **chưa hề xuất hiện** trong 3 000 lần chạy không? Con số đúng cần báo cho kiểu đó là gì?

## 4. Importance sampling, ESS và độ suy biến (bước C3)

**Bảng (`estimate.json`, `n = 600` mỗi kiểu):**

| seed | kiểu | p | hits | ess | top_share |
|---|---|---|---|---|---|
| 0 | bus_bypass | 2.26e-04 | 45 | 112 | 0.365 |
| 0 | cut_in | 0 | 0 | 127 | 0 |
| 0 | crosswalk_block | 1.76e-06 | 61 | 124 | 0.671 |
| 0 | rear_ended | 0 | 0 | 129 | 0 |
| 1 | bus_bypass | 1.25e-03 | 43 | 130 | 0.842 |
| 1 | cut_in | 0 | 0 | 107 | 0 |
| 1 | crosswalk_block | 1.46e-07 | 47 | 112 | 0.849 |
| 1 | rear_ended | 0 | 0 | 134 | 0 |
| 2 | bus_bypass | 4.15e-04 | 54 | 121 | 0.311 |
| 2 | cut_in | 0 | 0 | 117 | 0 |
| 2 | crosswalk_block | 2.16e-06 | 49 | 134 | 0.911 |
| 2 | rear_ended | 0 | 0 | 130 | 0 |

- `total` theo seed: 2.28e-04, 1.25e-03, 4.17e-04; `budget = 16800`.
- `other`: `estimate.json: other = {k: 0, n: 3000, hi: 1.279e-03, source: mc.json}`, lấy từ Monte Carlo; không săn vì chưa từng thấy va chạm nào thuộc loại này.
- `top_share > 0.5`: `crosswalk_block` ở cả 3 seed (0.671, 0.849, 0.911) và `bus_bypass` seed 1 (0.842); khi đó một mẫu có trọng số gần như quyết định con số.
- Các seed chỉ đồng ý về bậc độ lớn, không đồng ý về giá trị: `bus_bypass` từ 2.26e-04 đến 1.25e-03 (lệch ~5 lần, seed 1 có `top_share = 0.842`), `crosswalk_block` từ 1.46e-07 đến 2.16e-06 (lệch ~15 lần). Các khoảng Wilson của `bus_bypass` vẫn chồng nhau. Không mâu thuẫn với Monte Carlo (`mc.json: crashes = 0`, `hi = 1.279e-03`): mọi `p` của IS đều nhỏ hơn cận trên đó, nhưng MC không phân biệt được chúng với 0.
- Khi `top_share` lớn, `p` thực chất là trọng số của một vài mẫu (ví dụ 84-91 % tổng trọng số nằm ở một mẫu), nên `ess` (107-134) nhìn có vẻ đủ nhưng không phản ánh độ tin cậy thật.
- `budget = 16800` lần chạy; để MC có độ chính xác tương đương (±20 % ở `p ≈ 6e-04`) cần ~1.5e5 lần, tức nhiều hơn khoảng 9 lần.

Bảng: với từng kiểu và từng seed trong `estimate.json`, ghi `p`, `hits`, `ess`, `top_share`. Ba seed có đồng ý với nhau không, và có đồng ý với Monte Carlo không? Ở đâu `top_share` lớn (> 0,5), và vì sao khi đó con số thực ra chỉ là trọng số của một mẫu? So ngân sách (`budget`) với số lần chạy Monte Carlo cần để có cùng độ chính xác. Phần `other` trong tổng lấy từ đâu, và vì sao không săn nó?

## 5. Các kiểu thất bại (bước C2 + 3D)

Mỗi kiểu bạn tìm thấy: một ảnh chụp trình xem 3D (chương 9) ở đúng lúc va chạm, và một câu mô tả chuỗi sự kiện (ai thấy gì, ai chờ gì). Kiểu nào cuộc săn **không** tìm thấy, và bạn tin điều đó ở mức nào? Nếu chỉ chạy **một** cuộc săn chung cho mọi kiểu thì điều gì xảy ra (gợi ý: `hunt.json: found` của từng cuộc săn)?

**Trả lời (một phần, điền từ evidence):**
- Các cuộc săn trong `hunt.json: found`: `bus_bypass` -> `{bus_bypass: 168}`; `cut_in` -> `{bus_bypass: 13}`; `crosswalk_block` -> `{crosswalk_block: 15}`; `rear_ended` -> `{bus_bypass: 5, crosswalk_block: 5}`.
- Tìm thấy 2 kiểu: `bus_bypass` và `crosswalk_block`. **Không** tìm thấy `cut_in` và `rear_ended`: cuộc săn của chúng chỉ ra các kiểu khác. `estimate.json` cũng cho `hits = 0` ở hai kiểu này với cả 3 seed (`upper_zero_hits = 0.025`), nên mức tin cậy chỉ là "không thấy với ngân sách 600 mẫu/seed", không phải bằng chứng là chúng không tồn tại.
- Nếu chỉ chạy một cuộc săn chung thì kết quả thiên về kiểu dễ nhất (`bus_bypass` chiếm 168 kết quả, gấp hơn 10 lần `crosswalk_block`), các kiểu hiếm bị che mất.
- `bus_bypass`: ảnh `evidence/shots/ch9_bus_bypass_collision.png` (chương 9, "Va chạm: xe ta và xe cắt làn lúc 12,4 s"). Chuỗi sự kiện: đường ướt (μ 0,5), xe ta lách qua xe buýt đang đỗ ở điểm dừng; xe cắt làn (xe tím) chen vào với khe chấp nhận chỉ 6,4 m ngay sau xe buýt, xe ta phanh không kịp nên đâm vào nó. Người đi bộ ở vạch qua đường có ngó qua xe buýt, nên xe ta cũng bị chậm lại vì lý do khác.
- `crosswalk_block`: chuỗi sự kiện (chạy lại `C-01` đến `C-04` từ `viewer.json`): sự kiện là `block` với người đi bộ `P1` lúc 18,9-19,6 s, không phải va chạm. Người đi bộ bước ra từ sau xe buýt đỗ, xe ta chỉ thấy khi đã gần vạch nên phanh muộn và dừng với thân xe nằm trên vạch qua đường, chắn lối người đang qua (README: kiểu `crosswalk_block` là planner phanh quá muộn).
- Ảnh `crosswalk_block`: `evidence/shots/ch9_crosswalk_block_C01.png` (chương 9, kịch bản `C-01`, 19,5 s, nhãn "chặn lối": xe ta đã dừng nhưng thân xe nằm trên vạch qua đường, người đi bộ bị chắn lối).

## 6. Sửa planner (bước E + 3D)

**Trả lời (một phần, điền từ evidence):**
- Policy được chọn (`fix.json: chosen.policy`): `{cutinLat: 3.0}`, gốc là `cutinLat = 2.15`.
- TRAIN: `fail = 1.01e-07`, `fail_hi = 2.035e-07`.
- TEST (chosen): `fail = 1.984e-07`, `fail_hi_seen = 3.646e-07`, `fail_hi = 3.878e-02`, `unseen = ['bus_bypass']`.
- TEST (policy gốc): `fail = 1.229e-03`, `fail_hi_seen = 2.774e-03`, `fail_hi = 4.053e-03`.
- `fail_hi` của bản sửa (3.88e-02) lớn hơn của policy gốc vì `bus_bypass` là kiểu `unseen` (0 hit, ess 74) nên phải báo cận trên rộng (`upper_zero_hits = 0.0375`).
- Quãng đường/phanh gấp (TEST): chosen `progress = 216.7 m`, `harsh = 7.675`; gốc `progress = 216.7 m`, `harsh = 8.015`.
- Núm "nghe an toàn hơn": `cautionV = 9.0` có `progress` cao nhất (229.7) nhưng `fail = 5.5e-04` cao hơn gốc trên TRAIN (5.2e-04); `T = 2.0` tăng `fail` lên 6.09e-04 (`fix.json: table`).
- Giải thích núm "nghe an toàn hơn": hạ `cautionV` hay tăng `T` làm xe chậm/giữ khoảng cách lớn, nhưng xe phía sau và các tác nhân khác phản ứng lại (xe sau áp sát hơn, người đi bộ băng qua trong khoảng trống mới), nên thất bại không giảm mà có khi tăng, như `T = 2.0` (`fail = 6.09e-04` so với gốc `5.16e-04` trên TRAIN).
- Lưu ý: `cutinLat = 3.0` được chọn trên TRAIN, nhưng `bus_bypass` ở đó là `unseen` (không có hit nào), nên `fail` thấp một phần là do không quan sát được kiểu này. `cutinLat = 2.6` (`fail = 3.23e-05`, `unseen = []`) là lựa chọn đáng tin hơn về mặt thống kê.
- Cảnh bản sửa tránh được: chạy lại 4 kịch bản `bus_bypass` đã xuất sang trình xem (`viewer.json`, `B-01` đến `B-04`): với **policy gốc** cả 4 đều va chạm với xe cắt làn `C2` (kiểu `contact`) lúc 12,3 s; 12,7 s; 12,8 s; 12,3 s, còn với **policy được chọn** (`cutinLat = 3.0`) cả 4 đều **không** va chạm. Ảnh `evidence/shots/ch9_bus_bypass_collision.png` (va chạm với xe cắt làn lúc 12,4 s, có bóng mờ) là một cảnh thuộc loại này; bóng mờ là xe với policy còn lại, không va chạm. Trong ảnh, xe chính (có vòng đỏ va chạm) chạy policy gốc, còn xe mờ phía sau là bản sửa và không va chạm; nhãn chữ trong bảng thông tin của trình xem lấy theo tên "chính sách của bạn" cho xe chính, nên đọc theo kết quả chạy lại ở trên.
- Với `crosswalk_block` (`C-01` đến `C-04`) bản sửa **không** giúp: cả policy gốc và policy được chọn đều bị `block` với người đi bộ `P1` lúc 18,9-19,6 s, khớp với `fix.json` (`crosswalk_block.p = 1.01e-07` ở cả hai).

`fix.json`: policy được chọn, `fail` và `fail_hi` trên TRAIN, trên TEST (`fail_hi_seen` và `fail_hi`), và của policy gốc trên TEST; vì sao `fail_hi` của bản sửa có thể lớn hơn `fail` của policy gốc dù bản sửa tốt hơn thật? quãng đường và số lần phanh gấp. Một ảnh chụp 3D có **bóng mờ** (policy gốc) cho một kịch bản mà bản sửa tránh được. Có núm chỉnh nào "nghe thì an toàn hơn" nhưng thực tế làm tăng thất bại? Giải thích bằng cách thế giới phản ứng lại.

## 7. Từ mô phỏng ra đường thật

Phân phối tự nhiên trong `space.py` là giả định. Chọn **ba** giả định ảnh hưởng nhiều nhất đến con số của bạn (ví dụ khe chấp nhận của xe chuyển làn, tỉ lệ người đi bộ ngó qua xe buýt, ma sát đường ướt). Với mỗi cái: dữ liệu thật nào sẽ thay nó, lấy từ đâu, và nếu nó sai thì con số của bạn sai theo hướng nào?

**Trả lời:**
1. *Khe chấp nhận của xe chuyển làn (`cutGap`, `cutDs`)*: thay bằng dữ liệu quỹ đạo thật (ví dụ NGSIM/highD hay camera giao thông của chính thị trấn). Nếu giả định khe quá rộng so với thực tế thì xác suất `cut_in` bị đánh giá quá thấp.
2. *Tỉ lệ người đi bộ ngó qua xe buýt (`pedLook`, `pedGap`)*: thay bằng đếm thủ công/video tại điểm dừng xe buýt. Nếu thực tế ít người ngó hơn thì `crosswalk_block` và `bus_bypass` bị đánh giá thấp.
3. *Ma sát đường (`mu`) và độ trễ xe buýt (`busGo`)*: thay bằng số liệu thời tiết và dữ liệu lịch xe buýt. Nếu đường ướt phổ biến hơn giả định thì `fail` bị đánh giá thấp; nếu `busGo` thưa hơn thì bị đánh giá cao.

## 8. Giới hạn

Ba điều mà xác suất bạn báo **không** nói, ví dụ: thế giới chỉ có một con phố; người lái chỉ là IDM; cảm biến hoàn hảo trừ phần bị xe buýt che. Nếu được thêm một tuần, bạn sẽ đo gì tiếp?

**Trả lời:**
- Thế giới chỉ có một con phố, một con đường và một thị trấn (`D`), nên xác suất không nói gì về các địa hình khác.
- Người lái chỉ là IDM và cảm biến hoàn hảo trừ phần bị xe buýt che, nên không bao gồm lỗi của cảm biến thật hay hành vi người lái không tuân thủ.
- Xác suất không nói gì về hậu quả (mức độ nặng) và `mc.json: crashes = 0` chỉ cho cận trên `1.279e-03`, không phải bằng 0.
- Nếu có thêm một tuần: đo khe chấp nhận và hành vi người đi bộ thật để thay giả định ở mục 7, và chạy thêm seed/ngân sách để thu hẹp độ lệch giữa các seed ở mục 4.

## 9. Rắc rối của bạn (bước G + chương 10)

**Trả lời (từ `hazard.json` và `mine.json`):**
- **Rắc rối:** người đi bộ chạy băng ngang đường ở chỗ không có vạch (`hazard.json: hazards[0]`: `type = ped`, `x = 150`, `v = 2.5`, `trig = 30`). Ngoài đời giống người vội băng qua đường ở đô thị mà không dùng vạch qua đường.
- **Căn cứ:** planner chỉ thận trọng gần vạch qua đường bị che (`cautionV`, `cautionRange`), nên người bước ra ở chỗ không có vạch lại không nằm trong vùng đó. Kết quả xác nhận: planner gốc đâm vào người đi bộ (`mine.json: exact.planner = [ego, H0, 19.25 s]`).
- **Giả định:** `space` (`h0_x` 120-200, `h0_v` 1.8-2.5, `h0_trig` 20-40) và `exposure = 0.01` lần/chuyến là ước lượng của tôi, chưa có số đo (`hazard.json: why`). Cần thay bằng đếm từ video camera trên đoạn đường tương tự.
- **Kết quả**, theo `mine.json`:
  - `accepted = 1498/1500` (100 %) phiên bản khả thi.
  - policy gốc: 146 va chạm = `avoidable 145` + `unavoidable 0` + `unattributed 1` (`low_speed = 0`).
  - policy được chọn (`cutinLat = 3.0`): 145 va chạm = `avoidable 145` + `unavoidable 0` + `unattributed 0` (`low_speed = 0`).
  - P(lỗi planner | rắc rối) = 9.68e-02, Wilson [8.28e-02, 1.13e-01] (policy gốc).
  - tần suất mỗi chuyến = `0.01 * 9.68e-02 = 9.68e-04` (cận trên 1.13e-03). So với `estimate.json: bus_bypass.p` của 3 seed (2.26e-04, 1.25e-03, 4.15e-04, trung bình ~6.3e-04): cùng bậc độ lớn, cao hơn khoảng 1,5 lần so với trung bình. Vì `exposure` là ước lượng nên so sánh này chỉ mang tính tham khảo.
- **Hình:** `evidence/shots/ch10_ped_collision.png` (chương 10: "Va chạm: xe ta và người băng ngang lúc 19,3 s").
- **Bạn tự lái thì sao:** theo `mine.json: exact`, bạn tự lái không va chạm trong cảnh này (bạn đã xem lại cảnh nhiều lần, còn planner thì không). Bạn tránh được vì phanh sớm hơn planner: thấy người đi bộ ở mép đường là giảm tốc ngay, còn planner chỉ phản ứng khi người đã vào làn (145/146 va chạm là `avoidable`, tức phanh sớm là tránh được).
- **Cách sửa:** policy được chọn ở bước `fix` (`cutinLat = 3.0`) không giúp gì: 145 va chạm, bằng policy gốc, vì núm đó chỉ liên quan xe lấn làn. 145/146 va chạm là `avoidable` (phanh hết cỡ khi biết trước là tránh được), nên lỗi là planner phản ứng quá muộn, không phải bất khả kháng. Cần thêm logic: coi người đi bộ ở mép đường, hoặc đang bước vào làn, là mối nguy ở **mọi** vị trí, không chỉ gần vạch qua đường, rồi phanh theo thời gian-đến-va-chạm (TTC) với người đó. Chưa thử núm `pedBuffer` trong mục này, đó là núm đáng thử trước.
