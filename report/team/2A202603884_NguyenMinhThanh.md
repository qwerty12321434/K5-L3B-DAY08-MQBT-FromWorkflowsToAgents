# Báo cáo cá nhân — Nguyễn Minh Thành

## 1. Thông tin thành viên

- Họ tên: Nguyễn Minh Thành
- MSSV: 2A202603884
- Email: thanhhy98@gmail.com.
- GitHub: thanhhy98 (theo [TEAM.md](../../TEAM.md)).

## 2. Phần việc phụ trách

- Shape B (`single`): ReAct giới hạn 8 bước, chỉ gọi tool đọc. Python tính quote và host kiểm điều kiện trước khi ghi refund.

## 3. Đóng góp cụ thể

- Triển khai Shape B trong `src/lab8/shapes/single.py`: bounded ReAct loop (tối đa 8 bước), tra claim/order bằng read tools, kiểm principal qua `lookup_order`, và lấy số tiền bằng `compute_compensation`.
- Quyết định nghiệp vụ và số tiền không lấy từ ticket. Khi quote dương, truyền quote đầy đủ cho `durability.issue_refund`; host kiểm approval, idempotency và trần hoàn tiền theo đơn.
- Chuyển tiếp `crash_at` đến host và để `InjectedCrash` truyền lên caller. Trace không ghi ticket, claim row hoặc order row thô.
- Phần triển khai được tạo với hỗ trợ ChatGPT/Codex; cần tự rà soát và có thể giải thích được các quyết định kỹ thuật trước khi nộp.
- Commit chính: `4b7f6d2` (`feat: implement shape B`).
- Test thêm/sửa: không thêm test mới. Đã chạy ma trận student hiện có; kết quả và phạm vi được ghi ở mục Bằng chứng.

## 4. Bằng chứng

- `python -m compileall -q lab8/shapes/single.py`: pass.
- `python -m lab8.smoke`: pass (`lab8.smoke pass`; manifest 1.1, MockLLM seeded, trace clean).
- Lệnh student matrix do thành viên chạy: `python -m pytest -q -m student -o addopts= -k single`; kết quả `182 passed, 60 failed, 60 deselected`. `-k single` khớp tên test `test_public_single_run_cases`, nên lọc cả bốn giá trị shape. Các traceback được cung cấp đều là `shape='orchestrator'`; 60 lỗi đó thuộc orchestrator stub, không phải Shape B. Phần báo cáo này không coi kết quả tổng là cả ma trận xanh.
- Benchmark của bản đóng góp tại `4b7f6d2`: `src/reports/benchmark.json`, run ID `R-single-C-001-s11`. Bản này đạt 70/70 trial, pass rate `1.0`, pass^5 `1.0`, seed min–max `1.0–1.0`, variance `0.0` và chi phí mô phỏng `29.562857 VND`/lần thành công. Latency là `unavailable`.
- Crash suite: `src/reports/crash.json`, `crash_safety` `20/20`. Đây là crash suite tổng hợp của harness, không có run id và không nên hiểu là báo cáo crash riêng của Shape B.
- Lệnh benchmark: `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`.
- Lệnh crash: `python -m lab8.crash_test --count 20 --out reports/crash.json`.
- Các lỗi Orchestrator ở trên thuộc lần chạy của thành viên trước tích hợp. Kết quả của bản Single kết hợp và benchmark đủ bốn shape được ghi trong [báo cáo nhóm](group_report.md#4-kết-quả).

## 5. Công cụ AI đã dùng

- ChatGPT/Codex: hỗ trợ triển khai Shape B, rà contract, sửa lỗi cú pháp, và diễn giải kết quả pytest. Mã Shape B trong working tree có hỗ trợ AI và điều này được khai báo tại đây.
- Tự kiểm: compileall, smoke, benchmark Shape B, và crash suite đã chạy. Đã xem traceback của pytest: các lỗi được cung cấp có `shape='orchestrator'`, vì `-k single` lọc theo tên test chứ không phải giá trị shape.
- Trước khi nộp, tự rà lại code và bảo đảm có thể giải thích: ticket là dữ liệu không tin cậy; code/tool tính quote, MockLLM không quyết tiền; vì sao host nhận quote đầy đủ; `crash_at` được forward; trace loại ticket và dữ liệu nhạy cảm.

## 6. Tự đánh giá

- Bản Shape B đạt 70/70 trial public và qua compileall/smoke. Tại thời điểm tự kiểm, ma trận còn 60 lỗi thuộc Orchestrator. Đóng góp kỹ thuật hiện truy được tại commit `4b7f6d2`. Mục test cá nhân vẫn cần bằng chứng test do thành viên thêm hoặc sửa.
