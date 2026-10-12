# Đánh giá LLM Judge Phase B

Nhóm: Vision

Người phụ trách: Lê Việt Hoàng - 2A202602596

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T11:31:22.501355+00:00.
Models generation/judge: gpt-5-nano-2025-08-07 và openai/gpt-4o-mini.

Trạng thái kiểm chứng: số liệu dưới đây được đọc nguyên trạng từ artifact live hiện có. Artifact ban đầu ghi hash trên checkout Windows dùng CRLF; nhóm đã xác minh các hash đó khớp chính xác biến thể CRLF của source hiện tại và chuẩn hóa metadata hash sang byte LF của checkout tổng hợp. Không thay đổi input JSON, judgment hoặc kết quả bias/calibration.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers_50q.json:answer; B từ
fixtures/human_labels_10q.json:model_answer; reference từ fixtures/test_set_50q.json:ground_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Candidate A | Candidate B | Winner Pass 1 | Winner Pass 2 (Quy đổi) | Final Winner | Position Consistent | Điểm Pass 1 (A/B) | Điểm Pass 2 (A/B) | Lý do chính |
|---|---|---|---|---|---|---|---|---|---|
| 1 | 3 ngày làm việc. | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | B | B | B | True | 0.6 / 1.0 | 0.5 / 1.0 | Candidate B cung cấp đầy đủ thông tin về tính chất có lương theo đúng chính sách. |
| 5 | Không tìm thấy. | Cần Giám đốc phòng ban phê duyệt. | B | A | tie | False | 0.0 / 0.5 | 0.8 / 0.2 | Bị ảnh hưởng vị trí: một lượt phạt hallucination của B, lượt kia phạt A vì thiếu thông tin. |
| 12 | 1 tháng lương. | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | A | B | tie | False | 1.0 / 0.8 | 0.7 / 1.0 | Bị ảnh hưởng vị trí: lượt 1 ưu tiên tính súc tích của A, lượt 2 ưu tiên tính đầy đủ của B. |
| 21 | Không tìm thấy. | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | B | B | B | True | 0.0 / 1.0 | 0.0 / 1.0 | Candidate B trả lời chính xác toàn bộ các ý đa bước, trong khi A không tìm thấy thông tin. |
| 41 | Không tìm thấy. | Nhân viên được nghỉ 12 ngày phép năm. | A | B | tie | False | 0.2 / 0.0 | 0.0 / 0.7 | Bị ảnh hưởng vị trí: Candidate B dùng chính sách cũ v2023, LLM phân vân giữa từ chối và thông tin cũ. |

Position bias count/rate: 3 cặp / 60.0% (0.6).
Verbosity win rate: 100.0% (1.0); decisive cases: 2 trường hợp (ID 1 và ID 21).

Phân tích định tính:
- Position bias (60.0%): Trong 5 cặp thử nghiệm, có tới 3 cặp (ID 5, 12, 41) cho kết quả bất đồng giữa hai lượt hoán đổi thứ tự. Điều này cho thấy LLM Judge chịu ảnh hưởng rất mạnh từ vị trí câu trả lời khi hai candidate có chất lượng ngang nhau hoặc một bên bị lỗi chính sách. Kỹ thuật swap-and-average đã phát huy tác dụng tuyệt đối khi quy tất cả 3 trường hợp mâu thuẫn này về `tie`, loại bỏ hoàn toàn các quyết định thiên vị ngẫu nhiên.
- Verbosity bias (100.0% trên các ca decisive): Cả 2 trường hợp phân định thắng thua rõ ràng (ID 1 và ID 21) đều có phần thắng thuộc về câu trả lời dài hơn (Candidate B). Tuy nhiên, phân tích nội dung cho thấy Candidate B chiến thắng không phải do dài dòng, mà do cung cấp thông tin thực sự chính xác và đầy đủ hơn (ở ID 1 có thêm chi tiết "có lương", ở ID 21 có đầy đủ số ngày phép thâm niên và khung lương so với "Không tìm thấy" của Candidate A).

## Calibration đúng model_answer

Helper đánh giá đúng question/model_answer đang có trong fixtures/human_labels_10q.json;
không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference.
Không gửi human_label, human_note hoặc phân bố nhãn human vào prompt.

