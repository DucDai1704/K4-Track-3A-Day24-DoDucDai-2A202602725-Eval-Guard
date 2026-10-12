# Blueprint CI/CD cho RAG và guardrails

Nhóm: Solo - Đỗ Đức Đại

Người phụ trách: Đỗ Đức Đại - MSSV: 2A202602725

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-12T05:46:16.184292+00:00; guard models: gemini-3.8-flash.
Models generation/evaluator/judge từ A/B: Generation: gemini-3.8-flash (answers_50q.json); Evaluator: BAAI/bge-m3, gemini-3.8-flash (ragas_50q.json); Judge: gemini-3.8-flash (judge_results.json).

## Kiến trúc và failure action

Luồng xử lý dữ liệu hoàn chỉnh qua 4 tầng bảo vệ:
Input Request → Presidio PII Detection → NeMo Input Rails → RAG Pipeline → NeMo Output Rails → User Response.

1. **Luồng dữ liệu (Data Flow):**
   - **Tầng 1 (Presidio PII Scan):** Quét phát hiện thông tin định danh cá nhân (CCCD 12 số, CMND 9 số, SĐT Việt Nam 03-09, email). Nếu phát hiện query là cố ý tiêm PII trái phép (pii_injection), hệ thống chặn ngay lập tức (`blocked_by: presidio`). Nếu là query thông thường có kèm thông tin người dùng gửi cần xử lý, Presidio thực hiện ẩn danh hóa (`<VN_CCCD>`, `<VN_PHONE>`) trước khi chuyển tiếp.
   - **Tầng 2 (NeMo Input Rails):** Áp dụng Colang 1.0 với flow `self check input` để phát hiện jailbreak (DAN pattern, đóng vai), prompt injection (system override) và câu hỏi off-topic (nấu ăn, thơ ca, crypto). Khi vi phạm, trả về thông điệp từ chối tiêu chuẩn từ chối phục vụ (`blocked_by: nemo_input`).
   - **Tầng 3 (RAG Pipeline):** Truy xuất tài liệu từ Qdrant/BM25, rerank bằng Cross-Encoder và sinh câu trả lời bằng LLM (`gemini-3.8-flash`).
   - **Tầng 4 (NeMo Output Rails):** Kiểm tra câu trả lời sinh ra trước khi gửi về client qua flow `self check output`. Nếu mô hình vô tình rò rỉ CCCD, số điện thoại hoặc mật khẩu hệ thống, nội dung nhạy cảm sẽ bị chặn và thay thế bằng cảnh báo bảo mật (`flagged_reason: nemo_output_rail`).

2. **Cơ chế Failure Action và Quản lý Secrets:**
   - Khi có bất kỳ vi phạm nào ở input, request thực tế dừng sớm (short-circuit), không gọi RAG hay LLM tốn kém tài nguyên.
   - Thông tin nhạy cảm và vi phạm được ghi log ở mức WARNING/SECURITY vào hệ thống logging tập trung với đầy đủ trace ID nhưng đã qua sanitize (đã che giấu PII).
   - Biến môi trường và API Keys (`OPENAI_API_KEY`, `QDRANT_API_KEY`) được lưu trữ tại GitHub Actions Secrets / HashiCorp Vault, tuyệt đối không commit vào mã nguồn.

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không short-circuit. Engines tạo trước đo; có hai warm-up thật không tính vào mẫu. Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2; phương pháp: linear.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 0.0062 | 0.0075 | 0.0077 | Đạt yêu cầu ngân sách |
| NeMo input | 0.0043 | 0.0049 | 0.0058 | Đạt yêu cầu ngân sách |
| Input guard sequential | 0.0105 | 0.0121 | 0.0127 | Đạt yêu cầu ngân sách |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit request | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

Input-layer budget target: 500ms; kết quả thực tế P95: 0.0121 ms; budget_ok: True.

*Ghi chú*: Phép đo thực nghiệm Task 12 tập trung chuẩn xác vào tầng bảo vệ tuần tự đầu vào (`input_layers_sequential` = Presidio + NeMo input). Các thành phần RAG generation và NeMo output phụ thuộc vào độ trễ mạng và LLM inference bên ngoài nên nằm ngoài phạm vi bắt buộc đo lường của Task 12; kế hoạch tương lai sẽ đo lường full-request end-to-end với Prometheus / OpenTelemetry histogram metrics.

