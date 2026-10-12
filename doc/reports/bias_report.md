# Đánh giá LLM Judge Phase B

Nhóm: Solo - Đỗ Đức Đại

Người phụ trách: Đỗ Đức Đại - MSSV: 2A202602725

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T07:03:17.101292+00:00.
Models generation/judge: Generation: gemini-3.8-flash; Judge: gemini-3.8-flash.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Câu hỏi | Winner P1 | Winner P2 | Final Winner | Consistent | Scores P1 (A/B) | Scores P2 (A/B) | Reasoning |
|---|---|---|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ bao nhiêu ngày khi kết hôn? | tie | tie | tie | True | 0.73 / 0.77 | 0.73 / 0.77 | Cả hai câu trả lời có chất lượng tương đương nhau về độ chính xác và độ đầy đủ. |
| 5 | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | A | A | A | True | 0.81 / 0.67 | 0.81 / 0.67 | Answer A chỉ rõ cấp Tổng Giám đốc (CEO) phê duyệt cho đơn trên 50 triệu, trong khi Answer B nêu sai là Giám đốc phòng ban. |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là bao nhiêu? | A | A | A | True | 0.77 / 0.65 | 0.77 / 0.65 | Answer A bám sát hơn nội dung quy định chính sách tham chiếu, cung cấp đầy đủ điều kiện 6 tháng và mức thưởng 1 tháng lương. |
| 21 | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | B | B | B | True | 0.61 / 0.67 | 0.61 / 0.67 | Answer B trả lời trọn vẹn cả 2 vế (phép năm 18 ngày và khung lương Senior 20-35 triệu), trong khi Answer A thiếu thông tin mức lương. |
| 41 | Nhân viên được nghỉ bao nhiêu ngày phép năm? | A | A | A | True | 0.58 / 0.52 | 0.58 / 0.52 | Answer A dẫn chứng rõ ràng xuất xứ văn bản quy định, mặc dù cả hai cần đối chiếu thêm với phiên bản hiện hành. |

Position bias count/rate: 0 câu không nhất quán / tỷ lệ 0.0 (0.0%).
Verbosity win rate: 0.75 (75.0%); decisive cases: 4 câu (trong 4 câu phân định thắng bại, có 3 câu câu trả lời dài hơn giành chiến thắng).

### Phân tích hiện tượng bias

1. **Position Bias:**
   - Cả 5 cặp đánh giá đều đạt kết quả position_consistent = True qua kỹ thuật swap-and-average (Pass 1 giữ nguyên thứ tự A-B, Pass 2 hoán đổi B-A rồi quy đổi về nhãn gốc). Tỷ lệ position bias là 0.0%, cho thấy kết quả phân định giữa hai câu trả lời rất ổn định và không phụ thuộc vào thứ tự xuất hiện của ứng viên trong prompt.
