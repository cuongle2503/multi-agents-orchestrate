# Yêu cầu Hệ thống: Multi-Agent Orchestrator

Tài liệu này xác định các yêu cầu chức năng (hệ thống làm gì) và yêu cầu phi chức năng (hệ thống vận hành như thế nào) cho dự án điều phối đa tác vụ.

---

## 1. Yêu cầu Chức năng (Functional Requirements)
Tập trung vào các tính năng và nghiệp vụ mà hệ thống phải thực hiện.

### **1.1. Mục tiêu Hệ thống**
Hệ thống phải có khả năng quản lý, điều phối và phân quyền nhiệm vụ giữa các tác nhân AI (Agents) khác nhau trong một môi trường làm việc thống nhất (Terminal/tmux).

### **1.2. Các Chức năng Chính**
*   **Khởi tạo Giám sát (Supervisor):** Cho phép người dùng khởi chạy một tác nhân chính để tiếp nhận yêu cầu tổng thể.
*   **Ủy quyền Nhiệm vụ (Delegation):**
    *   **Bàn giao (Handoff):** Chuyển nhiệm vụ cho tác nhân khác và chờ kết quả trả về.
    *   **Giao việc (Assign):** Tạo một tác nhân chạy ngầm để thực hiện nhiệm vụ mà không làm gián đoạn tác nhân chính.
*   **Giao tiếp liên Tác nhân:** Hỗ trợ gửi tin nhắn trực tiếp giữa các tác nhân đang hoạt động thông qua hệ thống hộp thư (Inbox).
*   **Quản lý Phiên (Session Management):**
    *   Liệt kê danh sách các tác nhân đang chạy.
    *   Dừng hoặc xóa bỏ các phiên làm việc (từng phần hoặc toàn bộ).
*   **Tự động hóa Quy trình (Flows):** Cho phép định nghĩa và chạy các kịch bản đa tác vụ theo lịch trình (Cron) hoặc theo điều kiện cụ thể.

### **1.3. Đầu vào (Inputs)**
*   **Lệnh người dùng:** Các câu lệnh từ giao diện dòng lệnh (CLI).
*   **Hồ sơ Tác nhân (Profiles):** Các tệp cấu hình định nghĩa vai trò, hướng dẫn và nhà cung cấp AI.
*   **Lời nhắc (Prompts):** Các yêu cầu bằng ngôn ngữ tự nhiên từ người dùng hoặc từ tác nhân khác.
*   **Dữ liệu ngữ cảnh:** Thư mục làm việc, các tệp tin mã nguồn và biến môi trường.

### **1.4. Đầu ra (Outputs)**
*   **Phản hồi của Tác nhân:** Văn bản kết quả, mã nguồn được tạo ra hoặc các hành động thực thi trên hệ thống.
*   **Giao diện Trạng thái:** Bảng hiển thị trạng thái của các terminal, tiến độ công việc.
*   **Nhật ký (Logs):** Lịch sử giao tiếp và các lỗi phát sinh trong quá trình điều phối.

---

## 2. Yêu cầu Phi chức năng (Non-functional Requirements)
Tập trung vào các đặc tính chất lượng và cách hệ thống vận hành.

### **2.1. Hiệu năng (Performance)**
*   **Độ trễ API:** Các lệnh điều phối nội bộ (gọi qua localhost) phải có phản hồi gần như tức thì (< 100ms).
*   **Tốc độ xử lý:** Việc tạo mới một phiên `tmux` và khởi chạy tác nhân không được gây treo máy hoặc chậm trễ đáng kể giao diện người dùng.

### **2.2. Khả năng Mở rộng (Scalability)**
*   **Số lượng Tác nhân:** Hệ thống có thể quản lý đồng thời ít nhất 5-10 tác nhân hoạt động cùng lúc trên một máy cục bộ mà không gặp xung đột tài nguyên.
*   **Tích hợp:** Dễ dàng thêm các nhà cung cấp AI mới (Providers) hoặc các công cụ MCP mới mà không cần thay đổi kiến trúc cốt lõi.

### **2.3. Khả năng Sẵn sàng (Availability)**
*   **Trạng thái Máy chủ:** `cao-server` phải hoạt động ổn định trong suốt thời gian phiên làm việc diễn ra. Nếu máy chủ bị tắt, hệ thống phải có cơ chế khôi phục trạng thái từ các phiên `tmux` đang tồn tại.

### **2.4. Tính nhất quán (Consistency)**
*   **Thứ tự tin nhắn:** Đảm bảo tin nhắn trong hộp thư đến (Inbox) của các tác nhân được duy trì đúng thứ tự gửi.
*   **Ngữ cảnh đồng bộ:** Các tác nhân làm việc trong cùng một thư mục phải nhận diện được các thay đổi tệp tin do tác nhân khác thực hiện.

### **2.5. Độ tin cậy (Reliability)**
*   **Xử lý lỗi:** Khi một tác nhân con gặp lỗi hoặc bị treo, hệ thống phải báo cáo lại cho tác nhân giám sát thay vì làm treo toàn bộ phiên làm việc.
*   **Cô lập môi trường:** Lỗi của một tác nhân (ví dụ: lệnh shell sai) không được gây ảnh hưởng đến tính toàn vẹn của các tác nhân khác.
