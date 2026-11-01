# Project 14 — LangGraph Stateful Agent

A minimal command-line chatbot built with **LangGraph** that keeps conversation history across turns. Each user message is passed through a single-node graph that calls an LLM with the full message history, so the model always sees the whole conversation so far. When you exit, the full transcript is saved to `logging.txt`.

This project is a focused demonstration of the core LangGraph pattern: a typed state, a node that updates it, and a compiled graph invoked in a loop.

---

## How It Works

```
        ┌───────┐      ┌───────────┐      ┌─────┐
        │ START │ ───► │  process  │ ───► │ END │
        └───────┘      └───────────┘      └─────┘
                             │
                  llm.invoke(state["messages"])
                  append AIMessage to state
```

1. **State** — `AgentState` is a `TypedDict` with one field, `messages`, a list of `HumanMessage` / `AIMessage` objects.
2. **Node** — `process` sends the full `messages` list to the LLM, appends the reply as an `AIMessage`, prints it, and returns the updated state.
3. **Graph** — `START → process → END`, compiled with `StateGraph(...).compile()`.
4. **Loop** — `main.py` keeps a `conversation_history` list outside the graph. On every turn it appends the new `HumanMessage`, invokes the agent with the history, and stores the returned messages for the next turn. This is what makes the agent "stateful".
5. **Exit** — typing `exit` ends the loop and writes the conversation to `logging.txt`.

## Features

- Multi-turn conversation with full context on every call
- Typed LangGraph state (`TypedDict`)
- OpenAI-compatible LLM client (`ChatOpenAI`) pointed at a custom base URL
- Conversation logging to a text file on exit
- API key loaded from a `.env` file (not committed)

## Tech Stack

| Component | Tool |
|---|---|
| Orchestration | LangGraph (`StateGraph`) |
| LLM client | `langchain-openai` (`ChatOpenAI`) |
| Message types | `langchain-core` |
| Config / secrets | `python-dotenv` |
| Language | Python 3 |

## Project Structure

```
Project-14-LangGraph_Stateful_Agent/
├── main.py         # Agent graph + chat loop + logging
├── config.toml     # Model/settings file (see note below)
├── logging.txt     # Sample conversation log (overwritten on each run)
├── .gitignore      # Ignores .env, lang-practise/, *.ipynb
└── README.md
```

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/satyam-sharma7/Project-14-LangGraph_Stateful_Agent.git
cd Project-14-LangGraph_Stateful_Agent
```

### 2. Install dependencies

There is no `requirements.txt` yet, so install the packages directly:

```bash
pip install langgraph langchain-openai langchain-core python-dotenv
```

### 3. Set your API key

Create a `.env` file in the project root:

```env
FREE_LLM_API=your_api_key_here
```

### 4. Run

```bash
python main.py
```

## Usage

```
Enter: hi
AI: Hi there! How can I help you today?
Enter: can you code? yes or No?
AI: Yes.
Enter: exit
Conversation saved to logging.txt
```

Type `exit` (exact match, lowercase) to quit. The transcript is written to `logging.txt` in this format:

```
Your Conversation Log:
You: <message>
AI: <reply>

End of Conversation
```

> Note: `logging.txt` is **overwritten** on every run, and it is only written on a clean `exit`. Stopping with Ctrl+C loses the session.

## Configuration

The LLM is configured directly in `main.py`:

| Setting | Value |
|---|---|
| `base_url` | `https://freellmapi-seyc.onrender.com/v1` |
| `model` | `auto` |
| `api_key` | read from the `FREE_LLM_API` environment variable |

`config.toml` also defines `MAX_LOOP`, `MAX_CHARS`, and a `[models.freellmapi]` block (temperature `0.3`, max tokens `10000`), but **`main.py` does not read this file yet**. It is a placeholder for moving the hardcoded settings out of the code.

## Limitations

- **Memory is in-process only.** History lives in a Python list and is lost when the program ends. There is no checkpointer or database, so a new run starts with an empty conversation.
- **Unbounded history.** The full history is sent on every call, with no trimming or summarization, so long chats will eventually hit the model's context limit.
- **Single node, no tools.** The graph is a plain chat pass-through: no routing, tool calling, or branching.
- **State is mutated in place** inside `process` rather than returning a new state.
- **No error handling** for API failures or empty input.

## Possible Next Steps

- Use LangGraph's built-in `add_messages` reducer and `MessagesState` instead of manually managing the list
- Add persistence with a checkpointer (SQLite/Postgres) and `thread_id`s for resumable sessions
- Load settings from `config.toml` and enforce `MAX_CHARS` / `MAX_LOOP`
- Add history trimming or summarization
- Add tool nodes and conditional edges to turn this into a ReAct-style agent
- Add a `requirements.txt` and error handling

## Author

**Satyam Sharma** — [GitHub](https://github.com/satyam-sharma7) · [LinkedIn](https://www.linkedin.com/in/satyam-sharma-778383379/)
