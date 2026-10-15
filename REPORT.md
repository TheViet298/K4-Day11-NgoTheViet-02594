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
- TODO (cần tự giải thích): vì sao CEM bị kéo về vùng bất khả thi, và ước lượng bị lệch thế nào nếu đếm chúng là va chạm.

Liệt kê các luật trong `is_feasible` và căn cứ vật lý của từng luật (ma sát, gia tốc, tốc độ người). Trong `hunt.json`, cuộc săn nào sinh nhiều kịch bản bất khả thi nhất, và vì sao CEM lại bị kéo về phía đó? Điều gì xảy ra với ước lượng nếu bạn **đếm** chúng là va chạm?

## 3. Monte Carlo cho bạn biết gì (bước C1)

**Trả lời (một phần, điền từ evidence):**
- `mc.json: n = 3000, crashes = 0, p = 0.0`, Wilson 95 %: `lo = 1.08e-19`, `hi = 1.279e-03`. Cả 5 kiểu (`per_mode`) đều `k = 0`.
- Với k = 0 không thể thu hẹp khoảng quanh p̂ = 0; chỉ có cận trên 0,13 % (khoảng 3/n). Muốn ±20 % của một p thật nhỏ như vài 1e-4 thì cần hàng triệu lần chạy (n ~ 100/p).
- Mọi kiểu đều chưa từng xuất hiện trong 3000 lần chạy; con số đúng cần báo là cận trên `hi = 1.279e-03` chứ không phải 0.
- TODO: tính cụ thể số lần chạy (n) cho ±20 %.

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
- TODO: nhận xét các seed có đồng ý với nhau/Monte Carlo không (bus_bypass lệch ~5 lần giữa seed 0 và 1) và so `budget` với số lần MC tương đương.

Bảng: với từng kiểu và từng seed trong `estimate.json`, ghi `p`, `hits`, `ess`, `top_share`. Ba seed có đồng ý với nhau không, và có đồng ý với Monte Carlo không? Ở đâu `top_share` lớn (> 0,5), và vì sao khi đó con số thực ra chỉ là trọng số của một mẫu? So ngân sách (`budget`) với số lần chạy Monte Carlo cần để có cùng độ chính xác. Phần `other` trong tổng lấy từ đâu, và vì sao không săn nó?

## 5. Các kiểu thất bại (bước C2 + 3D)

Mỗi kiểu bạn tìm thấy: một ảnh chụp trình xem 3D (chương 9) ở đúng lúc va chạm, và một câu mô tả chuỗi sự kiện (ai thấy gì, ai chờ gì). Kiểu nào cuộc săn **không** tìm thấy, và bạn tin điều đó ở mức nào? Nếu chỉ chạy **một** cuộc săn chung cho mọi kiểu thì điều gì xảy ra (gợi ý: `hunt.json: found` của từng cuộc săn)?

## 6. Sửa planner (bước E + 3D)

**Trả lời (một phần, điền từ evidence):**
- Policy được chọn (`fix.json: chosen.policy`): `{cutinLat: 3.0}`, gốc là `cutinLat = 2.15`.
- TRAIN: `fail = 1.01e-07`, `fail_hi = 2.035e-07`.
- TEST (chosen): `fail = 1.984e-07`, `fail_hi_seen = 3.646e-07`, `fail_hi = 3.878e-02`, `unseen = ['bus_bypass']`.
- TEST (policy gốc): `fail = 1.229e-03`, `fail_hi_seen = 2.774e-03`, `fail_hi = 4.053e-03`.
- `fail_hi` của bản sửa (3.88e-02) lớn hơn của policy gốc vì `bus_bypass` là kiểu `unseen` (0 hit, ess 74) nên phải báo cận trên rộng (`upper_zero_hits = 0.0375`).
- Quãng đường/phanh gấp (TEST): chosen `progress = 216.7 m`, `harsh = 7.675`; gốc `progress = 216.7 m`, `harsh = 8.015`.
- Núm "nghe an toàn hơn": `cautionV = 9.0` có `progress` cao nhất (229.7) nhưng `fail = 5.5e-04` cao hơn gốc trên TRAIN (5.2e-04); `T = 2.0` tăng `fail` lên 6.09e-04 (`fix.json: table`).
- TODO: ảnh chụp 3D có bóng mờ; giải thích thế giới phản ứng.

`fix.json`: policy được chọn, `fail` và `fail_hi` trên TRAIN, trên TEST (`fail_hi_seen` và `fail_hi`), và của policy gốc trên TEST; vì sao `fail_hi` của bản sửa có thể lớn hơn `fail` của policy gốc dù bản sửa tốt hơn thật? quãng đường và số lần phanh gấp. Một ảnh chụp 3D có **bóng mờ** (policy gốc) cho một kịch bản mà bản sửa tránh được. Có núm chỉnh nào "nghe thì an toàn hơn" nhưng thực tế làm tăng thất bại? Giải thích bằng cách thế giới phản ứng lại.

## 7. Từ mô phỏng ra đường thật

Phân phối tự nhiên trong `space.py` là giả định. Chọn **ba** giả định ảnh hưởng nhiều nhất đến con số của bạn (ví dụ khe chấp nhận của xe chuyển làn, tỉ lệ người đi bộ ngó qua xe buýt, ma sát đường ướt). Với mỗi cái: dữ liệu thật nào sẽ thay nó, lấy từ đâu, và nếu nó sai thì con số của bạn sai theo hướng nào?

## 8. Giới hạn

Ba điều mà xác suất bạn báo **không** nói, ví dụ: thế giới chỉ có một con phố; người lái chỉ là IDM; cảm biến hoàn hảo trừ phần bị xe buýt che. Nếu được thêm một tuần, bạn sẽ đo gì tiếp?

## 9. Rắc rối của bạn (bước G + chương 10)

- **Rắc rối:** bạn dựng gì, và ngoài đời nó giống tình huống nào.
- **Căn cứ:** vì sao bạn nghĩ planner sẽ xử lý kém.
- **Giả định:** `space` và `exposure` lấy từ đâu (`hazard.json: why`).
- **Kết quả**, theo `mine.json`:
  - số va chạm `avoidable` (tránh được bằng phanh hết cỡ, biết trước), `unavoidable`, `unattributed`, và `low_speed`, với policy gốc và với policy bạn chọn; `accepted` (bao nhiêu phần `space` của bạn là khả thi);
  - P(lỗi planner | rắc rối) kèm khoảng Wilson;
  - tần suất mỗi chuyến, đặt cạnh tần suất lách xe buýt của bạn.
- **Hình:** một ảnh chương 9 hoặc 10 ở lúc va chạm.
- **Bạn tự lái thì sao:** nếu bạn đã thử lái, bạn tránh được không, và bạn làm gì khác planner?
- **Cách sửa:** núm chỉnh nào sửa được rắc rối này? Nếu không núm nào sửa được, planner cần thêm **logic gì** (mô tả, không cần code)?
