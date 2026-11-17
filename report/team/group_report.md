# Báo cáo nhóm — Lab 8 LogiX-VN

Tối đa 5 trang ngoài phụ lục. Số liệu lấy từ file trong `src/reports/`, không chép tay từ terminal.

## 1. Thông tin nhóm

- Tên nhóm: MQBT
- Lớp: L3B
- Thành viên: Trần Quang Minh, Nguyễn Hải Quang, Nguyễn Thế Bảo và Nguyễn Minh Thành. Xem [TEAM.md](../../TEAM.md).
- Commit chốt (trùng `report/team/decision_adr.md`): `f46f236`

## 2. Mục tiêu và cách tiếp cận

Nhóm triển khai quy trình xử lý claim của LogiX-VN, từ xác minh đơn và quyền sở hữu đến áp dụng chính sách, tính số tiền đề xuất và ghi refund qua host. Mục tiêu là chọn mức tự chủ thấp nhất đáp ứng độ đúng và an toàn cần thiết. Nhóm triển khai Workflow, Single và Orchestrator trên cùng contract, rồi chạy cả ba cùng Swarm tham chiếu trên chung dữ liệu, seed và giá mô phỏng. Phân công và các commit được đối chiếu trong [TEAM.md](../../TEAM.md).

## 3. Giải pháp

- **Workflow** tra claim và đơn theo quyền, tính quote từ dữ liệu có cấu trúc và policy hiện hành theo các bước Python cố định.
- **Single** dùng vòng ReAct giới hạn 8 bước với dispatcher chỉ đọc. Sau khi tra đơn, mã phản hồi của MockLLM chọn giữa gọi tool tính bồi thường và đọc policy rồi dùng cùng bộ tính quote trong Python.
- **Orchestrator** tạo handoff theo allowlist và dùng investigator chỉ đọc để tra đơn, tính quote. Handoff dùng mã claim, nhiệm vụ và các tool được phép. Nội dung ticket và parent context được loại trước khi giao việc.

Các shape chuyển quote đầy đủ cho `durability.issue_refund` để host kiểm approval, trừ phần đã hoàn theo đơn và chống ghi trùng. Trace ghi các bước tra cứu đã lọc, quote, quyết định và kết quả gọi host. Lần mô phỏng crash có trace để quét và khi resume vẫn giữ một dòng refund cho mỗi claim. [Kiến trúc](../../docs/architecture.md) · [Contract](../../docs/contracts.md)

## 4. Kết quả

Benchmark public chạy 4 shape trên cùng 14 claim với 5 seed `11, 22, 33, 44, 55`. Mỗi shape có 70 trial. Số bước được tính theo số event trong trace. Chi phí là VND mô phỏng, latency là `unavailable`. [Dữ liệu benchmark](../../src/reports/benchmark.json)

| Shape | Trial pass | Pass rate | Pass^5 | Seed min–max | Seed variance | VND/lần thành công | Steps mean / p95 |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Workflow | 70/70 | 100% | 100% | 100%–100% | 0 | 27,86 | 5,07 / 6 |
| Single | 70/70 | 100% | 100% | 100%–100% | 0 | 28,41 | 4,93 / 6 |
| Orchestrator | 70/70 | 100% | 100% | 100%–100% | 0 | 38,16 | 4,36 / 5 |
| Swarm | 69/70 | 98,57% | 92,86% | 92,86%–100% | 0,000816 | 49,57 | 2,80 / 3 |

Ca lệch duy nhất là Swarm ở `C-002` (T02), seed `55`. Ticket báo trễ 5 ngày, dữ liệu đơn ghi trễ 2 ngày. Swarm chọn `resolved`, trong khi nhãn đúng là `denied`. Nhóm chọn Workflow cho ADR vì đạt cùng độ đúng với Single và Orchestrator, có chi phí mô phỏng thấp nhất và dùng các bước cố định.

Crash suite đạt **20/20**. Ma trận student đạt **248/248**, gồm T11 gọi lại và T12 crash/resume cho cả bốn shape. Kiểm tra bổ sung trên `C-001` ở seed `11` xác nhận cả bốn shape có trace ở cả ba điểm crash và sau khi chạy lại chỉ có một refund cùng một audit `refund_committed`. [Crash report](../../src/reports/crash.json)

Số liệu public đã được đối chiếu với code, test và report tại commit `f46f236`.

Ablation R2 trên Swarm chặn việc ghi đè số ngày trễ của đơn bằng số ngày trong ticket. Kết quả tăng từ 69/70 lên 70/70, với một trial thay đổi là T02 seed 55. Kết quả ablation được ghi riêng với benchmark bốn shape. [Phiếu nghiên cứu](research_card.md) · [Quyết định kiến trúc](decision_adr.md)

