# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`.

## 1. Thông tin nhóm

- Tên nhóm: hihihaha
- Thành viên: xem `TEAM.md`
  - Nguyễn Quang Vũ (MSSV: 2A202603407) 
  - Nguyễn Tiến Đạt (MSSV: 2A202603452) - Nhóm trưởng
  - Nguyễn Quang Trung (MSSV: 2A202603389)

## 2. Mục tiêu và cách tiếp cận

Bài toán đặt ra cho LogiX-VN là xây dựng và so sánh thực nghiệm 4 kiến trúc phần mềm xử lý claim bồi hoàn phí vận chuyển:
- **Shape A (Deterministic Workflow):** Quy trình đường ống tuần tự mã hóa cứng.
- **Shape B (Single ReAct Agent):** Một tác nhân lặp suy nghĩ và gọi công cụ đọc (tối đa 8 bước).
- **Shape C (Multi-Agent Orchestrator):** Tác nhân điều phối ủy thác điều tra viên thông qua hợp đồng handoff bảo mật.
- **Shape D (Swarm Reference):** Các tác nhân cộng tác ngang hàng dùng làm đối chứng baseline.

**Cách tiếp cận & Thứ tự triển khai:**
1. **Phân tách ranh giới Code vs Model:** Toàn bộ việc tính toán tiền (`compute_compensation`) và thẩm định/ghi sổ (`durability.issue_refund`) bắt buộc phải do code đảm nhiệm; mô hình ngôn ngữ chỉ đóng vai trò phân tích hoặc soạn thảo văn bản phản hồi.
2. **Quy trình chia 3 phần việc độc lập:**
   - *Phần việc 1 (Workflow):* Xây dựng Shape A xác định hoàn toàn và viết bộ test riêng `test_workflow_custom.py`.
   - *Phần việc 2 (Agents):* Xây dựng Shape B & C với vòng lặp ReAct giới hạn và cơ chế handoff allowlist, viết test `test_agents_custom.py`.
   - *Phần việc 3 (Durability & Benchmark):* Kiểm thử crash recovery 20/20, benchmark 4 shapes × 5 seeds, viết test `test_durability_custom.py` và hoàn thiện toàn bộ báo cáo, ADR, Research Card.

## 3. Giải pháp

### 3.1. Ranh giới Code / Model và Luồng xử lý của 3 Shape

- **Shape A (Workflow):**
  - Đọc claim view an toàn qua `get_claim_view` (loại bỏ `parent_secret`).
  - Kiểm tra điều kiện tiên quyết: nếu thiếu `order_id` trả `needs_info`.
  - Tra cứu đơn hàng qua `lookup_order`: nếu không thấy đơn hoặc không thuộc quyền sở hữu của principal, trả `escalated`.
  - Tính tiền hoàn tự động bằng `compute_compensation(conn, order)`. Nếu `amount_vnd <= 0` trả `denied` (không gọi `issue_refund`).
  - Khi quote > 0, gọi `durability.issue_refund(conn, claim_id, quote, crash_at=crash_at)` với **quote đầy đủ** (không tự trừ khoản đã hoàn trước đó vì host sẽ tự trừ).
  - Bắt ngoại lệ `InjectedCrash` để kịp thời ghi trace và re-raise cho grader.
- **Shape B (Single Agent):**
  - Khởi tạo vòng lặp ReAct `run_react` với trần tối đa 8 bước (`max_steps=8`).
  - Hàm `decide()` đóng vai trò tác nhân suy luận qua MockLLM (`complete(..., purpose="think")`), chỉ được phép chọn các tool đọc (`lookup_order`, `compute_compensation`). Tuyệt đối không cấp tool ghi (`issue_refund`).
  - Agent tuyệt đối không tin lời khai trễ hạn trong ticket của khách hàng mà luôn căn cứ trên `order["delay_days"]` thật.
  - Kết thúc vòng lặp, code của shape trực tiếp nhận kết quả quote và gọi cổng `issue_refund`.
- **Shape C (Orchestrator):**
  - Orchestrator khởi tạo handoff an toàn thông qua `make_handoff` với hợp đồng `handoff.v1`.
  - Danh sách công cụ cấp cho investigator chỉ gồm các tool đọc allowlist: `lookup_order`, `lookup_policy`, `compute_compensation`.
  - Chạy điều tra viên chỉ đọc bằng `run_readonly_investigators`.
  - Orchestrator thu thập dữ kiện từ investigator, kiểm tra tính hợp lệ và trực tiếp thực hiện ghi tiền qua `issue_refund`.

### 3.2. Bảo mật Handoff và Trace Contract v1

- **Handoff Sanitization:** Hàm `make_handoff` và `sanitize` chỉ cho phép các trường thuộc `ALLOWED_FIELDS` (`contract_version`, `request_id`, `claim_id`, `principal_scope`, `task`, `facts`, `allowed_tools`, `max_output_tokens`). Mọi trường nhạy cảm như `parent_secret`, `raw_headers` hay lịch sử chat bị loại bỏ hoàn toàn. Nếu phát hiện chuỗi chứa `CANARY_SECRET_`, hệ thống lập tức ném `HandoffError`.
- **Trace Contract v1:** Toàn bộ trace được ghi vào `traces/{run_id}.json` trong chính lần chạy hiện tại. Trace chỉ ghi nhận các sự kiện: `claim`, `lookup_order`, `quote`, `decision`, `react_step`, `handoff`, `investigator`, `crash`. Tuyệt đối không ghi thông tin cá nhân khách hàng (SĐT, địa chỉ, hàng hóa nhạy cảm) hay secret nội bộ.

## 4. Kết quả

