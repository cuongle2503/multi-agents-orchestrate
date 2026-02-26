# High-Level Architecture: CLI Agent Orchestrator (CAO)

The **CLI Agent Orchestrator (CAO)** is a lightweight, hierarchical multi-agent system designed to coordinate multiple AI agent sessions within `tmux` terminals. It enables complex problem-solving by delegating tasks from a **Supervisor** agent to specialized **Worker** agents.

## 1. System Overview

CAO follows a **Supervisor-Worker** (Hierarchical) architecture. A central orchestrator manages the lifecycle and communication of various agents, each running in its own isolated environment.

### Key Architectural Pillars:
*   **Hierarchical Structure:** A central "Supervisor" agent maintains project context and delegates sub-tasks to specialized "Workers" (e.g., developers, testers, reviewers).
*   **Session Isolation:** Each agent operates within a dedicated `tmux` session or window, ensuring context (files, environment) remains clean and focused.
*   **Centralized Orchestration:** The `cao-server` acts as the system's brain, tracking terminal states and routing messages via a REST API.

---

## 2. Core Components

### A. Orchestrator (`cao-server`)
The local HTTP server (running on port 9889) that maintains the global state of the orchestration.
*   **Terminal Management:** Tracks every agent terminal via a unique `CAO_TERMINAL_ID` and its status (`IDLE`, `PROCESSING`, `COMPLETED`, `ERROR`).
*   **Messaging Hub:** Implements an "Inbox" system for each terminal to allow asynchronous communication between agents.

### B. Agents
Agents are defined by **Profiles** (YAML/Markdown) that specify roles and instructions.
*   **Supervisor Agent:** The primary interface for the user, responsible for planning and delegation.
*   **Worker Agents:** Specialized agents spawned for specific tasks (e.g., `code_reviewer`, `security_auditor`).
*   **Providers:** Underlying AI engines (Kiro CLI, Claude Code, Codex CLI, Amazon Q Developer CLI).

### C. Tools (MCP Integration)
CAO uses the **Model Context Protocol (MCP)** to expose orchestration as tools:
*   `handoff`: Synchronous task transfer (caller waits for result).
*   `assign`: Asynchronous task delegation (runs in background).
*   `send_message`: Direct inter-agent communication for feedback loops.

---

## 3. Workflow & Data Flow

1.  **Initialization:** The user starts `cao-server` and launches the **Supervisor**.
2.  **Task Analysis:** The Supervisor receives a goal and identifies the need for specialized assistance.
3.  **Delegation:** The Supervisor uses an MCP tool (e.g., `assign`) to spawn a **Worker** in a new `tmux` window.
4.  **Execution:** The Worker performs the task independently in its isolated environment.
5.  **Reporting:** Upon completion, the Worker sends a message back to the Supervisor's inbox.
6.  **Synthesis:** The Supervisor monitors its inbox, collects all results, and provides a final response to the user.

---

## 4. Technology Stack

*   **Language:** Python 3.10+
*   **Terminal Multiplexer:** `tmux` (for session management and UI isolation)
*   **Protocol:** Model Context Protocol (MCP) & REST API
*   **Environment Management:** `uv` (for fast Python package handling)
*   **Communication:** Asynchronous Messaging (Inbox-based)

---

## 5. Design Patterns

*   **Hierarchical Multi-Agent System (HMAS):** Clear division of labor between a coordinator and specialists.
*   **Sidecar/Orchestrator Pattern:** Using a separate server process to manage state and communication for CLI tools.
*   **Event-Driven Communication:** Decoupling agents via inboxes and status tracking.
