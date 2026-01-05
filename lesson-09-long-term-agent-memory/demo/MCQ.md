## Question 1

**Scenario:**  
You are designing an agent that needs to store and recall simple, structured
user preferences (e.g., "The user's favorite color is blue" or "The user prefers
window seats"). You are deciding between using ADK's **User State** and **Vertex
AI Agent Engine Memory Bank**.

**Question:**  
Why would **User State** be the more appropriate choice for this specific
requirement?

**Options:**

- **A)** User State is persistent across sessions and designed for simple
  key-value pairs, making it efficient for direct preference storage.
- **B)** Agent Engine Memory Bank cannot store any structured data at all.
- **C)** User State is automatically summarized by an LLM, ensuring the "gist"
  of the preference is captured.
- **D)** User State data is shared globally across all users of the application.

**Rationale:**

- **A)** Correct — User State is optimized for explicit, structured attributes (
  key/value) tied to a user identity. Memory Bank is better for fuzzy, semantic
  recall of unstructured conversation history.
- **B)** Wrong — Memory Bank stores facts which can be structured-like, but it's
  not its primary optimization vs. a simple KV store.
- **C)** Wrong — Summarization is a feature of Memory Bank, not User State (
  which is exact).
- **D)** Wrong — User State is scoped to the specific user.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand the difference between Session State, User
State, and Long-Term Memory.

## Question 2

**Scenario:**  
You have implemented the `memory_agent` as shown in the demo. However, when you
run the agent, you notice that while the agent can recall information from
*previous* sessions, it seems to have no memory of what you just said 5 minutes
ago in the *current* active session.

**Question:**  
What is the most likely architectural explanation for this behavior?

**Options:**

- **A)** The `preload_memory_tool` only loads memories from the database (past
  sessions); current session context is managed separately by the LLM's context
  window (history).
- **B)** The `auto_save_session_to_memory_callback` is overwriting the current
  session data with old data.
- **C)** The Agent Engine Memory Bank takes 24 hours to index new information.
- **D)** You forgot to add `preload_memory_tool` to the `tools` list.

**Rationale:**

- **A)** Correct — Long-term memory (Memory Bank) bridges *gaps* between
  sessions. "Short-term" memory within a session is typically handled by passing
  the conversation history (turns) to the LLM. If the agent "forgets" recent
  turns, it's usually a context window or history management issue, not a Memory
  Bank issue (which focuses on *past* sessions).
- **B)** Wrong — The callback *adds* to memory; it doesn't destructively
  overwrite the live session state.
- **C)** Wrong — Indexing is near real-time for this purpose.
- **D)** Wrong — If missing, it wouldn't recall *past* sessions either.

**Difficulty:** Advanced  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand how to maintain context over extended
sessions balancing performance and context length.

## Question 3

**Scenario:**  
You are reviewing the `agent.py` code. You see the following line:

```python
tools = [preload_memory_tool]
```

**Question:**  
How does the `preload_memory_tool` differ in execution from a standard tool like
`google_search` or a calculator?

**Options:**

- **A)** The LLM decides when to call it based on user intent.
- **B)** It is a "system tool" that the ADK framework invokes automatically at
  the start of a turn to inject context, rather than being called by the LLM
  during generation.
- **C)** It requires the user to type "/load_memory" to function.
- **D)** It runs locally on the user's device to save bandwidth.

**Rationale:**

- **A)** Wrong — Standard tools work this way. Memory preloading happens
  *before* the LLM even sees the prompt.
- **B)** Correct — This tool works in the background (middleware layer). Its
  purpose is to fetch relevant summaries and append them to the system prompt so
  the LLM has the context *before* it generates a response.
- **C)** Wrong — It is transparent to the user.
- **D)** Wrong — It interacts with the backend Memory Service.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Learn how to maintain context over extended sessions
balancing performance and context length.

## Question 4

**Scenario:**  
You have successfully deployed your agent. A user has a 2-hour long conversation
about their entire life history.

**Question:**  
How does the Vertex AI Agent Engine Memory Bank handle this large volume of
information to ensure it fits into the context window of future sessions?

**Options:**

- **A)** It stores every single word verbatim and truncates the oldest messages
  when the limit is reached.
- **B)** It uses an LLM to generate concise summaries ("facts") of the
  conversation and stores those semantic summaries instead of the raw logs.
- **C)** It compresses the text using GZIP before storing it.
- **D)** It refuses to save the session if it exceeds 4000 tokens.

**Rationale:**

- **A)** Wrong — This is a naive "sliding window" approach, not what Memory Bank
  does.
