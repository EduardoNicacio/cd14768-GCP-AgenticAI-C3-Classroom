## Question 1

**Scenario:**  
Your agent has a tool `save_nickname` that sets
`state["user:nickname"] = "Spike"`.
A user chats with the agent and sets their nickname. They then close their
browser and return the next day to start a new chat session.

**Question:**  
What happens when the agent tries to read `state["user:nickname"]` in this new
session?

**Options:**

- **A)** It retrieves "Spike".
- **B)** It retrieves `None` because the session ended.
- **C)** It retrieves `None` unless the user explicitly reminds the agent of
  their name.
- **D)** It raises a `StateExpiredError`.

**Rationale:**

- **A)** Correct — The `user:` prefix indicates user-scoped state, which is
  designed to persist across multiple sessions for the same user identity.
- **B)** Wrong — This describes session-scoped state (no prefix).
- **C)** Wrong — Persistence is automatic for user-scoped keys; no user reminder
  is needed.
- **D)** Wrong — No such error exists in this context.

**Difficulty:** Easy  
**Cognitive Level:** Conceptual  
**Learning Objective:** Understand the difference between request-scoped,
session-scoped and user-scoped state in ADK.

## Question 2

**Scenario:**  
You want to implement a "Shopping Cart" agent.
Items added to the cart should disappear if the user closes the tab,
but their shipping address should be remembered for next time.

**Question:**  
How should you implement the `add_to_cart` and `save_address` tools?

**Options:**

- **A)** `cart` uses `user:cart`; `address` uses `address`.
- **B)** `cart` uses `cart`; `address` uses `user:address`.
- **C)** `cart` uses `temp:cart`; `address` uses `user:address`.
- **D)** `cart` uses `cart`; `address` uses `cart`.

**Rationale:**

- **A)** Wrong — This persists the cart forever (User scope) and loses the
  address (Session scope).
- **B)** Correct — `cart` (Session scope) allows the cart to be temporary.
  `user:address` (User scope) persists the address across sessions.
- **C)** Wrong — `temp:cart` would wipe the cart after every single user
  interaction, making it impossible to add more than one item.
- **D)** Wrong — Both session scoped; address would be lost.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Understand the difference between request-scoped,
session-scoped and user-scoped state in ADK.

## Question 3

**Scenario:**  
You are using `run_task` from the demo.
`current_iteration` is session-scoped.
The user chats for a while, reaching iteration 5.
Then, the server restarts (the Python process dies and restarts).

**Question:**  
Assuming the ADK uses in-memory storage, what happens to
`current_iteration`?

**Options:**

- **A)** It is preserved because session scope is always written to disk.
- **B)** It is lost (resets to 0 or None) because default session storage is
  in-memory.
- **C)** It is recovered from the LLM's context window.
- **D)** It is saved in the user's browser cookies.

**Rationale:**

- **A)** Wrong — Default is usually memory.
- **B)** Correct — Without an external persistence layer (like Redis/Memorystore
  or a database) configured, "Session" state in a basic Python app usually lives
  in RAM. Restarting the process wipes it. (Note: The user-scoped state might
  also be lost if that was only in-memory too, though the concept of "User"
  scope implies longer-term intent).
- **C)** Wrong — Context window is ephemeral to the conversation, not the state
  store.
- **D)** Wrong — Server-side state.

**Difficulty:** Advanced  
**Cognitive Level:** Evaluation  
**Learning Objective:** Understand the limitations of memory-based storage.

## Question 4

**Scenario:**  
Why do we "bootstrap" the prompt with `{user:num_iterations_instructions?}`
instead of just reading the value `user:num_iterations` directly in the prompt?

**Question:**  
Select the best design reason.

**Options:**

- **A)** The LLM cannot read numbers.
- **B)** We want to change the *instructions* (logic), not just provide data.
- **C)** State variables can only store strings, not integers.
- **D)** It saves token costs.

**Rationale:**

- **A)** Wrong.
- **B)** Correct — As seen in the code, the stored value is a block of text: *"
  But we do know... start asking questions"*. This overrides the default "Ask
  the user for a number" logic. Merely injecting "3" might not be enough to stop
  the agent from asking "How many do you want?" if the prompt logic isn't
  complex enough. Injecting *instructions* is a powerful pattern.
- **C)** Wrong — They can store anything.
- **D)** Wrong — It actually uses *more* tokens than just a number.

**Difficulty:** Advanced  
**Cognitive Level:** Analysis  
**Learning Objective:** Use state to control a multi-step conversation flow.


## Question 5

**Scenario:**  
Your agent has a tool `calculate_tax` that computes the tax for a specific
purchase request.
If you store the result in `tax_amount` (session scope) instead of
`temp:tax_amount` (request scope), what potential bug could occur?

**Question:**  
Select the most likely issue.

**Options:**

- **A)** The tax amount will be lost before the agent can answer the user.
- **B)** The tax calculation will take twice as long.
- **C)** If the user asks a follow-up question about a *different* item without
  triggering a new calculation, the agent might hallucinate or use the *old*
  `tax_amount` from the previous turn.
- **D)** Session scope is read-only, so the tool will crash.

**Rationale:**

- **A)** Wrong — Session scope persists longer than temp.
- **B)** Wrong.
- **C)** Correct — Session scope persists across the *entire conversation*.
  Intermediate calculation values that are specific to a single interaction (
  like the tax for one specific item) should be cleared (using `temp:`) so they
  don't pollute the context for subsequent unrelated queries.
- **D)** Wrong.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand the difference between request-scoped,
session-scoped and user-scoped state in ADK.

## Question 6

**Scenario:**  
An agent has two state variables set:

1. `state["theme"] = "blue"`
2. `state["user:theme"] = "red"`

**Question:**  
When the agent reads `state["theme"]`, what value does it get, and why?

**Options:**

- **A)** "red" — because User scope overrides Session scope.
- **B)** "blue" — because it looks for they key match in the Session scope.
- **C)** "purple" — because they mix.
- **D)** It raises an `AmbiguousKeyError`.

**Rationale:**

- **A)** Wrong — Scopes are namespaces, not an inheritance hierarchy where one
  automatically shadows the other without explicit logic.
- **B)** Correct — In ADK state, keys are literal. `theme` and `user:theme` are
  two completely different keys in the underlying storage. Accessing `theme`
  retrieves the session-scoped variable.
- **C)** Wrong.
- **D)** Wrong.

**Difficulty:** Medium  
**Cognitive Level:** Conceptual  
**Learning Objective:** Understand the difference between request-scoped,
session-scoped and user-scoped state in ADK.