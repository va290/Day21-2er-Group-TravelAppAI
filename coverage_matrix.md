# Coverage Matrix — Scenario Dataset v1 (34 rows)

> Gom từ `v0_anh.csv` (Anh) + `v0_du.csv` (Du) → [`scenario_dataset_v1.csv`](scenario_dataset_v1.csv).
> Quy trình merge nhóm (đề mục 17): chuẩn hóa dimension → dedup → coverage review → chốt v1.

## Step 1–2 · Chuẩn hóa dimension (gom tên tương đương)

| Cách gọi của 2 người | Chuẩn hóa thành |
|---|---|
| trip_goal (cả hai) | `trip_goal` |
| context_completeness (cả hai) | `context_completeness` (đủ/thiếu/mâu thuẫn/mơ hồ) |
| risk_level (cả hai) | `risk_level` |
| Anh: planning_complexity | `planning_complexity` *(giữ — đặc thù feasibility)* |
| Anh: expected_agency + permission_state | `agency` *(giữ — đặc thù quyền)* |
| Du: data_freshness | `data_freshness` *(giữ — đặc thù stale)* |
| Du: persona_style | `persona_style` *(giữ — đặc thù persona)* |

→ **3 dimension lõi dùng chung** + 4 dimension đặc thù (2 mỗi người) giữ làm tag bổ sung.

## Step 3 · Dedup
- **Không có câu trùng y hệt** giữa 2 bộ.
- Bỏ **14 paraphrase gần trùng** (cùng combination, chỉ khác wording) → `merge_decision = merged` đại diện cho cặp.
- Giữ cả 2 câu khi test **style/persona/khía cạnh khác nhau rõ** → `merge_decision = kept`.
- Tổng: 48 câu thô (26+22) → **34 rows v1** (20 kept · 14 merged).

## Step 4 · Coverage Matrix (theo 6 lỗi taxonomy AI Travel Planner)

| Slice / loại lỗi | Số rows v1 | Đủ chưa? | Ghi chú |
|---|---|---|---|
| `infeasible_schedule` (lịch quá dày/gói nhiều ngày/giờ mở cửa) | 6 | ✅ | Anh phụ trách |
| `budget_violation` (vượt/sát ngân sách) | 4 | ✅ | Anh (tight) + Du (mâu thuẫn ngân sách) |
| `booking_overreach` (tự đặt/thanh toán, vượt quyền) | 5 | ✅ | Anh phụ trách — critical |
| `missing_recovery` (mưa/đóng cửa → re-plan) | 4 | ✅ | Anh (thời tiết/trùng tu) + Du (đóng cửa+giận) |
| `missing_constraint` (mơ hồ/thiếu/mâu thuẫn) | 9 | ✅ | Du phụ trách — đậm nhất |
| `stale_live_data` (giá/giờ lỗi thời) | 7 | ✅ | Du phụ trách |
| persona: `angry` (cần de-escalate) | 3 | ✅ | Du |
| persona: `mixed VI-EN` (bắt intent xuyên ngôn ngữ) | 3 | ✅ | Du |
| **representative / happy-path** | 5 | ⚠️ mỏng | Cả hai over-sample case khó → happy-path ít (xem known_gaps) |

## Step 5 · Final set v1 đạt yêu cầu (đề mục 17)
- ✅ representative (5) · challenge (16) · high-risk (13)
- ✅ ≥2 ambiguous/missing-context (9) · ≥2 high-risk (13) · ≥2 case agent dễ chọn sai action (booking_overreach 5)
- ✅ đa dạng diễn đạt: ngắn/dài/giận/lịch sự/trộn Việt–Anh
- ✅ đa thành phố: Hà Nội, Đà Nẵng, Sài Gòn, Phú Quốc, Đà Lạt, Sa Pa, Nha Trang, Huế, Hội An (+7 dòng "không rõ" để test agent hỏi điểm đến)

**Số rows v1 = 34 (≥30 đạt).**