### Tự kiểm trên bộ hidden

Nhóm chạy Workflow, Single và Orchestrator trên cùng 40 claim từ `student_drop/` với 5 seed `11, 22, 33, 44, 55`. Mỗi shape chạy 200 trial, tổng cộng **600 trial**. Số liệu lấy từ `src/reports/hidden_selfcheck.json` trên máy nhóm.

| Shape | Trial đã chạy | Lỗi chạy | Lỗi trace | Cảnh báo `leak_suspect` | Event trace trung bình |
| --- | ---: | ---: | ---: | ---: | ---: |
| Workflow | 200 | 0 | 0 | 0 | 5,15 |
| Single | 200 | 0 | 0 | 0 | 5,08 |
| Orchestrator | 200 | 0 | 0 | 0 | 4,50 |

Cả 600 trial có `reason: unlabeled`. Bộ tự kiểm thiếu nhãn đáp án nên chưa tính được pass rate, pass^5 hay cost/success. Grader của giảng viên sẽ chấm Workflow theo ADR để xác nhận ngưỡng **75% (ít nhất 150/200 trial đúng)** và các cổng an toàn.

## 5. Khó khăn và cách xử lý

- **PowerShell chặn script kích hoạt venv.** Chạy `Activate.ps1` báo `running scripts is disabled on this system`. Nhóm gọi trực tiếp Python trong venv, ví dụ `.\.venv\Scripts\python.exe -m lab8.smoke`. Kết quả `lab8.smoke pass`.
- **Trace khi crash.** Orchestrator ban đầu chưa có trace trên đĩa khi `InjectedCrash` xảy ra trước `finish()`. Bản tích hợp lưu trace trước khi gọi host trong lần mô phỏng crash. Test kiểm trace ngay sau ba điểm crash rồi xác nhận resume vẫn chỉ ghi một lần.
- **Policy bị đọc lặp.** Orchestrator bỏ lượt đọc policy riêng vì tool tính quote đã đọc policy hiện hành. Single dùng lại policy trong nhánh đọc trực tiếp. Ma trận public vẫn qua sau khi tích hợp.

## 6. Tổng hợp đóng góp

Lịch sử Git ghi nhận Trần Quang Minh triển khai Orchestrator và test handoff/crash, Nguyễn Thế Bảo triển khai Workflow và Nguyễn Minh Thành triển khai Single. Chi tiết commit, bằng chứng test và bốn báo cáo cá nhân được liên kết trong [TEAM.md](../../TEAM.md).

Nguyễn Hải Quang phụ trách review và tích hợp A/B/C. Với Workflow, Quang rà hợp đồng kết quả và bổ sung comment giải thích quote đầy đủ, tra đơn theo quyền và trace khi crash. Với Single, Quang so sánh hai bản trên cùng benchmark rồi tích hợp cách để MockLLM chọn tool đọc mà vẫn dùng chung logic tính quote. Với Orchestrator, Quang bổ sung lưu trace khi mô phỏng crash và bỏ lượt đọc policy trùng. Quang bổ sung test cho hai đường đọc của Single và kiểm trace khi crash cho Single, Orchestrator, sau đó chạy test và benchmark đủ bốn shape, tổng hợp kết quả, lập ADR chọn Workflow và ghi lại ablation R2. [Báo cáo cá nhân Nguyễn Hải Quang](2A202603936_NguyenHaiQuang.md)

## 7. Hạn chế và hướng cải thiện

Benchmark sử dụng 14 claim giả lập, 5 seed và bảng giá mô phỏng. Latency là `unavailable`. Kết luận chọn Workflow cần được xem xét lại nếu tập claim thực tế có nhiều trường hợp phải chọn đường tra cứu linh hoạt hoặc kết quả hidden cho thấy khác biệt về độ đúng. Mọi phương án ghi refund vẫn phải qua các cổng approval, trace và crash.

Phần mở rộng **B3 — Swarm verifier** được trình bày riêng trong [bonus_report.md](bonus_report.md), với code, 12 claim đối kháng, test và số liệu sinh bằng script. Biến thể đạt 70/70 public và 60/60 đối kháng; chi phí cao hơn Workflow nên không đổi lựa chọn ADR. Điều kiện tính bonus trên hidden chờ giảng viên xác nhận.

## 8. Tài liệu tham khảo

- [Luật nghiệp vụ LogiX-VN](../../docs/business_rules.md)
- [Định nghĩa metric của lab](../../docs/metrics.md)
- [Cemri và cộng sự (NeurIPS 2025), Why Do Multi-Agent LLM Systems Fail?](https://openreview.net/pdf/e5dfd95c87e815f4c3773cfb5e9ffc17220fd006.pdf)
