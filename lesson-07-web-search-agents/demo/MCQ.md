## Question 1

**Scenario:**  
You are building a customer support agent for a company that frequently updates
its product pricing and policies. You decide to integrate "Grounding with Google
Search" to help the agent answer questions.

**Question:**  
What is the primary technical mechanism by which "Grounding with Google Search"
improves the agent's reliability compared to a standard, non-grounded LLM?

**Options:**

- **A)** It retrains the model in real-time using the search results as new
  training data.
- **B)** It forces the model to generate a response based strictly on the
  retrieved search snippets, reducing the likelihood of hallucination.
- **C)** It uses a separate "fact-checking" model to verify the LLM's output
  against Wikipedia.
- **D)** It increases the model's temperature setting to generate more creative
  answers.

**Rationale:**

- **A)** Wrong — Grounding is a retrieval-time inference technique, not a
  training technique.
- **B)** Correct — Grounding connects the model's generation to verifiable
  sources retrieved at runtime. The model is instructed (implicitly or
  explicitly via the API) to synthesize the answer from the retrieved context.
- **C)** Wrong — While fact-checking models exist, Google Search Grounding
  integrates the retrieval directly into the generation process of the main
  model.
- **D)** Wrong — Higher temperature increases randomness and hallucinations,
  which is the opposite of the goal.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand the concept of "Grounding" in LLMs.

## Question 2

**Scenario:**  
You have implemented the `web_search_agent` from the demo. A user asks, "What is
the capital of France?" The agent responds correctly ("Paris") but does *not*
trigger a web search.

**Question:**  
Why did the agent choose not to use the `google_search` tool for this specific
query?

**Options:**

- **A)** The ADK `google_search` tool is broken and needs to be re-initialized.
- **B)** The `agent-prompt.txt` explicitly forbids searching for geography
  questions.
- **C)** The model's internal confidence for this common fact was high enough
  that it determined external retrieval was unnecessary.
- **D)** The Google Search API was down at the time of the request.

**Rationale:**

- **A)** Wrong — The tool works; the decision not to use it is key.
- **B)** Wrong — The prompt encourages search for uncertain facts, but doesn't
  forbid geography.
- **C)** Correct — Modern models are trained to be efficient. For well-known,
  static facts present in their training data (like capitals), they typically
  skip the overhead of a tool call unless explicitly forced.
- **D)** Wrong — While possible, it is more likely that the LLM will return 
  an error in this case.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Observe how the model automatically decides when to
search the web.

## Question 3

**Scenario:**  
A user asks the agent: "Who won the latest Super Bowl and what was the score?"
The agent searches Google and returns an answer. You inspect the response object
in the ADK (or the web UI).

**Question:**  
Besides the text answer, what critical piece of "Grounding" data does the
response object contain that is essential for user trust?

**Options:**

- **A)** The raw HTML of the top 10 search result pages.
- **B)** A list of "Grounding Chunks" or citations that link specific parts of
  the answer to source URLs.
- **C)** The IP address of the server that processed the request.
- **D)** The search query string that the user typed.

**Rationale:**

- **A)** Wrong — You get snippets/summary data, not raw HTML pages (which would
  be huge).
- **B)** Correct — The API returns metadata linking the generated text to the
  source. This allows the UI to render footnotes or "Learn more" links, which is
  the core value proposition of grounding (verification).
- **C)** Wrong — Irrelevant implementation detail.
- **D)** Wrong — The response contains the *model's generated* search query, but
  the citations are the "trust" element.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Interpret the grounded responses, including citations
and search metadata.

## Question 4

**Scenario:**  
You are reviewing the `agent.py` code provided in the demo.

```python
root_agent = Agent(
  # ...
  tools=[google_search],
)
```

**Question:**  
Where is the logic for the `google_search` tool defined?

**Options:**

- **A)** It is defined in a hidden `tools.yaml` file inside the `google-adk`
  package.
- **B)** It is a client-side web scraper implemented in Python within the ADK.
- **C)** It is a native capability of the Vertex AI platform; the ADK object is
  just a flag/configuration to enable this feature on the API call.
- **D)** It is a hardcoded set of responses for testing purposes.

**Rationale:**

- **A)** Wrong — It's not a local YAML definition.
- **B)** Wrong — It's not a local scraper (which would be slow and brittle).
- **C)** Correct — "Grounding with Google Search" is a feature of the model
  serving infrastructure (Vertex AI). The `google_search` object in Python tells
  the ADK to set the appropriate `tools` configuration in the API request
  payload, instructing Vertex to handle the search.
- **D)** Wrong — It performs real searches.

**Difficulty:** Advanced  
**Cognitive Level:** Analysis  
**Learning Objective:** Integrate the `google_search` tool into an ADK agent.

