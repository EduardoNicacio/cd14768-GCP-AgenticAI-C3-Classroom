## Question 1

**Scenario:**  
You have deployed your ADK agent to a production environment. You are using
Google Cloud Trace to monitor its performance. You notice that a specific trace
shows a very long "span" for an `execute_tool` operation, but the subsequent
`generate_content` call to the LLM is fast.

**Question:**  
What does this specific trace pattern indicate about your agent's performance
bottleneck?

**Options:**

- **A)** The Gemini model is taking too long to process the prompt.
- **B)** The external tool (API or function) being called is slow/latent, and
  optimization should focus there.
- **C)** The ADK framework is introducing overhead in serializing the response.
- **D)** The user's internet connection is slow.

**Rationale:**

- **A)** Wrong — The trace shows the `generate_content` (LLM call) is fast.
- **B)** Correct — In a distributed trace, if the span for `execute_tool` is
  long, it directly points to the execution time of that tool (e.g., a slow
  database query or API call) as the bottleneck.
- **C)** Wrong — Framework overhead is typically negligible compared to I/O
  operations.
- **D)** Wrong — Server-side traces don't measure client-side network latency in
  this specific span.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Visualize and analyze agent activity in the Google Cloud
Log Explorer.

## Question 2

**Scenario:**  
You are configuring your agent for a production deployment in a highly regulated
industry (e.g., healthcare). You need to enable observability to track errors,
but you are strictly forbidden from logging any PII (Personally Identifiable
Information) such as patient names or medical queries.

**Question:**  
Which environment variable configuration represents the correct balance of
observability and compliance for this scenario?

**Options:**

- **A)** `OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT=true`
- **B)** `OTEL_INSTRUMENTATION_GENAI_CAPTURE_MESSAGE_CONTENT=false`
- **C)** `OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED=false`
- **D)** `OTEL_SERVICE_NAME=healthcare-agent-secure`

**Rationale:**

- **A)** Wrong — This captures the full text of prompts and responses, which
  would log the PII.
- **B)** Correct — Setting this to `false` (or omitting it/defaulting to false)
  ensures that while you still get traces of *what* happened (e.g., "LLM
  called", "Tool executed", "Error occurred"), you do not log the *content* (the
  actual sensitive text) of the interaction.
- **C)** Wrong — Disabling logging entirely hurts observability for
  non-sensitive system errors.
- **D)** Wrong — Service name doesn't control content capture.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Configure OpenTelemetry environment variables for ADK.

## Question 3

**Scenario:**  
You run the command `adk web` in your terminal to test your agent. You verify
that `OTEL_SERVICE_NAME` is set in your `.env` file. However, when you check the
Google Cloud Trace console, no traces are appearing even after waiting 10
minutes.

**Question:**  
What is the most likely missing step in your execution command?

**Options:**

- **A)** You forgot to set `GOOGLE_CLOUD_PROJECT`.
- **B)** You did not include the `--otel_to_cloud` flag when running `adk web`.
- **C)** The Google Cloud Trace API is down.
- **D)** Your internet connection is unstable.

**Rationale:**

- **A)** Wrong — ADK usually errors out earlier if the project is missing.
- **B)** Correct — As emphasized in the lesson, ADK defaults to local/no-op
  telemetry for development speed. You must explicitly pass `--otel_to_cloud` to
  initialize the OpenTelemetry exporter that sends data to Google Cloud.
- **C)** Wrong — Highly unlikely.
- **D)** Wrong — Unlikely to be the primary cause if other things work.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Run the agent with telemetry enabled using
`adk web --otel_to_cloud`.

## Question 4

**Scenario:**  
In the "Trace Explorer" view of Google Cloud Console, you see a hierarchical
visualization of an agent interaction. At the top level is the agent invocation.
Nested underneath are "generate_content" calls and "execute_tool" calls.

**Question:**  
What architectural concept does this nested visualization represent?

**Options:**

- **A)** The recursive nature of Python functions.
- **B)** A "Distributed Trace," showing the causal relationship and timing of
  operations across the system components.
- **C)** A standard log file viewer.
- **D)** The memory stack of the underlying server.

**Rationale:**

- **A)** Wrong — It captures more than just Python functions (e.g., network
  calls).
- **B)** Correct — This is the definition of distributed tracing. It shows how a
  single user request ("trace") flows through various services and operations ("
  spans"), allowing you to see that Operation A caused Operation B.
- **C)** Wrong — Logs are linear/flat lists of events.
- **D)** Wrong.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Visualize and analyze agent activity in the Google Cloud
Log Explorer.

## Question 5

