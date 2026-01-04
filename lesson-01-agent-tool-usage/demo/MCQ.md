## Question 1

**Scenario:**  
You have defined a Python function `calculate_mortgage_rate(principal, years)`
and registered it as a tool with your ADK agent. However, when you ask the
agent "What is the mortgage rate for a $500k loan over 30 years?", the agent
hallucinates an answer instead of calling the tool.

**Question:**  
Based on best practices for tool definition, what is the most likely cause of
this failure?

**Options:**

- **A)** The function name is too long for the LLM to process.
- **B)** The function lacks a descriptive docstring explaining its purpose and
  parameters.
- **C)** The agent's model temperature is set too low.
- **D)** The function was not defined as an asynchronous `async def`.

**Rationale:**

- **A)** Wrong — LLMs can handle long function names; clarity is preferred.
- **B)** Correct — The LLM relies heavily on the docstring to understand the
  semantic purpose of a function and when to apply it to a user's query.
- **C)** Wrong — Low temperature usually makes the model more deterministic,
  which typically helps, not hinders, function calling.
- **D)** Wrong — The ADK supports synchronous functions; being async is not a
  strict requirement for recognition.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Learn ADK's `Tool` interface and requirements for
effective function calling.

## Question 2

**Scenario:**  
In the demo, the initial configuration used the `gemini-2.5-flash-lite` model.
During the walkthrough, the instructor observes that the agent fails to invoke
the weather tools consistently and decides to switch to `gemini-2.5-flash`.

**Question:**  
What is the primary technical reason for this change?

**Options:**

- **A)** The `flash-lite` model does not support Python 3.12 syntax.
- **B)** The `flash` model has a larger context window required for the weather
  data.
- **C)** The `flash` model generally exhibits higher reliability and reasoning
  capabilities for function calling compared to the `lite` version.
- **D)** The `flash-lite` model is strictly for text generation and cannot
  output JSON.

**Rationale:**

- **A)** Wrong — Models are language-agnostic regarding the Python version of
  the runtime.
- **B)** Wrong — The weather data is small; context window is not the bottleneck
  here.
- **C)** Correct — As noted in the troubleshooting steps, larger or more capable
  models often handle the logic of tool selection and argument generation more
  robustly.
- **D)** Wrong — `flash-lite` can output JSON, but may struggle with the
  specific logic of *when* to do so in complex agent loops.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Examine how different models perform to handle tools.

## Question 3

**Scenario:**  
Your agent has a tool `get_stock_price(ticker)` that returns `"150.00"`. You
ask "How is Apple doing?". The agent calls the tool with `ticker="AAPL"`,
receives `"150.00"`, and then responds to the user: *"150.00"*. You want the
agent to say *"Apple is currently trading at $150.00."*

**Question:**  
Where should you make the change to ensure a conversational response?

**Options:**

- **A)** Modify the `get_stock_price` tool to return a full sentence string.
- **B)** Update the `agent-prompt.txt` to instruct the agent to synthesize tool
  outputs into natural language.
- **C)** Hardcode a print statement in `agent.py` after the tool execution.
- **D)** Change the variable name `root_agent` to `conversational_agent`.

**Rationale:**

- **A)** Wrong — Tools should ideally return raw, structured data so they can be
  reused in different contexts; the agent should handle the phrasing.
- **B)** Correct — The system instruction (prompt) defines the persona and
  output style (e.g., "structured as if it was being said on the radio"),
  guiding how the LLM interprets tool results.
- **C)** Wrong — This bypasses the LLM's generation capabilities and breaks the
  agent abstraction.
- **D)** Wrong — The variable name in Python has no effect on the LLM's
  behavior.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Parse tool responses and integrate them into LLM
outputs.

## Question 4

**Scenario:**  
You are architecting a system where the agent needs to perform a secure database
deletion. The LLM generates the function call `delete_user(user_id=123)`.

**Question:**  
Which component is responsible for the actual execution of this database
deletion command?

**Options:**

- **A)** The Gemini Model itself directly connects to the database.
- **B)** The `agent-prompt.txt` file executes the command.
- **C)** The ADK runtime (Python interpreter) executes the function after
  receiving the call details from the LLM.
- **D)** The Google Cloud Secret Manager.

**Rationale:**

- **A)** Wrong — LLMs are text-in/text-out engines; they cannot "run" code or
  connect to networks directly.
- **B)** Wrong — This is a text file for instructions, not executable code.
- **C)** Correct — The LLM outputs the *intent* (function name and arguments),
  and the ADK framework runs the actual Python code locally.
- **D)** Wrong — Secret Manager stores keys but does not execute logic.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand the architectural separation between the
LLM's reasoning (Function Calling) and the runtime execution (Tool Usage).

## Question 5

**Scenario:**  
You want to add a new tool to your agent that calculates the square root of a
number. You define the function as:

```python
def calc(x):
  return x ** 0.5
```

You register it, but the agent rarely uses it correctly.

**Question:**  
Which modification would most improve the agent's ability to use this tool?

**Options:**

- **A)** Rename the function to `calculate_square_root` and add type hints and a
  docstring like `"""Calculates the square root of a number."""`.
- **B)** Move the function to a separate file named `math_utils.py`.
- **C)** Increase the `expected_replacements` parameter in the ADK config.
- **D)** Add a print statement inside the function: `print("Calculating...")`.

**Rationale:**

- **A)** Correct — Clear, semantic naming and type hints/docstrings provide the
  necessary metadata for the LLM to map a user's intent ("square root of 9") to
  the specific code capability.
- **B)** Wrong — File location is irrelevant to the LLM's context.
- **C)** Wrong — This parameter does not exist in the standard tool definition.
- **D)** Wrong — Side effects like printing do not help the LLM understand the
  tool's purpose.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Create and use a custom ADK `Tool`.

## Question 6

**Scenario:**  
A user asks: "What is the weather in London and Paris?" The agent has a
`get_weather(city)` tool.

**Question:**  
How does the ADK agent typically handle this multi-part request?

**Options:**

- **A)** It forces the user to ask one question at a time.
- **B)** It calls `get_weather` once with `city="London, Paris"`.
- **C)** The LLM can generate multiple function calls (sequentially or in
  parallel) for `London` and `Paris`, then synthesize both results.
- **D)** It fails because agents can only handle one tool call per turn.

**Rationale:**

- **A)** Wrong — Modern agents handle compound queries.
- **B)** Wrong — Unless the tool is explicitly designed to accept lists, the LLM
  will usually generate separate calls for separate entities.
- **C)** Correct — The agent loop allows for multiple tool executions to gather
  all necessary information before formulating the final answer.
- **D)** Wrong — Multiple tool calls are a standard feature of the ADK and
  Gemini.

**Difficulty:** Advanced  
**Cognitive Level:** Analysis  
**Learning Objective:** Loop through relevant tools to collect the answer.