2. **Verbosity Bias:**
   - Trong 4 trường hợp có quyết định phân thắng bại rõ ràng (decisive cases: câu 5, 12, 21, 41), có 3 câu mà câu trả lời dài hơn giành chiến thắng (tỷ lệ 75.0%).
   - Tuy nhiên, qua phân tích nội dung, việc câu dài hơn chiến thắng ở câu 5 và câu 12 bắt nguồn từ việc câu trả lời cung cấp đầy đủ thông tin ngữ cảnh chính sách (nêu rõ căn cứ vượt ngưỡng 50 triệu và thời gian công tác 6 tháng), chứ không đơn thuần chỉ do độ dài cơ học. Ở câu 21, Answer B ngắn hơn một chút nhưng trả lời đủ 2 vế câu hỏi nên đã giành chiến thắng xứng đáng trước Answer A dài nhưng thiếu vế mức lương.

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json; không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference. Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| Question ID | Model Answer Gốc | Nhãn Judge | Human Label | Agree | Reasoning của Judge |
|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | 1 | 1 | True | Câu trả lời chính xác và đầy đủ theo chính sách tham chiếu. |
| 5 | Cần Giám đốc phòng ban phê duyệt. | 0 | 0 | True | Câu trả lời mâu thuẫn hoặc không chính xác với chính sách tham chiếu (vượt ngưỡng 50 triệu cần CEO). |
| 12 | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | 1 | 1 | True | Câu trả lời chính xác và đầy đủ theo chính sách tham chiếu. |
| 21 | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1 | 1 | True | Câu trả lời chính xác và đầy đủ theo chính sách tham chiếu. |
| 23 | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | 1 | True | Câu trả lời chính xác và đầy đủ theo chính sách tham chiếu. |
| 29 | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu. | 0 | 0 | True | Thiếu thẩm quyền phê duyệt của Kế toán trưởng theo quy định tạm ứng. |
| 33 | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng. | 1 | 1 | True | Câu trả lời chính xác và đầy đủ theo chính sách tham chiếu. |
| 41 | Nhân viên được nghỉ 12 ngày phép năm. | 0 | 0 | True | Câu trả lời mâu thuẫn hoặc không chính xác với chính sách tham chiếu (dùng chính sách cũ v2023). |
| 46 | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần. | 1 | 1 | True | Câu trả lời chính xác và đầy đủ theo chính sách tham chiếu. |
| 50 | Được, miễn là đảm bảo kết nối an toàn. | 0 | 0 | True | Câu trả lời mâu thuẫn hoặc không chính xác với chính sách tham chiếu (VPN cá nhân bị cấm). |

- **Hệ số Cohen's κ**: 1.0 (Thỏa thuận hoàn hảo giữa LLM Judge và nhãn con người).
- **Observed / Chance Agreement**:
  - Tỷ lệ quan sát đồng thuận: P_observed = 10/10 = 1.0 (100%).
  - Tỷ lệ đồng thuận ngẫu nhiên: P_chance = (6/10)*(6/10) + (4/10)*(4/10) = 0.36 + 0.16 = 0.52.
  - Công thức tính: κ = (P_observed - P_chance) / (1 - P_chance) = (1.0 - 0.52) / (1.0 - 0.52) = 1.0.
  - Diễn giải: Mức độ tương đồng hoàn toàn tuyệt đối giữa nhãn Judge và Human chuyên gia (κ > 0.8 là mức Near Perfect Agreement theo thang đo Landis & Koch).

## Giới hạn và cách dùng judge

1. **Giới hạn của LLM-as-Judge trong thực tế:**
   - Kích thước tập mẫu: Bộ câu hỏi kiểm chuẩn gồm 10 câu nhãn con người và 5 cặp pairwise. Mặc dù hệ số κ đạt 1.0 trên tập 10 câu này do các tiêu chí sai/đúng rất rành mạch (như sai quyền phê duyệt, sai chính sách cũ v2023), trong môi trường sản xuất với các câu hỏi mở rộng hoặc câu hỏi có tính nhập nhằng (ambiguous), LLM Judge vẫn có thể gặp sai sót nếu không có tài liệu ground truth chuẩn xác.
   - Verbosity và Style Bias: Mô hình LLM Judge có xu hướng thiên vị các câu trả lời dài và định dạng markdown đẹp mắt nếu không được kiểm soát chặt chẽ bằng prompt rubric chuẩn mực.
2. **Khuyến nghị sử dụng trong hệ thống RAG Production:**
   - Luôn áp dụng kỹ thuật Swap-and-Average khi chạy Pairwise Judge để triệt tiêu Position Bias.
   - Không sử dụng LLM Judge như nguồn sự thật duy nhất (sole source of truth); cần kết hợp với Human-in-the-loop định kỳ (lấy mẫu ngẫu nhiên 5-10% các câu hỏi biên) để tái hiệu chuẩn.
   - Tách biệt rõ ràng vai trò giữa Evaluator tự động (RAGAS / automated metrics) và LLM Judge phân tích chất lượng chuyên sâu.
