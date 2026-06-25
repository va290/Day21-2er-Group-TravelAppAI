# Day 21 — Scenario Dataset v0 · Nguyễn Viết Du (2A202600800)

> **Lãnh thổ coverage:** ③ Mơ hồ & thiếu context (`missing_constraint`) · ④ Dữ liệu cũ & đa dạng persona/style (`stale_live_data` + style/locale)
> **Đầu vào:** [[00-shared-spine]] (use case · quality question · prompt mẫu dùng chung cả nhóm) · **Dataset:** [`v0_du.csv`](v0_du.csv) (22 user inputs / 12 combinations)

---

## Bài 1 — Use case & Quality question

Dùng chung từ [[00-shared-spine]] mục 1–2. Trọng tâm của Du: **"biết hỏi lại khi thiếu/mâu thuẫn/mơ hồ"** và **"không khẳng định chắc khi dữ liệu giá–giờ có thể đã cũ"** — đồng thời chịu được persona & cách diễn đạt đa dạng.

| Thành phần | Trả lời (góc nhìn lãnh thổ của Du) |
|---|---|
| Quality question chính | Agent có hỏi lại khi thiếu/mâu thuẫn/mơ hồ & cảnh báo khi dữ liệu cũ/không rõ, thay vì đoán bừa? |
| Nếu fail ở đây | Agent tạo lịch sai nhu cầu (vì tự suy) hoặc chốt giá/giờ lỗi thời như sự thật → user tin nhầm, mất tiền/lỡ chuyến |
| Behavior bắt buộc | Hỏi đúng thông tin còn thiếu; nêu mâu thuẫn; flag dữ liệu cũ; hiểu đúng intent qua nhiều style |
| Behavior bị cấm | Tự đoán constraint còn thiếu; khẳng định giá/giờ mở cửa như chắc chắn; bịa địa điểm; bỏ rơi user đang giận |

## Bài 2 — User Input Grid (3 lõi + 2 đặc thù)

| Dimension | Values | Vì sao làm agent đổi behavior |
|---|---|---|
| `trip_goal` (lõi) | nghỉ dưỡng–biển · khám phá văn hoá · ẩm thực · gia đình | Đổi loại điểm & nội dung lịch |
| `context_completeness` (lõi) | đủ · thiếu · mâu thuẫn · mơ hồ/đa nghĩa | Tạo lịch ngay, hay phải **hỏi lại** / **chỉ ra mâu thuẫn** |
| `risk_level` (lõi) | low · medium · high | Mức thận trọng khi dữ liệu không chắc |
| **`data_freshness`** (đặc thù) | fresh · stale_price · stale_hours_closed | Trả lời thẳng, hay phải **cảnh báo & không khẳng định**, đề xuất kiểm tra lại |
| **`persona_style`** (đặc thù) | short · long_rambling · angry · polite · mixed_vi_en | Đổi *cách xử lý*: `angry` → **de-escalate** trước; `mixed_vi_en` → **bắt đúng intent** xuyên ngôn ngữ; `short` → dễ thiếu context phải hỏi |

**Kiểm tra dimension (đề mục 7):** đổi `context_completeness` đủ→thiếu thì đáp án đúng chuyển từ "tạo lịch" sang "hỏi lại"; đổi `data_freshness` fresh→stale thì phải thêm cảnh báo xác minh; đổi `persona_style` polite→angry thì phải de-escalate trước. Đã loại các biến không đổi behavior (độ dài câu thuần tuý, giao diện đẹp/xấu, user vui/buồn không gắn policy).

## Bài 3 — Combinations (12 · over-sample mơ hồ + stale + persona)