## Question 5

**Scenario:**  
Your agent is deployed in a production environment. Users are complaining that
for some queries, the agent provides outdated information despite
`google_search` being enabled.

**Question:**  
Which of the following prompt engineering strategies would most effectively 
address this issue?

**Options:**

- **A)** "Always answer in JSON format."
- **B)** "Use web search for current events, recent news, or facts you're
  uncertain about."
- **C)** "Never admit that you don't know the answer."
- **D)** "Keep your answers short and concise."

**Rationale:**

- **A)** Wrong — Formatting doesn't affect information freshness.
- **B)** Correct — Explicitly instructing the model *when* to use the tool helps
  override its default tendency to rely on internal weights. This prompt acts as
  a "trigger" for the tool usage policy.
- **C)** Wrong — This leads to hallucinations.
- **D)** Wrong — Length doesn't affect freshness.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Observe how the model automatically decides when to
search the web.

## Question 6

**Scenario:**  
You are explaining the limitations of "Grounding with Google Search" to a
stakeholder. They ask: "Does this mean the agent can read my private Google Docs
or emails if I ask it to?"

**Question:**  
What is the correct technical answer?

**Options:**

- **A)** "Yes, Google Search has access to everything."
- **B)** "No, 'Grounding with Google Search' only searches the public web. To
  search private data, we would need to implement a RAG (Retrieval-Augmented
  Generation) system with a different data source."
- **C)** "Yes, but only if you provide your password in the prompt."
- **D)** "No, because the ADK blocks private data access by default."

**Rationale:**

- **A)** Wrong — Public search does not index private authenticated content.
- **B)** Correct — This distinguishes between *Public Web Search* (the topic of
  this lesson) and *Private Enterprise Search* (RAG/Vertex AI Search, covered in
  other lessons). Grounding with Google Search is strictly for public internet
  data.
- **C)** Wrong — This is a security misunderstanding.
- **D)** Wrong — It's not an ADK feature; it's a fundamental scope difference of
  the search engine.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Understand the concept of "Grounding" in LLMs.

## Question 7

**Scenario:**
In the exercise solution, the `structured_search_agent` is initialized with
`tools=[AgentTool(agent=search_agent)]`.

**Question:**
What is the primary function of the `AgentTool` class in this context?

**Options:**

- **A)** It converts the agent's Python code into a REST API endpoint.
- **B)** It wraps an ADK Agent instance so that it can be invoked by another
  agent as a standard tool.
- **C)** It forces the wrapped agent to only use Google Search.
- **D)** It is a debugging utility that logs all internal state changes of the
  agent.

**Rationale:**

- **A)** Wrong — It remains an in-process Python object.
- **B)** Correct — `AgentTool` adapts the `Agent` interface (which processes
  prompts) into the `Tool` interface (which takes function arguments and returns
  a string), allowing for hierarchical composition.
- **C)** Wrong — The wrapped agent can have any tools.
- **D)** Wrong — It is for orchestration, not just debugging.

**Difficulty:** Medium
**Cognitive Level:** Conceptual
**Learning Objective:** Implement `AgentTool` to wrap agents as tools.

## Question 8

**Scenario:**
The solution architecture creates a chain: `root_agent` calls
`structured_search_agent`, which calls `search_agent`. The `search_agent` has
the `google_search` tool, while the `root_agent` has the calculation tools.

**Question:**
Why is this separation into different agents necessary, rather than giving one
single agent all the tools and the `output_schema`?

**Options:**

- **A)** The ADK enforces a limit of 5 tools per agent to optimize the size 
  of the tool list provided to the LLM. Too many tools can cause the LLM to 
  behave erratically.
- **B)** To improve code readability, although a single agent would work
  technically.
- **C)** "Grounding with Google Search" often cannot be combined with custom
  Function Calling tools or structured output enforcement in the same inference
  step due to API limitations.
- **D)** The `google_search` tool consumes too many tokens to run alongside
  other tools.

**Rationale:**

- **A)** Wrong — There is no such strict low limit in the ADK. While too 
  many tools can cause problems, this does not address that issue.
- **B)** Wrong — While modularity is good, the primary driver here is the
  technical constraint imposed by the `google_search` tool.
- **C)** Correct — As noted in the lesson, mixing the specific "Grounding"
  feature with other tool definitions or JSON schema constraints can lead to
  conflicts or errors in the model's processing pipeline. Separation ensures
  each agent handles one "mode" of operation.
- **D)** Wrong — Token consumption is not the primary conflict.

**Difficulty:** Advanced
**Cognitive Level:** Analysis
**Learning Objective:** Orchestrate a chain of agents to work around API
constraints.