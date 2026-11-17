# Research card

File này nằm cố định ở `report/team/research_card.md`; `python -m lab8.check_submission` đọc cột "Nhóm điền" bằng máy.

Phần này nối tiếp phần mini research ở buổi lý thuyết (slide 62–63). Chọn **một** câu R1–R4, điền 6 dòng, rồi biến nó thành **một ablation** chạy được trong lab: giữ nguyên model (MockLLM), tool, seed, corpus và giá, chỉ thay đổi một biến.

| Câu | Nguồn (thư viện tham khảo, slide 68–69) | Câu hỏi gợi ý cho lab |
| --- | --- | --- |
| R1 · Scaling | Google Research 2026, *Towards a Science of Scaling Agent Systems* | Task claims chủ yếu tuần tự hay song song được? Dự đoán thêm investigator sẽ làm pass rate và cost/success thay đổi thế nào. |
| R2 · Failure taxonomy | MAST, NeurIPS 2025, *Why Do Multi-Agent LLM Systems Fail?* | Các trial fail của swarm (seed 55) thuộc nhóm nào: lỗi giao việc, lệch mục tiêu, hay lỗi kiểm chứng? |
| R3 · Architecture choices | AgentArch, AACL 2026 | Nếu đổi topology mà giữ nguyên mọi thứ khác, biến nào phải cố định để kết luận còn đúng? |
| R4 · Evaluation confounders | EMNLP 2026, *Rethinking Evaluation of MAS Efficiency* | Làm sao chứng minh chênh lệch giữa A/B/C/D là do topology, không phải do prompt, tool hay seed? |

## Phiếu của nhóm

Điền vào cột cuối. Không xóa hai cột đầu.

| Field | Hướng dẫn | Nhóm điền |
| --- | --- | --- |
| CÂU | R1 / R2 / R3 / R4 | R2 · Failure taxonomy |
| SOURCE | Paper/blog, năm, URL, loại bằng chứng (benchmark, production report, blog kinh nghiệm) | Cemri và cộng sự, [Why Do Multi-Agent LLM Systems Fail?](https://openreview.net/pdf/e5dfd95c87e815f4c3773cfb5e9ffc17220fd006.pdf), NeurIPS 2025, Datasets and Benchmarks Track. Bằng chứng từ tập trace có chú giải và phân tích thực nghiệm. |
| CLAIM | Một kết luận gốc của nguồn. Không viết rộng hơn phạm vi paper | MAST phân loại 14 kiểu lỗi thành ba nhóm. Trong nhóm kiểm chứng, FM-3.3 là việc kiểm tra chéo chưa đúng đối với thông tin hoặc quyết định quan trọng. |
| BOUNDARY | Model / dataset / setup giới hạn kết luận đó | Bài báo phân tích 1.642 trace từ 7 hệ thống đa agent, chủ yếu với họ GPT-4 và Claude, trên tác vụ lập trình, toán và tác vụ agent tổng quát. Việc gắn nhãn FM-3.3 cho LogiX-VN là cách nhóm áp dụng taxonomy trên lab MockLLM. |
| HYPOTHESIS | Nếu đổi shape A→B (hoặc bật/tắt một thành phần), metric X sẽ đổi theo hướng nào, khoảng bao nhiêu | Nếu chặn việc dùng số ngày trễ trong ticket để ghi đè dữ liệu đơn đã xác minh, Swarm dự kiến sửa đúng một trial public, từ 69/70 lên 70/70 (+1,43 điểm phần trăm). Cost/success dự kiến giảm nhẹ vì bớt lượt tính lại quote và có thêm một trial pass. |
| METHOD | Biến giữ cố định: model, tools, seed, corpus, giá, evaluator. Biến duy nhất được thay đổi: … | So Swarm gốc với cùng Swarm khi vô hiệu hóa đầu vào của nhánh ghi đè ngày trễ. Giữ nguyên MockLLM và các lượt gọi model, bộ tool, 14 claim, seed 11/22/33/44/55, pricing, host và evaluator. Mỗi trial dùng SQLite mới. Chỉ script ablation vô hiệu hóa nhánh trong tiến trình đo. |
| FAILURE / METRIC | Một lỗi dự kiến và cách đo nó: pass rate, pass^5, variance, cost/success, leak, bypass | Dự kiến T02 ở seed 55 ghi refund sai khi tin số ngày khách tự khai. Đối chiếu `status` và kết quả chấm ledger trong trial này, sau đó so pass rate, pass^5, seed variance và cost/success. |

## Kết quả ablation

- Lệnh đã chạy (copy nguyên văn): `.\.venv\Scripts\python.exe -m scripts.ablate_swarm_ticket --out reports/ablation_swarm_ticket.json` từ `src/`.
- Run id / file report: `R-swarm-C-002-s55` ở cả hai nhánh trong [ablation_swarm_ticket.json](../../src/reports/ablation_swarm_ticket.json). Dùng `python -m scripts.ablate_swarm_ticket --out reports/ablation_swarm_ticket.json` nếu venv đã kích hoạt.
- Kết quả so với hypothesis (đúng / sai / không kết luận được), kèm số liệu: Đúng trên bộ public. T02 seed 55 từ `resolved` sai thành `denied` đúng. Đây là trial duy nhất thay đổi. Pass rate từ 98,57% (69/70) lên 100% (70/70), pass^5 từ 92,86% lên 100%, seed variance từ 0,000816 về 0. Cost/success giảm từ 49,57 xuống 48,72 VND. Nhóm phân loại ca này là FM-3.3 vì Swarm đã có dữ liệu đơn đã xác minh nhưng dùng nguồn sai khi quyết định.
- Giới hạn: Một ca sai có chủ đích trong 70 trial giả lập. Kết quả chỉ đo tác động của quy tắc chọn nguồn dữ liệu trong lab. Latency là `unavailable`.

Muốn đi xa hơn một ablation để lấy điểm cộng thì xem [docs/bonus_topics.md](../../docs/bonus_topics.md).
