## Question 1

**Scenario:**  
You are building a RAG agent to answer questions about a large collection of
technical manuals stored as PDFs. You want the agent to cite specific paragraphs
rather than summarizing entire documents. You configure the `discoveryengine`
search request in your code.

**Question:**  
Which specific configuration setting in the `SearchRequest` is critical for
retrieving small, relevant text snippets instead of full document records?

**Options:**

- **A)** Set `page_size` to 1.
- **B)** Set `query_expansion_spec` to `AUTO`.
- **C)** Set `search_result_mode` in `content_search_spec` to `CHUNKS`.
- **D)** Set the `filter` parameter to limit by file type.

**Rationale:**

- **A)** Wrong — This just limits the number of results, not their granularity.
- **B)** Wrong — This helps with synonyms, not chunking.
- **C)** Correct — The `CHUNKS` mode tells the Discovery Engine to return parsed
  segments of text (chunks) that are semantically relevant, which is the core
  retrieval unit for RAG.
- **D)** Wrong — This filters the result set but doesn't change the return
  format (documents vs. chunks).

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Implement a search tool using the
`google.cloud.discoveryengine` library.

## Question 2

**Scenario:**  
Your RAG agent is failing to retrieve information about "Fiscal Year 2024" even
though you uploaded the relevant PDF to the GCS bucket 5 minutes ago. You check
the Vertex AI Agent Builder console and see the Data Store status.

**Question:**  
What is the most likely reason for the empty search results?

**Options:**

- **A)** The documents have not finished indexing.
- **B)** The agent's `model` parameter is set to `gemini-2.5-pro` instead of
  `flash`.
- **C)** You forgot to enable the "PDF Parser" API.
- **D)** The GCS bucket permissions are set to private.

**Rationale:**

- **A)** Correct — Indexing unstructured documents in Vertex AI Search is an
  asynchronous process that takes time (minutes to hours depending on volume).
  Until indexing is complete, queries will return empty results.
- **B)** Wrong — The LLM model choice does not affect the retrieval tool's
  ability to find documents.
- **C)** Wrong — Vertex AI Search handles parsing automatically for supported
  formats.
- **D)** Wrong — Vertex AI Search accesses the bucket via a service agent,
  independent of public/private status, as long as the initial setup linkage was
  successful.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Configure Google Cloud Storage and Vertex AI Search for
RAG.

## Question 3

**Scenario:**  
A user asks: "What were the 'earnings' for Q3?" The relevant document uses the
term "Revenue" and "Net Income" but never explicitly uses the word "earnings".
However, your RAG agent successfully finds the correct text chunks about
Revenue.

**Question:**  
Which feature of the Vertex AI Search (Discovery Engine) configuration in
`datastore.py` likely enabled this successful retrieval?

**Options:**

- **A)** `spell_correction_spec` set to `AUTO`.
- **B)** `query_expansion_spec` set to `AUTO`.
- **C)** The use of `page_size: 10`.
- **D)** The `datastore_search_tool` wrapper function.

**Rationale:**

- **A)** Wrong — "Earnings" is spelled correctly, so spell correction isn't the
  primary factor.
- **B)** Correct — Query Expansion automatically expands terms to include
  synonyms and related concepts (like mapping "earnings" to "revenue" or
  "profit"), improving recall for semantic matches.
- **C)** Wrong — Page size only affects the count of results.
- **D)** Wrong — The wrapper just calls the API; it doesn't add intelligence.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand the `search` function configuration (content
specs, query expansion).

## Question 4

**Scenario:**  
You have deployed your agent. A user asks a question, and the
`datastore_search_tool` returns a list of 5 relevant text chunks. However, the
agent's final answer ignores these chunks and provides a generic response from
its training data.

**Question:**  
What is the most effective change to fix this behavior?

**Options:**

- **A)** Increase the temperature of the model to 1.0.
- **B)** Add an instruction to your agent prompt: "Base your answers *only* on 
  the retrieved information from the `datastore_search_tool`. If the info is 
  not in the results, say you don't know."
- **C)** Configure the tool to return 50 chunks instead of 5.
- **D)** Switch the model to `gemini-2.5-pro` because it is better at RAG.

**Rationale:**

- **A)** Wrong — Higher temperature increases hallucination.
- **B)** Correct — This is a standard Grounding prompt technique. You must
  explicitly constrain the model to the context provided by the tool to prevent
  it from falling back on its internal (potentially outdated) weights.
- **C)** Wrong — More chunks might dilute the context or hit token limits;
  quality of instruction is the issue.
- **D)** Wrong — Newer models like 1.5/2.0 are generally better at context
  handling; downgrading isn't the fix.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Connect the search tool to an ADK agent for grounded
responses.

## Question 5

**Scenario:**  
You are setting up the environment variables for `datastore.py`. You have the
Project ID and the Location. You need the `DATASTORE_ENGINE_ID`.

**Question:**  
Where in the Google Cloud Console do you find the correct `DATASTORE_ENGINE_ID`
for a "Custom Search" app?

**Options:**

- **A)** It is the name of the GCS bucket.
- **B)** It is the "App ID" listed in the Agent Builder / AI Applications list (
  a string of letters/numbers).
- **C)** It is the "Data Store ID" listed in the Data Stores tab.
- **D)** It is the project number found on the dashboard.

**Rationale:**

- **A)** Wrong.
- **B)** Correct — For a Search App, the `engine_id` in the API corresponds to
  the App ID in the console. This is a common point of confusion vs. the Data
  Store ID (which is linked to the app).
- **C)** Wrong — The API calls the *Engine* (App), which queries the Data Store.
  Using the Data Store ID often results in 404s.
- **D)** Wrong.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Configure Google Cloud Storage and Vertex AI Search for
RAG.

## Question 6

**Scenario:**  
Your code in `datastore.py` uses `ClientOptions` to set the `api_endpoint`.

```python
if location != "global":
  client_options = ClientOptions(
    api_endpoint=f"{location}-discoveryengine.googleapis.com")
```

**Question:**  
Why is this conditional logic necessary?

**Options:**

- **A)** It is not necessary; all requests can go to
  `discoveryengine.googleapis.com`.
- **B)** Vertex AI Search (Discovery Engine) is a regional service. If your
  data/engine is in `us-central1`, you *must* target the `us-central1` endpoint,
  or the API will return a 404/Error.
- **C)** It is for load balancing purposes only.
- **D)** `global` locations are deprecated.

**Rationale:**

- **A)** Wrong — The default endpoint assumes global/standard routing which
  might fail for strictly regional resources.
- **B)** Correct — Google Cloud APIs often require regional endpoints for data
  residency and routing. If you created a regional engine, you must talk to that
  region's API endpoint.
- **C)** Wrong.
- **D)** Wrong.

**Difficulty:** Advanced  
**Cognitive Level:** Analysis  
**Learning Objective:** Implement a search tool using the
`google.cloud.discoveryengine` library.
