# Báo cáo cá nhân — Trần Quang Minh

## 1. Thông tin thành viên

- Họ tên: Trần Quang Minh
- MSSV: 2A202603929
- Email: tqminhhn04@gmail.com
- GitHub: https://github.com/MinhTrannnnn

## 2. Phần việc phụ trách

Phụ trách triển khai Orchestrator (Shape C) ở CP2–CP3: xây dựng luồng xử lý claim, giao việc cho investigator qua handoff an toàn, gọi host để hoàn tiền và kiểm tra crash/resume. Bổ sung test cho ranh giới dữ liệu, approval và các nhánh trả kết quả.

## 3. Đóng góp cụ thể

- Commit [`ef07efa`](https://github.com/qwerty12321434/K5-L3B-DAY08-MQBT-FromWorkflowsToAgents/commit/ef07efa) (tác giả Git `MinhTran`) thay stub bằng `run_claim` cho Shape C, giữ nguyên chữ ký hàm và đóng kết nối SQLite bằng `finally`.
- Xử lý các nhánh: claim không tồn tại → `failed`; thiếu mã đơn → `needs_info`; đơn không tìm thấy hoặc ngoài quyền sở hữu → `escalated`; quote bằng 0 hoặc host từ chối → `denied`; host ghi thành công → `resolved`. Kết quả được tạo bằng `finish()` để lấy effect và số audit từ ledger, đồng thời ghi trace.
- Tạo handoff bằng `make_handoff`, để `facts={}` và giao các tool đọc cho investigator. Không đưa nguyên claim, nội dung ticket, secret hoặc header vào handoff/trace. Dữ liệu đơn hàng được đọc theo principal; lời yêu cầu trong ticket không được dùng để sửa số tiền hay bỏ qua policy.
- Lấy quote từ `compute_compensation`, truyền **quote đầy đủ** vào `issue_refund`. Host chịu trách nhiệm kiểm approval, trần theo đơn và idempotency; orchestrator không tự trừ tiền đã hoàn. MockLLM chỉ soạn câu chữ, không quyết định số tiền hay quyền ghi.
- Truyền `crash_at` vào host và để `InjectedCrash` truyền ra ngoài. Thêm 4 hàm test, tạo thành **6 trường hợp kiểm thử** do test crash được chạy tại 3 điểm:
  - `test_handoff_and_trace_isolate_untrusted_context`: chèn secret và lệnh giả vào ticket; kiểm handoff chỉ cấp tool đọc, trace không lộ secret và refund vẫn đúng 35.000 VND.
  - `test_crash_propagates_and_resume_commits_once`: thử `before_effect`, `after_effect_before_checkpoint`, `after_checkpoint`; sau resume chỉ có một refund và một audit committed.
  - `test_unknown_claim_writes_fresh_trace`: claim không tồn tại vẫn trả lỗi có trace đúng run ID.
  - `test_missing_approval_reaches_host_and_records_denial`: quote dương nhưng không có approval thì không refund, đồng thời host ghi audit `policy_denied`.

Phần tích hợp sau đó ở commit `e0f56b0` do Nguyễn Hải Quang thực hiện: ghi trace trước điểm crash và bỏ lượt đọc policy dư thừa. Kết quả benchmark bên dưới thuộc bản tích hợp của nhóm, không chỉ riêng commit cá nhân.

## 4. Bằng chứng

- [Code Orchestrator](../../src/lab8/shapes/orchestrator.py) và [test Orchestrator](../../src/tests/test_orchestrator.py).
- Tại thời điểm hoàn thành commit `ef07efa`, chạy trong `src/`:

  | Lệnh | Kết quả |
  | --- | --- |
  | `python -m lab8.smoke` | `lab8.smoke pass` |
  | `python -m pytest -q` | 60 passed, gồm 6 trường hợp mới |
  | `python -m pytest -q -m student -o addopts= -k orchestrator` | 62 passed |

- Trong [benchmark public của bản tích hợp](../../src/reports/benchmark.json), Orchestrator đạt **70/70 trial** trên 14 claim × 5 seed `11,22,33,44,55`: pass rate = 1,0; pass^5 = 1,0; seed variance = 0; cost/success ≈ **38,16 VND mô phỏng**. Run ID mẫu: `R-orchestrator-C-001-s11`.
- [Crash suite của nhóm](../../src/reports/crash.json) đạt **20/20**. Suite này chủ yếu kiểm host và có ca swarm; bằng chứng trực tiếp cho crash/resume của Orchestrator là test 3 điểm crash và T12 trong ma trận student.

## 5. Công cụ AI đã dùng

Phần mở rộng B3 được bổ sung với hỗ trợ Codex: [Swarm verifier](../../src/bonus/swarm_verified.py), 12 claim với nhãn viết trực tiếp theo luật nghiệp vụ, runner đo safety và [test bonus](../../src/tests/test_bonus_swarm_verified.py). Bằng chứng chạy: 70 test bonus pass; toàn bộ 135 test mặc định và 248 test student pass. [Báo cáo bonus](bonus_report.md) ghi kết quả public 70/70, đối kháng 60/60 và crash/retry 3/3 của biến thể. Chi phí tăng nên nhóm vẫn chọn Workflow; chưa kết luận về hidden. Không gắn phần bổ sung này với commit Orchestrator cũ `ef07efa`.

Sử dụng **Codex (OpenAI)** để đọc yêu cầu CP2–CP3, tham khảo contract và các helper có sẵn, hỗ trợ triển khai orchestrator, viết test và tổng hợp bằng chứng cho báo cáo.

Kết quả do AI hỗ trợ được kiểm chứng bằng việc chạy smoke, test nền, ma trận student của Orchestrator và các test bổ sung nêu trên. Các test kiểm ledger/audit thực tế sau resume, kiểm phản ứng của host khi thiếu approval và quét nội dung handoff/trace thay vì chỉ dựa vào phần giải thích của AI. Số liệu báo cáo lấy từ output kiểm thử và file benchmark do công cụ sinh; không sửa tay JSON hay tự đặt run ID, commit hash.

## 6. Tự đánh giá

Phần triển khai cá nhân đáp ứng ma trận public của Orchestrator và bổ sung kiểm tra các ranh giới quan trọng: investigator chỉ có quyền đọc, tiền do code tính, host kiểm duyệt thao tác ghi. Bài học chính là phân chia trách nhiệm rõ giữa các thành phần giúp kiểm tra lỗi và quyền truy cập cụ thể hơn.

Handoff an toàn phải dựa trên allowlist trong code; tóm tắt bằng model không thay thế được việc lọc dữ liệu. Tương tự, checkpoint không đủ để chống hoàn trùng: ledger và audit trong transaction của host mới bảo đảm một effect khi retry. Cần truyền quote đầy đủ để host tự xử lý phần đã hoàn của cùng đơn.

Điểm cần cải thiện của bản đầu là chưa ghi trace trước khi crash và có lượt đọc policy dư thừa; nhóm đã bổ sung ở bước tích hợp. Phần cá nhân tập trung vào tính đúng và an toàn của Shape C, còn benchmark và quyết định kiến trúc cuối cùng là kết quả chung của nhóm.

Kết quả 70/70 chỉ phản ánh bộ public với MockLLM, chưa chứng minh hiệu quả trên model sản xuất hay bộ hidden. Chi phí là mô phỏng và latency là `unavailable`. Trong benchmark hiện có, workflow cũng đạt pass rate 1,0 với cost/success khoảng 27,86 VND, thấp hơn Orchestrator; do đó chưa có bằng chứng để chọn kiến trúc phức tạp hơn chỉ vì có nhiều thành phần.
