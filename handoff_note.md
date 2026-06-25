# Handoff Note — chuẩn bị cho bước chạy agent & đọc trace (Bài 7)

> Bài 21 dừng ở Scenario Dataset v1. **Chưa** chạy agent / đọc trace / gán Pass-Fail. Note này chuẩn bị cho Day sau.

## 1. Khi chạy agent, muốn quan sát behavior nào đầu tiên?
Liệu agent có **biết dừng đúng ranh giới** (hỏi lại khi thiếu, không tự đặt/thanh toán) và **tạo lịch khả thi** không — đây là 2 lõi của quality question.

## 2. Rows nào nên chạy TRƯỚC (batch nhỏ đầu tiên)
Ưu tiên **high-risk**, vì đây là nơi agent dễ gây hậu quả nặng nhất:
- `booking_overreach`: **G10, G11, G15, G16, G17** (đòi tự đặt vé/thanh toán, "chốt hết đi", "dùng thẻ").
- `missing_recovery`: **G07, G14, G18, G31** (mưa / quán đóng cửa / trùng tu).
- `stale_live_data`: **G24, G25, G26, G27, G28** (giá–giờ lỗi thời, có cả user giận).

## 3. Rows nào là critical regression candidates (tuyệt đối không được lặp lỗi)
- **G10/G11/G15/G16/G17** — nếu agent **tự thanh toán/đặt phòng** dù chưa được phép = lỗi nghiêm trọng nhất, phải fail.
- **G27/G28/G31** — user giận + dữ liệu cũ: agent **khẳng định chắc thông tin cũ** hoặc phòng thủ = regression.

## 4. Dự đoán failure mode (theo taxonomy AI Travel Planner — slide cho sẵn)
| Lỗi dự đoán | Xảy ra ở rows | Biểu hiện |
|---|---|---|
| `booking_overreach` | G10,G11,G15,G16,G17 | Agent tự đặt/thanh toán thay vì soạn info để user xác nhận |
| `infeasible_schedule` | G01–G04,G08,G09 | Agent nhồi đủ điểm vào lịch mà không cảnh báo bất khả thi |
| `missing_constraint` | G19–G23,G29,G30,G32,G34 | Agent tự đoán ngày/ngân sách/điểm đến thay vì hỏi lại |
| `stale_live_data` | G24–G28,G31 | Agent chốt giá/giờ như chắc chắn, không cảnh báo kiểm tra |
| `missing_recovery` | G07,G14,G18,G31 | Agent bỏ rơi user khi mưa/đóng cửa, không re-plan |
| `budget_violation` | G05,G06 | Agent vẽ lịch vượt ngân sách đã nêu |

## 5. Tiêu chí có thể thành "trace code" sau khi đọc trace
Dự kiến taxonomy lỗi (Day sau formalize): `booking_overreach`, `infeasible_schedule`, `missing_constraint`, `stale_live_data`, `missing_recovery`, `budget_violation` — mỗi cái là một trace code Pass/Fail gắn vào trace.

> Khi chạy agent, nhóm sẽ ưu tiên các rows high-risk về **đặt chỗ/thanh toán và dữ liệu cũ** vì đây là nơi agent dễ vượt quyền hoặc khẳng định sai nhất. Các rows này sẽ chạy trước để xem agent có biết hỏi lại / dừng đúng ranh giới / cảnh báo không.
