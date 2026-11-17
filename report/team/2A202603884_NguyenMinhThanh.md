# Báo cáo cá nhân — Nguyễn Minh Thành

## 1. Thông tin thành viên

- Họ tên: Nguyễn Minh Thành
- MSSV: 2A202603884
- Email: thanhhy98@gmail.com.
- GitHub: Chưa cung cấp.

## 2. Phần việc phụ trách

- Shape B (`single`): một ReAct agent giới hạn 8 bước, chỉ gọi read tool; host chịu trách nhiệm tính quote và ghi refund.

## 3. Đóng góp cụ thể

- Triển khai Shape B trong `src/lab8/shapes/single.py`: bounded ReAct loop (tối đa 8 bước), tra claim/order bằng read tools, kiểm principal qua `lookup_order`, và lấy số tiền bằng `compute_compensation`.
- Quyết định nghiệp vụ và số tiền không lấy từ ticket. Khi quote dương, truyền quote đầy đủ cho `durability.issue_refund`; host kiểm approval, idempotency và trần hoàn tiền theo đơn.
- Chuyển tiếp `crash_at` đến host và để `InjectedCrash` truyền lên caller. Trace không ghi ticket, claim row hoặc order row thô.
- Phần triển khai được tạo với hỗ trợ ChatGPT/Codex; cần tự rà soát và có thể giải thích được các quyết định kỹ thuật trước khi nộp.
- Commit chính: chưa có commit chứa thay đổi này; `7355ff7` là HEAD hiện tại và không chứa thay đổi Shape B đang nằm trong working tree.
- Test thêm/sửa: không thêm test mới. Đã chạy ma trận student hiện có; kết quả và phạm vi được ghi ở mục Bằng chứng.

## 4. Bằng chứng

- `python -m compileall -q lab8/shapes/single.py`: pass.
- `python -m lab8.smoke`: pass (`lab8.smoke pass`; manifest 1.1, MockLLM seeded, trace clean).
- Lệnh student matrix do thành viên chạy: `python -m pytest -q -m student -o addopts= -k single`; kết quả `182 passed, 60 failed, 60 deselected`. `-k single` khớp tên test `test_public_single_run_cases`, nên lọc cả bốn giá trị shape. Các traceback được cung cấp đều là `shape='orchestrator'`; 60 lỗi đó thuộc orchestrator stub, không phải Shape B. Phần báo cáo này không coi kết quả tổng là cả ma trận xanh.
- Benchmark: `src/reports/benchmark.json`, run id mẫu Shape B `R-single-C-001-s11`. Shape B đạt 70/70 trial, pass rate `1.0`, pass^5 `1.0`, seed min–max `1.0–1.0`, variance `0.0`, simulated cost/success `29.562857 VND`; latency unavailable. Đây là chi phí mô phỏng, không phải hóa đơn API.
- Crash suite: `src/reports/crash.json`, `crash_safety` `20/20`. Đây là crash suite tổng hợp của harness, không có run id và không nên hiểu là báo cáo crash riêng của Shape B.
- Lệnh benchmark: `python -m lab8.benchmark --shapes workflow,single,orchestrator,swarm --cases data/public_claims.jsonl --seeds 11,22,33,44,55 --out reports/benchmark.json`.
- Lệnh crash: `python -m lab8.crash_test --count 20 --out reports/crash.json`.
- Giới hạn bằng chứng: benchmark cũng ghi shape orchestrator còn stub (0/70), nên chưa phải benchmark nộp hợp lệ cho toàn nhóm. Ma trận hiện có 60 lỗi orchestrator và chưa xanh hoàn toàn.

## 5. Công cụ AI đã dùng

- ChatGPT/Codex: hỗ trợ triển khai Shape B, rà contract, sửa lỗi cú pháp, và diễn giải kết quả pytest. Mã Shape B trong working tree có hỗ trợ AI và điều này được khai báo tại đây.
- Tự kiểm: compileall, smoke, benchmark Shape B, và crash suite đã chạy. Đã xem traceback của pytest: các lỗi được cung cấp có `shape='orchestrator'`, vì `-k single` lọc theo tên test chứ không phải giá trị shape.
- Trước khi nộp, tự rà lại code và bảo đảm có thể giải thích: ticket là dữ liệu không tin cậy; code/tool tính quote, MockLLM không quyết tiền; vì sao host nhận quote đầy đủ; `crash_at` được forward; trace loại ticket và dữ liệu nhạy cảm.

## 6. Tự đánh giá

- Shape B đạt kết quả benchmark public 70/70 và các kiểm tra compileall/smoke hiện có đều pass. Ma trận student tổng thể vẫn có 60 lỗi thuộc orchestrator; chưa có commit kỹ thuật hoặc test mới được ghi nhận cho cá nhân. Cần rà soát code, hoàn tất thông tin liên hệ, và tạo commit thật trước khi coi báo cáo này là bản nộp cuối.
