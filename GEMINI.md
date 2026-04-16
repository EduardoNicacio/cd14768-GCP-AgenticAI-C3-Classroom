# Building module demos and exercises

Your goal is to assist in writing code for demos and examples for different
modules. Each module has its own folder that is identified by number using
the pattern "lesson-##".
So for Module 1 the name of the folder would be "lesson-01".

What is needed for each demo or example is provided in the
"module-outline.md" file. This file is divided by modules and each module
contains a section about the demo program and the example program that are
required. Pay attention to all the requirements and components of the demo
or exercise that are outlined in their respective sections.

The demo program will be in the "demo" folder under the module folder.
The example program will be under the "exercises/solution" folder under the
module folder.

To assist when building a demo or exercise you should base it on previous
demo or exercise files, either from this module or a previous module, or
from the files in "project/solution".

Other requirements for the code:

* It must be written in python and using the ADK toolkit
* There must be an "__init__.py" file. You can copy it from a previous demo
  or exercise folder. It must not be empty.
* There must be an "agent.py" file that creates the `root_agent`
  * You must look at previous code to see how to structure "agent.py"
  * The name of the adk package is "google.adk".
  * Make sure you add a "name" parameter based on the name of the module,
    with any spaces converted to underscore
  * It should also have a description based on the description of the module
  * The instruction parameter should be read from an "agent-prompt.txt"
    file. You can see other modules to see how this file should be loaded in.
  * Make sure there is a model variable that is set to "gemini-2.5-flash"
    and make sure you include it when constructing the Agent
* You should add a "requirements.txt"
* When creating new module demos and exercises, always include the
  .env file by copying it from a previous module.
* When creating new module demos and exercises, each should also contain a
  README.md file that contains environment information and any additional
  setup information. You can look at the README.md from previous modules for
  examples.
* It should be clear and well documented

# Building the exercise starter

You are an expert technical educator.
You will be helping create starter files that students should build upon.
These files will be in the "exercises/starter" directory and should be based
on the files that are in "exercises/solution".
Information about the lesson being taught can be found in the
"module-outline.md" file.

Guidelines:

* The starter files should contain boilerplate code, or other code that is
  incidental to the lesson being taught.
* You should only use code that is present in the solution files. You should
  not create any new code.
* You should not edit the original solution in any way. Just produce the
  files in the starter that can be used to create the solution.
* For parts of the files that the student will need to fill in, they should
  be marked with a comment starting with "TODO:"
* You should create an ".env-sample" file based on the environment
  information provided in the README.md file in the solution directory.
* The __init__.py file should be copied verbatim
* Prompt files should just have the contents "TODO"

# Generate "CLASSROOM-README.md" from Code Repository

You are an expert technical educator.
Analyze a code repository and generate a README based on the specified type.

## INPUT

* The lesson number

* README type: __demo__, __starter__, or __solution__
* You can get the topic / lesson focus based on the "module-outline.md" file

Based on the lesson number and the type, you will know the root directory of
the files you will be working with.

* For __demo__ types, this will be "demo"
* For __starter__ types, this will be "exercises/starter"
* For __solution__ types, this will be "exercises/solution"

## README TYPES

### Type 1: DEMO README

__Purpose__: Explain and describe existing code walkthrough
__Tone__: Descriptive - "This code does...", "Here's how it works..."

### Type 2: STARTER README

__Purpose__: Give instructions for learners to implement
__Tone__: Instructional - "Build a...", "Implement...", "Your task is..."

### Type 3: SOLUTION README

__Purpose__: Explain and describe the solution code
__Tone__: Descriptive - "The solution works by...", "This implementation..."

---

## OUTPUT STRUCTURE

# {Lesson Title}

{One sentence describing what this lesson teaches}

---

## Overview

### What You'll Learn

{2-3 sentences explaining what learners will be able to do}

Learning objectives:

* {Objective 1}
* {Objective 2}
* {Objective 3}

### Prerequisites

* {Prerequisite 1}
* {Prerequisite 2}

---

## Understanding the Concept

### The Problem

{What problem does this solve? Use a relatable scenario}

### The Solution

{How does this concept/technique solve the problem?}

### How It Works

{Explain the mechanism, architecture, or workflow}

{If multiple components/steps:}

__Step 1: {Name}__
{Explanation}

__Step 2: {Name}__
{Explanation}

### Key Terms

__{Term 1}__: {Definition}

__{Term 2}__: {Definition}

---

## CODE WALKTHROUGH (for demo/solution) OR EXERCISE INSTRUCTIONS (for starter)

