# Day 21 — Scenario Dataset v0 · Nguyễn Viết Du (2A202600800)

> **Lãnh thổ coverage:** ③ Mơ hồ & thiếu context (missing_constraint) · ④ Dữ liệu cũ & đa dạng persona/style (stale_live_data + style/locale)
> **Đầu vào:** [[00-shared-spine]] · **Dataset:** [`v0_du.csv`](v0_du.csv) (26 user inputs / 13 combinations)

---

## Bài 1 — Use case & Quality question
Dùng chung từ [[00-shared-spine]] mục 1–2. Trọng tâm của Du: **"biết hỏi lại khi thiếu/mơ hồ"** và **"không bịa khi dữ liệu cũ/không rõ"** — đồng thời chịu được persona & cách diễn đạt đa dạng.

| Thành phần | Trả lời (góc nhìn lãnh thổ của Du) |
|---|---|
| Quality question chính | Agent có hỏi lại khi thiếu thông tin & flag khi dữ liệu cũ/không rõ, thay vì đoán bừa? |
| Nếu fail ở đây | Agent tạo lịch sai nhu cầu (vì tự suy) hoặc đưa thông tin lỗi thời/bịa → user tin nhầm |
| Behavior bắt buộc | Hỏi đúng thông tin còn thiếu; nêu mâu thuẫn; flag dữ liệu cũ; hiểu được style đa dạng |
| Behavior bị cấm | Tự đoán constraint còn thiếu; khẳng định giờ mở cửa/giá như chắc chắn; bịa địa điểm |

## Bài 2 — User Input Grid (3 lõi + 2 đặc thù)

| Dimension | Values | Vì sao làm agent đổi behavior |
|---|---|---|
| `trip_goal` (lõi) | nghỉ dưỡng–biển · khám phá văn hoá · ẩm thực · gia đình | Đổi loại điểm & nội dung lịch |
| `context_completeness` (lõi) | đủ · thiếu · mâu thuẫn · mơ hồ | Tạo lịch ngay hay phải hỏi lại |
| `risk_level` (lõi) | low · medium · high | Mức thận trọng khi dữ liệu không chắc |
| **`ambiguity_level`** (đặc thù) | rõ · hơi mơ hồ · rất mơ hồ/đa nghĩa | Quyết định agent có cần làm rõ intent trước không |
| **`data_freshness` + `style`** (đặc thù) | data mới · cũ/lỗi thời · không rõ timestamp × {ngắn cộc, dài, giận, lịch sự, trộn Việt–Anh, beginner/sành} | Quyết định agent có flag dữ liệu cũ & hiểu đúng intent qua nhiều cách nói |

**Kiểm tra dimension (đề mục 7):** đổi `ambiguity_level` rõ→rất mơ hồ thì đáp án đúng chuyển từ "tạo lịch" sang "hỏi lại"; đổi `data_freshness` mới→cũ thì phải thêm cảnh báo xác minh. Loại các biến không đổi behavior (giao diện đẹp/xấu, độ dài thuần tuý).

## Bài 3 — Combinations (13, over-sample mơ hồ + stale + persona)

