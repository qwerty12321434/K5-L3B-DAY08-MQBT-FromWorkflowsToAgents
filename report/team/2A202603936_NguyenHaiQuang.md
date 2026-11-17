# Báo cáo cá nhân — Nguyễn Hải Quang

## 1. Thông tin thành viên

- Họ tên: Nguyễn Hải Quang
- MSSV: 2A202603936
- Email: quangnh2004@gmail.com
- GitHub: qwerty12321434

## 2. Phần việc phụ trách

Review và tích hợp ba shape do các thành viên triển khai. Công việc gồm kiểm tra contract, cải thiện khả năng bảo trì, sửa các lỗi khi tích hợp, chạy kiểm thử và tổng hợp hồ sơ CP5.

## 3. Đóng góp cụ thể

Các commit dưới đây tách lần lượt phần review A/B/C, kiểm thử và hồ sơ CP5.

| Thứ tự | Commit | Thay đổi của mình |
| ---: | --- | --- |
| 1 | `4e81a7f` · `docs(workflow): explain tracing and refund invariants` | Review Shape A từ commit `7355ff7`. Bổ sung comment về dữ liệu được ghi vào trace, quyền tra đơn, quote đầy đủ và lý do lưu trace trước khi gọi host. [Workflow](../../src/lab8/shapes/workflow.py) |
| 2 | `e8854c4` · `refactor(single): integrate bounded read-tool selection` | Review Shape B từ `4b7f6d2` và đối chiếu với bản local. Tích hợp hai đường đọc do MockLLM chọn trong vòng ReAct giới hạn 8 bước. Hai đường dùng chung bộ tính quote và tránh đọc policy lặp. [Single](../../src/lab8/shapes/single.py) |
| 3 | `e0f56b0` · `fix(orchestrator): preserve crash traces and reuse policy reads` | Review Shape C từ `ef07efa`. Thêm lưu trace trước khi gọi host trong ca mô phỏng crash và bỏ lượt `lookup_policy` riêng vì `compute_compensation` đã đọc policy hiện hành. Thêm comment về dữ liệu trong handoff và thứ tự tool. [Orchestrator](../../src/lab8/shapes/orchestrator.py) |
| 4 | `f46f236` · `test(lab8): verify shape safety and record evaluation results` | Thêm test buộc Single đi qua từng đường đọc và kiểm trace ở ba điểm crash. Mở rộng test crash của Orchestrator để kiểm file trace trước khi resume. Chạy toàn bộ suite, benchmark 4 shape, crash suite và ablation R2. [Test Single](../../src/tests/test_single_react_contract.py) · [Test Orchestrator](../../src/tests/test_orchestrator.py) · [Script ablation](../../src/scripts/ablate_swarm_ticket.py) |
| 5 | `docs(submission): finalize CP5 decision and team reports` | Tổng hợp số liệu, giải thích lựa chọn Workflow, ghi phương pháp và kết quả ablation, cập nhật bảng đóng góp và báo cáo cá nhân. [Báo cáo nhóm](group_report.md) · [ADR](decision_adr.md) · [Research card](research_card.md) |

## 4. Bằng chứng

- Toàn bộ **313 test** pass, gồm 65 test mặc định và 248 test student. Các test mình bổ sung cho Single là `test_single_model_choice_preserves_the_quote` và `test_single_writes_trace_before_crash`. Với Orchestrator, mình bổ sung kiểm tra trace vào test có sẵn `test_crash_propagates_and_resume_commits_once`.
- Benchmark public theo cùng 14 claim và 5 seed. Workflow, Single và Orchestrator đều **70/70**. Bản Single tích hợp có chi phí mô phỏng **28,41 VND/lần thành công**, so với **29,56 VND** của bản thành viên và **32,38 VND** của bản local ban đầu. Orchestrator giảm từ **47,45 xuống 38,16 VND** sau khi bỏ lượt đọc policy trùng. Workflow là bản được chọn với **27,86 VND/lần thành công**, run ID mẫu `R-workflow-C-001-s11`. [Benchmark](../../src/reports/benchmark.json)
- Crash suite đạt **20/20**. Ablation R2 sửa đúng ca Swarm T02 seed 55, đưa kết quả từ **69/70 lên 70/70**. [Crash report](../../src/reports/crash.json) · [Ablation](../../src/reports/ablation_swarm_ticket.json)

## 5. Công cụ AI đã dùng

Mình dùng ChatGPT/Codex để hỗ trợ rà code, giải thích contract, bổ sung comment, viết test, so sánh bản Single và tổng hợp báo cáo. Kết quả được đối chiếu với Git diff, test suite và các file JSON do công cụ của lab sinh. Mình sẽ rà lại diff, các số liệu và nội dung báo cáo trước khi nộp.

## 6. Tự đánh giá

Bản tích hợp đạt toàn bộ kiểm tra public. Khi so sánh hai bản Single, điểm cần giữ là khả năng cho MockLLM chọn đường đọc trong giới hạn tool, đồng thời dùng dữ liệu đã đọc và giữ logic tính tiền ở Python. Trường hợp Orchestrator cho thấy cần kiểm file trace ngay tại điểm crash để xác nhận grader có dữ liệu quét. Kết luận chọn Workflow dựa trên benchmark public và sẽ được đối chiếu thêm khi có kết quả hidden.
