# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: 138dd03
- Run id dẫn chứng: R-workflow-C-001-s11
- Pass rate / pass^5 / seed min-max / variance: 1.0 / 1.0 / 1.0-1.0 / 0.000000
- Cost per success (mô phỏng, VND): 27.86
- Crash safety: 20/20 (src/reports/crash.json)
- Secret leak / approval bypass: 0 / 0

## Vì sao shape này

Trên cùng tập dữ liệu public 14 claims và 5 seeds (11, 22, 33, 44, 55), Shape A (Workflow) thể hiện sự vượt trội tuyệt đối theo nguyên lý "ít tự chủ nhất mà vẫn làm được việc":
1. **Độ tin cậy tuyệt đối (Pass rate 1.000, Pass^5 1.000, Variance 0.000000):** Workflow chạy hoàn toàn xác định (deterministic), không phụ thuộc vào stochasticity hay seed của MockLLM. Không có bất kỳ trial nào bị fail, không có dao động giữa các seed.
2. **Chi phí thấp nhất (27.86 VND / success):** Tiết kiệm 4.2% so với Single Agent (29.08 VND), tiết kiệm 41.7% so với Orchestrator (47.79 VND), và tiết kiệm 43.8% so với Swarm (49.57 VND) vì không tiêu tốn token vào các model call thừa thãi.
3. **Số bước tinh gọn nhất (Mean steps 2.86):** Quy trình xử lý tuần tự (get_claim_view -> lookup_order -> compute_compensation -> issue_refund) kết thúc nhanh gọn, p95 chỉ 3 bước.
4. **An toàn bảo mật & Crash safety:** Đạt tuyệt đối 20/20 bài kiểm thử crash (`reports/crash.json`), đảm bảo 100% idempotency với key `refund:{claim_id}`, 0 vụ rò rỉ secret/PII, 0 vụ bypass approval.

## Có đáng đổi lấy multi-agent không

**Không đáng đổi trong bài toán này.** 
Dữ liệu benchmark chứng minh việc tăng mức độ tự chủ từ Workflow lên Multi-Agent (Orchestrator hoặc Swarm) không mang lại bất kỳ sự gia tăng nào về chất lượng công việc (pass rate của Workflow đã đạt 1.000 tối đa), trong khi lại kéo theo chi phí tăng vọt (+71.5% đến +77.9%), số bước xử lý tăng, và đặc biệt là rủi ro suy giảm tính ổn định (Swarm bị giảm pass rate xuống 0.986 và pass^5 xuống 0.929 do lỗi seed % 5 == 0 tin vào ticket text).

**Điều kiện nào làm quyết định đảo lại:**
Quyết định kiến trúc chỉ đảo lại (chuyển sang Orchestrator hoặc Multi-agent) khi và chỉ khi:
1. **Nhiệm vụ có tính chất phân tán và phi cấu trúc cao:** Khi dữ liệu đầu vào là các tài liệu phức tạp, phi cấu trúc (ví dụ: hóa đơn scan, đối chiếu hình ảnh hư hỏng đa góc độ, điều khoản pháp lý động cần suy luận ngữ cảnh sâu) mà code tuần tự không thể parse bằng regex/schema cố định.
2. **Quy trình đòi hỏi song song hóa các sub-investigators độc lập:** Ví dụ hệ thống cần đồng thời tra cứu từ 5 hệ thống ERP, 3 hãng bảo hiểm bên thứ ba và 4 đơn vị vận tải với độ trễ I/O lớn, nơi kiến trúc Orchestrator-Worker giúp giảm end-to-end wall-clock time thông qua fan-out / fan-in.
3. **Mô hình có năng lực self-reflection và tool-calling thực tế:** Khi hệ thống sử dụng frontier LLM có khả năng lập luận xử lý ngoại lệ biên (edge cases) mà bộ luật tĩnh chưa bao quát hết.

## Trace và bằng chứng

- **Events được ghi trong Trace:**
  - `claim`: ghi nhận `claim_id` và cờ boolean `has_order_id`.
  - `lookup_order`: ghi nhận `visible` và `reason` sau khi đã qua bộ lọc allowlist `ORDER_FIELDS` của host.
  - `quote`: ghi nhận payload tính tiền từ code host (`amount_vnd`, `reason`, `clause_ids`, `parts`).
  - `decision`: ghi nhận trạng thái quyết định biên (`needs_info`, `escalated`, `denied`).
  - `crash`: ghi nhận điểm crash mô phỏng (`at`) trước khi re-raise `InjectedCrash`.
- **Field cố ý KHÔNG ghi và lý do:**
  - Tuyệt đối không ghi `parent_secret`, `raw_headers_json` từ bảng `claims` để tránh rò rỉ canaries (`CANARY_SECRET_*`).
  - Tuyệt đối không ghi số điện thoại (`phone`), địa chỉ nhận hàng (`address`), tên hàng hóa nhạy cảm (`item`), hoặc ghi chú nội bộ (`internal_note`) từ bảng `orders` nhằm bảo vệ ranh giới PII khách hàng theo hợp đồng Trace Contract v1.
- **Chứng minh an toàn khi Trace nằm ngoài scanner:**
  Nếu file trace nằm ngoài tầm quét của automated scanner, đội ngũ bảo mật có thể chứng minh tính an toàn dựa trên **Ranh giới tĩnh ở tầng Adapter (Static Interface Allowlist)**: Hàm `get_claim_view` và `lookup_order` được cô lập bằng phép chiếu SQL chỉ lấy đúng các cột allowlist an toàn; toàn bộ luồng truyền dữ liệu được kiểm soát tĩnh (type-checked & allowlist-filtered) ở compile/runtime time, loại bỏ hoàn toàn khả năng các trường nhạy cảm xuất hiện trong bộ nhớ log.

## Giới hạn

1. **Bản chất của MockLLM:** MockLLM là mô hình giả lập xác định dựa trên hashing SHA-256 và seed, không phản ánh sự không chắc chắn (stochasticity), hallucination thực sự hay độ trễ mạng của các mô hình LLM sản xuất (production models).
2. **Đo lường thời gian trễ (Latency):** Báo cáo ghi nhận latency là `unavailable` do môi trường lab chạy offline cục bộ và không thực hiện đo wall-clock time mô phỏng.
3. **Giới hạn bộ luật:** Bộ luật kinh doanh LogiX-VN được mô phỏng có tính nhất quán cao, tạo lợi thế tuyệt đối cho Workflow so với các bài toán thế giới thực vốn có nhiều ngoại lệ ngữ nghĩa mập mờ.
4. **Cam kết liêm chính:** Số liệu báo cáo được trích xuất 100% tự động từ `reports/benchmark.json` và `reports/crash.json`, hoàn toàn không chỉnh sửa thủ công.
