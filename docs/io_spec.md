# Đặc tả Đầu vào/Đầu ra: CLI Agent Orchestrator (CAO)

Tài liệu này chi tiết hóa các cấu trúc dữ liệu, giao thức giao tiếp và đặc tả giao diện cho hệ thống CAO.

---

## 1. Giao diện CLI (`cao`)
Điểm nhập liệu chính cho người dùng để quản lý các phiên điều phối và quy trình công việc (flows).

### **Đầu vào (Lệnh)**
*   **Điều khiển Phiên làm việc:**
    *   `cao launch --agents <tên_hồ_sơ> [--provider <nhà_cung_cấp>] [--yolo]`: Khởi chạy một tác nhân giám sát.
    *   `cao shutdown [--all | --session <id_phiên>]`: Đóng các môi trường điều phối.
    *   `cao list`: Hiển thị các phiên hoạt động và trạng thái terminal.
*   **Quản lý Quy trình (Tự động hóa):**
    *   `cao flow add <file.md>`: Đăng ký một quy trình làm việc mới theo lịch trình.
    *   `cao flow run <tên_quy_trình>`: Kích hoạt thủ công một quy trình cụ thể.

### **Đầu ra**
*   **Giao diện Terminal:** Các bảng trạng thái (sử dụng thư viện `rich`), nhật ký phiên và các cửa sổ `tmux` tương tác.
*   **Mã phản hồi (Return Codes):** Các mã thoát tiêu chuẩn (0 cho thành công, khác 0 cho lỗi).

---

## 2. Orchestrator API (`cao-server`)
API REST nội bộ được các tác nhân và công cụ sử dụng để giao tiếp với máy chủ trung tâm (Mặc định: `http://localhost:9889`).

### **Các Điểm cuối (Endpoints) & Cấu trúc Dữ liệu**

#### **`POST /v1/orchestration/handoff`**
*   **Đầu vào (JSON):**
    ```json
    {
      "agent_profile": "string (bắt buộc)",
      "message": "string (bắt buộc)",
      "working_directory": "string (tùy chọn)",
      "context": "object (tùy chọn)"
    }
    ```
*   **Đầu ra (JSON):**
    ```json
    {
      "status": "COMPLETED",
      "output": "Kết quả văn bản từ tác nhân thực thi",
      "terminal_id": "T_12345"
    }
    ```

#### **`POST /v1/orchestration/assign`**
*   **Đầu vào (JSON):** Tương tự như `handoff`.
*   **Đầu ra (JSON):**
    ```json
    {
      "status": "PROCESSING",
      "terminal_id": "T_67890",
      "message": "Tác nhân đã được tạo trong một phiên làm việc ngầm mới."
    }
    ```

---

## 3. Các Công cụ MCP (Model Context Protocol)
Các công cụ được cung cấp cho các tác nhân AI để điều phối giữa các tác nhân.

| Tên Công cụ | Tham số Đầu vào | Đầu ra Mong đợi |
| :--- | :--- | :--- |
| `handoff` | `agent_profile`, `message`, `working_directory` | Phản hồi văn bản đầy đủ từ tác nhân được ủy quyền (chế độ chờ). |
| `assign` | `agent_profile`, `message`, `working_directory` | `terminal_id` (chế độ không chờ). |
| `send_message`| `terminal_id`, `message` | Xác nhận tin nhắn đã được gửi đến hộp thư đích. |

---

## 4. Hồ sơ Tác nhân & Quy trình (Cấu hình)
Cấu hình được xử lý thông qua các tệp Markdown với phần đầu (frontmatter) bằng YAML.

### **Định dạng Hồ sơ Tác nhân (`.md`)**
*   **Đầu vào (YAML Frontmatter):**
    ```yaml
    name: "developer"
    description: "Tác nhân lập trình chuyên gia"
    provider: "kiro_cli" # ví dụ: claude_code, amazon_q, codex_cli
    instructions: "Tuân thủ các hướng dẫn PEP 8..."
    ```
*   **Đầu ra (System Prompt):** Phần còn lại của tệp Markdown sẽ được thêm vào làm lời nhắc hệ thống (system prompt) cơ sở cho tác nhân.

### **Cấu hình Quy trình - Flow (`.md`)**
*   **Đầu vào (YAML Frontmatter):**
    ```yaml
    name: "daily_standup"
    schedule: "0 9 * * 1-5" # Cron: Thứ 2-6 lúc 9 giờ sáng
    script: "./health_check.sh" # Chỉ chạy nếu script trả về mã 0
    agents: ["summary_bot"]
    ```

---

## 5. Giao thức Giao tiếp
*   **Giữa các tác nhân:** Nhắn tin bất đồng bộ thông qua hệ thống "Hộp thư đến" (Inbox).
*   **Tác nhân tới Máy chủ:** JSON thông qua giao thức REST (HTTP).
*   **Máy chủ tới Terminal:** Các biến môi trường (`CAO_TERMINAL_ID`, `CAO_SERVER_URL`) và tiêm lệnh (command injection) vào `tmux`.
