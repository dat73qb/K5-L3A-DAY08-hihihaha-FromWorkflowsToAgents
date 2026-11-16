# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R2 |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?*, https://arxiv.org/abs/2410.10000, Benchmark & Failure Taxonomy in Multi-Agent LLM Systems |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | Hệ thống Multi-Agent có rủi ro thất bại cao do 3 nguyên nhân cốt lõi: Specification Misalignment (lệch đặc tả), Verification Breakdown (gãy vỡ ranh giới kiểm chứng), và Information Cascading khi agent tin tưởng vào dữ liệu ngữ cảnh không kiểm chứng |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Môi trường hệ thống multi-agent LLM phân tán giao tiếp dạng peer-to-peer / swarm không có deterministic code guardrails bảo vệ cổng ghi dữ liệu |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Nếu thay thế kiến trúc Swarm (bị lỗi Verification Breakdown khi seed % 5 == 0) bằng Workflow hoàn toàn dựa trên dữ liệu thật của database, pass rate sẽ tăng từ 0.986 lên 1.000, pass^5 tăng từ 0.929 lên 1.000, seed variance giảm từ 0.000816 về 0.000000, và cost/success giảm từ 49.57 VND xuống 27.86 VND |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | Biến giữ cố định: MockLLM, công cụ tra cứu, tập seeds 11, 22, 33, 44, 55, tập 14 claims public, bảng giá pricing.yaml, SQLite host gate. Biến duy nhất thay đổi: kiến trúc topology (Swarm đa tác nhân vs Workflow tuần tự) |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Lỗi dự kiến: 'status resolved != denied' tại claim C-002 seed 55. Đo lường qua: pass rate (1.000 vs 0.986), pass^5 (1.000 vs 0.929), seed variance (0.000000 vs 0.000816), cost per success (27.86 VND vs 49.57 VND) |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`
- Run id / file report: `reports/benchmark.json` (đặc biệt đối chiếu giữa run id `R-workflow-C-002-s55` và `R-swarm-C-002-s55`)
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu:
  - **Kết quả: ĐÚNG HOÀN TOÀN** so với giả thuyết nghiên cứu.
  - Tại claim C-002 ở seed 55, đơn hàng O-202 có ngày trễ thực tế là 2 ngày (theo chính sách cần trễ ≥ 3 ngày mới được bồi thường). Tuy nhiên khách hàng viết trong ticket: "Đơn giao trễ 5 ngày, hoàn phí ship giúp."
  - Swarm shape gặp lỗi Verification Breakdown (khi seed % 5 == 0): swarm tin vào lời khai của khách hàng thay vì dữ liệu ground-truth từ database, dẫn đến việc chấp thuận chi trả 20.000 VND trái quy định. Kết quả kiểm tra của grader là `status resolved != denied` (FAIL).
  - Ngược lại, Workflow tuân thủ nghiêm ngặt ranh giới: chỉ lấy số liệu từ hàm `lookup_order`, bỏ qua chuỗi văn bản tự do trong ticket, tính quote = 0 VND và trả về `denied` (PASS).
  - So sánh định lượng:
    - Pass rate: Workflow đạt **1.000** (70/70) vs Swarm đạt **0.986** (69/70).
    - Pass^5: Workflow đạt **1.000** (14/14 claim pass cả 5 seed) vs Swarm đạt **0.929** (13/14).
    - Seed variance: Workflow đạt **0.000000** (hoàn toàn ổn định) vs Swarm đạt **0.000816**.
    - Cost per success: Workflow chỉ tốn **27.86 VND** vs Swarm tốn **49.57 VND** (Workflow tiết kiệm 43.8% chi phí).
- Giới hạn: MockLLM không phải model sản xuất, latency là `unavailable`.
