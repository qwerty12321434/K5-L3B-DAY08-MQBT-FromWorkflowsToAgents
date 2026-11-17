# Bonus report: B3

- Topic + nguồn: Verifier chống khuếch đại lỗi; Google Research, *Towards a Science of Scaling Agent Systems* (bài gốc 2025, bài giới thiệu 28/01/2026), slide 17/22/42. [Nguồn chính thức](https://research.google/blog/towards-a-science-of-scaling-agent-systems-when-and-why-agent-systems-work/).
- CLAIM: kiểm chứng tập trung có thể hạn chế lỗi lan truyền giữa các agent.
- BOUNDARY: nguồn nghiên cứu 180 cấu hình, 5 kiến trúc, 3 họ model trên Finance-Agent, BrowseComp-Plus, PlanCraft, Workbench. Lab này chỉ thử một bước verifier bằng code trên MockLLM; không tái lập quy mô hay mức cải thiện của nghiên cứu.
- HYPOTHESIS: verifier đưa Swarm về độ đúng của Workflow trên public và bộ đối kháng, giảm variance, đổi lại chi phí đọc lại dữ liệu và sửa quote.
- Biến cố định: claim, policy, host, MockLLM, seed 11/22/33/44/55, pricing, evaluator; mỗi trial SQLite mới. Biến thay đổi: thêm đúng một bước verifier vào Swarm. Giữ peer/parse/draft và nhánh candidate lấy delay từ ticket; draft dùng proposal cuối cùng nên token cost có thể thay đổi.
- Code: `src/bonus/swarm_verified.py`, phát triển từ Swarm tham chiếu của đề. Verifier đọc lại order theo principal, đối chiếu toàn bộ `ORDER_FIELDS` (gồm delay/status/damage và giá), tính lại quote tối đa một lần khi lệch, ghi `verifier_override`. Kiểm trước cả nhánh quote=0; host vẫn nhận quote đầy đủ.
- Nhãn: 12 claim mới trong `src/bonus/cases.jsonl`, `world.json`; đáp án viết trực tiếp theo luật, không sinh bằng bộ tính quote. Giải thích từng nhãn ở `src/bonus/CASES.md`.

## Lệnh tái lập

Chạy trong `src/`, Python 3.11–3.13, đã cài requirements và kích hoạt venv:

```bash
python -m pytest -q tests/test_bonus_swarm_verified.py
python -m bonus.evaluate --out reports/bonus_swarm_verified.json
```

Runner đăng ký biến thể qua `register_extra_shapes`, gọi nguyên `run_benchmark` của đề và quan sát ledger/trace trước khi SQLite tạm bị xóa. Không sửa evaluator; safety counts là số trial vi phạm, không phải số nhãn fail. Có thể chạy độc lập qua CLI chuẩn:

```bash
python -m lab8.benchmark --shapes workflow,swarm,swarm_verified --extra-shape swarm_verified=bonus.swarm_verified:run_claim --cases bonus/cases.jsonl --world bonus/world.json --seeds 11,22,33,44,55 --out reports/bonus_adversarial_cli.json
```

## Kết quả

Nguồn số liệu: [bonus_swarm_verified.json](../../src/reports/bonus_swarm_verified.json), các mục `public`, `adversarial`, `crash_retry`. VND là chi phí mô phỏng; latency `unavailable`.

| Corpus | Shape | Trial pass | Pass rate | Pass^5 | Seed variance | VND/success | Override |
| --- | --- | ---: | ---: | ---: | ---: | ---: | ---: |
| Public: 14 claim | Workflow | 70/70 | 100% | 100% | 0 | 27,86 | 0 |
| Public | Swarm | 69/70 | 98,57% | 92,86% | 0,000816 | 49,57 | 0 |
| Public | Swarm verified | 70/70 | 100% | 100% | 0 | 57,57 | 1 |
| Đối kháng: 12 claim | Workflow | 60/60 | 100% | 100% | 0 | 29,17 | 0 |
| Đối kháng | Swarm | 56/60 | 93,33% | 66,67% | 0,017778 | 56,96 | 0 |
| Đối kháng | Swarm verified | 60/60 | 100% | 100% | 0 | 63,99 | 10 |

Cả 390 trial được quan sát safety đầy đủ: secret leak, PII leak, approval bypass, ownership bypass, duplicate refund, unexpected ledger và trace contract failure đều **0** cho từng shape/corpus. 70 test bonus pass; ba điểm crash/resume của biến thể đạt **3/3**, retry hai lần vẫn đúng một refund/audit. Suite host 20/20 là bằng chứng riêng, không gộp thành 23/23.

Trace minh họa được nhúng trong `public.example_trace` để không commit thư mục traces: `R-swarm_verified-C-002-s55`, field `delay_days` lệch, proposal 20.000 → 0, một vòng sửa. Bộ đối kháng sửa bốn trial seed 55: B3-01/06 (khai quá), B3-02/11 (khai thiếu). 10 override không đồng nghĩa 10 trial được cứu: một số sửa input nhưng entitlement không đổi do status/damage/approval đã quyết định kết quả.

## Kết luận và giới hạn

Dữ liệu ủng hộ hypothesis về độ đúng: verifier đạt ngang Workflow, variance về 0. Cost/success tăng so với Swarm: public 49,57 → 57,57; đối kháng 56,96 → 63,99. Khác ablation R2 chỉ tắt nhánh sai, B3 giữ chi phí candidate và tính cả đọc lại/sửa quote.

Workflow vẫn rẻ nhất và giữ nguyên lựa chọn ADR. Trong task có luật xác định sẵn, peer/parse/draft chưa chứng minh giá trị tăng thêm; verifier đang khôi phục nguồn sự thật mà Workflow dùng trực tiếp. Kết luận có thể đổi nếu corpus đại diện cần phân tích ngữ cảnh và Swarm đem lại lợi ích đo được lớn hơn chi phí kiểm chứng.

MockLLM có lỗi chủ đích; 12 claim được thiết kế quanh rủi ro đã biết, không phải mẫu ngẫu nhiên. Verifier tin order và bộ tính quote chính thức, không phát hiện sai sót bên trong hai nguồn đó. Không suy rộng sang model thật hoặc hidden. Bonus chỉ được tính khi shape chọn qua must-pass và hidden ≥ 0,80 do giảng viên xác nhận; self-check không nhãn không chứng minh gate này.
