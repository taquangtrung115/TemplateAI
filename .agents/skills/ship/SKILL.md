---
name: ship
description: Chạy trọn dây chuyền bốn agent (planner, coder, tester, reviewer) để phân tích, lập kế hoạch, code, test và review cho một tính năng.
---

# Ship Workflow — Dây chuyền 4 Subagents

Chạy trọn dây chuyền phát triển và kiểm định tính năng theo quy trình bốn bước độc lập.

## Nguyên tắc cốt lõi
- Làm lần lượt các chặng dưới đây, **không nhảy cóc**.
- Sau mỗi chặng, kiểm tra file bàn giao đã tồn tại và đầy đủ trong thư mục `PromtAI/.gemini/results/` (hoặc `PromtAI/.codex/results/`) rồi mới chuyển sang chặng kế tiếp.
- Tuyệt đối không đọc hoặc ghi kết quả ra ngoài root (`.gemini/` hay `results/`).
- Không gộp nhánh, không push git, không tạo pull request. Giữ nguyên nhánh làm việc để người dùng tự kiểm tra và nghiệm thu.

---

## Quy trình thực hiện chi tiết

### Chặng 0: Kiểm tra Git Branch & Dọn dẹp
1. Chạy lệnh kiểm tra nhánh Git hiện tại (`git branch --show-current`).
   - Nếu đang đứng trên nhánh chính (`main` hoặc `master`), **DỪNG LẠI NGAY LẬP TỨC** và thông báo cho người dùng chuyển sang một feature branch riêng. Tuyệt đối không chạy tiếp trên nhánh chính.
2. Dọn dẹp thư mục kết quả (`PromtAI/.gemini/results/`):
   - Xóa các file bàn giao của lần chạy trước nếu có: `plan.md`, `change.md`, `result-test.md`, `evaluate.md` để tránh đọc nhầm dữ liệu cũ.

### Chặng 1: Planner (Lập kế hoạch)
1. Đọc nội dung hướng dẫn vai trò tại `PromtAI/.gemini/agents/planner.md`.
2. Giao việc cho subagent **planner** với yêu cầu tính năng được truyền vào.
3. Chờ cho đến khi `PromtAI/.gemini/results/plan.md` được tạo đầy đủ theo cấu trúc:
   - Các file cần sửa hoặc tạo mới kèm đường dẫn chính xác.
   - Chữ ký hàm / interface.
   - Các trường hợp biên cần xử lý.
   - Quy ước bám theo.
   - Mục **CÂU HỎI CÒN BỎ NGỎ** (nếu có điểm mơ hồ).

### Chặng 2: Coder (Triển khai code)
1. Kiểm tra file `PromtAI/.gemini/results/plan.md`:
   - Nếu có mục **CÂU HỎI CÒN BỎ NGỎ**, **DỪNG LẠI** và đưa các câu hỏi đó cho người dùng. Không tự ý suy đoán yêu cầu.
2. Khi không có câu hỏi còn bỏ ngỏ (hoặc người dùng đã trả lời xác nhận):
   - Đọc hướng dẫn vai trò tại `PromtAI/.gemini/agents/coder.md`.
   - Giao việc cho subagent **coder** triển khai đúng theo bản kế hoạch.
   - Tuân thủ các nguyên tắc của repo: không sửa code ngoài phạm vi kế hoạch, viết code chuẩn quy ước.
3. Chờ cho đến khi `PromtAI/.gemini/results/change.md` được tạo với tóm tắt các file đã đổi và vị trí cần soi kỹ.

### Chặng 3: Tester (Viết và chạy test)
1. Đọc hướng dẫn vai trò tại `PromtAI/.gemini/agents/tester.md`.
2. Giao việc cho subagent **tester**.
3. `tester` viết test bao quát các trường hợp biên, luồng thuận lợi và trường hợp kỳ vọng thất bại, sau đó chạy test.
4. Chờ cho đến khi có `PromtAI/.gemini/results/result-test.md`.
5. Nếu có bất kỳ test nào thất bại:
   - **DỪNG LẠI**, hiển thị chi tiết phần rớt cho người dùng. Không tự ý sửa code sản phẩm để lách test.

### Chặng 4: Reviewer (Đánh giá và phán quyết)
1. Đọc hướng dẫn vai trò tại `PromtAI/.gemini/agents/reviewer.md`.
2. Giao việc cho subagent **reviewer**.
3. `reviewer` đọc các file trong `PromtAI/.gemini/results/`, chạy `git diff` để rà soát toàn bộ thay đổi và trả lời 3 câu hỏi cốt lõi:
   - Code có khớp bản kế hoạch không?
   - Test có giá trị thực tế không?
   - Có vấn đề bảo mật, hiệu năng, tính đúng đắn không?
4. Ghi phán quyết vào `PromtAI/.gemini/results/evaluate.md` với định dạng dòng đầu:
   `PHAN QUYET: CHOT / CAN SUA / CHAN`
5. Hiển thị nội dung `PromtAI/.gemini/results/evaluate.md` và báo cáo phán quyết cuối cùng cho người dùng.