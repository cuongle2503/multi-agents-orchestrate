# Đề xuất Công nghệ (Technology Stack)

Tài liệu này liệt kê các công nghệ, thư viện và công cụ được đề xuất để xây dựng hệ thống **Multi-Agent Orchestrator**.

---

## 1. Ngôn ngữ & Trình quản lý Gói
*   **Ngôn ngữ chính:** **Python 3.10+**
    *   *Lý do:* Python có hệ sinh thái thư viện AI/LLM phong phú nhất và hỗ trợ tốt cho các giao thức như MCP.
*   **Quản lý gói & Môi trường:** **uv**
    *   *Lý do:* Tốc độ cài đặt cực nhanh, thay thế hoàn hảo cho `pip` và `venv`, giúp việc quản lý phụ thuộc trở nên nhất quán.

## 2. Lõi Điều phối (Orchestration Core)
*   **Máy chủ Orchestrator:** **FastAPI**
    *   *Lý do:* Nhẹ, hiệu suất cao, hỗ trợ tốt cho các tác vụ bất đồng bộ (asyncio) và tự động tạo tài liệu API (Swagger).
*   **Giao thức giao tiếp:** **Model Context Protocol (MCP)**
    *   *Lý do:* Tiêu chuẩn hóa cách các tác nhân AI tương tác với công cụ và dữ liệu, giúp dễ dàng tích hợp với các hệ sinh thái như Anthropic hay OpenAI.
*   **Quản lý Terminal:** **libtmux**
    *   *Lý do:* Thư viện Python mạnh mẽ để điều khiển `tmux` theo lập trình, cho phép tạo, quản lý và gửi lệnh tới các cửa sổ terminal một cách chính xác.

## 3. Giao diện Dòng lệnh (CLI)
*   **Xây dựng CLI:** **Typer**
    *   *Lý do:* Xây dựng dựa trên `Click`, dễ sử dụng, hỗ trợ gợi ý kiểu dữ liệu (type hints) và tự động tạo menu trợ giúp đẹp mắt.
*   **UI/UX Terminal:** **Rich**
    *   *Lý do:* Hỗ trợ hiển thị bảng biểu, màu sắc, thanh tiến trình (progress bars) và định dạng Markdown ngay trong terminal, giúp trải nghiệm người dùng chuyên nghiệp hơn.

## 4. Xử lý Dữ liệu & Cấu hình
*   **Validation dữ liệu:** **Pydantic V2**
    *   *Lý do:* Đảm bảo tính nhất quán của dữ liệu đầu vào/đầu ra giữa CLI và API.
*   **Định dạng cấu hình:** **YAML** (sử dụng `PyYAML`)
    *   *Lý do:* Dễ đọc, dễ viết, phù hợp để định nghĩa Hồ sơ Tác nhân (Agent Profiles).

## 5. Kiểm thử & Chất lượng Mã nguồn
*   **Kiểm thử:** **Pytest**
    *   *Lý do:* Tiêu chuẩn công nghiệp cho kiểm thử Python, hỗ trợ tốt cho cả unit test và integration test.
*   **Linting & Formatting:** **Ruff**
    *   *Lý do:* Công cụ cực nhanh thay thế cho Flake8, Black, và Isort, giúp duy trì chất lượng mã nguồn đồng nhất.

## 6. Hạ tầng & Triển khai
*   **Môi trường chạy:** **tmux** (bắt buộc trên Linux/macOS hoặc WSL trên Windows).
*   **CI/CD:** **GitHub Actions**
    *   *Lý do:* Tự động hóa việc kiểm tra mã nguồn và đóng gói khi có thay đổi được đẩy lên repository.
