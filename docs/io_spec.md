# Input/Output Specifications: CLI Agent Orchestrator (CAO)

This document details the data structures, communication protocols, and interface specifications for the CAO system.

---

## 1. CLI Interface (`cao`)
The primary entry point for users to manage orchestration sessions and flows.

### **Inputs (Commands)**
*   **Session Control:**
    *   `cao launch --agents <profile_name> [--provider <provider>] [--yolo]`: Starts a supervisor agent.
    *   `cao shutdown [--all | --session <session_id>]`: Terminates orchestration environments.
    *   `cao list`: Shows active sessions and terminal states.
*   **Flow Management (Automation):**
    *   `cao flow add <file.md>`: Registers a new scheduled workflow.
    *   `cao flow run <flow_name>`: Manually triggers a specific flow.

### **Outputs**
*   **Terminal UI:** Status tables (using `rich` library), session logs, and interactive `tmux` windows.
*   **Return Codes:** Standard exit codes (0 for success, non-zero for errors).

---

## 2. Orchestrator API (`cao-server`)
The internal REST API used by agents and tools to communicate with the central server (Default: `http://localhost:9889`).

### **Endpoints & Data Structures**

#### **`POST /v1/orchestration/handoff`**
*   **Input (JSON):**
    ```json
    {
      "agent_profile": "string (required)",
      "message": "string (required)",
      "working_directory": "string (optional)",
      "context": "object (optional)"
    }
    ```
*   **Output (JSON):**
    ```json
    {
      "status": "COMPLETED",
      "output": "The text result from the worker agent",
      "terminal_id": "T_12345"
    }
    ```

#### **`POST /v1/orchestration/assign`**
*   **Input (JSON):** Identical to `handoff`.
*   **Output (JSON):**
    ```json
    {
      "status": "PROCESSING",
      "terminal_id": "T_67890",
      "message": "Agent spawned in a new background session."
    }
    ```

---

## 3. MCP Tools (Model Context Protocol)
Tools exposed to AI agents for inter-agent coordination.

| Tool Name | Input Parameters | Expected Output |
| :--- | :--- | :--- |
| `handoff` | `agent_profile`, `message`, `working_directory` | Full text response from the delegated agent (blocking). |
| `assign` | `agent_profile`, `message`, `working_directory` | `terminal_id` (non-blocking). |
| `send_message`| `terminal_id`, `message` | Confirmation of delivery to the target inbox. |

---

## 4. Agent Profiles & Flows (Configuration)
Configuration is handled via Markdown files with YAML frontmatter.

### **Agent Profile Format (`.md`)**
*   **Input (YAML Frontmatter):**
    ```yaml
    name: "developer"
    description: "Expert coder agent"
    provider: "kiro_cli" # e.g., claude_code, amazon_q, codex_cli
    instructions: "Follow PEP 8 guidelines..."
    ```
*   **Output (System Prompt):** The rest of the Markdown file is appended as the base system prompt for the agent.

### **Flow Configuration (`.md`)**
*   **Input (YAML Frontmatter):**
    ```yaml
    name: "daily_standup"
    schedule: "0 9 * * 1-5" # Cron: Mon-Fri at 9 AM
    script: "./health_check.sh" # Only run if script exits with 0
    agents: ["summary_bot"]
    ```

---

## 5. Communication Protocol
*   **Inter-agent:** Asynchronous messaging via the "Inbox" system.
*   **Agent-to-Server:** RESTful JSON over HTTP.
*   **Server-to-Terminal:** Environment variables (`CAO_TERMINAL_ID`, `CAO_SERVER_URL`) and `tmux` command injection.
