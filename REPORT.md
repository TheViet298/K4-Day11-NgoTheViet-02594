MSSV: 202602594

# Lab 11 · Scenario Forge — báo cáo

Mỗi câu: trả lời ngắn, **trích số từ `evidence/`** (ghi tên file và trường, ví dụ `mc.json: crashes = 4`). Grader chạy lại với MSSV của bạn; số không khớp `evidence/` thì mất điểm phần D (RUBRIC), và câu dựa trên số sai khó được điểm đầy đủ. Không cần văn hay: một bảng nhỏ và hai, ba câu giải thích là đủ.

Thị trấn của bạn (`meta.json: _run.town`): 

## 1. Phát lại log có trung thực không? (bước A)

**Tr? l?i (?i?n t? evidence):**
- `replay.json: dropped_human_crash_logs = 0` (n = 400, n_used = 400): kh?ng k?ch b?n n?o b? b? v? ng??i l?i g?y va ch?m.
- `replay.json: replay.crashes = 106`, theo lo?i ti?p x?c `rear = 100`, `side = 6` (kh?ng c? `front`).
- `replay.json: reactive.crashes = 0`, `crash_in_both = 0`.
- 100/106 va ch?m c?a ph?t l?i l? `rear`: trong log ph?t l?i, xe ph?a sau l? b?n ghi c? ??nh, kh?ng ph?n ?ng v?i vi?c planner phanh, n?n n? ??m v?o ego. Trong th? gi?i ph?n ?ng, xe ?? bi?t gi? kho?ng c?ch n?n kh?ng va ch?m. ??y l? va ch?m **gi?** do ph?t l?i.
- `replay_only` c? 20 k?ch b?n, v? d? `scenario = 5` (`contact = rear`, `min_ttc_reactive = 2.41`): ph?t l?i b?o va ch?m, th? gi?i ph?n ?ng kh?ng.

Trong `replay.json`: bao nhiêu kịch bản bị bỏ vì người lái gây va chạm, bao nhiêu va chạm khi planner chạy trên log phát lại, bao nhiêu trong thế giới phản ứng, theo từng loại tiếp xúc. Giải thích vì sao phần lớn va chạm của "phát lại" là `rear`, và vì sao đó là va chạm **giả**. Chọn một kịch bản trong `replay_only` và nói chuyện gì đã xảy ra.

## 2. Kịch bản bất khả thi (bước B)

**Tr? l?i (m?t ph?n, ?i?n t? evidence):**
- C?c lu?t trong `is_feasible` ([feasible.py](lab11/feasible.py)): (1) gia t?c ngang c?a l?n chuy?n l?n `(pi/cutDur)^2 * 3.5/2` kh?ng v??t `mu*g` (gi?i h?n b?m ???ng); (2) phanh sau chuy?n l?n `cutBrake <= mu*g` (gi?i h?n ma s?t); (3) `pedV <= 2.5` m/s (ng??i ?i b? ch?y nhanh nh?t); (4) `0 < cutV <= 22` m/s (t?c ?? h?p l? c?a xe).
- `hunt.json`: s? k?ch b?n b?t kh? thi: `bus_bypass` 1, `cut_in` 1, `crosswalk_block` 9, `rear_ended` 7. Cu?c s?n `crosswalk_block` nhi?u nh?t (9).
- TODO (c?n t? gi?i th?ch): v? sao CEM b? k?o v? v?ng b?t kh? thi, v? ??c l??ng b? l?ch th? n?o n?u ??m ch?ng l? va ch?m.

Liệt kê các luật trong `is_feasible` và căn cứ vật lý của từng luật (ma sát, gia tốc, tốc độ người). Trong `hunt.json`, cuộc săn nào sinh nhiều kịch bản bất khả thi nhất, và vì sao CEM lại bị kéo về phía đó? Điều gì xảy ra với ước lượng nếu bạn **đếm** chúng là va chạm?

## 3. Monte Carlo cho bạn biết gì (bước C1)

**Tr? l?i (m?t ph?n, ?i?n t? evidence):**
- `mc.json: n = 3000, crashes = 0, p = 0.0`, Wilson 95 %: `lo = 1.08e-19`, `hi = 1.279e-03`. C? 5 ki?u (`per_mode`) ??u `k = 0`.
- V?i k = 0 kh?ng th? thu h?p kho?ng quanh p? = 0; ch? c? c?n tr?n 0.13 % (kho?ng 3/n). Mu?n ?20 % c?a m?t p th?t nh? nh? v?i 1e-4 th? c?n h?ng tri?u l?n ch?y (n ~ 100/p).
- M?i ki?u ??u ch?a t?ng xu?t hi?n trong 3000 l?n ch?y; con s? ??ng c?n b?o l? c?n tr?n `hi = 1.279e-03` ch? kh?ng ph?i 0.
- TODO: t?nh c? th? s? l?n ch?y (n) cho ?20 %.