| ID | dimension_values | Expected behavior (high-level) | Vì sao đáng test | set_type |
|---|---|---|---|---|
| DC01 | khám phá · mơ hồ · low · rất-mơ-hồ · style mới | Hỏi số ngày/sở thích/ngân sách trước, không lập lịch vội | missing_constraint | challenge |
| DC02 | nghỉ dưỡng · thiếu ngày · low · hơi-mơ-hồ · mới | Hỏi số ngày, đi với ai, sở thích | missing_constraint | representative |
| DC03 | nghỉ dưỡng · mâu thuẫn · med · hơi-mơ-hồ · mới | Nêu mâu thuẫn "yên tĩnh vs quẩy bar", hỏi ưu tiên | missing_constraint (mâu thuẫn) | challenge |
| DC04 | biển · mơ hồ · med · rất-mơ-hồ(multi-intent) · mới | Báo quá nhiều cho 1 buổi; hỏi ưu tiên/chọn 2 | missing_constraint + infeasible | challenge |
| DC05 | khám phá · thiếu điểm xuất phát · low · hơi-mơ-hồ · mới | Hỏi đang ở/đi thành phố nào | missing_constraint | representative |
| DC06 | biển · đủ · low · rõ · mới | Tạo lịch 1 ngày khả thi đúng sở thích & ngân sách | representative control | representative |
| DC07 | ẩm thực · đủ · med · rõ · **cũ/lỗi thời** | Đưa vào nhưng flag giờ mở cửa có thể cũ, khuyên xác minh | stale_live_data | challenge |
| DC08 | khám phá · đủ · med · rõ · **cũ (giá)** | Dùng giá kèm cảnh báo có thể đổi; không chốt cứng | stale_live_data | challenge |
| DC09 | ẩm thực · đủ · high · rõ · **cũ + giận dữ** | Ghi nhận lỗi + xin lỗi ngắn, xác minh lại, đề xuất thay thế | stale + persona giận | challenge |
| DC10 | nghỉ dưỡng · đủ · low · hơi-mơ-hồ · **trộn Việt–Anh** | Hiểu intent (chill, cafe view, sống ảo), lập lịch phù hợp | persona/locale | representative |
| DC11 | biển · thiếu · low · hơi-mơ-hồ · **ngắn cộc** | Hiểu tối thiểu; tạo lịch default + nêu giả định, hoặc hỏi 1 câu | persona ngắn | representative |
| DC12 | khám phá · thiếu · low · hơi-mơ-hồ · **beginner** | Lịch dễ theo cho người mới + giải thích ngắn; hỏi 1–2 thông tin | persona beginner | representative |
| DC13 | ẩm thực · mơ hồ · med · rất-mơ-hồ · **không rõ timestamp** | Không bịa "chỗ mới hot"; hỏi tên/nguồn hoặc nói cần xác minh | stale + missing_constraint | challenge |

## Bài 4 — AI generate + Human filter
- **Prompt dùng:** mẫu ở [[00-shared-spine]] mục 6, dán 13 combinations trên.
- **Output thô:** ~30 câu.
- **Human filter (đề mục 12) — đã loại 4 câu:**
  1. *"Đà Nẵng 3 ngày 2 đêm thích biển ngân sách 5 triệu lên lịch"* (DC01) → AI **tự thêm context** (ngày/ngân sách) làm case hết mơ hồ → đổi mất mục đích test. Loại.
  2. *"Lên lịch Đà Lạt 2 ngày giúp mình"* (DC02) → AI tự điền "2 ngày", phá tình huống "thiếu ngày". Loại.
  3. *"Quán Bún Chả Cá 109 mở tới 22h nhé"* (DC07) → AI khẳng định giờ như chắc chắn (đáng ra để mơ hồ cho agent flag). Sửa còn D13/D14 dạng hỏi.
  4. *"Cho mình lịch chill Đà Lạt"* (DC10) → mất phần trộn Việt–Anh cần test → giữ bản mixed D19/D20.
- **Giữ lại:** 26 user inputs → `v0_du.csv`.

## Bài 5 — Scenario Dataset v0 + Coverage note
Dataset: [`v0_du.csv`](v0_du.csv) — 26 rows, map rõ combination + expected_behavior high-level.

**Coverage note (đề mục 15):**
- **Cover tốt:** input mơ hồ/thiếu/mâu thuẫn (`missing_constraint`), dữ liệu cũ/không rõ (`stale_live_data`), và đa dạng persona/style (giận, trộn Việt–Anh, ngắn cộc, beginner).
- **Chưa cover (Anh đã bù):** tính khả thi thời gian/ngân sách (`infeasible_schedule`, `budget_violation`) và ranh giới quyền (`booking_overreach`, `missing_recovery`).
- **Cố tình chưa chọn:** case đa ngôn ngữ ngoài Việt/Anh — ngoài phạm vi persona chính.
- **High-risk nhất:** DC09 (user giận + thông tin cũ → dễ phòng thủ/khẳng định sai).
- **Boundary khó nhất:** DC13 ("chỗ mới hot chưa ai biết") — dễ khiến agent **bịa địa điểm** để chiều user.

### Checklist cá nhân
- [x] Use case + Unit of AI Work · [x] Quality question · [x] Grid ≥3 dim (có 5)
- [x] ≥10 combinations (13) · [x] Prompt generate · [x] ≥20 inputs sau lọc (26)
- [x] Scenario Dataset v0 · [x] Coverage note

*v0 · Nguyễn Viết Du · Day 21 · AI Travel Planner.*
