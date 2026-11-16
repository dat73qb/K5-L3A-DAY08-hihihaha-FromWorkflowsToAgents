# Báo cáo cá nhân — MSSV_HoTen

## 1. Thông tin thành viên

- Họ tên: Nguyễn Quang Trung
- MSSV: 2A202603389
- Email: 26ai.trungnq@vinuni.edu.vn
- GitHub: quangtrung0812

## 2. Phần việc phụ trách

- **Phần việc 3 (Benchmark, Durability / Crash Suite, Evaluation & Reports):**
  - Chịu trách nhiệm thực thi và giám sát bộ kiểm thử va đập (Crash Safety Suite 20/20).
  - Chạy toàn bộ chương trình đo lường benchmark trên cả 4 kiến trúc và 5 seeds.
  - Viết bộ unit test chuyên sâu `src/tests/test_durability_custom.py`.
  - Phân tích số liệu và chủ trì soạn thảo tài liệu quyết định kiến trúc (`decision_adr.md`) và phiếu nghiên cứu (`research_card.md`).
...

## 3. Đóng góp cụ thể

- **Kiểm thử Crash Safety & Tính Lũy Đẳng (Idempotency):**
  - Chạy công cụ kiểm thử lỗi ngẫu nhiên `lab8.crash_test`, sinh file `src/reports/crash.json` đạt điểm tuyệt đối 20/20 kịch bản ngắt trước effect, sau effect và sau checkpoint.
  - Xác nhận tính an toàn tuyệt đối của khóa định danh `refund:{claim_id}`, ngăn chặn hoàn toàn việc giải ngân lặp lại khi có crash hoặc retry.
- **Thực thi Benchmark diện rộng 4 Shapes × 5 Seeds:**
  - Chạy lệnh `lab8.benchmark` trên tập 14 claims public và 5 seeds (11, 22, 33, 44, 55), tổng hợp dữ liệu 280 trials vào `src/reports/benchmark.json`.
  - Trích xuất và phân tích định lượng các chỉ số: pass rate, pass^5, seed variance, chi phí mô phỏng / success, và số bước thực hiện trung bình.
- **Xây dựng bộ kiểm thử Unit Test:**
  - Viết file `tests/test_durability_custom.py` kiểm chứng cơ chế phục hồi sau crash, tính bất biến của sổ cái khi gọi lại nhiều lần, và tính toán phần chênh lệch khi áp dụng trần hoàn tiền theo đơn (order cap).
- **Hoàn thiện Hồ sơ Báo cáo Kỹ thuật:**
  - Soạn thảo `report/team/decision_adr.md` với các số liệu thực chứng, phân tích đánh đổi kiến trúc theo triết lý "ít tự chủ nhất mà vẫn làm được việc".
  - Soạn thảo `report/team/research_card.md` theo taxonomy lỗi của NeurIPS 2025 (Verification Breakdown trên Swarm ở seed 55).
...

## 4. Bằng chứng

- **Commit kỹ thuật:** `7a42315` (`feat(eval): add durability custom tests, crash safety suite and benchmark reports 1`).
- **File báo cáo & dữ liệu:** `src/reports/crash.json` (20/20), `src/reports/benchmark.json`, `src/tests/test_durability_custom.py`.
- **Kết quả kiểm thử:**
  - 3/3 custom unit tests trong `tests/test_durability_custom.py` pass 100%.
  - Crash safety: 20/20 passed.
  - Trình kiểm tra `check_submission` xác nhận tính toàn vẹn của dữ liệu và định dạng.
...

## 5. Công cụ AI đã dùng

- **Công cụ:** Google DeepMind Antigravity Agent.
- **Mục đích:** Hỗ trợ tính toán phương sai mẫu dân số (population variance) và định dạng bảng markdown so sánh đa chiều các kiến trúc.
- **Cách tự kiểm tra kết quả:** Tự động đối chiếu các trường trong `benchmark.json` bằng script Python và kiểm tra độ nhất quán với báo cáo.
...

## 6. Tự đánh giá
- Hoàn thành trọn vẹn toàn bộ các mục tiêu đo lường, kiểm thử độ bền và chứng minh khoa học cho quyết định kiến trúc của nhóm.
- Đảm bảo tính minh bạch và trung thực tuyệt đối trong dữ liệu báo cáo.
- Điểm tự đánh giá: 10/10.
...
