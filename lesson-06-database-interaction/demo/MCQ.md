## Question 1

**Scenario:**  
You are designing an agent that needs to access sensitive customer data in a
production database. You are choosing between a "Text-to-SQL" approach (where
the LLM generates the SQL query string) and an MCP-based approach using the
Google MCP Database Toolkit.

**Question:**  
From a security and reliability standpoint, what is the primary advantage of
using the MCP Database Toolkit over a "Text-to-SQL" approach?

**Options:**

- **A)** The MCP server can automatically generate more optimized SQL queries
  than an LLM.
- **B)** It eliminates the risk of SQL injection and unauthorized schema access
  by restricting the agent to pre-defined, parameterized SQL statements.
- **C)** The MCP server allows the LLM to directly modify the database schema at
  runtime.
- **D)** It removes the need for any database authentication or user
  permissions.

**Rationale:**

- **A)** Wrong — Optimization depends on the pre-written SQL, not an automatic
  feature of MCP.
- **B)** Correct — By using pre-defined statements in `tools.yaml`, the LLM only
  provides parameters. It cannot change the SQL structure, preventing attacks
  like `DROP TABLE` or accessing sensitive columns not in the query.
- **C)** Wrong — This would be a major security risk, not an advantage.
- **D)** Wrong — Authentication is still required, but it's handled by the MCP
  server, not the LLM.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Understand the role of the Model Context Protocol (MCP)
in agent architectures.

## Question 2

**Scenario:**  
A developer is configuring the `tools.yaml` file for a new MCP server. They have
included the following source definition:

```yaml
sources:
  inventory_db:
    kind: mysql
    host: 1.2.3.4
    user: root
    password: SuperSecretPassword123
```

**Question:**  
Which best practice for database integration is being violated in this
configuration, and how should it be corrected?

**Options:**

- **A)** **Performance**: The `kind` should be `postgres` for better speed;
  switch to a PostgreSQL instance.
- **B)** **Least Privilege**: Using the `root` user is unsafe; create a
  restricted user with only `SELECT` permissions on required tables and use
  environment variables for sensitive data.
- **C)** **Scalability**: Hardcoded IPs are faster; no correction is needed.
- **D)** **Redundancy**: A single source is insufficient; you must define at
  least two sources for every tool.

**Rationale:**

- **A)** Wrong — Performance is not the primary issue here, and the database
  type depends on the existing infrastructure.
- **B)** Correct — Using `root` provides too much power to the tool server.
- **C)** Wrong — Hardcoding is a maintenance and security risk.
- **D)** Wrong — Multiple sources are not required for a single tool.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Configure and connect an ADK agent to a Google Cloud
SQL (MySQL) instance using best practices.

## Question 3

**Scenario:**  
In the `tools.yaml` file, a tool is defined as follows:

```yaml
  books-by-author:
    kind: mysql-sql
    source: demo
    description: Get the books written by a specific author
    parameters:
      - name: author
        type: string
        description: The name of the author we're looking for
    statement:
      SELECT title FROM books WHERE author_name = ?
```

**Question:**  
How does the MCP Database Toolkit handle the `?` in the `statement` when the
agent calls this tool?

**Options:**

- **A)** It performs a string replacement, inserting the value provided by the
  LLM directly into the query string before execution.
- **B)** It uses parameterized queries (prepared statements), where the database
  driver safely handles the substitution to prevent SQL injection.
- **C)** It asks the LLM to provide the full `WHERE` clause to replace the `?`.
- **D)** The `?` is a wild-card that tells the database to return all authors if
  the parameter is empty.

**Rationale:**

- **A)** Wrong — Simple string replacement is the cause of SQL injection.
- **B)** Correct — Parameterized queries are the standard security mechanism for
  separating SQL logic from user-provided data.
- **C)** Wrong — This would allow the LLM to inject SQL.
- **D)** Wrong — The behavior of the parameter is defined by the SQL syntax, not
  the toolkit itself (though the toolkit facilitates the parameter passing).

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Define safe SQL tools using a declarative `tools.yaml`
configuration.

## Question 4

**Scenario:**  
You have updated your `tools.yaml` file with a new tool called `get-author-bio`.
You have restarted the MCP server (`toolbox`). However, when you ask your ADK
agent about an author's biography, it says it doesn't have a tool for that.

**Question:**  
Looking at the `agent.py` code, what is the most likely reason the agent cannot
use the new tool?

**Options:**

- **A)** The `agent-prompt.txt` needs to contain the full SQL query for the new
  tool.
- **B)** The `root_agent` must be explicitly updated in `agent.py` to include
  `db_client.load_tool("get-author-bio")` in its `tools` list.
- **C)** The LLM (`gemini-2.5-flash`) does not support more than two tools at a
  time.
- **D)** The database needs to be restarted for the MCP server to see the new
  tool definition.

**Rationale:**

- **A)** Wrong — The agent doesn't need to know the SQL; it just needs the tool
  definition.
- **B)** Correct — In ADK, tools must be explicitly registered with the agent.
  `ToolboxSyncClient.load_tool()` fetches the definition from the server, but it
  must be added to the agent's `tools` parameter.
- **C)** Wrong — Modern LLMs support many tools simultaneously.
- **D)** Wrong — The MCP server reads the `tools.yaml` at startup; the database
  itself is independent of the tool definitions.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Load specific tools defined in `tools.yaml` into an ADK
agent.

## Question 5

**Scenario:**  
An MCP server is running on `http://127.0.0.1:5001`. The `agent.py` initializes
the client with `db_client = ToolboxSyncClient("http://127.0.0.1:5000")`.

**Question:**  
What will happen when the agent attempts to call
`db_client.load_tool("books-by-author")`?

**Options:**

