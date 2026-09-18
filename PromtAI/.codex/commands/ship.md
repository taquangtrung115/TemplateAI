---
description: Chạy trọn dây chuyền bốn agent cho một yêu cầu tính năng.
---

Chạy trọn dây chuyền làm tính năng cho: $ARGUMENTS

Làm lần lượt các chặng dưới đây, không nhảy cóc. Sau mỗi chặng, kiểm tra
file bàn giao đã tồn tại rồi mới sang chặng kế tiếp.

0. Xem đang đứng trên nhánh git nào. Nếu là nhánh chính (main hoặc
   master), dừng lại và báo cho tôi, không chạy tiếp.
   Sau đó dọn thư mục PromtAI/.codex/results/, tức xoá plan.md, change.md,
   result-test.md và evaluate.md của lần chạy trước (nếu có), để không ai đọc
   nhầm file cũ.

1. Giao việc cho subagent planner kèm yêu cầu tính năng ở trên.
   Chờ tới khi có PromtAI/.codex/results/plan.md.

2. Nếu bản kế hoạch có mục CÂU HỎI CÒN BỎ NGỎ, dừng lại và đưa các câu
   hỏi đó cho tôi. Nếu không có, giao việc cho subagent coder.
   Chờ tới khi có PromtAI/.codex/results/change.md.

3. Giao việc cho subagent tester.
   Chờ tới khi có PromtAI/.codex/results/result-test.md.
   Có test rớt thì dừng lại và cho tôi xem phần rớt.

4. Giao việc cho subagent reviewer. Cho tôi xem PromtAI/.codex/results/evaluate.md.

Báo lại phán quyết cuối cùng. Không gộp nhánh, không push, không tạo pull
request. Cứ để nguyên nhánh đó cho tôi tự xem.