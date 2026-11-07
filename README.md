# 🤖 AI Multiple Agent (CrewAI)

A **Python multi-agent system built on [CrewAI](https://github.com/crewAIInc/crewAI)** where a **manager LLM** coordinates three specialised worker agents — **Researcher**, **Developer** and **Reviewer**. The manager delegates work in a hierarchical process, and the agents can call tools exposed by a local **MCP (Model Context Protocol) server**.

Everything runs locally on **Ollama**, so no cloud API key is needed.

> A hand-built variant of this idea (manager loop written from scratch with the OpenAI SDK) lives at
> [mishraabhishek11/ai-multiple-agent](https://github.com/mishraabhishek11/ai-multiple-agent).
> This repository is the CrewAI version.

---

## 🚀 Features

### 🧭 Hierarchical Crew

- `crew.py` assembles the agents and tasks into a `Crew` with `Process.hierarchical`
- A separate `manager_llm` plans, delegates and validates the work of the workers
- `verbose=True` prints every step the agents take

### 🔎 Researcher Agent

- Role: **AI Researcher**
- Researches AI agent concepts (agent loop, tools, state, observations)
- Has access to the **MCP Current Time** tool

### 💻 Developer Agent

- Role: **Python Developer**
- Receives the researcher's output as context
- Produces a complete Python example with a short explanation

### ✅ Reviewer Agent

- Role: **Code Reviewer**
- Receives the developer's output as context
- Returns identified problems and recommended improvements

### 🔌 MCP Server and Tools

- `mcp_server/server.py` is a [FastMCP](https://gofastmcp.com) server named **Time Server**
- `mcp_server/client.py` connects to it, discovers tools and executes them
- `mcp_tools.py` wraps the MCP `current_time` tool as a CrewAI tool (`MCP Current Time`)

| MCP tool            | Description                                   | Used by an agent |
| ------------------- | --------------------------------------------- | ---------------- |
| `current_time`      | Returns the current date and time             | Researcher       |
| `roll_dice`         | Rolls a six-sided die                         | Not wired yet    |
| `generate_password` | Generates a password (default length 12)      | Not wired yet    |

### 🧠 Memory

- `memory=True` on the crew, with an **Ollama embedder** (`nomic-embed-text`)
- Lets the crew remember information (for example the user's name) across interactions

### 🔌 Central LLM Configuration

- `llm.py` defines `llm` (workers) and `manager_llm` (manager)
- Both use `ollama/qwen3:4b` by default; change the model string in one place

---

## 🛠 Tech Stack

### Core

- Python 3.12 (CrewAI supports 3.10 – 3.13)

### Libraries

| Package   | Purpose                                    |
| --------- | ------------------------------------------ |
| `crewai`  | Agents, tasks, crew and memory             |
| `fastmcp` | MCP server and client                      |
| `ollama`  | Python client required by the Ollama embedder |

### External

- [Ollama](https://ollama.com) running locally, with the models below

---

## 📂 Project Structure

```
ai-multiple-agent-crewai/
├─ mcp_server
│  ├─ client.py          # MCP client: connect, discover tools, execute, disconnect
│  └─ server.py          # FastMCP "Time Server" with current_time, roll_dice, generate_password
├─ agents.py             # Researcher, Developer and Reviewer agents
├─ crew.py               # Hierarchical crew with memory and Ollama embedder
├─ llm.py                # LLM and manager LLM configuration
├─ main.py               # Entry point: kicks off the crew and prints the result
├─ mcp_tools.py          # Wraps MCP tools as CrewAI tools
├─ tasks.py              # Research, coding and review tasks
└─ requirements.txt      # Pinned dependencies
```

---

## ⚙ Installation & Setup

### 1⃣ Clone the repository

```
git clone <your-repository-url>
cd ai-multiple-agent-crewai
```

### 2⃣ Create and activate a virtual environment

```
py -3.12 -m venv .venv
.venv\Scripts\activate
```

### 3⃣ Install dependencies

```
pip install -r requirements.txt
```

### 4⃣ Install and start Ollama, then pull the models

```
ollama pull qwen3:4b
ollama pull nomic-embed-text
```

- `qwen3:4b` — chat model for the agents and the manager
- `nomic-embed-text` — embedding model used by crew memory

The Ollama app (or `ollama serve`) must be running before you start the crew.

### 5⃣ Run the system

```
python main.py
```

---

## 🧑‍💻 Usage

1. Run `python main.py`
2. Watch the manager delegate to the researcher, developer and reviewer (verbose output)
3. Read the **FINAL RESULT** section printed at the end

```
============================================================
FINAL RESULT
============================================================
<output of the last task>
============================================================
```

To change what the agents work on, edit the task descriptions in `tasks.py`.

---

## 🔁 How It Works

1. `main.py` imports the crew from `crew.py` and calls `crew.kickoff()`
2. The crew runs in **hierarchical** mode: `manager_llm` decides which agent handles each task
3. `research_task` → the **Researcher** explains the agent loop, tools, state and observations, and calls the **MCP Current Time** tool
4. `coding_task` → the **Developer** uses the research as context to write a simple agent-loop example
5. `review_task` → the **Reviewer** uses the code as context to list problems and improvements
6. Crew memory (Ollama embeddings) stores information for later interactions
7. The final result is printed

```
Manager LLM ──delegates──▶ Researcher ──context──▶ Developer ──context──▶ Reviewer
                               │
                               └── MCP Current Time ──▶ FastMCP Time Server
```

---

## ➕ Adding Your Own Agent or Tool

**New MCP tool**

1. Add a function decorated with `@mcp.tool()` in `mcp_server/server.py`
2. Wrap it in `mcp_tools.py` with CrewAI's `@tool("Name")`, calling `execute_tool(client, "<tool name>")`
3. Pass it to an agent through `tools=[...]` in `agents.py`

**New agent**

1. Define an `Agent` in `agents.py` (role, goal, backstory, `llm`)
2. Define its `Task` in `tasks.py`, using `context=[...]` to receive earlier output
3. Add the agent and task to the lists in `crew.py`

---

## 🩺 Troubleshooting

| Error | Cause and fix |
| ----- | ------------- |
| `The ollama python package is not installed` | Run `pip install -r requirements.txt` inside the venv |
| Connection refused to `localhost:11434` | Ollama is not running. Start the Ollama app or run `ollama serve` |
| `model "qwen3:4b" not found` / `nomic-embed-text` not found | Run `ollama pull qwen3:4b` and `ollama pull nomic-embed-text` |
| `py -3.11` says no suitable Python runtime | That version is not installed. Use an installed one, for example `py -3.12 -m venv .venv` (`py -0p` lists them) |
| Poor or looping agent output | `qwen3:4b` is small. Try a larger Ollama model in `llm.py` |

---

## 🎯 Learning Objectives

- Multi-agent orchestration with CrewAI's hierarchical process
- Specialised agents with focused roles, goals and backstories
- Passing context between tasks
- Exposing and consuming tools through MCP
- Local LLMs and embeddings with Ollama
- Crew memory

---

## 🔮 Future Enhancements

- Wire `roll_dice` and `generate_password` to agents
- Accept the task from user input instead of hard-coding it in `tasks.py`
- Move the model name and Ollama URL into environment variables
- Tests for the MCP tools and the crew
- Web or chat interface

---

## 🤝 Contributing

1. Fork the repository
2. Create a new branch  
   git checkout -b feature/your-feature

3. Commit changes  
   git commit -m "feat(scope): your message"

4. Push changes  
   git push origin feature/your-feature

5. Open Pull Request

---

## 👨‍💻 Author

Abhishek Mishra  
GitHub: https://github.com/mishraabhishek11

---

## ⭐ Support

Give a ⭐ if you like this project!
