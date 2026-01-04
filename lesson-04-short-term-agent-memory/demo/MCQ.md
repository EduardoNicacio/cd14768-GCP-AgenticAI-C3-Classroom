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
A developer mistakenly uses `user:current_step` to track progress in a
multi-step wizard.
A user gets stuck on Step 3 and rage-quits (closes the browser).
They come back the next day to try again.

**Question:**  
What happens, and why is this likely a poor user experience?

**Options:**

- **A)** The wizard starts at Step 1, which is fine.
- **B)** The wizard starts at Step 3 immediately, potentially without context or
  the necessary previous data, confusing the user.
- **C)** The agent refuses to talk to the user.
- **D)** The database crashes because of a stale lock.

**Rationale:**

- **A)** Wrong — That would happen if it were session-scoped.
- **B)** Correct — Because `user:` state persists, the "current_step" is still
  3. The agent might immediately ask "What is your credit card?" (Step 3)
  without introducing itself or confirming the item (Steps 1 & 2), which feels
  broken.
- **C)** Wrong.
- **D)** Wrong.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Observe how persistent user preferences affect agent
behavior across multiple sessions.

## Question 4

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

## Question 5

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
