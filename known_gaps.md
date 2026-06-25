# Known Gaps & Priority — Scenario Dataset v1

> Trả lời nhanh các câu hỏi review nhóm (đề mục 19) + những chỗ còn thiếu để bổ sung sau.

## Group coverage review (mục 19)

| Câu hỏi | Trả lời |
|---|---|
| Dataset v1 cover tốt slice nào? | `missing_constraint` (9), `stale_live_data` (7), `infeasible_schedule` (6), `booking_overreach` (5); persona giận/trộn-ngôn-ngữ. |
| Slice nào còn thiếu/yếu? | **Happy-path representative (chỉ 5)**; `permission_state = có quyền` (Act) ít; locale ngoài Việt/Anh = 0; lịch dài >3 ngày = 0 (ngoài scope). |
| Có over-sample happy path không? | **Ngược lại — UNDER-sample.** Cả hai cố ý lấy nhiều case khó → pass rate trên v1 sẽ thấp hơn production thật (đúng cảnh báo slide: "pass rate trên challenge set ≠ production success rate"). |
| Có row high-risk nào chưa rõ expected behavior? | Không — mọi high-risk đều có expected_behavior + risk_if_fail. |
| AI generation có bóp méo combination ở đâu? | Đã chặn ở human-filter (Anh loại 5, Du loại 3 câu AI tự thêm context/đổi intent/làm mất style). |
| Nếu chỉ chạy 1 batch nhỏ đầu tiên, chọn rows nào? | Các high-risk `booking_overreach` + `stale_live_data` + `missing_recovery` (xem handoff_note). |

## Known gaps (ưu tiên bổ sung ở vòng sau)

1. ~~**Happy-path mỏng (P1)**~~ → **ĐÃ VÁ:** nhóm bổ sung 6 case representative (G35–G40) "user nói rõ, đủ thông tin, 1–3 ngày". Representative giờ 11/40.
2. **`agency = Act/có quyền` ít (P2):** đa số case quyền là Ask/Don't Act; thêm vài case agent ĐƯỢC PHÉP tự làm (thêm điểm, đổi giờ) để test agent không quá rụt rè.
3. **City "không rõ" (P3):** 7 dòng (chủ yếu của Du) chưa gắn thành phố — bản thân việc thiếu điểm đến là chủ ý test `missing_constraint`, nhưng nên ghi chú rõ trong cột để khỏi nhầm là thiếu dữ liệu.
4. **Đa ngôn ngữ ngoài Việt/Anh (P3):** ngoài scope persona chính, chưa test.
5. **Combination chưa chọn có chủ đích:** lịch >3 ngày, đặt nhóm đông (>4 người) — để vòng sau khi mở rộng Unit of AI Work.
