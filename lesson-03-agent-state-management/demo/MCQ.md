## Question 1

**Scenario:**  
Your agent performs a calculation that takes 5 steps. You store the intermediate
result in a variable named `temp:result`. After the agent returns the final
answer to the user, the user asks: "Can you double that result?"

**Question:**  
What value will the agent see in `temp:result` when processing this new request?

**Options:**

- **A)** The final value calculated in the previous turn.
- **B)** `None` (or the default value), because `temp:` variables are cleared
  after the response is sent.
- **C)** The value stored in the database.
- **D)** It depends on the LLM's context window.

**Rationale:**

- **A)** Wrong — Request-scoped state does not persist across user turns.
- **B)** Correct — The `temp:` prefix specifically designates request-scoped
  state in the ADK, which is ephemeral and lasts only for the duration of a
  single request execution loop.
- **C)** Wrong — `temp:` state is typically in-memory and transient, not
  database-persisted for sessions.
- **D)** Wrong — State storage is independent of the LLM's token context limits.

**Difficulty:** Medium  
**Cognitive Level:** Conceptual  
**Learning Objective:** Understand the concept of "temp:" state for
request-scoped variables.

## Question 2

**Scenario:**  
You are building a state machine for an order processing agent. The states are
`INIT`, `VALIDATING`, `PAYING`, `SHIPPED`.
The tool code is:

```python
current = ctx.state.get("temp:state", "INIT")
if current == "INIT":
  ctx.state["temp:state"] = "VALIDATING"
```

The user asks "Check my order status". The agent calls the tool.

**Question:**  
If the prompt contains `Current State: {temp:state?}`, what will the LLM see in
the prompt *immediately after* this tool execution?

**Options:**

- **A)** Current State: INIT
- **B)** Current State: VALIDATING
- **C)** Current State: SHIPPED
- **D)** The prompt will not change until the next user turn.

**Rationale:**

- **A)** Wrong — This was the state *before* execution.
- **B)** Correct — The execution loop runs the tool, the tool updates the state,
  and then the ADK re-generates the prompt with the *new* state value before
  passing control back to the LLM for the next step.
- **C)** Wrong — Logic only advances it to VALIDATING
- **D)** Wrong — Agents are iterative; the prompt updates within the same
  request loop.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Observe how the agent's prompt updates dynamically based
on state changes

## Question 3

**Scenario:**  
Why is using a state machine with explicit variables preferred over relying
solely on conversation history for tracking a multi-step process?

**Question:**  
Select the primary reason related to LLM limitations.

**Options:**

- **A)** State machines save money on API costs.
- **B)** Conversation history is limited to 10 messages.
- **C)** LLMs struggle with maintaining precise tracking of changing
  attributes (like counters or aggregated state) over long contexts, leading to
  hallucinations.
- **D)** State machines are faster than LLMs.

**Rationale:**

- **A)** Wrong — Not the primary driver; sometimes state machines add tokens.
- **B)** Wrong — Context windows are much larger now.
- **C)** Correct — Remember that LLMs are not good at counting and 
  that the conversation history contains all past states. These can
  confuse the model about the *current* truth. Explicit state injection provides
  a single, unambiguous source of truth.
- **D)** Wrong — While state machines might be faster, speed is not the main 
  consideration; reliability is.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Difference between stateless and stateful systems.

## Question 4

**Scenario:**  
An agent's tool tries to access `tool_context.state["temp:stage"]` but crashes
with a `KeyError`.

**Question:**  
How should the code be modified to prevent this crash when the state hasn't been
set yet?

**Options:**

- **A)** Wrap the entire tool in a `try...except` block that returns "Error".
- **B)** Use `tool_context.state.get("temp:stage", "DEFAULT_VALUE")` to provide
  a fallback.
- **C)** Hardcode the state to "START" at the top of the file.
- **D)** Require the user to type "Start" to initialize the variable.

**Rationale:**

- **A)** Wrong — Too broad; doesn't fix the logic.
- **B)** Correct — Standard Python dictionary practice, essential for
  initializing state variables lazily on the first run.
- **C)** Wrong — Global variables don't work for per-request state.
- **D)** Wrong — Poor UX.

**Difficulty:** Easy  
**Cognitive Level:** Application  
**Learning Objective:** Read and write state within a tool using `ToolContext`.

## Question 5

**Scenario:**  
In the demo, the `agent-prompt.txt` includes `{temp:stage?}`. What does the `?`
symbol signify to the ADK?

**Question:**  
Select the correct function of this syntax.

**Options:**

- **A)** It asks the LLM to guess the value.
- **B)** It marks the variable as a secret password.
- **C)** It tells the ADK to ignore the variable if it doesn't exist (render as
  empty string), preventing a crash.
- **D)** It forces the variable to be boolean.

**Rationale:**

- **A)** Wrong.
- **B)** Wrong.
- **C)** Correct — Standard template syntax for optional injection. Without it,
  a missing key might raise an error or print a placeholder literal.
- **D)** Wrong.

**Difficulty:** Medium  
**Cognitive Level:** Conceptual  
**Learning Objective:** Understand dynamic prompt injection syntax.

## Question 6

**Scenario:**  
Your tool successfully updates `temp:stage` to "END", but instead of replying to
the user, the agent calls the tool again with `command="SHOW"`, causing an
infinite loop or unnecessary extra step.

**Question:**  
What is the most likely missing component in your `agent-prompt.txt`?

**Options:**

- **A)** You forgot to remove the `{temp:stage?}` placeholder.
- **B)** You missed an explicit instruction telling the agent what to do when
  the state reaches "END".
- **C)** The "END" string must be uppercase for the LLM to recognize it.
- **D)** You need to manually call `sys.exit()` in your Python tool.

**Rationale:**

- **A)** Wrong — The placeholder is needed to show the current state.
- **B)** Correct — The LLM follows instructions. Even if the state is "END", if
  the prompt doesn't say "Stop when END", the LLM might think "I need to do
  something else" or "I should double-check". Explicit termination conditions
  are crucial for state machines driven by LLMs.
- **C)** Wrong — LLMs are generally case-insensitive regarding semantic meaning,
  though consistency helps.
- **D)** Wrong — Tools should never exit the process; they just return data.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Implement a simple state machine using ADK tools.

## Question 7

**Scenario:**  
User A and User B are both interacting with your "Color Mixer" agent at the
exact same time. User A issues a command that sets their `temp:stage` to "RED".
User B then immediately sends their first message "Start mixing".

**Question:**  
What value for `temp:stage` will be injected into the prompt for User B's
request?

**Options:**

- **A)** "RED" — The state is shared globally across the agent instance.
- **B)** "BLACK" (or None) — `temp:` state is isolated to the specific
  request/user context.
- **C)** "RED" — But only if they are on the same WiFi network.
- **D)** It alternates between "RED" and "BLACK" depending on the thread
  scheduler.

**Rationale:**

- **A)** Wrong — This would be a major race condition and security flaw.
- **B)** Correct — `temp:` state is request-scoped (and by extension,
  thread/context-safe). One user's temporary calculation state does not leak
  into another user's request.
- **C)** Wrong — Network has no bearing on internal memory scope.
- **D)** Wrong — The ADK handles context isolation.

**Difficulty:** Medium  
**Cognitive Level:** Conceptual  
**Learning Objective:** Understand the concept of "temp:" state for
request-scoped variables.