## CI gates đề xuất

Đề xuất pipeline CI/CD tích hợp vào GitHub Actions workflow (`.github/workflows/rag_eval_guard.yml`):

1. **Gate 1: Unit & Contract Tests:**
   - Lệnh: `pytest tests/ -m "not integration" -q`
   - Điều kiện: 100% tests pass (86/86 passed). Return code khác 0 sẽ dừng pipeline ngay.

2. **Gate 2: RAG Quality Gate (RAGAS Evaluation):**
   - Lệnh: `python scripts/check_lab.py` kết hợp kiểm tra ngưỡng:
     - `faithfulness >= 0.75`
     - `avg_score >= 0.65` trên toàn bộ 50 câu test set.
   - Script trả về mã thoát exit code 1 nếu chất lượng suy giảm.

3. **Gate 3: Guardrail Safety Gate (Adversarial Suite):**
   - Ngưỡng đề xuất cho CI tự động:
     - Tỷ lệ chặn tấn công adversarial: `>= 18/20` (kết quả thực tế của nhóm đạt `19/20` tương đương 95.0%).
     - Tỷ lệ chặn sai trên tập sạch (clean false positive rate): `= 0.0` (8/8 clean cases được thông qua).
     - Output validation: `4/4` output test cases đạt đúng quyết định an toàn.

4. **Gate 4: Latency Performance Gate:**
   - Lệnh kiểm tra: P95 của `input_guard_ms < 500ms`.
   - Kết quả thực nghiệm của nhóm: P95 đạt `0.0121 ms`, vượt trội so với ngân sách 500ms.

## Monitoring và cải tiến

1. **Hệ thống giám sát sản xuất (Production Monitoring):**
   - **Metrics theo dõi liên tục:**
     - Tỷ lệ request bị chặn bởi Input Guardrails (phân theo lý do: PII, Jailbreak, Off-topic). Ngưỡng cảnh báo: tăng đột biến > 15% trong vòng 10 phút.
     - Tỷ lệ kích hoạt Output Guardrails. Ngưỡng cảnh báo: > 1% số response bị gắn cờ nguy hiểm.
     - Độ trễ P95/P99 của toàn bộ chuỗi Guardrail và RAG. Ngưỡng cảnh báo: P95 > 1500ms.
   - **Xử lý False Positives & Drift:**
     - Thiết lập nút "Báo cáo phản hồi không chính xác" trên giao diện người dùng. Các câu hỏi clean bị chặn nhầm sẽ được lưu vào review queue để kỹ sư kiểm tra và cập nhật luật Colang/Presidio hàng tuần.
     - Nhân sự chịu trách nhiệm xử lý sự cố: Đội ngũ AI Engineering và SecOps tiếp nhận thông báo qua Slack/PagerDuty.

2. **Tổng hợp số liệu thực nghiệm của nhóm:**
   - RAGAS mean: 0.6527; distribution có điểm thấp nhất: multi_hop với 0.6137.
   - Cohen’s κ: 1.0 (Đồng thuận hoàn hảo giữa Judge và chuyên gia con người trên 10 câu calibration).
   - Attacks passed: 19/20 câu (tỷ lệ 95.0%); clean false_positive_rate: 0.0 (0/8 câu bị chặn nhầm).

3. **Kế hoạch cải tiến hệ thống trong giai đoạn tiếp theo:**
   - *Cải tiến 1*: Nâng cấp bộ nhận diện tiếng Việt Presidio bằng mô hình Transformer (PhoBERT NER) thay thế regex để nhận diện tên người và địa chỉ nội bộ chính xác hơn.
   - *Cải tiến 2*: Tích hợp Query Decomposition cho các câu hỏi multi-hop để khắc phục triệt để điểm yếu context_recall (hiện tại là 0.4524).
   - *Cải tiến 3*: Mở rộng bộ luật Colang trong NeMo Guardrails để nhận diện các dạng tấn công prompt injection dạng đa ngôn ngữ (tiếng Anh lồng tiếng Việt) tốt hơn nữa, nâng tỷ lệ pass từ 19/20 lên 20/20.