### 4.1. Bảng số liệu Benchmark 4 Shape trên bộ Public (14 claims × 5 seeds)

Trích xuất trực tiếp từ file kết quả do công cụ sinh ra: `src/reports/benchmark.json`:

| Kiến trúc (Shape) | Pass Rate | Pass^5 | Seed Variance | Seed Min | Seed Max | Chi phí / Success (VND) | Mean Steps | P95 Steps | Model Calls | Tool Calls |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **A · Workflow** | **1.000** | **1.000** | **0.000000** | **1.000** | **1.000** | **27.86** | **2.86** | **3.0** | **0.0** | **2.79** |
| **B · Single ReAct** | 1.000 | 1.000 | 0.000000 | 1.000 | 1.000 | 29.08 | 5.64 | 6.0 | 2.79 | 2.79 |
| **C · Orchestrator** | 1.000 | 1.000 | 0.000000 | 1.000 | 1.000 | 47.79 | 3.79 | 4.0 | 1.93 | 4.64 |
| **D · Swarm (Ref)** | 0.986 | 0.929 | 0.000816 | 0.929 | 1.000 | 49.57 | 2.80 | 3.0 | 3.00 | 4.66 |

### 4.2. Độ an toàn Crash Safety & Kiểm định Must-Pass

- **Crash Safety:** Đạt tuyệt đối **20/20** kịch bản crash theo kết quả từ `src/reports/crash.json`. Mọi điểm ngắt trước effect, sau effect trước checkpoint, và sau checkpoint đều đảm bảo tính lũy kế (idempotency key `refund:{claim_id}`), không bao giờ tạo dòng refund thứ hai.
- **Approval Bypass:** `0` (không có trường hợp nào giải ngân khi thiếu approval hoặc approval sai claim).
- **Secret & PII Leak:** `0` (không có token canary nào bị rò rỉ trong trace hay file phụ).

Quyết định kiến trúc chính thức và phân tích đánh đổi: xem `report/team/decision_adr.md`.
Phiếu nghiên cứu R2 và thí nghiệm ablation: xem `report/team/research_card.md`.

## 5. Khó khăn và cách xử lý

1. **Khó khăn về ranh giới Crash Injection:**
   - *Vấn đề:* Ban đầu khi chạy thử nghiệm crash, nếu shape vô tình bắt và nuốt ngoại lệ `InjectedCrash`, bài kiểm thử T12 sẽ thất bại do grader kiểm tra ngoại lệ có được đẩy lên tầng caller hay không.
   - *Cách xử lý:* Bổ sung khối `try ... except InjectedCrash:` trong cả ba shape: khi gặp `InjectedCrash`, shape lập tức gọi `trace.write()` để flush file log trước khi dùng lệnh `raise` đẩy ngoại lệ ra ngoài.
2. **Khó khăn về Trần Hoàn tiền theo Đơn (Order Cap):**
   - *Vấn đề:* Với các đơn đã từng được hoàn phí trước đó (như claim C-013 và C-014), nếu shape tự ý trừ số tiền đã hoàn trước khi gọi `issue_refund`, host sẽ trừ thêm một lần nữa dẫn đến sai số tiền (T14 fail).
   - *Cách xử lý:* Luôn truyền **quote đầy đủ** (full entitlement) vào `issue_refund`. Cổng host sẽ tự động tính toán phần chênh lệch (`difference`) dựa trên lịch sử sổ cái `refund_ledger`.

## 6. Tổng hợp đóng góp

Chi tiết đóng góp và commit kỹ thuật được tổng hợp trong `TEAM.md` và các báo cáo cá nhân:
- **Nguyễn Quang Vũ (2A202603407):** Phụ trách Phần việc 1 — Shape A Workflow và bộ unit test `tests/test_workflow_custom.py` (Commit ).
- **Nguyễn Tiến Đạt (2A202603452):** Phụ trách Phần việc 2 — Shape B Single Agent, Shape C Orchestrator và bộ unit test `tests/test_agents_custom.py` (Commit ).
- **Nguyễn Quang Trung (2A202603389):** Phụ trách Phần việc 3 — Độ bền vững, Crash Test, Benchmark, `tests/test_durability_custom.py`, ADR và Research Card (Commit ).

## 7. Hạn chế và hướng cải thiện

1. **Giới hạn môi trường MockLLM:** Hệ thống sử dụng MockLLM giả lập qua mã băm SHA-256 thay vì mô hình LLM sản xuất thực tế. Trong môi trường thực, LLM có thể bị trôi dạt ngữ cảnh (drift), gặp ảo giác (hallucination) hoặc sinh định dạng tool call sai.
2. **Đo lường Latency:** Do môi trường lab chạy offline không kết nối mạng, số liệu độ trễ hiển thị là `unavailable`. Cần đánh giá thêm độ trễ wall-clock khi tích hợp API thực tế.
3. **Mở rộng tương lai:** Khi các quy tắc kinh doanh trở nên phức tạp và phi cấu trúc (ví dụ: cần đọc hình ảnh kiện hàng hư hỏng đa góc độ, trích xuất hóa đơn VAT scan), mô hình Orchestrator với các Investigator chuyên biệt sẽ phát huy giá trị vượt trội so với Workflow mã hóa cứng.

## 8. Tài liệu tham khảo

1. Google Research (2026), *Towards a Science of Scaling Agent Systems*.
2. NeurIPS (2025), *Why Do Multi-Agent LLM Systems Fail?* (MAST taxonomy).
3. AgentArch (AACL 2026), *Topology and Coordination Trade-offs in Multi-Agent Architectures*.
4. EMNLP (2026), *Rethinking Evaluation of Multi-Agent System Efficiency*.
