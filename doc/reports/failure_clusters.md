# Phân tích lỗi Phase A

Nhóm: Solo - Đỗ Đức Đại

Người phụ trách: Đỗ Đức Đại - MSSV: 2A202602725

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-12T05:48:07.528706+00:00.
Generation mode/fallback và điều kiện chạy: Hệ thống kết hợp generation trực tiếp qua LLM gemini-3.8-flash và cơ chế extractive_fallback khi chạm ngưỡng giới hạn quota 20 lượt/ngày của gói Free Tier. Pipeline retrieval sử dụng BM25 search kết hợp Qdrant dense vector store và cross-encoder reranking.
Models generation/evaluator: Generation: gemini-3.8-flash; Evaluator: BAAI/bge-m3, gemini-3.8-flash.

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.7863 | 0.9500 | 0.9500 |
| answer_relevancy | 0.5442 | 0.4157 | 0.5901 |
| context_precision | 0.7858 | 0.6369 | 0.5982 |
| context_recall | 0.7574 | 0.4524 | 0.3649 |
| avg_score | 0.7184 | 0.6137 | 0.6258 |

## Bottom 10 và nguyên nhân

Bảng dưới đây liệt kê 10 câu hỏi có điểm trung bình thấp nhất từ thực nghiệm:

| Rank | ID | Distribution | Câu hỏi | avg_score | Worst Metric | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|---|
| 1 | 40 | multi_hop | Nhân viên cấp Manager có thời hạn thử việc 90 ngày, đang trong tháng thứ 3, phát hiện vi phạm bảo mật. Họ nên và không nên làm gì theo chính sách? | 0.4099 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |
| 2 | 7 | factual | Nghỉ phép không lương 20 ngày cần ai phê duyệt? | 0.4240 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 3 | 50 | adversarial | Nhân viên Manager có thể dùng VPN cá nhân (như NordVPN) khi WFH để tăng bảo mật thêm không? | 0.4472 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |
| 4 | 8 | factual | Nhân viên được nghỉ bao nhiêu ngày khi cha hoặc mẹ mất? | 0.4577 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 5 | 33 | multi_hop | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm theo v2024 là bao nhiêu? | 0.4935 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |
| 6 | 22 | multi_hop | Nếu cần mua một chiếc laptop 30 triệu cho nhân viên mới, ai phê duyệt và cần gì từ phòng CNTT? | 0.5141 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |
| 7 | 44 | adversarial | Bao lâu phải đổi mật khẩu một lần? | 0.5384 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |
| 8 | 21 | multi_hop | Một nhân viên Senior có 9 năm thâm niên được nghỉ bao nhiêu ngày phép năm và lương trong khoảng nào? | 0.5488 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |
| 9 | 28 | multi_hop | Nhân viên cấp Lead nhận tổng phụ cấp hàng tháng (ăn trưa + điện thoại) là bao nhiêu? | 0.5684 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 10 | 49 | adversarial | Theo chính sách nghỉ phép cũ (v2023), nhân viên được nghỉ bao nhiêu ngày? Còn chính sách nào đang có hiệu lực hiện tại? | 0.5719 | context_recall | Missing relevant chunks | Improve chunking or add BM25 |

### Phân tích nguyên nhân lỗi đại diện

1. **Lỗi Multi-hop Retrieval (Câu ID 40, 33, 21):**
   - *Câu 40*: Câu hỏi đòi hỏi kết hợp đồng thời chính sách thời gian thử việc của cấp quản lý, quyền lợi trong tháng thứ 3 và quy trình xử lý sự cố an toàn thông tin. Pipeline retrieval dạng single-query chỉ truy xuất được văn bản về quy trình thử việc mà bỏ sót chunk tài liệu về chính sách an ninh mạng IT Policy. Hậu quả là `context_recall` chỉ đạt 0.20, kéo điểm tổng thể xuống 0.4099.
   - *Câu 33*: Đòi hỏi liên kết bảng phụ cấp theo chức vụ Manager với bảng tính ngày phép tăng theo thâm niên 12 năm. Đoạn trích chỉ lấy được thông tin ngày phép v2024 nhưng thiếu bảng phụ cấp quản lý.

2. **Lỗi Answer Relevancy & Prompt Generation (Câu ID 7, 8):**
   - *Câu 7*: Câu hỏi hỏi thẩm quyền phê duyệt khi nghỉ phép không lương trên 14 ngày (cụ thể 20 ngày). Hệ thống trích xuất được chunk điều kiện chung nhưng câu trả lời diễn đạt chung chung về thủ tục xin nghỉ mà chưa nêu bật trực diện cấp phê duyệt thẩm quyền (CEO/Director), khiến `answer_relevancy` thấp.