---

### IF README TYPE = "demo" OR "solution"

## Code Walkthrough

### Repository Structure

```bash
{file-structure}
├── {file}  # {Description}
└── {file}  # {Description}
```

### Step 1: {Implementation Step}

{Describe what this code does and why}

```{language}
{Actual code from repository with inline comments}
```

__Key points:__

* {Important detail}
* {Important detail}

### Step 2: {Implementation Step}

{Describe what this code does and why}

```{language}
{Actual code from repository with inline comments}
```

__Key points:__

* {Important detail}
* {Important detail}

### Complete Example

{Show the full working code}

```{language}
{Complete code with comments}
```

__How it works:__

1. {Explanation of key section}
2. {Explanation of key section}
3. {Explanation of key section}

__Expected output:__

```bash
{What this code produces when run}
```

---

### IF README TYPE = "starter"

## Exercise Instructions

### Your Task

{Clear description of what the learner needs to build}

### Requirements

Your implementation must:

1. {Requirement 1}
2. {Requirement 2}
3. {Requirement 3}
4. {Requirement 4}

### Repository Structure

```bash
{file-structure}
├── {file}  # {Description - what learner will modify}
└── {file}  # {Description}
```

Make sure you copy ".env-sample" to ".env" and edit it to add the Google
Cloud project you are working with.

{Provide any other instructions about the environment variables necessary}

Remember that you should __never__ check-in your .env file to git.

### Starter Code

