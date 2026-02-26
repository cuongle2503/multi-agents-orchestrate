# Kiến trúc Tổng quan: CLI Agent Orchestrator (CAO)

**CLI Agent Orchestrator (CAO)** là một hệ thống đa tác vụ (multi-agent) phân cấp, nhẹ, được thiết kế để điều phối nhiều phiên làm việc của các tác nhân AI (AI agents) bên trong các cửa sổ terminal `tmux`. Nó cho phép giải quyết các vấn đề phức tạp bằng cách ủy quyền nhiệm vụ từ một tác nhân **Giám sát (Supervisor)** cho các tác nhân **Thực thi (Worker)** chuyên biệt.

## 1. Tổng quan Hệ thống

CAO tuân theo kiến trúc **Giám sát - Thực thi (Supervisor-Worker)**. Một bộ điều phối trung tâm quản lý vòng đời và sự giao tiếp của các tác nhân khác nhau, mỗi tác nhân chạy trong môi trường biệt lập của riêng mình.

### Các trụ cột kiến trúc chính:
*   **Cấu trúc Phân cấp:** Một tác nhân "Giám sát" trung tâm duy trì ngữ cảnh của dự án và ủy quyền các nhiệm vụ con cho các "Nhân viên" chuyên biệt (ví dụ: lập trình viên, người kiểm thử, người đánh giá).
*   **Cô lập Phiên làm việc:** Mỗi tác nhân hoạt động trong một phiên hoặc cửa sổ `tmux` riêng biệt, đảm bảo ngữ cảnh (tệp tin, môi trường) luôn sạch sẽ và tập trung.
*   **Điều phối Tập trung:** `cao-server` đóng vai trò là bộ não của hệ thống, theo dõi trạng thái của các terminal và định tuyến tin nhắn thông qua REST API.

---

## 2. Các Thành phần Chính

### A. Bộ điều phối (Orchestrator - `cao-server`)
Máy chủ HTTP cục bộ (chạy trên cổng 9889) duy trì trạng thái toàn cầu của quá trình điều phối.
*   **Quản lý Terminal:** Theo dõi mọi terminal của tác nhân thông qua một `CAO_TERMINAL_ID` duy nhất và trạng thái của nó (`IDLE`, `PROCESSING`, `COMPLETED`, `ERROR`).
*   **Trung tâm Nhắn tin:** Triển khai hệ thống "Hộp thư đến" (Inbox) cho mỗi terminal để cho phép giao tiếp bất đồng bộ giữa các tác nhân.

### B. Tác nhân (Agents)
Các tác nhân được định nghĩa bởi các **Hồ sơ (Profiles)** (YAML/Markdown) quy định vai trò và hướng dẫn.
*   **Tác nhân Giám sát (Supervisor Agent):** Giao diện chính cho người dùng, chịu trách nhiệm lập kế hoạch và ủy quyền.
*   **Tác nhân Thực thi (Worker Agents):** Các tác nhân chuyên biệt được tạo ra cho các nhiệm vụ cụ thể (ví dụ: `code_reviewer`, `security_auditor`).
*   **Nhà cung cấp (Providers):** Các công cụ AI nền tảng (Kiro CLI, Claude Code, Codex CLI, Amazon Q Developer CLI).

### C. Công cụ (Tools - Tích hợp MCP)
CAO sử dụng **Giao thức Ngữ cảnh Mô hình (Model Context Protocol - MCP)** để cung cấp các khả năng điều phối dưới dạng công cụ:
*   `handoff`: Bàn giao nhiệm vụ đồng bộ (người gọi chờ kết quả).
*   `assign`: Giao nhiệm vụ bất đồng bộ (chạy ngầm).
*   `send_message`: Giao tiếp trực tiếp giữa các tác nhân để tạo vòng lặp phản hồi.

---

## 3. Quy trình & Luồng Dữ liệu

1.  **Khởi tạo:** Người dùng khởi động `cao-server` và chạy tác nhân **Giám sát**.
2.  **Phân tích Nhiệm vụ:** Giám sát nhận mục tiêu và xác định nhu cầu hỗ trợ từ các chuyên gia.
3.  **Ủy quyền:** Giám sát sử dụng công cụ MCP (ví dụ: `assign`) để tạo một **Nhân viên** trong một cửa sổ `tmux` mới.
4.  **Thực thi:** Nhân viên thực hiện nhiệm vụ một cách độc lập trong môi trường biệt lập của mình.
5.  **Báo cáo:** Khi hoàn thành, Nhân viên gửi tin nhắn phản hồi về hộp thư đến của Giám sát.
6.  **Tổng hợp:** Giám sát theo dõi hộp thư, thu thập tất cả kết quả và đưa ra phản hồi cuối cùng cho người dùng.

---

## 4. Công nghệ Sử dụng

*   **Ngôn ngữ:** Python 3.10+
*   **Trình quản lý Terminal:** `tmux` (để quản lý phiên và cô lập giao diện)
*   **Giao thức:** Model Context Protocol (MCP) & REST API
*   **Quản lý Môi trường:** `uv` (để xử lý nhanh các gói Python)
*   **Giao tiếp:** Nhắn tin bất đồng bộ (dựa trên Inbox)

---

## 5. Các Mẫu Thiết kế (Design Patterns)

*   **Hệ thống Đa tác vụ Phân cấp (HMAS):** Phân chia rõ ràng công việc giữa bộ điều phối và các chuyên gia.
*   **Mẫu Sidecar/Orchestrator:** Sử dụng một tiến trình máy chủ riêng biệt để quản lý trạng thái và giao tiếp cho các công cụ CLI.
*   **Giao tiếp hướng Sự kiện:** Ghép nối lỏng lẻo các tác nhân thông qua hộp thư và theo dõi trạng thái.
