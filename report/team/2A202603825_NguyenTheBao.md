# Báo cáo cá nhân — Nguyễn Thế Bảo

## 1. Thông tin thành viên

- Họ tên: Nguyễn Thế Bảo
- MSSV: 2A202603825
- Email: nguyenthebao2506@gmail.com
- GitHub: NTheBao

## 2. Phần việc phụ trách

Workflow (Shape A).

## 3. Đóng góp cụ thể

- Commit `7355ff7` (tác giả Git `Bảo Nguyễn Thế`) triển khai `run_claim` theo các bước Python cố định. Workflow tra claim và đơn theo quyền, tính quote từ dữ liệu đơn và policy hiện hành, rồi chuyển quote đầy đủ và `crash_at` cho host.
- Chờ xác nhận tên test hoặc commit chứa test do thành viên thêm hoặc sửa.

## 4. Bằng chứng

- [Code Workflow](../../src/lab8/shapes/workflow.py).
- Trong benchmark public của bản tích hợp, Workflow đạt **70/70** trial với chi phí mô phỏng **27,86 VND/lần thành công**. Run ID mẫu là `R-workflow-C-001-s11`. [Benchmark nhóm](../../src/reports/benchmark.json)

## 5. Công cụ AI đã dùng

Tôi dùng Copilot để hỗ trợ triển khai và rà soát luồng Workflow, tập trung vào contract `RunResult`, trace, quote đầy đủ và cách chuyển tiếp `crash_at`. Tôi đối chiếu thay đổi với luật nghiệp vụ, xem diff và chạy ma trận student của Workflow thay vì dựa riêng vào gợi ý của AI.

## 6. Tự đánh giá

Qua Workflow, tôi hiểu rõ hơn vì sao phải dùng dữ liệu đơn và policy làm nguồn sự thật, còn ticket chỉ là đầu vào không đáng tin. Tôi cũng học được cách để host kiểm soát refund, idempotency và crash thay vì tự xử lý ở shape. Lần sau, tôi sẽ kiểm tra kỹ bộ lọc test để chắc chắn lệnh chỉ chạy đúng shape cần kiểm tra.