- **B)** Correct — The core value of Memory Bank is intelligent summarization.
  It distills "I played tennis for 3 hours and won..." into "User enjoys
  tennis." This compact summary allows long-term retention without blowing up
  token budgets.
- **C)** Wrong — Compression doesn't help the LLM understand it; it needs text.
- **D)** Wrong — It summarizes rather than rejects.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand why we wish to update, prune, and manage an
agent’s long-term memory.

## Question 5

**Scenario:**  
You are running the agent locally using `adk web`. You receive an error:
`ValueError: memory_service_uri must be provided`.

**Question:**  
Where should you have provided this URI, and what does it represent?

**Options:**

- **A)** In the `agent.py` file as a hardcoded string; it represents the Google
  Cloud Storage bucket URL.
- **B)** As a command-line argument (`--memory_service_uri`) when starting
  `adk web`; it represents the resource name of the Agent Engine instance.
- **C)** In the `requirements.txt` file; it represents the library version.
- **D)** In the `agent-prompt.txt`; it tells the agent where to look for
  memories.

**Rationale:**

- **A)** Wrong — Hardcoding config is bad practice, and the lesson explicitly
  uses the CLI arg.
- **B)** Correct — The lesson instructions highlight:
  `adk web --memory_service_uri agentengine://...`. This decouples the
  infrastructure config from the code.
- **C)** Wrong.
- **D)** Wrong.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Learn how to maintain context over extended sessions
balancing performance and context length.

## Question 6

**Scenario:**  
You want to modify the `auto_save_session_to_memory_callback` to only save the
session if the user explicitly says "Save this conversation."

**Question:**  
Which object passed to the callback function would you inspect to access the
user's last message?

**Options:**

- **A)** `callback_context._invocation_context.session.history`
- **B)** `callback_context.memory_service`
- **C)** `os.environ`
- **D)** `root_agent.instruction`

**Rationale:**

- **A)** Correct — The `CallbackContext` provides access to the
  `InvocationContext`, which holds the `session` object. The `session` contains
  the conversation `history` (messages), which you can inspect to implement
  custom logic before deciding to call `add_session_to_memory`.
- **B)** Wrong — This is the service for saving, not the data source.
- **C)** Wrong.
- **D)** Wrong — This is the static system prompt.

**Difficulty:** Advanced  
**Cognitive Level:** Application  
**Learning Objective:** Understand why we wish to update, prune, and manage an
agent’s long-term memory.

## Question 7

**Scenario:**  
Your organization is concerned about "Memory Poisoning" — where a user tells the
agent false facts (e.g., "The sky is green") and the agent remembers this
forever, corrupting future interactions.

**Question:**  
How can you manually intervene to correct such false memories in the Vertex AI
Agent Engine?

**Options:**

- **A)** You cannot; once written, memory is immutable blockchain storage.
- **B)** You must delete the entire Agent Engine instance and recreate it.
- **C)** You can navigate to the "Memories" tab in the Google Cloud Console,
  view the stored facts for that user, and edit or delete the specific incorrect
  fact.
- **D)** You must write a Python script to overwrite the memory with a blank
  session.

**Rationale:**

- **A)** Wrong.
- **B)** Wrong — Overkill.
- **C)** Correct — The lesson "Viewing Memory in Google Cloud Console" section
  explicitly mentions the ability to "Edit" facts via the three-dot menu. This
  is a crucial administrative feature for maintaining memory quality.
- **D)** Wrong — While possible programmatically, the Console provides a direct
  UI for this.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Understand why we wish to update, prune, and manage an
agent’s long-term memory.

## Question 8

**Scenario:**  
In the `agent.py` code, the callback is registered as:
`after_agent_callback=auto_save_session_to_memory_callback`.

**Question:**  
What happens if you remove this line?

**Options:**

- **A)** The agent stops working entirely and raises an error on startup.
- **B)** The agent works, but it will no longer persist new conversations to the
  Memory Bank (making it effectively stateless across sessions).
- **C)** The agent continues to save memory because the `preload_memory_tool`
  handles both reading and writing.
- **D)** The agent reverts to using a local file `memory.json` for storage.

**Rationale:**

- **A)** Wrong — The parameter is optional.
- **B)** Correct — The `after_agent_callback` is the *trigger* for the save
  action. Without it, the function `auto_save...` is never called, so nothing
  gets written to the Memory Bank. The `preload` tool only reads.
- **C)** Wrong — Tools generally perform specific actions when invoked; the
  preload tool is for loading context, not saving session state.
- **D)** Wrong — ADK doesn't default to local file storage for this service.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand why we wish to update, prune, and manage an
agent’s long-term memory.
