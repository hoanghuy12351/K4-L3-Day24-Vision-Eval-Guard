# Nhóm thực hiện K4 Track 3A

Tên nhóm: Vision

Repo chung: https://github.com/hoanghuy12351/K4-L3-Day24-Vision-Eval-Guard

Trưởng nhóm/người nộp: Võ Huy Hoàng - 2A202602548.

Ngày lab/deadline áp dụng: 23:59 ngày 12/10/2026 — Asia/Ho_Chi_Minh (UTC+07:00).

Commit main dùng làm mốc tổng hợp trước khi hoàn thiện hồ sơ nộp: `c59ffb7b1c8ddde8688574103363de1e1c64f85a` (merge PR #3).

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
|---|---|---|---|---|
| 1 | Nguyễn Lê Phước Tiến | 2A202602616 | Member 1 — tích hợp pipeline RAG Day 18 (`m1`–`m5`), cấu hình `gpt-5-nano`, chạy pipeline và tạo 50 answers. | Commit `dcb8a6551e9dc72478f933e27127488b973d0a0f`; `src/m1_chunking.py`–`src/m5_enrichment.py`; `tests/test_member1_rag.py`; `artifacts/answers_50q.json`. |
| 2 | Lê Việt Hoàng | 2A202602596 | Phase B — Tasks 5–8: pairwise judge, swap-and-average, Cohen’s κ và phân tích bias. | Commit `dd64756746e4b327713a4e661f050cfd0e0a681d`; merge PR #2 `0a79e369ea73d3d46a6233c64879f022778c925b`; `artifacts/judge_results.json`; `doc/reports/bias_report.md`. |
| 3 | Võ Huy Hoàng | 2A202602548 | Trưởng nhóm/người nộp; Phase A — Tasks 1–4: RAGAS 50 câu, bottom 10 và failure clusters. | Commit `0293373fc4524a79c57b97716828e11b354ad86a`; merge PR #3 `c59ffb7b1c8ddde8688574103363de1e1c64f85a`; `artifacts/ragas_50q.json`; `doc/reports/failure_clusters.md`. |
| 4 | Nguyễn Hoàng Nam | 2A202602485 | Phase C — Tasks 9a–12: PII/input/output rails, adversarial suite và latency; viết blueprint. | Commit `5e63f8c7ab0b249c07acb5d8678c167e30977b08`; merge PR #1 `b5aec6893724573602e91bfe88eafecdef3a1395`; `artifacts/guard_results.json`; `doc/reports/blueprint.md`. |

Người tích hợp/chạy batch: Nguyễn Lê Phước Tiến - 2A202602616; tích hợp RAG và chạy batch 50 answers. Người tổng hợp nhánh `main` và nộp repo chung: Võ Huy Hoàng - 2A202602548, trước 23:59 ngày 12/10/2026.

## Môi trường và chuẩn bị trước lớp

- OS/Python/compiler: macOS 15.7.2; Python 3.11.17 trong `.venv`.
- Virtualenv/dependencies/imports: `python -m pip check` báo không có dependency hỏng; unit tests hiện đạt 92 tests, 3 integration tests được deselect.
- Models/Qdrant/50 answers: Qdrant `v1.9.7`; embedding `BAAI/bge-m3`; reranker `BAAI/bge-reranker-v2-m3`; generation `gpt-5-nano-2025-08-07`. `artifacts/answers_50q.json` có 50 ID duy nhất, phân bố 20 factual/20 multi-hop/10 adversarial, mỗi câu có 3 contexts và `generation_mode=llm`.
- Giới hạn corpus: `BCTC.pdf` và file Nghị định 13 không có text layer nên loader đã bỏ qua và ghi warning trong lượt chạy.
- Trạng thái bằng chứng: source và unit tests đã đạt. Artifacts được tạo trên checkout Windows dùng CRLF; checkout tổng hợp dùng LF. Nhóm đã xác minh toàn bộ hash cũ khớp chính xác biến thể CRLF của các file hiện tại, rồi chuẩn hóa riêng metadata hash sang byte LF hiện tại; không thay đổi input JSON, metrics hoặc kết quả thực nghiệm.

## INDIVIDUAL

### NguyenLePhuocTien-2A202602616

- Phần được giao và phần thực tế đã làm: tích hợp năm module RAG `m1`–`m5`, kết nối chunking/enrichment, hybrid BM25 + Qdrant retrieval, cross-encoder reranking và generation; chuyển generation sang `gpt-5-nano`; tạo bộ 50 answers dùng cho các phase sau.
- Bằng chứng: commit `dcb8a6551e9dc72478f933e27127488b973d0a0f`; `artifacts/answers_50q.json`; năm module trong `src/`; `tests/test_member1_rag.py`.
- Cách kiểm tra: `python scripts/setup_answers.py --live`; artifact có 50 ID, đúng 20/20/10, 50/50 `llm`, không answer/context rỗng; test Member 1 đạt 6/6.
- Điều đã học: cách giữ metadata và `parent_id` xuyên suốt pipeline; phân biệt answer, contexts và ground truth; tách model generation khỏi evaluator/judge/guard; ghi đúng fallback/model provenance.
- Giới hạn/cách xử lý: pin `pyarrow==17.0.0` để tương thích NumPy 1.26; đổi tham số GPT-5 sang `max_completion_tokens`, bỏ `temperature`; ghi nhận hai PDF không có text layer.
- Nguồn/AI hỗ trợ: Codex hỗ trợ rà soát interface, sửa compatibility/dependency, chạy pipeline, validation và Git workflow; kết quả được kiểm tra bằng schema, unit tests và checker.

### LeVietHoang-2A202602596

- Phần được giao và phần thực tế đã làm: triển khai Phase B Tasks 5–8, gồm judge hai chiều cho 5 cặp, quy đổi winner sau swap, chấm nhị phân 10 `model_answer`, Cohen’s κ và báo cáo position/verbosity bias.
- Bằng chứng: commit `dd64756746e4b327713a4e661f050cfd0e0a681d`, merge PR #2 `0a79e369ea73d3d46a6233c64879f022778c925b`; `artifacts/judge_results.json`; `doc/reports/bias_report.md`.
- Cách kiểm tra: unit tests Phase B đạt 8/8; artifact có 5 pairwise results, 10 binary judgments, κ = 1.0, position inconsistency 3/5 và verbosity win rate 1.0 trên 2 decisive cases.
- Điều đã học: pairwise A/B/tie khác nhãn tốt/xấu 0/1; cần swap thứ tự để phát hiện position bias; κ phải tính cả chance agreement.
- Giới hạn/cách xử lý: chỉ có 5 cặp và 10 nhãn; position bias cao nên không dùng một lượt judge làm kết luận; các tie cần human review.
- Nguồn/AI hỗ trợ: sử dụng AI hỗ trợ triển khai/rà soát; kết quả được đối chiếu với fixtures, tests và artifact live. Artifact hiện stale theo source hash của checkout mới và chưa chạy lại theo yêu cầu nhóm.

### VoHuyHoang-2A202602548

- Phần được giao và phần thực tế đã làm: Phase A Tasks 1–4 trong `src/phase_a_ragas.py`; điều chỉnh adapter M4; chạy RAGAS trên 50 answers; phân tích bottom 10, matrix và nhóm yếu.
- Bằng chứng: commit `0293373fc4524a79c57b97716828e11b354ad86a`, merge PR #3 `c59ffb7b1c8ddde8688574103363de1e1c64f85a`; `artifacts/ragas_50q.json`; `doc/reports/failure_clusters.md`.
- Cách kiểm tra: unit tests Phase A đạt 6/6; artifact có 50 kết quả, 10 bottom rows và metadata evaluator. Nhóm `multi_hop` yếu nhất với mean 0.5424; `faithfulness` là worst metric của 23/50 câu.
- Điều đã học: RAGAS là tín hiệu chẩn đoán; phải đọc answer/contexts/reference trước khi kết luận lỗi retrieval hay generation; nhóm yếu được chọn theo mean thay vì số lượng.
- Giới hạn/cách xử lý: nhiều answer từ chối “Không tìm thấy”; một số điểm tự động không phản ánh rõ chứng cứ hiển thị, nên báo cáo giữ phân tích thủ công và không sửa reference để tăng điểm.
- Nguồn/AI hỗ trợ: Codex hỗ trợ triển khai, xử lý adapter và rà soát báo cáo; artifact được tạo bằng batch live. Artifact hiện stale theo source hash của checkout mới và chưa chạy lại theo yêu cầu nhóm.

### NguyenHoangNam-2A202602485

- Phần được giao và phần thực tế đã làm: Phase C Tasks 9a–12: PII scan/anonymization, NeMo input/output rails, chạy attacks + clean controls và đo latency input layers; hoàn thiện blueprint CI/CD.
- Bằng chứng: commit `5e63f8c7ab0b249c07acb5d8678c167e30977b08`, merge PR #1 `b5aec6893724573602e91bfe88eafecdef3a1395`; `artifacts/guard_results.json`; `doc/reports/blueprint.md`.
- Cách kiểm tra: unit tests Phase C đạt 12/12; artifact ghi 19/20 attacks, 8/8 clean, 4/4 output cases và 20 latency samples sau 2 warm-ups.
- Điều đã học: cần clean controls để phát hiện always-block; output rail phải kiểm tra answer có sẵn; latency Task 12 chỉ đại diện Presidio + NeMo input tuần tự.
- Giới hạn/cách xử lý: input guard P95 = 1749.34 ms vượt target 500 ms, bottleneck ở NeMo cloud call; blueprint đề xuất SLM local và theo dõi false positives.
- Nguồn/AI hỗ trợ: sử dụng AI hỗ trợ triển khai/rà soát Phase C và báo cáo; kết quả được kiểm tra bằng unit tests và artifact. Artifact hiện stale theo source hash của checkout mới và chưa chạy lại theo yêu cầu nhóm.

## Cập nhật sau khi nộp

Chưa nộp bản cuối. Nếu có sửa sau deadline, nhóm phải ghi commit, thời điểm, lý do và xác nhận của coach tại đây.