`mc.json`: k, n, p̂ và khoảng Wilson 95 %. Với p̂ đó, cần bao nhiêu lần chạy để khoảng tin cậy hẹp lại còn ±20 % của p? Có kiểu thất bại nào **chưa hề xuất hiện** trong 3 000 lần chạy không? Con số đúng cần báo cho kiểu đó là gì?

## 4. Importance sampling, ESS và độ suy biến (bước C3)

**B?ng (`estimate.json`, `n = 600` m?i ki?u):**

| seed | ki?u | p | hits | ess | top_share |
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
- `other`: `estimate.json: other = {k: 0, n: 3000, hi: 1.279e-03, source: mc.json}`, l?y t? Monte Carlo; kh?ng s?n v? ch?a t?ng th?y va ch?m n?o thu?c lo?i n?y.
- `top_share > 0.5`: `crosswalk_block` ? c? 3 seed (0.671, 0.849, 0.911) v? `bus_bypass` seed 1 (0.842); khi ?? m?t m?u c? tr?ng s? g?n nh? quy?t ??nh con s?.
- TODO: nh?n x?t c?c seed c? ??ng ? v?i nhau/Monte Carlo kh?ng (bus_bypass l?ch ~5x gi?a seed 0 v? 1) v? so `budget` v?i s? l?n MC t??ng ???ng.

Bảng: với từng kiểu và từng seed trong `estimate.json`, ghi `p`, `hits`, `ess`, `top_share`. Ba seed có đồng ý với nhau không, và có đồng ý với Monte Carlo không? Ở đâu `top_share` lớn (> 0,5), và vì sao khi đó con số thực ra chỉ là trọng số của một mẫu? So ngân sách (`budget`) với số lần chạy Monte Carlo cần để có cùng độ chính xác. Phần `other` trong tổng lấy từ đâu, và vì sao không săn nó?

## 5. Các kiểu thất bại (bước C2 + 3D)

Mỗi kiểu bạn tìm thấy: một ảnh chụp trình xem 3D (chương 9) ở đúng lúc va chạm, và một câu mô tả chuỗi sự kiện (ai thấy gì, ai chờ gì). Kiểu nào cuộc săn **không** tìm thấy, và bạn tin điều đó ở mức nào? Nếu chỉ chạy **một** cuộc săn chung cho mọi kiểu thì điều gì xảy ra (gợi ý: `hunt.json: found` của từng cuộc săn)?

## 6. Sửa planner (bước E + 3D)

**Tr? l?i (m?t ph?n, ?i?n t? evidence):**
- Policy ???c ch?n (`fix.json: chosen.policy`): `{cutinLat: 3.0}`, g?c l? `cutinLat = 2.15`.
- TRAIN: `fail = 1.01e-07`, `fail_hi = 2.035e-07`.
- TEST (chosen): `fail = 1.984e-07`, `fail_hi_seen = 3.646e-07`, `fail_hi = 3.878e-02`, `unseen = ['bus_bypass']`.
- TEST (policy g?c): `fail = 1.229e-03`, `fail_hi_seen = 2.774e-03`, `fail_hi = 4.053e-03`.
- `fail_hi` c?a b?n s?a (3.88e-02) l?n h?n c?a policy g?c v? `bus_bypass` l? ki?u `unseen` (0 hit, ess 74) n?n ph?i b?o c?n tr?n r?ng (`upper_zero_hits = 0.0375`).
- Qu?ng ???ng/phanh g?p (TEST): chosen `progress = 216.7 m`, `harsh = 7.675`; g?c `progress = 216.7 m`, `harsh = 8.015`.
- N?m "nghe an to?n h?n": `cautionV = 9.0` c? `progress` cao nh?t (229.7) nh?ng `fail = 5.5e-04` cao h?n g?c tr?n TRAIN (5.2e-04); `T = 2.0` t?ng `fail` l?n 6.09e-04 (`fix.json: table`).
- TODO: ?nh ch?p 3D c? b?ng m?; gi?i th?ch th? gi?i ph?n ?ng.

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
