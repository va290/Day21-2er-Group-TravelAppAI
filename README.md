# Day 21 Lab — Thiết kế Test Inputs cho AI Evals

**Sản phẩm:** AI Travel Planner (nối tiếp Day 18–20) · **Use case:** lập lịch trình 1 ngày.
**Mục tiêu lab:** thiết kế **Scenario Dataset** (bộ test inputs) cho eval — *chưa* chạy agent/đọc trace.

> Nguyên tắc cốt lõi (lý thuyết Day 21): **coverage đến từ cách chọn dimension & combination, không từ số rows**; **human thiết kế coverage, AI chỉ paraphrase**.

## Nhóm (2 người)
| Họ tên | Mã HV | Lãnh thổ coverage |
|---|---|---|
| Đỗ Việt Anh | 2A202601008 | Tính khả thi (`infeasible_schedule`, `budget_violation`) + Ranh giới quyền/khôi phục (`booking_overreach`, `missing_recovery`) |
| Nguyễn Viết Du | 2A202600800 | Mơ hồ/thiếu context (`missing_constraint`) + Dữ liệu cũ & persona/style (`stale_live_data`) |

## Cấu trúc repo
| File | Nội dung |
|---|---|
| [`00-shared-spine.md`](00-shared-spine.md) | Xương sống chung: Unit of AI Work, quality question, 3 dimension lõi, schema CSV, prompt paraphrase |
| [`01-report-anh.md`](01-report-anh.md) | **Pha cá nhân của Anh** — Bài 1→5 (grid, combinations, human filter, coverage note) |
| [`v0_anh.csv`](v0_anh.csv) | **Scenario Dataset v0 của Anh** — 26 user inputs / 13 combinations / 8 thành phố |

## Tiến độ
- [x] Pha cá nhân — **Anh** (v0: 13 combinations, 26 inputs, đa thành phố, cân bằng feasibility–agency)
- [ ] Pha cá nhân — **Du** (Du tự làm & push riêng)
- [ ] Pha nhóm — merge → Scenario Dataset **v1** (≥30 rows) + coverage matrix + known gaps + handoff note

## Schema dataset (đề mục 14, +`city`)
`scenario_id, owner, use_case, quality_question, combination_id, city, dimension_values, user_input, style, expected_behavior, why_included, set_type`

`set_type` ∈ representative / challenge / high-risk · *(Anh nghiêng challenge/high-risk vì lãnh thổ là các case khó; happy-path representative để dành cho merge v1.)*

> Không đưa API key/token/credential vào repo.
