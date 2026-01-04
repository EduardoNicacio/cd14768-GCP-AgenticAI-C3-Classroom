## Question 1

**Scenario:**  
You are building an agent to process customer feedback. You define a Pydantic
model for the output with a field
`sentiment: Literal["positive", "negative", "neutral"]`. However, when you run
the agent with `output_schema` configured, the LLM sometimes returns "mixed" as
the sentiment, causing a validation error.

**Question:**  
What is the most robust way to fix this issue without changing the business
requirement that sentiment must be one of the three specific values?

**Options:**

- **A)** Remove the `Literal` constraint and use `str` instead to accept any
  value.
- **B)** Update the `agent-prompt.txt` to explicitly list the allowed values and
  instruct the model to choose the "closest fit" if the sentiment is mixed.
- **C)** Add "mixed" to the `Literal` definition in the Python code.
- **D)** Switch to a larger model like `gemini-2.5-pro` which never makes
  mistakes.

**Rationale:**

- **A)** Wrong — This removes type safety and breaks the contract with
  downstream systems expecting specific enums.
- **B)** Correct — The prompt (system instruction) works in tandem with the
  schema. Explicitly guiding the LLM's reasoning process helps it map complex
  inputs to the strict schema constraints.
- **C)** Wrong — The scenario implies the business requirement is strict ("must
  be one of the three").
- **D)** Wrong — All models can hallucinate; architectural and prompt
  engineering safeguards are still required.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Use Pydantic to define the desired output schema and
guide the LLM via prompts.

## Question 2

**Scenario:**  
You have an agent configured with `output_schema=RobotCommands`. A user
inputs: "Move forward 10 feet". The agent returns:

```json
{
  "commands": [
    {
      "command": "walk",
      "distance": {
        "num": 10,
        "units": "feet"
      }
    }
  ]
}
```

Now the user inputs: "Hello there!". The agent returns a generic text string:
"Hello! I am a robot controller." instead of JSON, causing your backend parser to
crash.

**Question:**  
Why did the agent fail to return the structured JSON format for the second
input?

**Options:**

- **A)** The input "Hello there!" did not contain any trigger words for the
  robot commands.
- **B)** The `output_schema` was temporarily disabled by the ADK.
- **C)** The agent's prompt likely lacks instructions on how to handle
  non-command inputs within the strict JSON structure (e.g., using an "error"
  or "message" field).
- **D)** Pydantic models automatically reject any input that isn't a valid
  command.

**Rationale:**

- **A)** Wrong — Even with irrelevant input, a properly configured structured
  output agent should return a valid JSON object (e.g., with an empty list or
  error field), not plain text.
- **B)** Wrong — The schema configuration is static.
- **C)** Correct — If the prompt doesn't tell the LLM how to map "chat" to the
  required schema (like the `error` field in the demo), the model might revert
  to its default chat behavior, breaking the structure.
- **D)** Wrong — Pydantic validates *output*, it doesn't filter *input*.

**Difficulty:** Advanced  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand how to handle ambiguous or invalid inputs
with structured error reporting.

## Question 3

**Scenario:**  
In the `RobotCommand` Pydantic model, you see the following field definition:

```python
direction: Optional[DirectionType] = Field(None,
                                           description="The direction for the robot.")
```

**Question:**  
What is the primary function of the `description` parameter in this `Field`
definition?

**Options:**

- **A)** It is solely for generating Python documentation (docstrings).
- **B)** It provides semantic context to the Gemini model, helping it understand
  *what* data to populate in this field.
- **C)** It sets the default value of the field if the LLM omits it.
- **D)** It is used by the database to name the column when saving the data.

**Rationale:**

- **A)** Wrong — While true for Python docs, in this context, its critical role
  is for the LLM.
- **B)** Correct — The ADK converts this description into the schema passed to
  the LLM. The model uses this text to infer intent and correct usage.
- **C)** Wrong — The `default` parameter (or the first argument to `Field`) sets
  the default.
- **D)** Wrong — It is metadata, not a database configuration.

**Difficulty:** Medium  
**Cognitive Level:** Conceptual  
**Learning Objective:** Use Pydantic to define the desired output schema for
Gemini.

## Question 4

**Scenario:**  
Your agent needs to extract a list of grocery items from a user's sentence. You
define `Item(BaseModel)` and then set `output_schema=Item`. When the user says
"Apples, bananas, and milk", the agent only returns the JSON for "Apples".

**Question:**  
How should you correct the Pydantic model structure to capture all items?

**Options:**

- **A)** Change the prompt to say "Return all items".
- **B)** Define a wrapper model (e.g., `GroceryList`) containing a field
  `items: list[Item]` and use that as the `output_schema`.
- **C)** Add `many=True` to the `Agent` constructor.
- **D)** Increase the `max_output_tokens` parameter.

**Rationale:**

- **A)** Wrong — The schema constraint `output_schema=Item` forces a single
  object return, regardless of the prompt.
- **B)** Correct — To return a JSON array or a list of objects, the top-level
  schema must define a field that is a list. This matches the `RobotCommands`
  pattern in the demo.
- **C)** Wrong — `many=True` is not a valid ADK parameter for this purpose.
- **D)** Wrong — Token limits don't change the structural constraints of the
  schema.