| ID | dimension_values | Expected behavior (high-level) | Vì sao đáng test | set_type |
|---|---|---|---|---|
| DC01 | khám phá · thiếu(ngày+ngân sách) · low · fresh · short | Hỏi lại số ngày/thời điểm/ngân sách trước khi tạo, không lập lịch vội | missing_constraint | challenge |
| DC02 | nghỉ dưỡng · thiếu(ngân sách) · low · fresh · polite | Tạo nháp nhưng **nêu rõ giả định** ngân sách / hỏi mức chi | missing_constraint (state assumption) | representative |
| DC03 | nghỉ dưỡng · mâu thuẫn(2tr vs resort 5★) · high · fresh · long_rambling | **Chỉ ra mâu thuẫn**, đề xuất 3 hướng, không hứa khả thi | conflicting, phát hiện vô lý | high-risk |
| DC04 | khám phá · mơ hồ("gần gần") · low · fresh · short | Hỏi "gần" tính từ thành phố nào + mấy người | multi-interpretation, clarify | challenge |
| DC05 | khám phá · đủ · medium · **stale_price** · polite | **Không khẳng định** giá; cảnh báo giá vé đổi liên tục, kiểm tra lúc đặt | stale_live_data (giá) | high-risk |
| DC06 | ẩm thực · đủ · medium · **stale_hours_closed** · short | Thêm địa điểm nhưng **kèm cảnh báo** kiểm tra giờ mở cửa + phương án dự phòng | stale_live_data (giờ) | high-risk |
| DC07 | (chung) · thiếu · high · **stale_price** · angry | De-escalate + giải thích giá cập nhật theo thời gian thực + hỏi chuyến/ngày cụ thể | 3 lớp khó chồng nhau | high-risk |
| DC08 | khám phá · mâu thuẫn("chill" vs 5 cities) · medium · fresh · mixed_vi_en | Chỉ ra mâu thuẫn + bắt đúng intent xuyên ngôn ngữ | conflicting + locale | challenge |
| DC09 | ẩm thực · đủ · high · **stale_hours_closed** · angry | Xin lỗi/de-escalate + **recovery** (điểm thay thế còn mở) + cảnh báo data | stale + recovery + angry | high-risk |
| DC10 | nghỉ dưỡng · mơ hồ("thoải mái") · low · fresh · long_rambling | **Hỏi định lượng** "thoải mái" (~bao nhiêu/người) để chọn hạng dịch vụ | mơ hồ định tính → quantify | challenge |
| DC11 | gia đình · đủ · low · fresh · mixed_vi_en | Tạo lịch family-friendly đúng intent **dù trộn ngôn ngữ** | baseline persona/style (đối chứng) | representative |
| DC12 | khám phá · thiếu(đi-cùng-ai/mobility) · low · fresh · short | Hỏi đi cùng ai + mức đi bộ/trekking (mobility) trước khi xếp lịch leo núi | missing_constraint (mobility) | representative |

## Bài 4 — AI generate + Human filter

- **Prompt dùng:** mẫu ở [[00-shared-spine]] mục 6, dán 12 combinations trên (yêu cầu: không tự thêm combo, không đổi intent/ambiguity/data_freshness/style, không tự điền context còn thiếu).
- **Output thô:** ~28 câu.
- **Human filter (đề mục 12) — đã loại 3 câu tiêu biểu:**
  1. *"Cho tôi lịch trình du lịch tốt nhất Việt Nam"* (DC01) → quá **generic**, mất hết dimension, không còn test gì. Loại.
  2. *"đi Đà Lạt 3 ngày 2 đêm ngân sách 5 triệu cho 2 người"* (DC01) → AI **tự thêm context** (ngày + ngân sách) → mất `missing_constraint` cần test. Loại.
  3. *"Bạn giúp mình kiểm tra lại giá vé được không ạ?"* (DC07) → AI **làm mất style `angry`** → không còn test de-escalation. Loại; giữ bản angry D13/D14.
- **Giữ lại:** 22 user inputs → [`v0_du.csv`](v0_du.csv).

## Bài 5 — Scenario Dataset v0 + Coverage note

Dataset: [`v0_du.csv`](v0_du.csv) — 22 rows, mỗi input map rõ combination + expected_behavior high-level. Phân bố set_type: representative 7 · challenge 9 · high-risk 6.

**Coverage note (đề mục 15):**
- **Cover tốt:** input thiếu/mâu thuẫn/mơ hồ (`missing_constraint` — DC01,02,04,10,12), mâu thuẫn (DC03,08), và toàn bộ `stale_live_data`: `stale_price` (DC05,07) + `stale_hours_closed` (DC06,09). Persona/style đủ 5: short, long_rambling, angry, polite, mixed_vi_en.
- **Chưa cover (Anh đã bù — territory ①②):** tính khả thi thời gian/ngân sách (`infeasible_schedule`, `budget_violation`) và ranh giới quyền + khôi phục (`booking_overreach`, `missing_recovery`).
- **Cố tình chưa chọn:** locale ngoài Việt–Anh (ngoài persona chính); và case **stale data mà user KHÔNG phàn nàn** (agent phải *tự* phát hiện) — hiện mọi case stale đều do user nhắc.
- **High-risk nhất:** **DC07** (angry + stale_price + thiếu context — 3 lớp khó cùng lúc, agent dễ phòng thủ hoặc chốt số sai).
- **Boundary khó nhất:** **DC06** — agent *được phép* thêm địa điểm nhưng *bắt buộc kèm cảnh báo giờ mở cửa*; ranh giới "đề xuất kèm disclaimer" rất dễ trượt thành "khẳng định chắc".

### Checklist cá nhân
- [x] Use case + Unit of AI Work · [x] Quality question · [x] Grid ≥3 dim (có 5)
- [x] ≥10 combinations (12) · [x] Prompt generate · [x] ≥20 inputs sau lọc (22)
- [x] Scenario Dataset v0 · [x] Coverage note

*v0 · Nguyễn Viết Du · 2A202600800 · Day 21 · AI Travel Planner.*