3. **Lỗi Xung đột phiên bản và phủ định (Câu ID 50, 49):**
   - *Câu 50*: Người dùng hỏi có được dùng NordVPN cá nhân để tăng cường bảo mật không. Thực tế VPN v1.3 cấm hoàn toàn VPN bên thứ ba và bắt buộc dùng WireGuard. Khung retrieval bị nhiễu do từ khóa "VPN", dẫn đến thiếu context giải thích cụ thể lệnh cấm.
   - *Câu 49*: Chứa câu hỏi bẫy về phiên bản cũ v2023 đã hết hiệu lực so với v2024 hiện hành.

## Matrix và nhóm yếu nhất

| Worst Metric | Factual (20 câu) | Multi-hop (20 câu) | Adversarial (10 câu) | Tổng số câu |
|---|---|---|---|---|
| faithfulness | 2 | 0 | 0 | 2 |
| context_recall | 2 | 8 | 9 | 19 |
| context_precision | 2 | 1 | 0 | 3 |
| answer_relevancy | 14 | 11 | 1 | 26 |
| Tổng | 20 | 20 | 10 | 50 |

- **Nhóm yếu nhất theo điểm trung bình**: Phân phối `multi_hop` có mean thấp nhất đạt 0.6137 (so với `factual` 0.7184 và `adversarial` 0.6258).
- **Worst metric xuất hiện nhiều nhất**: `answer_relevancy` là metric yếu nhất ở 26/50 câu hỏi; tiếp theo là `context_recall` yếu nhất ở 19/50 câu hỏi.

### Nguyên nhân và Đề xuất cải tiến

1. **Tại sao `multi_hop` là nhóm yếu nhất:**
   - Trong `multi_hop`, thông tin nằm rải rác ở ít nhất 2 chính sách khác nhau (ví dụ thâm niên nhân sự kết hợp khung lương phòng ban, hoặc mua sắm thiết bị kết hợp ngân sách tài chính). Pipeline hiện tại chỉ thực hiện truy xuất đơn tầng (single-step retrieval) bằng dense vector search + BM25, không có cơ chế Sub-Question Query Decomposition hay Multi-Query Generation, nên top-K chunks luôn bị thiếu ít nhất một nửa số thông tin cần thiết.
2. **Kế hoạch sửa đổi:**
   - Triển khai kỹ thuật Query Decomposition để phân tách câu hỏi multi-hop thành 2 câu hỏi đơn lẻ, truy xuất độc lập rồi hợp nhất tập chunks.
   - Bổ sung Reciprocal Rank Fusion (RRF) giữa BM25 và Dense Search để tăng context_recall.
   - Đo lường lại: Chạy `python scripts/run_experiments.py --phase a --live` và so sánh mean của distribution multi_hop, kỳ vọng tăng từ 0.6137 lên trên 0.7500.

## Bẫy adversarial và giới hạn

1. **Phân tích bẫy Adversarial trong tập dữ liệu:**
   - Các câu hỏi adversarial (như câu 41, 44, 49, 50) chứa các bẫy tinh vi như:
     - Xung đột phiên bản: Cố tình hỏi về số ngày phép theo chính sách cũ v2023 (12 ngày) thay vì v2024 (15 ngày).
     - Phủ định ngầm: Hỏi dùng VPN ngoài để tăng bảo mật, lừa mô hình nghĩ rằng điều đó là tốt trong khi quy định công ty nghiêm cấm.
   - Mặc dù điểm faithfulness trong nhóm adversarial đạt 0.9500 (do câu trả lời bám sát context được cung cấp), nhưng `context_recall` chỉ đạt 0.3649 vì retrieval không tìm đủ các điều khoản cảnh báo phủ định. Điểm thấp này phản ánh hạn chế của retrieval chứ không đồng nghĩa với việc pipeline đã có khả năng tự nhận biết và vô hiệu hóa bẫy adversarial nếu không có tầng Guardrails (Phase C) bảo vệ phía trước.

2. **Giới hạn thực nghiệm:**
   - Số lượng mẫu đánh giá là 50 câu (20 factual, 20 multi-hop, 10 adversarial).
   - Mô hình thế hệ ngôn ngữ: Sử dụng Google Gemini (gemini-3.8-flash) thông qua API OpenAI compatibility layer. Do hạn chế quota 20 lượt gọi mỗi ngày của gói Free Tier, một số câu hỏi kích hoạt cơ chế extractive_fallback để đảm bảo tính liên tục và không làm gián đoạn pipeline.
   - Evaluator sử dụng bộ embedding BAAI/bge-m3 kết hợp metric tính toán RAGAS.
