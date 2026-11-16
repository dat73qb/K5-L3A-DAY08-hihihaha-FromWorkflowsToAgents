# Báo cáo cá nhân — 2A202603407_NguyenQuangVu

## 1. Thông tin thành viên

- Họ tên: Nguyễn Quang Vũ
- MSSV: 2A202603407
- Email: 
- GitHub: 

## 2. Phần việc phụ trách

- **Phần việc 1 (Workflow Architecture & Deterministic Engine):**
  - Chịu trách nhiệm thiết kế và cài đặt Shape A (`src/lab8/shapes/workflow.py`).
  - Xây dựng quy trình tuần tự xác định, tách biệt ranh giới logic tính tiền của host và việc ghi log trace.
  - Viết bộ unit test chuyên sâu `src/tests/test_workflow_custom.py`.

## 3. Đóng góp cụ thể

- **Cài đặt Shape A Workflow:**
  - Hoàn thiện toàn bộ logic của Shape A theo hướng dẫn chuẩn: mở kết nối store, đọc claim view qua allowlist `get_claim_view`, xử lý trường hợp thiếu mã đơn (`needs_info`), kiểm tra quyền sở hữu đơn hàng (`escalated`).
  - Đảm bảo tiền hoàn được tính toán bởi hàm nghiệp vụ `compute_compensation`, hoàn toàn không phụ thuộc vào chuỗi văn bản tự do trong ticket của khách hàng.
  - Tích hợp cổng host `durability.issue_refund` với quote đầy đủ, xử lý trơn tru các phản hồi `committed`, `idempotent_replay`, `amount_conflict`, và `policy_denied`.
- **Đảm bảo cơ chế Crash Injection & Trace Contract:**
  - Đặt lệnh gọi `issue_refund` trong khối xử lý ngoại lệ `InjectedCrash`: tự động ghi trace và flush ra đĩa trước khi re-raise ngoại lệ lên grader (vượt qua test T12).
  - Khởi tạo trace hợp lệ theo chuẩn Trace Contract v1, cam kết không ghi bất kỳ secret (`CANARY_SECRET_*`) hay PII khách hàng vào log.
- **Xây dựng bộ kiểm thử Unit Test:**
  - Viết file `tests/test_workflow_custom.py` bao phủ 5 kịch bản kiểm thử: unknown claim, missing order_id, out of scope, zero entitlement quote denied, và crash propagation.

## 4. Bằng chứng

- **Commit kỹ thuật:**(`feat(workflow): implement Shape A deterministic workflow and custom unit tests`).
- **File code:** `src/lab8/shapes/workflow.py`, `src/tests/test_workflow_custom.py`.
- **Kết quả kiểm thử:**
  - Toàn bộ 62/62 bài kiểm thử student của shape workflow đều xanh (`pytest -q -m student -k workflow`).
  - 5/5 custom unit tests trong `tests/test_workflow_custom.py` pass 100%.

## 5. Công cụ AI đã dùng

- **Công cụ:** Google DeepMind Antigravity Agent.
- **Mục đích:** Hỗ trợ phân tích mã nguồn starter, rà soát các nhánh điều kiện biên của hợp đồng Output Contract v1 và gợi ý cấu trúc các ca kiểm thử unit test.
- **Cách tự kiểm tra kết quả:** Trực tiếp thực thi các lệnh kiểm thử tự động của pytest (`python -m pytest tests/test_workflow_custom.py`), đọc trực tiếp nội dung các file JSON trong `src/traces/` để xác nhận không có dữ liệu nhạy cảm hay canaries bị rò rỉ.

## 6. Tự đánh giá

- Hoàn thành xuất sắc nhiệm vụ được phân công, đảm bảo đúng tiến độ và chất lượng cao nhất.
- Shape A đạt kết quả tuyệt đối: Pass rate 1.000, Pass^5 1.000, Variance 0.000000, chi phí thấp nhất hệ thống (27.86 VND).
- Điểm tự đánh giá: 10/10.