- **A)** The agent will successfully load the tool because the MCP protocol
  handles port discovery automatically.
- **B)** It will fail with a connection error because the client is attempting
  to connect to the wrong port.
- **C)** The agent will default to a "Text-to-SQL" mode if the MCP server is
  unreachable.
- **D)** The Python script will hang indefinitely waiting for a response.

**Rationale:**

- **A)** Wrong — Port numbers must match exactly for TCP/HTTP connections.
- **B)** Correct — The port mismatch (5000 vs 5001) is a common configuration
  error that prevents communication between the agent and the toolkit server.
- **C)** Wrong — ADK does not automatically switch between tool-based and direct
  SQL modes.
- **D)** Wrong — Most HTTP clients have a timeout and will raise an exception.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Identify and resolve common connectivity issues between
agents and MCP servers.

## Question 6

**Scenario:**  
You are using the Model Context Protocol (MCP) to connect your agent to a
database. You decide to move the MCP server from your local machine to a
separate secure server in your internal network.

**Question:**  
What is a major architectural benefit of this decoupling (separating the Agent
from the MCP Server)?

**Options:**

- **A)** The Agent code no longer needs any knowledge of the database
  technology (e.g., MySQL vs. PostgreSQL) or connection credentials.
- **B)** The LLM will be able to process database results faster if they come
  from a remote server.
- **C)** You no longer need to define parameters for your tools in `tools.yaml`.
- **D)** It allows the LLM to access the database even when the Agent is
  offline.

**Rationale:**

- **A)** Correct — This is the "Separation of Concerns" principle. The agent
  only cares about the tool interface (name, description, parameters). The MCP
  server handles the "how" (driver, credentials, SQL syntax).
- **B)** Wrong — Network latency usually makes remote calls slightly slower than
  local ones.
- **C)** Wrong — Parameters are still required for the tool to function.
- **D)** Wrong — The agent is the orchestrator; if it's offline, the process
  stops.

**Difficulty:** Advanced  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand the architectural benefits of the Model
Context Protocol (MCP).

## Question 7

**Scenario:**  
You are writing the `description` for a tool in `tools.yaml`.

```yaml
  books-in-year-range:
    description: "Executes the SELECT query on the books table with a BETWEEN clause."
```

**Question:**  
Why is this description suboptimal for an LLM-based agent, and how should it be
improved?

**Options:**

- **A)** It is too short; it needs to include the full database schema.
- **B)** It describes the implementation (SQL) rather than the utility; it
  should explain *what* the tool helps the user achieve (e.g., "Find books
  published between two specific years").
- **C)** Descriptions are ignored by LLMs; you should focus only on the
  parameter names.
- **D)** It should be written in Python code instead of plain English.

**Rationale:**

- **A)** Wrong — Schema details are usually unnecessary for the tool caller.
- **B)** Correct — LLMs use descriptions to map user intent to tool calls. An
  LLM understands "find books in 1920" better if the tool description mentions "
  Find books published between two years" rather than technical SQL terms.
- **C)** Wrong — Descriptions are critical metadata for function calling.
- **D)** Wrong — YAML/Plain text is the correct format for `tools.yaml`.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Write effective tool descriptions to guide LLM tool
selection.

## Question 8

**Scenario:**  
The `agent-prompt.txt` contains the instruction: "You are a helpful assistant.
Use your tools to answer questions about books." The `tools.yaml` contains a
tool called `get-books` which returns a list of titles. A user asks: "Who is the
most popular author in the database?"

**Question:**  
Given that there is no tool to calculate "popularity," how is the agent likely
to behave based on the lesson's principles?

**Options:**

- **A)** It will try to write a new SQL query to calculate author popularity and
  execute it directly.
- **B)** It should politely inform the user that it can only find books by
  author or year, as it is restricted to the capabilities provided by its tools.
- **C)** It will call the `get-books` tool and try to count the results
  manually.
- **D)** It will crash because the user's request doesn't match a tool.

**Rationale:**

- **A)** Wrong — The agent is restricted to the MCP tools; it cannot "
  Text-to-SQL" unless explicitly programmed and permitted to do so (which
  violates the safety goal of this lesson).
- **B)** Correct — This follows the "Fail-safe design" and "Constrained
  Capability" principles. The agent should stay within its defined domain.
- **C)** Wrong — Unless it has a tool for "all books" and enough context window
  to count, this is unlikely and inefficient.
- **D)** Wrong — Well-behaved agents handle out-of-domain requests gracefully.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Understand agent behavior and constraints when using
tool-based database access.

## Question 9

**Scenario:**  
A developer has finished building their agent and wants to share their
`tools.yaml` file with the rest of their team by checking it into their shared
git repository.

**Question:**  
What critical step should they take to ensure they are following security best
practices before they commit and push the file?

**Options:**

- **A)** Encrypt the entire `tools.yaml` file using a specialized tool.
- **B)** Ensure that all sensitive information, such as passwords and hostnames,
  has been replaced with environment variable references (e.g.,
  `${MYSQL_PASSWORD}`).
- **C)** Delete the `tools.yaml` file and only share the `agent.py` file.
- **D)** Change the file extension from `.yaml` to `.txt` to prevent
  unauthorized execution.

**Rationale:**

- **A)** Wrong — While encryption is a technique, the standard practice for
  configuration files like this is to use environment variables to keep secrets
  out of version control.
- **B)** Correct — Using environment variables ensures that sensitive data is
  not stored in plain text in the repository, while still allowing the
  configuration to be shared and used.
- **C)** Wrong — The `tools.yaml` file is essential for the MCP server to
  function.
- **D)** Wrong — Changing the extension provides no security and breaks the
  tool.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Configure and connect an ADK agent to a Google Cloud
SQL (MySQL) instance using best practices.