{Explain what's provided and what needs to be completed}

```{language}
{Partial code with TODO comments}


# TODO: {Instruction for what to implement}


# TODO: {Instruction for what to implement}
```

### Expected Behavior

{Describe what the completed implementation should do}

__Running the agent:__

You will run the agent using the `adk web` tool. This tool launches a chat
environment that lets you test the agent interactively and examine the
internal processing that ADK and Gemini go through.

The `adk web` application is meant to be run from a directory that has a
collection of agents, which is usually the parent directory from where your
agent's code is. Typically, you will run this on the same machine where your
browser is located with a command such as:

```bash
adk web
```

{Provide any other information about starting it that are needed}

__Example usage:__

```bash
{How to run/test the code}
```

__Expected output:__

```bash
{What correct implementation produces}
```

### Implementation Hints

1. {Hint without giving away solution}
2. {Hint without giving away solution}
3. {Hint without giving away solution}

---

## Important Details

### Common Misconceptions

__Misconception__: "{Common misunderstanding}"
__Reality__: {Correct explanation}

__Misconception__: "{Common misunderstanding}"
__Reality__: {Correct explanation}

### Best Practices

1. __{Practice 1}__: {Explanation based on repository code}
2. __{Practice 2}__: {Explanation based on repository code}

### Common Errors

__Error__: {Description of typical error}

* __Cause__: {Why it happens}
* __Solution__: {How to fix it}

__Error__: {Description of typical error}

* __Cause__: {Why it happens}
* __Solution__: {How to fix it}

---

## GUIDELINES

* Extract actual code from the repository - do not invent examples
* For demo/solution: Explain what the code does and why it works
* For starter: Give clear instructions on what to build, not how to build it
* Use code blocks with inline comments
* Keep explanations concise (3-5 sentences per section)
* Include expected output examples
* Use the term "customer" when talking about the person who is using the agent

# Building a "video-outline.md"

You are an expert technical educator.
Based on the code for either a module demo or exercise solution, the
information about a module in the "module-outline.md" file, and the contents
of the "CLASSROOM-README.md" file for the demo or exercise, you will create
an outline that will be used to present the demo or solution.

Guidelines:

* This outline will be used by the instructor to create a walk-through of
  the code and then running and testing the code.
  * For the demo, the instructor will be introducing the concepts that this
    module is intended to teach and that the code is intended to illustrate.
    It should be making concepts clear as they are walking through the code.
  * For the exercise solution walk-through, the code will already be written,
    so the instructor is going over the code, not talking about what needs
    doing.
* Focus on the new elements that are being introduced and taught
* Each presentation should be about 5 minutes spoken
* The outline will be guidance for the instructor, not a script
* The outline should be presented sequentially, so the instructor can go
  linearly from point to point.
  * The code should be presented in a top-down format, introducing the main
    `root_agent` first and then seeing how those parts are broken down.
* Be specific - mention specific files, functions, commands, and prompts.
  Remember that the instructor will be following this to make their
  presentation.

Format:

* Produce a list of bullet points of the major things to highlight
* Be specific
  * mention specific files, functions, and line numbers
  * give specific commands to run or prompts to try to illustrate
  * when covering setup, give specific steps to follow
* Each bullet point should be about one line long
* If there are additional things to mention related to the major point, use
  an indented sub-list under the bullet
* The title will include the name of the project directory (either "cd14768"
  or "cd14769")

## Output structure

# {project directory} - Lesson {module number} - {"demo" or "exercise"}

{name of lesson}

* {objective of the lesson}
* Setup
  * {list of detailed setup instructions}
* [{file with change}] {highlight code or prompt change}
  * {reason for change}
  * {relationship to overall lesson}
  * {important concept to learn from this, if relevant}
* {...repeat above as necessary to show all new code, configuration, and prompts}
* running the code
  * start `adk web` in another window. {other command line parameters as needed}
  * {other startup instructions as needed}
* demonstration
  * {steps to illustrate}
* {...repeat any of the above as necessary to show changes}
* {conclusion and summary}

# Building Multiple Choice Questions (MCQs)

You are an __expert Subject Matter Expert (SME) and Instructional Designer__.  
Your goal is to design __high-quality, technically rigorous Multiple Choice
Questions (MCQs)__ that assess __deep understanding, reasoning, and
real-world application__, not surface-level recall.  

---

## __Task__

I will ask you to generate the multiple-choice questions, or MCQs, based on
the demo and exercise material for a specific lesson. You should read the files
associated with that lesson, focusing on the `CLASSROOM-README.md` and
`video-outline.md` files, but also being sure to incorporate the major
concepts from other files.

Based on this material, you should generate __exactly 8 MCQs__.
They should be saved using the format below in a file named `MCQ.md` under
each lesson demo directory.

The questions should collectively evaluate __analysis, application, and evaluation skills__.

---

## __Core Quality Expectations__

### __1. No Basic Recall__

* Do __not__ ask definition-only or fact-memorization questions.
* Every question must require interpretation, reasoning, or decision-making.

### __2. Cognitive Depth__

* Target __Medium to Advanced difficulty__.
* Align with __Bloom’s levels__:
  * Application
  * Analysis
  * Evaluation

### __3. Scenario-Driven Thinking__

* Anchor questions in __realistic scenarios__, such as:
  * System or architectural decisions
  * Code execution behavior
  * Debugging unexpected outcomes
  * Business-to-technical translation

---

## __Question Design Guidance__

The six questions should demonstrate __variety and balance__.  
You may draw inspiration from the following __idea patterns__, but __do not force-fit__ them if the material does not naturally support them:

* Foundations / Conceptual questions
* Architectural or design trade-offs
* Logic flow or code execution interpretation
* Workflow or process ordering
* Scenerio-based problem solving
* Cause-and-effect analysis or debugging
* Multi-condition evaluation (e.g., “select all that apply”)

__Important:__  
These are __examples of high-value question styles__, not a fixed sequence or required mapping. Prioritize __conceptual integrity and realism__ over category coverage.

---

## __Required Output Format__

Use __Markdown__ for each question.

```md
## Question [Number]

**Scenario:**  
[Relevant technical or business context providing background for the question]

**Question:**  
[Focused question testing a specific learning objective]

**Options:**  
- **A)** [Option text]  
- **B)** [Option text]  
- **C)** [Option text]  
- **D)** [Option text]

**Rationale:**  
- **A)** Correct/Wrong — [Explanation grounded in the material]  
- **B)** Correct/Wrong — [Explanation grounded in the material]  
- **C)** Correct/Wrong — [Explanation grounded in the material]  
- **D)** Correct/Wrong — [Explanation grounded in the material]

**Difficulty:** Medium / Advanced  
**Cognitive Level:** Application / Analysis / Evaluation  
**Learning Objective:** [Specific skill or capability assessed]
```

---

## __Design Principles__

* __Start with the “Why”__  
  Identify key insights, failure modes, or misconceptions learners commonly have.
* __Use Material as the Source of Truth__  
  Base reasoning strictly on the provided content (especially code or architecture).
* __Application over Definition__  
  Prefer:  
  “Given this setup, what happens if X changes?”  
  over:  
  “What is X?”
* __Plausible Distractors__  
  Incorrect options should reflect:
  * Common misunderstandings
  * Near-correct logic
  * Realistic implementation mistakes

---

## __Assessment Intent__

These MCQs should resemble the kind of reasoning required in __real-world
engineering, architecture, or advanced professional scenarios__, not
textbook exercises.  
