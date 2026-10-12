# Nhóm thực hiện K4 Track 3A

Tên nhóm: DoDucDai-Solo

Repo chung: https://github.com/DucDai1704/K4-Track-3A-Day24-DoDucDai-2A202602725-Eval-Guard

Trưởng nhóm/người nộp: Đỗ Đức Đại - 2A202602725

Ngày lab/deadline áp dụng: 2026-10-12 23:59 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: b9f8d5b53bc4f94899aa30fdc52c30be0e96eb6d

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thành viên thực hiện độc lập toàn bộ các phase của dự án.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Đỗ Đức Đại | 2A202602725 | Toàn bộ Phase A, Phase B, Phase C, tích hợp pipeline, runner và báo cáo | Commit git, phase_*.py, artifacts JSON và báo cáo markdown |

Người tích hợp/chạy batch/nộp link: Đỗ Đức Đại - 2A202602725, tích hợp toàn bộ module, thực thi kiểm thử và nộp repo.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: Windows 11, Python 3.12, MSVC Build Tools.
Virtualenv/dependencies/imports: pytest 86/86 passed, Presidio, NeMo Guardrails, RAGAS adapter.
Models/Qdrant/50 answers đã chuẩn bị: Qdrant Docker container đang chạy trên port 6333; BGE-M3 embedding; 5 module RAG từ Day 18 đã tích hợp.

## INDIVIDUAL

### DoDucDai-2A202602725

- Phần được giao và phần thực tế đã làm: Tự thực hiện độc lập toàn bộ các nhiệm vụ trong lab:
  + Tích hợp 5 module RAG (m1-m5) từ Day 18.
  + Hoàn thiện Phase A trong phase_a_ragas.py (group distribution, chạy RAGAS evaluation, lọc bottom 10, phân tích failure clusters).
  + Hoàn thiện Phase B trong phase_b_judge.py (pairwise LLM judge, swap-and-average khử position bias, tính Cohen kappa với human labels, phân tích verbosity bias).
  + Hoàn thiện Phase C trong phase_c_guard.py (tích hợp Presidio PII scanner với custom Vietnamese CCCD và phone recognizers, async NeMo input rail, adversarial test suite 28 câu, async output rail, đo độ trễ tuần tự P95 với 2 warm-up runs).
  + Tổng hợp các file artifacts JSON và hoàn thiện 3 báo cáo failure_clusters.md, bias_report.md, blueprint.md.
- Bằng chứng: Toàn bộ commit trên git repo DucDai1704/K4-Track-3A-Day24-DoDucDai-2A202602725-Eval-Guard, 86 unit tests offline vượt qua 100%, 4 artifacts JSON trong artifacts/.
- Cách kiểm tra: Chạy pytest tests/ (86 passed), chạy python scripts/check_lab.py --with-tests để xác thực toàn bộ hợp đồng và schema.
- Điều đã học:
  + Hiểu sâu cách đánh giá hệ thống RAG đa chiều thông qua 4 chỉ số RAGAS (faithfulness, answer_relevancy, context_precision, context_recall).
  + Nắm vững kỹ thuật đánh giá bằng LLM-as-a-Judge, nhận diện và khử position bias qua kỹ thuật swap-and-average, đo độ tương đồng với con người qua Cohen kappa.
  + Xây dựng kiến trúc Guardrail Stack đa tầng (Presidio PII + NeMo Guardrails) cân bằng giữa an toàn thông tin (chặn tấn công, bảo mật PII) và độ trễ (latency budget P95).
- Giới hạn/cách xử lý: NeMo Guardrails và LLM evaluation phụ thuộc vào API và network; thiết kế hệ thống có cơ chế extractive fallback và warm-up để đảm bảo độ tin cậy.
- Nguồn/AI hỗ trợ nếu có: Antigravity IDE coding assistant hỗ trợ pair-programming, tự kiểm chứng lại từng hàm và chạy unit tests độc lập.

## Cập nhật sau khi nộp

Bản nộp đầu tiên đúng hạn trước 23:59 ngày 2026-10-12. Không có cập nhật sau deadline.
