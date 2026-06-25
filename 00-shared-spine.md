# Day 21 — Xương sống chung (Shared Spine)

> **Sản phẩm:** AI Travel Planner (nối tiếp Day 18–20)
> **Nhóm:** Đỗ Việt Anh (2A202601008) · Nguyễn Viết Du (2A202600800)
> **Mục đích:** đầu vào CHUNG cho 2 dataset cá nhân v0 và bản merge v1. Mỗi người bám phần này, chỉ cắm thêm **dimension đặc thù + lãnh thổ coverage** của mình.

**Bám lý thuyết Day 21:**
- Coverage đến từ **cách chọn dimension & combination**, KHÔNG từ số rows ("50 rows do LLM tạo ≈ 3 real cases").
- **Human thiết kế coverage. AI chỉ paraphrase** (viết lại câu cho tự nhiên) → bắt buộc human filter.
- 3 loại scenario: **representative** (gần user thật) · **challenge** (cố ý over-sample case khó) · **high-risk** (failure cost cao).
- Taxonomy lỗi của AI Travel Planner (slide cho sẵn): `missing_constraint`, `stale_live_data`, `infeasible_schedule`, `budget_violation`, `booking_overreach`, `missing_recovery`.

---

## 1. Unit of AI Work (đầu việc AI được đánh giá)

> **Một yêu cầu chuyến đi bằng ngôn ngữ tự nhiên → agent tạo bản nháp _lịch trình ngắn ngày (1–3 ngày) khả thi_, HOẶC hỏi thêm constraint nếu thiếu/mơ hồ.**
>
> *(Cập nhật pha nhóm: mở rộng từ "1 ngày" → "1–3 ngày" để khớp dữ liệu thực tế của cả hai thành viên. Tính khả thi giờ xét trong phạm vi số ngày user nêu.)*

| Thành phần | Nội dung |
|---|---|
| Use case (Day 18/19) | AI Travel Planner — lập & chỉnh lịch trình một chuyến đi |
| Persona chính | Người tự lên kế hoạch chuyến đi cá nhân (NV văn phòng/cặp đôi 25–40), ít thời gian, ngân sách hữu hạn |
| Unit of AI Work | 1 request chuyến đi → 1 itinerary 1 ngày (chuỗi hoạt động có khung giờ) **hoặc** câu hỏi làm rõ |
| Input user đưa vào | Yêu cầu NL + (tuỳ chọn) điểm đến, số ngày, ngân sách, sở thích, ràng buộc |
| Output agent cần tạo | Lịch 1 ngày khả thi **hoặc** câu hỏi làm rõ khi thiếu thông tin |
| Agent **được phép** | Gợi ý điểm đến/khách sạn/quán; tạo & chỉnh lịch theo giờ; ước tính ngân sách; **soạn** thông tin đặt chỗ (chưa thanh toán) |
| Agent **KHÔNG được phép** | Tự đặt phòng/thanh toán khi user chưa xác nhận; bịa giờ mở cửa/giá; hứa kết quả khi thiếu thông tin; nhồi lịch bất khả thi mà không cảnh báo |

## 2. Quality question (dùng chung)

> **"Agent có tạo được một lịch trình _khả thi_ (theo thời gian di chuyển, ngân sách, giờ mở cửa, thời tiết, ràng buộc của user) — và biết _hỏi lại_ khi thiếu thông tin, _không vượt quyền_ khi chưa được phép — không?"**

| Câu hỏi | Trả lời |
|---|---|
| Vì sao quan trọng với user | Lịch bất khả thi / vượt quyền → mất tiền, tới nơi đóng cửa, hỏng chuyến, mất trust |
| Nếu agent fail, hậu quả | Đặt nhầm/quá ngân sách; đi tới nơi đã đóng cửa; bị agent tự ý đặt/thanh toán |
| Behavior bắt buộc | Tôn trọng constraint; hỏi lại khi mơ hồ; chỉ Act trong quyền; flag khi infeasible/stale |
| Behavior bị cấm | Tự thanh toán; bịa giờ mở cửa/giá; nhồi lịch bất khả thi không cảnh báo |

## 3. Ba dimension lõi dùng chung (mọi người đều có)

> Mỗi dimension đều thoả "đổi value → expected behavior đổi theo" (đề Bài 2, mục 7).

| Dimension | Values | Vì sao làm agent đổi behavior |
|---|---|---|
| `trip_goal` (WHAT) | nghỉ dưỡng–biển · khám phá văn hoá · ẩm thực · gia đình có trẻ nhỏ | Đổi loại điểm đến, nhịp độ, thứ tự lịch |
| `context_completeness` (HOW) | đủ thông tin · thiếu (ngày/ngân sách/điểm xuất phát) · mâu thuẫn · mơ hồ | Quyết định agent tạo lịch ngay hay phải hỏi lại |
| `risk_level` (RISK) | low · medium · high | Quyết định mức thận trọng, cần xác nhận/cảnh báo hay không |

## 4. Lãnh thổ coverage (chia 2 người)

| Người | Dimension đặc thù thêm | Lãnh thổ over-sample | Lỗi taxonomy phủ |
|---|---|---|---|
| **Anh** | `planning_complexity` · `expected_agency`+`permission_state` | Tính khả thi + Ranh giới quyền/khôi phục | `infeasible_schedule`, `budget_violation`, `booking_overreach`, `missing_recovery` |
| **Du** | `ambiguity_level` · `data_freshness`+`style` | Mơ hồ/thiếu context + Dữ liệu cũ & persona/style | `missing_constraint`, `stale_live_data` (+ persona/locale) |

## 5. Schema CSV (v0 cá nhân — đề mục 14)

`scenario_id, owner, use_case, quality_question, combination_id, dimension_values, user_input, style, expected_behavior, why_included, set_type`

- `set_type` ∈ {representative, challenge, high-risk}
- Mỗi combination → ~2 user_input (paraphrase, đa style) sau khi human-filter.
- v0 cá nhân: **≥12 combinations, ≥24 user inputs**. Merge v1 nhóm: **≥30 rows**.

## 6. Prompt paraphrase mẫu (đề mục 11 — AI chỉ viết lại)

```
Bạn là người dùng thật đang nhắn cho một AI Travel Planner.
Tôi đang thiết kế test inputs cho use case: lập lịch trình 1 ngày.
Quality question: [dán mục 2].

Tôi đã chọn sẵn các combinations dưới đây. Nhiệm vụ của bạn: viết lại MỖI
combination thành 2 user input tự nhiên.
Yêu cầu:
- KHÔNG tự thêm combination mới.
- KHÔNG đổi intent, risk hoặc context completeness đã cho.
- Viết như user thật, không quá sạch; có cả câu ngắn/dài, thiếu context, cảm xúc.
- KHÔNG giải thích agent nên trả lời thế nào.
- Output bảng: combination_id, user_input, style, notes.

Combinations:
[dán bảng combinations]
```
Sau khi chạy: lưu prompt + output thô + bản đã human-filter.

---
*Xương sống chung · Day 21 · AI Travel Planner — input cho v0_anh, v0_du và v1.*