**Scenario:**  
You have two different agents, "SalesBot" and "SupportBot", running in the same
Google Cloud Project. You want to easily filter their traces separately in the
Cloud Console.

**Question:**  
Which OpenTelemetry configuration variable should be unique for each agent
deployment to achieve this?

**Options:**

- **A)** `GOOGLE_CLOUD_PROJECT`
- **B)** `OTEL_SERVICE_NAME`
- **C)** `OTEL_TRACES_EXPORTER`
- **D)** `OTEL_PYTHON_LOGGING_AUTO_INSTRUMENTATION_ENABLED`

**Rationale:**

- **A)** Wrong — They share the project.
- **B)** Correct — `OTEL_SERVICE_NAME` is the standard attribute for identifying
  the distinct application or service generating the telemetry. You would set it
  to `sales-bot` and `support-bot` respectively.
- **C)** Wrong — This defines *how* (protocol) traces are sent, not *who* sent
  them.
- **D)** Wrong — This is a global toggle.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Configure OpenTelemetry environment variables for ADK.

## Question 6

**Scenario:**  
You are debugging an issue where the agent is hallucinating answers. You want to
see exactly what "System Instructions" were sent to the LLM to verify if the
prompt logic is correct.

**Question:**  
Using the Google Cloud Log Explorer technique demonstrated in the lesson, how
can you find the specific log entry containing the system prompt?

**Options:**

- **A)** Search for logs with `severity="ERROR"`.
- **B)** Search for logs where `jsonPayload.role` equals `"user"`, as the system
  prompt is often bundled or associated with the user interaction context in the
  logs.
- **C)** Download the entire database of the agent.
- **D)** It is impossible; system instructions are never logged.

**Rationale:**

- **A)** Wrong — Prompts are info/debug level, not errors.
- **B)** Correct — The lesson demonstrates filtering by `role: "user"` (and
  potentially unhiding fields) to see the full interaction payload, which
  includes the list of messages sent to the model (system instructions, user
  query, tool outputs).
- **C)** Wrong.
- **D)** Wrong — If content capture is enabled, they are logged.

**Difficulty:** Advanced  
**Cognitive Level:** Application  
**Learning Objective:** Visualize and analyze agent activity in the Google Cloud
Log Explorer.

## Question 7

**Scenario:**  
Why is "Observability" described as a critical step for *evaluation* in this
lesson, rather than just using a static test set?

**Question:**  
Which of the following insights can **only** be gained through observability of
real-world traffic?

**Options:**

- **A)** Whether the code compiles without syntax errors.
- **B)** Whether the LLM can answer "What is 2+2?".
- **C)** Understanding the unexpected phrasing, intent, or "long tail" queries
  that actual users are typing, which may differ from your test cases.
- **D)** Checking if the Google Cloud Project billing is enabled.

**Rationale:**

- **A)** Wrong — CI/CD checks this.
- **B)** Wrong — Unit tests check this.
- **C)** Correct — Static tests only check what you *expect*. Observability
  reveals what you *didn't expect* (e.g., users asking about competitors, using
  slang, or trying to break the agent), which is essential for true evaluation
  and improvement.
- **D)** Wrong.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Visualize and analyze agent activity in the Google Cloud
Log Explorer.

## Question 8

**Scenario:**  
You are using `demo-2` which involves nested agents (`root_agent` ->
`search_agent_tool` -> `search_agent`). You notice that in the `adk web` UI, you
only see the high-level tool call to `search_agent_tool` and then a long pause
before the result.

**Question:**  
Why does the `adk web` UI not show the internal steps of the `search_agent` (
like its own LLM calls or Google Search)?

**Options:**

- **A)** `adk web` is designed to show the perspective of the *current* agent
  being tested; it treats the sub-agent as a "black box" tool.
- **B)** The `search_agent` crashed silently.
- **C)** OpenTelemetry does not support nested spans.
- **D)** You need to run `adk web` twice in parallel.

**Rationale:**

- **A)** Correct — The `adk web` UI visualizes the execution flow of the agent
  you connected to. Since `search_agent` is wrapped as a tool (
  `search_agent_tool`), the root agent just sees it as a function call. To see
  the internals, you must use the Trace Explorer (as shown in the lesson) which
  captures the full distributed trace across all components.
- **B)** Wrong — It returned a result eventually.
- **C)** Wrong — OTEL excels at nested spans; the limitation here is the
  specific view of the local testing UI vs. the full backend trace.
- **D)** Wrong.

**Difficulty:** Advanced  
**Cognitive Level:** Analysis  
**Learning Objective:** Visualize and analyze agent activity in the Google Cloud
Log Explorer.
