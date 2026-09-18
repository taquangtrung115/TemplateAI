## Agent Workspace & Routing (Quy tắc phân luồng độc lập)

Bắt buộc xác định danh tính trước khi thực thi. Mỗi bên chỉ được phép hoạt động trong đúng không gian của mình, tuyệt đối KHÔNG can thiệp chéo:

### 1. Nếu môi trường đang chạy là GEMINI (hoặc Antigravity):
- Không gian duy nhất được phép truy cập: `PromtAI/.gemini/`
  - Đọc kịch bản tại: `PromtAI/.gemini/commands/`
  - Đọc vai trò tại: `PromtAI/.gemini/agents/`
  - Đọc và ghi kết quả bàn giao tại: `PromtAI/.gemini/results/`
- CẤM: Không đọc hoặc ghi vào `PromtAI/.codex/`, không xuất file ra `.gemini/` hay `results/` ngoài root.

### 2. Nếu môi trường đang chạy là CODEX:
- Không gian duy nhất được phép truy cập: `PromtAI/.codex/`
  - Đọc kịch bản tại: `PromtAI/.codex/commands/`
  - Đọc vai trò tại: `PromtAI/.codex/agents/`
  - Đọc và ghi kết quả bàn giao tại: `PromtAI/.codex/results/`
- CẤM: Không đọc hoặc ghi vào `PromtAI/.gemini/`, không xuất file ra `.codex/` hay `results/` ngoài root.