**Difficulty:** Medium  
**Cognitive Level:** Application  
**Learning Objective:** Configure Gemini calls to return structured JSON.

## Question 5

**Scenario:**  
The demo code includes this line in `agent.py`:

```python
output_schema = RobotCommands,  # See what happens if we remove this line
```

If you were to remove this line, what would be the most likely immediate
consequence for downstream applications consuming this agent's response?

**Options:**

- **A)** The agent would stop responding entirely.
- **B)** The agent would automatically default to XML output.
- **C)** The model would always return non-deterministic plain text (e.g., 
  "Sure, I'll turn left"), breaking any code expecting strictly formatted JSON. 
- **D)** The output may become non-deterministic plain text (e.g., "Sure, I'll
  turn left"), breaking any code expecting strictly formatted JSON.

**Rationale:**

- **A)** Wrong — The service would still run.
- **B)** Wrong — The LLM would still generate a response.
- **C)** Wrong — Without the forced schema, it *may* give the wrong 
  formatting, but the instructions may be enough to coerce it into the 
  correct format much of the time. But without forcing the schema, this is 
  not guaranteed.
- **D)** Correct — Without the forced schema, the LLM reverts to natural
  language generation. Even if prompted to output JSON, it might wrap it in
  markdown blocks or include conversational filler, breaking strict parsers.

**Difficulty:** Medium  
**Cognitive Level:** Evaluation  
**Learning Objective:** Configure ADK agents to return structured JSON.

## Question 6

**Scenario:**  
You are designing an agent to control a smart home. You want the
`set_temperature` command to strictly accept values between 60 and 80.

**Question:**  
Using Pydantic and ADK best practices, how should you enforce this constraint?

**Options:**

- **A)** Write a post-processing Python function to check the value after the
  agent returns.
- **B)** Use Pydantic's `Field(..., ge=60, le=80)` validation in the schema
  definition.
- **C)** Trust the LLM to follow the instruction "only pick values between 60
  and 80" in the text prompt.
- **D)** Use a `Literal` with all 21 possible integers listed.

**Rationale:**

- **A)** Wrong — While possible, it's reactive. It's better to guide the
  generation or catch it at the validation layer first.
- **B)** Correct — Pydantic supports numeric validation constraints (`ge` =
  greater than or equal, `le` = less than or equal). This information is often
  passed to the model (depending on provider support) or used to validate the
  output immediately, raising an error if the model hallucinates a value outside
  the range.
- **C)** Wrong — Instructions are soft constraints; schema validation is a hard
  constraint (or at least a hard check).
- **D)** Wrong — Clunky and doesn't handle floats (e.g., 72.5).

**Difficulty:** Advanced  
**Cognitive Level:** Application  
**Learning Objective:** Use Pydantic to define the desired output schema.

## Question 7

**Scenario:**  
A user inputs "Go north". The `DirectionType` Literal is defined as
`Literal["left", "right"]`. The agent, trying to be helpful, outputs JSON with
`"direction": "north"`.

**Question:**  
What happens when the ADK attempts to process this response against the
`RobotCommand` schema?

**Options:**

- **A)** It auto-corrects "north" to "left".
- **B)** It raises a `ValidationError` because "north" is not a valid member of
  the `Literal` set.
- **C)** It accepts the value and passes it to the application.
- **D)** It re-runs the prompt with a higher temperature.

**Rationale:**

- **A)** Wrong — Pydantic does not auto-correct data semantics.
- **B)** Correct — Pydantic enforces strict type and value checking. If a value
  is not in the `Literal` definition, validation fails.
- **C)** Wrong — This violates the schema definition.
- **D)** Wrong — While some advanced frameworks *might* implement auto-retries
  on validation errors, the immediate result of the schema check itself is an
  error.

**Difficulty:** Medium  
**Cognitive Level:** Analysis  
**Learning Objective:** Understand Pydantic validation mechanics.

## Question 8

**Scenario:**  
Why does the `RobotCommand` model include an optional `error` field?

```python
error: Optional[str] = Field(None,
                             description="If the instruction was invalid...")
```

**Question:**  
Select the best architectural reason for this design choice.

**Options:**

- **A)** To allow the LLM to gracefully handle ambiguity within the strict JSON
  constraints, rather than breaking the schema or hallucinating a valid command.
- **B)** To debug Python exceptions that occur during execution.
- **C)** To allow the backend to send error logs to the user.
- **D)** It is a mandatory field required by the ADK `Agent` class.

**Rationale:**

- **A)** Correct — By providing a valid schema path for "I don't understand"or "
  That's invalid", we keep the system deterministic. The parsing logic can
  checks `if command.error: ...` instead of dealing with broken JSON or
  unexpected plain text.
- **B)** Wrong — This is for *semantic* errors in the user's intent, not runtime
  exceptions.
- **C)** Wrong — This is output *from* the agent, not *to* the user.
- **D)** Wrong — The schema is user-defined; ADK does not mandate specific
  fields.

**Difficulty:** Medium  
**Cognitive Level:** Conceptual  
**Learning Objective:** Handle ambiguous or invalid inputs with structured error
reporting.
