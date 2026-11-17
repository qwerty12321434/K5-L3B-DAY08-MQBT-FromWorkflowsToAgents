# Decision record

Điền sau dấu `:` trên cùng dòng, không xóa nhãn. Giảng viên và `python -m lab8.check_submission` (chạy trong `src/`) đọc các dòng này bằng máy. Không đổi tên hay dời file này khỏi `report/team/`.
`Shape chọn` phải là đúng một trong: `workflow`, `single`, `orchestrator`, `swarm`. Đây là shape được chấm cổng hidden.

- Shape chọn: workflow
- Commit hash: f46f236
- Run id dẫn chứng: R-workflow-C-001-s11 (một trial trong src/reports/benchmark.json)
- Pass rate / pass^5 / seed min-max / variance: 100% (70/70) / 100% (14/14) / 100%–100% / 0
- Cost per success (mô phỏng, VND): 27,86 VND/lần thành công
- Crash safety: 20/20 (src/reports/crash.json). Workflow cũng qua T11, T12 và resume đúng ở cả ba điểm crash kiểm bổ sung.
- Secret leak / approval bypass: 0 ca phát hiện trong 70 trial public của Workflow. Ca kiểm riêng với approval của claim khác bị từ chối và có audit `policy_denied`. Điểm hidden chờ giảng viên xác nhận.

## Vì sao shape này

Workflow đạt 70/70 trial và qua cả 5 seed trên từng claim. Single và Orchestrator đạt cùng độ đúng trên 14 claim và 5 seed, với chi phí lần lượt là 28,41 và 38,16 VND/lần thành công. Workflow có chi phí thấp nhất (27,86 VND) và các bước tra đơn, áp policy, tính quote và gọi host đều có thể định trước. Swarm đạt 69/70 và tốn 49,57 VND/lần thành công. [Benchmark public](../../src/reports/benchmark.json)

## Có đáng đổi lấy multi-agent không

Giữ Workflow cho bài toán hiện tại. Xem xét thêm agent nếu bộ claim đại diện mới có các trường hợp cần chọn cách tra cứu theo ngữ cảnh và Workflow xử lý sai lặp lại. Bản thay thế cần cải thiện độ đúng trên cùng bộ claim và seed, qua các cổng an toàn và có mức chi phí, độ dao động nhóm chấp nhận được.

## Trace và bằng chứng

Workflow ghi các event `claim_view`, `lookup_order`, `quote`, `refund_attempt`, `refund_result` và `decision` tùy nhánh. `claim_view` chỉ ghi mã claim, mã đơn và cờ tìm thấy. Kết quả tra đơn dùng view đã lọc theo quyền và allowlist. Ticket thô, `parent_secret`, header, số điện thoại, địa chỉ, ghi chú nội bộ và tên hàng được loại khỏi trace và kết quả. Trace có `run_id` khớp với `RunResult` và được ghi trong lần chạy. File phụ, nếu có, phải nằm cạnh trace với tên bắt đầu bằng `run_id` để scanner đọc được. Log ngoài phạm vi quét không cung cấp bằng chứng kiểm rò cho lần chạy, nên cần đưa log về đúng phạm vi rồi kiểm lại. [Trace contract](../../docs/contracts.md#trace-contract-v1)

## Giới hạn

Benchmark dùng MockLLM, 14 claim giả lập và giá mô phỏng. Latency là `unavailable`. Grader của giảng viên xác định pass rate và các cổng an toàn trên bộ hidden.
