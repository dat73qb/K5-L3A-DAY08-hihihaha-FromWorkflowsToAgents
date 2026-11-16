# Team

## Thông tin nhóm

- Tên nhóm: hihihaha
- Lớp: L3A
- Repo nhóm: `K5-L3A-DAY08-hihihaha-FromWorkflowsToAgents` (xem [SUBMISSION.md](SUBMISSION.md))
- URL repo, nhánh nộp, commit chốt:
  - URL repo: https://github.com/VinUni-AI20k/K5-L3A-DAY08-hihihaha-FromWorkflowsToAgents
  - Nhánh nộp: 
  - Commit chốt: 

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
|   1 | Nguyễn Quang Vũ | 2A202603407 |  |  | Shape A: Workflow & Deterministic Engine |
|   2 | Nguyễn Quang Trung | 2A202603389 |  |  | Durability, Benchmark & Evaluation |
|   3 | Nguyễn Tiến Đạt | 2A202603452 |  |  | Shape B & C: Single ReAct & Multi-Agent Orchestrator  |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.
- Nhóm 3 người: workflow; agent systems (single + orchestrator); durability, benchmark & report.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Nguyễn Quang Vũ | Workflow & Deterministic Engine |  | tests/test_workflow_custom.py |
| Nguyễn Tiến Đạt | Single ReAct & Multi-Agent Orchestrator |  | tests/test_agents_custom.py |
| Nguyễn Quang Trung | Durability, Benchmark & Evaluation |  | tests/test_durability_custom.py |

## Cá nhân

Mỗi thành viên đồng thời tạo một báo cáo riêng trong `report/` theo dạng:

```text
report/team/MSSV_HoTen.md
```.


## 2A202603407_NguyenQuangVu

### Phần việc phụ trách

- Cài đặt kiến trúc Shape A (Deterministic Workflow).
- Xây dựng luồng xử lý tuần tự, tích hợp an toàn với host cổng `durability.issue_refund`.
- Viết bộ unit test `tests/test_workflow_custom.py`.

### Đóng góp cụ thể

- Commit chính:.
- Xử lý các edge cases của workflow: claim không tồn tại, thiếu `order_id` (trả `needs_info`), đơn không thuộc sở hữu (trả `escalated`), quote 0 trả `denied`, quote > 0 gọi host an toàn.
- Đảm bảo cơ chế crash injection (`crash_at`) đẩy ngoại lệ `InjectedCrash` lên host và flush trace thành công.

### Bằng chứng

- Commit:.
- File test: `tests/test_workflow_custom.py` (5 tests passed).
- Báo cáo chi tiết: `report/team/2A202603407_NguyenQuangVu.md`.
