# Team

## Thông tin nhóm

- Tên nhóm: MQBT
- Lớp: L3B
- Repo nhóm: [K5-L3B-DAY08-MQBT-FromWorkflowsToAgents](https://github.com/qwerty12321434/K5-L3B-DAY08-MQBT-FromWorkflowsToAgents)
- Nhánh nộp: `main`
- Commit chốt: `f46f236`

## Thành viên

| STT | Họ tên | MSSV | Email | GitHub | Vai trò |
| --: | ------ | ---- | ----- | ------ | ------- |
| 1 | Trần Quang Minh | 2A202603929 | | | Orchestrator / Shape C |
| 2 | Nguyễn Hải Quang | 2A202603936 | quangnh2004@gmail.com | qwerty12321434 | Review A/B/C, tích hợp, kiểm thử và báo cáo |
| 3 | Nguyễn Thế Bảo | 2A202603825 | nguyenthebao2506@gmail.com | NTheBao | Workflow / Shape A |
| 4 | Nguyễn Minh Thành | 2A202603884 | thanhhy98@gmail.com | thanhhy98 | Single / Shape B |

Gợi ý chia vai:

- Nhóm 6 người: integration; workflow; single agent; orchestrator; ledger/policy/crash; eval/report.
- Nhóm 4 người: workflow + integration; single; orchestrator + handoff; ledger + evaluation.

## Bảng đóng góp

`python -m lab8.check_submission` đọc bảng này bằng máy: giữ nguyên tiêu đề `## Bảng đóng góp` và các cột. Mỗi người một dòng: commit có nội dung và ít nhất một test đã thêm hoặc sửa.

| Người | Vai | Commit | Test đã thêm hoặc sửa |
| --- | --- | --- | --- |
| Trần Quang Minh (tác giả Git: `MinhTran`) | Orchestrator / Shape C | `ef07efa` | `test_handoff_and_trace_isolate_untrusted_context`, `test_crash_propagates_and_resume_commits_once` |
| Nguyễn Hải Quang | Review A/B/C, tích hợp, kiểm thử và báo cáo | `4e81a7f`, `e8854c4`, `e0f56b0`, `f46f236` | Thêm `test_single_model_choice_preserves_the_quote`, `test_single_writes_trace_before_crash`. Bổ sung kiểm tra trace trong `test_crash_propagates_and_resume_commits_once`. |
| Nguyễn Thế Bảo (tác giả Git: `Bảo Nguyễn Thế`) | Workflow / Shape A | `7355ff7` | Chưa xác nhận test do thành viên thêm hoặc sửa. |
| Nguyễn Minh Thành | Single / Shape B | `4b7f6d2` | Báo cáo cá nhân ghi đã chạy ma trận student. Chưa có test do thành viên thêm hoặc sửa trong commit này. |

## Cá nhân

- [Trần Quang Minh](report/team/2A202603929_TranQuangMinh.md)
- [Nguyễn Hải Quang](report/team/2A202603936_NguyenHaiQuang.md)
- [Nguyễn Thế Bảo](report/team/2A202603825_NguyenTheBao.md)
- [Nguyễn Minh Thành](report/team/2A202603884_NguyenMinhThanh.md)