| Question ID | Model Answer Gốc | Nhãn Judge (0/1) | Nhãn Human (0/1) | Đồng thuận (Agree) | Lý do đánh giá của Judge |
|---|---|---|---|---|---|
| 1 | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn. | 1 | 1 | Yes | Câu trả lời chính xác theo tài liệu tham khảo, nêu rõ số ngày nghỉ và điều kiện không trừ phép năm. |
| 5 | Cần Giám đốc phòng ban phê duyệt. | 0 | 0 | Yes | Câu trả lời sai cấp phê duyệt; theo chính sách cần Tổng Giám đốc phê duyệt cho hạn mức 55 triệu. |
| 12 | Nhân viên được thưởng Tết tối thiểu 1 tháng lương. | 1 | 1 | Yes | Câu trả lời chính xác, nêu đúng điều kiện nhân viên chính thức từ 6 tháng trở lên được tối thiểu 1 tháng lương. |
| 21 | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1 | 1 | Yes | Chính xác theo reference cho câu hỏi multi-hop, tính đúng ngày phép cơ bản + thâm niên và khoảng lương. |
| 23 | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết. | 1 | 1 | Yes | Nêu đúng nghĩa vụ bồi hoàn 100% kinh phí đào tạo khi nghỉ việc trước thời hạn cam kết 1 năm. |
| 29 | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu. | 0 | 0 | Yes | Thiếu Kế toán trưởng đồng phê duyệt và áp dụng sai quy định thời gian tính phạt tạm ứng. |
| 33 | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng. | 1 | 1 | Yes | Tính toán chính xác tổng ngày phép thâm niên Manager và phụ cấp trách nhiệm hàng tháng. |
| 41 | Nhân viên được nghỉ 12 ngày phép năm. | 0 | 0 | Yes | Câu trả lời vi phạm do sử dụng phiên bản chính sách cũ (v2023: 12 ngày) thay vì bản hiện hành (v2024: 15 ngày). |
| 46 | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần. | 1 | 1 | Yes | Chính xác, phản ánh đúng quy định cấm nghỉ phép năm đối với nhân viên trong giai đoạn thử việc. |
| 50 | Được, miễn là đảm bảo kết nối an toàn. | 0 | 0 | Yes | Câu trả lời sai nghiêm trọng; chính sách bảo mật cấm tuyệt đối việc sử dụng VPN thương mại cá nhân khi WFH. |

Cohen’s κ: 1.0 (Đạt mốc bonus >0.6).
Observed agreement: $p_o = 10/10 = 1.0$ (100% đồng thuận).
Chance agreement: $p_e = (6/10 \times 6/10) + (4/10 \times 4/10) = 0.36 + 0.16 = 0.52$.
Phép tính: $\kappa = \frac{p_o - p_e}{1 - p_e} = \frac{1.0 - 0.52}{1.0 - 0.52} = 1.0$.
Nhận xét: Theo thang đo Landis-Koch, hệ số $\kappa = 1.0$ thể hiện mức "Almost Perfect Agreement" (sự đồng thuận hoàn hảo) giữa LLM Judge và nhãn chuyên gia con người trên tập 10 câu hỏi chuẩn hóa.

## Giới hạn và cách dùng judge

- Giới hạn mô hình và cỡ mẫu: Đánh giá được thực hiện với mô hình `openai/gpt-4o-mini` ở `temperature=0`. Dù kết quả calibration đạt $\kappa = 1.0$, cỡ mẫu 10 câu nhị phân và 5 cặp so sánh là tập kiểm thử tối thiểu. Trong các bài toán thực tế có độ biến thiên ngôn ngữ cao, độ tin cậy có thể bị suy giảm.
- Rủi ro Position Bias: Thực nghiệm chứng minh tỷ lệ position bias lên tới 60.0% nếu chỉ chấm một chiều. Do đó, trong hệ thống production, bắt buộc phải triển khai cơ chế swap-and-average hai chiều để triệt tiêu thiên kiến vị trí.
- Hướng dẫn vận hành: Không được xem LLM Judge là chân lý tuyệt đối. Các trường hợp LLM trả về `tie` hoặc có điểm số sai lệch biên độ nhỏ cần được đưa vào hàng đợi kiểm duyệt thủ công (Human-in-the-loop). Đồng thời cần kết hợp các deterministic rule checkers để kiểm tra số liệu cứng (ngày tháng, số tiền, phiên bản tài liệu).
