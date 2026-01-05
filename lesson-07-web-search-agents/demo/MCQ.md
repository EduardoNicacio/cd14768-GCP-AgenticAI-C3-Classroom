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
- **D)** Wrong — While possible, the efficient behavior logic (C) is the
  intended design explanation in the lesson.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Observe how the model automatically decides when to
search the web.

## Question 3

**Scenario:**  
You want to modify the demo agent to *also* have a calculator tool. You update
the code as follows:

```python
from google.adk.tools import google_search


def calculator(a: int, b: int) -> int:
  """Adds two numbers."""
  return a + b


tools = [google_search, calculator]
```

**Question:**  
How does the ADK handle the coexistence of the native `google_search` tool and
your custom Python `calculator` function?

**Options:**

- **A)** It will fail; you cannot mix native platform tools with custom Python
  functions.
- **B)** It works seamlessly; the ADK registers both, and the model selects the
  appropriate one based on the user's query (e.g., "current news" vs. "add
  numbers").
- **C)** The `google_search` tool will override the calculator because it has
  higher priority.
- **D)** You must write a wrapper function around `google_search` to make it
  look like a Python function.

**Rationale:**

- **A)** Wrong — ADK is designed to support hybrid tool sets.
- **B)** Correct — The model sees a list of available tool definitions.
  `google_search` is just another tool in the schema. The model uses its
  reasoning capabilities to pick the right tool for the job.
- **C)** Wrong — Priority is determined by relevance to the query.
- **D)** Wrong — `google_search` is already an object compatible with the ADK's
  tool registration system.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Integrate the `google_search` tool into an ADK agent.

## Question 4

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

## Question 5

**Scenario:**  
You are reviewing the `agent.py` code provided in the demo.

```python
root_agent = Agent(
  # ...
  tools=[google_search],
)
```

**Question:**  
Unlike the database tools in the previous lesson (where we defined SQL in
`tools.yaml`), where is the logic for the `google_search` tool defined?

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

## Question 6

**Scenario:**  
Your agent is deployed in a production environment. Users are complaining that
for some queries, the agent provides outdated information despite
`google_search` being enabled.

**Question:**  
Which of the following prompt engineering strategies (as seen in
`agent-prompt.txt`) would most effectively address this issue?

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

## Question 7

**Scenario:**  
You attempt to run the demo code, but you receive a `403 Permission Denied`
error related to the Grounding service.

**Question:**  
What is the most likely administrative cause for this error?

**Options:**

- **A)** You forgot to run `adk web`.
- **B)** Your computer is not connected to the internet.
- **C)** The Google Search Grounding API (or Vertex AI API) is not enabled in
  your Google Cloud Project, or your project lacks access to this feature.
- **D)** The `agent.py` file has a syntax error.

**Rationale:**

- **A)** Wrong — `adk web` is the UI; the error comes from the backend
  execution.
- **B)** Wrong — Connection errors usually manifest as timeouts or DNS errors,
  not 403.
- **C)** Correct — 403 is a standard permissions error. Grounding is a specific
  cloud service that must be enabled (and potentially billed) in the project
  console.
- **D)** Wrong — Syntax errors cause parse failures, not runtime API permissions
  errors.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Integrate the `google_search` tool into an ADK agent.

## Question 8

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
