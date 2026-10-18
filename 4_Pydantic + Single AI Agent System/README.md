# 🧱 Pydantic for AI Agents + Building a Single AI Agent System (Beginner → Pro)

> **Who this is for:** Beginners who know basic Python and the agent basics (LLM, tools, loop).
> **Goal:**
> 1. Learn **Pydantic**: how to force AI output into a **clean, checked structure** instead of messy text.
> 2. Build a **complete single AI agent** with **LangChain**: tools, memory/context, planning loop, structured output, and a preview of **deployment** (Docker + cloud).
>
> 📖 Every new term is explained in a **"📖 New term"** box, and all terms are collected in the **[Glossary](#13-glossary--every-term-explained-simply)** at the end.

---

## 📚 Table of Contents

**PART 1: Pydantic**
1. [The Problem: LLMs Return Messy Text](#1-the-problem-llms-return-messy-text)
2. [What is Pydantic?](#2-what-is-pydantic)
3. [Data Modeling with `BaseModel`](#3-data-modeling-with-basemodel)
4. [`Field`: Rules, Limits & Descriptions](#4-field--rules-limits--descriptions)
5. [Advanced Types & Custom Validators (Pro)](#5-advanced-types--custom-validators-pro)
6. [Structured Output from LLMs](#6-structured-output-from-llms)
7. [Why Validation Matters in Agents](#7-why-validation-matters-in-agents)

**PART 2: Single AI Agent System**
8. [Prerequisites: LangChain Basics](#8-prerequisites--langchain-basics)
9. [What is a Single AI Agent System?](#9-what-is-a-single-ai-agent-system)
10. [Build It End-to-End (Step by Step)](#10-build-it-end-to-end-step-by-step)
11. [Deployment Overview (Docker + Cloud)](#11-deployment-overview-docker--cloud)
12. [System Revision: Architecture Recap, Cheat Sheet & Interview Qs](#12-system-revision--architecture-recap)
13. [Glossary](#13-glossary--every-term-explained-simply)

---

# PART 1: PYDANTIC FOR AI AGENTS

## 1. The Problem: LLMs Return Messy Text

Ask an LLM: *"Extract the name, age and city from: 'Ashish is 28 and lives in Hyderabad'"*

You might get any of these:
```
Name: Ashish, Age: 28, City: Hyderabad
```
```
Sure! Here's the info 😊
- Name → Ashish
- Age → twenty-eight
- City → Hyderabad
```
```json
{"name": "Ashish", "age": "28 years", "city": "Hyd"}
```

Same question, **3 different formats**. Now imagine your code needs to save this into a database:
- `age = "twenty-eight"` → 💥 your DB expects a number
- missing `city` → 💥 crash
- extra chatty text → 💥 JSON parsing fails

> 🔑 **The core problem:** LLMs are **creative and unpredictable**. Code and APIs need **exact, predictable** data.
> **Pydantic is the bridge between the two.**

> 📖 **New terms**
> - **Parse / Parsing:** reading text and converting it into a usable data structure (e.g., turning a JSON string into a Python dict).
> - **Downstream service:** any system that **receives** your agent's output next: a database, an API, a dashboard, another agent.

---

## 2. What is Pydantic?

**Pydantic** is a Python library that **checks (validates) data** against a structure you define. If the data is wrong, it **fixes it when it safely can** or **raises a clear error**.

> 📖 **New terms**
> - **Validation:** checking that data has the right shape, types, and values *before* using it.
> - **Schema:** a blueprint describing what data should look like (field names, types, rules).
> - **Type:** the kind of a value: `str` (text), `int` (whole number), `float` (decimal), `bool` (True/False), `list`, `dict`.

### 🛂 Analogy: Airport security check
- You define the rules: *"Passport required, must be valid, bag under 15 kg."*
- Every passenger (**data**) passes through the security check (**Pydantic**).
- ✅ Follows the rules → allowed in.
- ❌ Breaks a rule → stopped with a clear reason ("Bag is 18 kg, max is 15").

### Install
```powershell
pip install pydantic
```
> This README uses **Pydantic v2** (current version). Some old tutorials use v1. Differences are noted where they matter.

### Who uses Pydantic?
**FastAPI, LangChain, LangGraph, OpenAI SDK, Anthropic SDK, CrewAI, PydanticAI**... almost the whole Python AI ecosystem. Learning it once pays off everywhere.

---

## 3. Data Modeling with `BaseModel`

> 📖 **New terms**
> - **Data model:** a Python class that describes the structure of your data.
> - **Class:** a blueprint for creating objects. (`class Person:` describes what every person object has.)
> - **`BaseModel`:** Pydantic's parent class. Any class that inherits from it gets automatic validation.
> - **Inherit:** a class "borrowing" all abilities from a parent class: `class Person(BaseModel)`.
> - **Type hint:** writing the expected type next to a variable: `age: int`.

### 3.1 Your first model
```python
from pydantic import BaseModel

class Person(BaseModel):
    name: str
    age: int
    city: str

p = Person(name="Ashish", age=28, city="Hyderabad")
print(p)            # name='Ashish' age=28 city='Hyderabad'
print(p.name)       # Ashish
```

### 3.2 Automatic type conversion (coercion)
```python
p = Person(name="Ashish", age="28", city="Hyderabad")   # age is a STRING "28"
print(p.age, type(p.age))    # 28 <class 'int'>  ✅ converted automatically
```
> 📖 **New term: Coercion** = automatically converting a value to the right type when it's safe (`"28"` → `28`).

### 3.3 Validation errors (bad data is stopped)
```python
from pydantic import ValidationError

try:
    Person(name="Ashish", age="twenty-eight")      # bad age + missing city
except ValidationError as e:
    print(e)
```
**Output (simplified):**
```
2 validation errors for Person
age
  Input should be a valid integer, unable to parse string as an integer
city
  Field required
```
Clear, exact reasons: perfect for debugging, and perfect for **sending back to the LLM so it can fix its own mistake** (see 7.3).

### 3.4 Optional fields & default values
```python
from typing import Optional

class Person(BaseModel):
    name: str
    age: int
    city: str = "Unknown"           # default value → not required
    email: Optional[str] = None     # can be missing or None
```
> 📖 **New terms**
> - **Default value:** the value used when none is given.
> - **`Optional[str]`:** the value can be a string **or** `None`. (Modern style: `str | None`.)
> - **`None`:** Python's "nothing / empty" value.

### 3.5 Lists and nested models
```python
class Address(BaseModel):
    city: str
    pincode: str

class Employee(BaseModel):
    name: str
    skills: list[str]               # list of strings
    address: Address                # 👈 a model inside a model (nested)

emp = Employee(
    name="Ashish",
    skills=["Python", "PySpark", "AWS"],
    address={"city": "Hyderabad", "pincode": "500032"},   # dict → auto-converted to Address
)
print(emp.address.city)             # Hyderabad
```
> 📖 **New term: Nested model** = a model used as a field inside another model, for data that has layers (like JSON inside JSON).

### 3.6 Converting to/from dict & JSON
```python
emp.model_dump()              # → Python dict
emp.model_dump_json()         # → JSON string
Employee.model_validate({...})            # dict → validated model
Employee.model_validate_json('{"name": ...}')   # JSON string → validated model
Employee.model_json_schema()  # → the JSON Schema (very important for LLMs!)
```
> 📖 **New terms**
> - **Serialize:** convert an object into a format to store or send (dict / JSON text).
> - **Deserialize:** the reverse: text/dict → object.
> - **JSON Schema:** a standard JSON document that **describes** the structure of data. LLMs read this to know exactly what to output.
>
> 🕰️ Pydantic v1 names: `.dict()`, `.json()`, `parse_obj()`. In v2 these became `model_dump()`, `model_dump_json()`, `model_validate()`.

---

## 4. `Field`: Rules, Limits & Descriptions

`Field()` adds **extra rules and information** to each field.

> 📖 **New term: Constraint** = a rule a value must follow (e.g., "must be between 1 and 5").

```python
from pydantic import BaseModel, Field

class Product(BaseModel):
    name: str  = Field(..., min_length=2, max_length=50, description="Product name")
    price: float = Field(..., gt=0, description="Price in INR, must be positive")
    rating: int  = Field(default=3, ge=1, le=5, description="Star rating from 1 to 5")
    tags: list[str] = Field(default_factory=list, max_length=5, description="Up to 5 tags")
```

### 📏 Common `Field` options

| Option | Meaning | Example |
|---|---|---|
| `...` | **Required** (no default) | `Field(...)` |
| `default=` | Default value | `Field(default=3)` |
| `default_factory=` | Creates a fresh default each time (use for lists/dicts) | `Field(default_factory=list)` |
| `gt` / `ge` | Greater than / greater-or-equal | `ge=1` |
| `lt` / `le` | Less than / less-or-equal | `le=5` |
| `min_length` / `max_length` | Length limits for text or lists | `max_length=50` |
| `pattern` | Must match a regex pattern | `pattern=r"^\d{6}$"` (6-digit pincode) |
| `description` | Explains the field (**the LLM reads this!**) | `description="Price in INR"` |
| `examples` | Example values | `examples=["Laptop"]` |
| `alias` | Different name in the input JSON | `alias="productName"` |

> 📖 **New term: Regex (Regular Expression)** = a mini-language for text patterns. `^\d{6}$` means "exactly 6 digits".

```python
try:
    Product(name="X", price=-10, rating=9)
except ValidationError as e:
    print(e)
# name   → String should have at least 2 characters
# price  → Input should be greater than 0
# rating → Input should be less than or equal to 5
```

> 💡 **Pro tip:** For AI agents, `description` is **gold**. When this model is sent to an LLM as a schema, the descriptions become **instructions** for each field. Good descriptions mean better LLM output.

---

## 5. Advanced Types & Custom Validators (Pro)

### 5.1 `Literal` and `Enum`: allow only fixed choices
```python
from typing import Literal
from enum import Enum

class Priority(str, Enum):
    LOW = "low"
    MEDIUM = "medium"
    HIGH = "high"

class Ticket(BaseModel):
    category: Literal["billing", "technical", "general"]   # only these 3 strings
    priority: Priority
```
> 📖 **New terms**
> - **`Literal`:** the value must be **exactly one** of the listed values.
> - **Enum (enumeration):** a fixed set of named choices.
>
> 💡 These are **super useful in agents**: they stop the LLM from inventing categories like `"Billing Issue!!"`.

### 5.2 Custom validation with `@field_validator`
```python
from pydantic import field_validator

class Signup(BaseModel):
    username: str
    email: str

    @field_validator("username")
    @classmethod
    def no_spaces(cls, v: str) -> str:
        if " " in v:
            raise ValueError("username must not contain spaces")
        return v.lower()          # you can also CLEAN the value
```
> 📖 **New terms**
> - **Decorator (`@something`):** a line placed above a function that adds extra behaviour to it.
> - **`@classmethod` / `cls`:** a method that belongs to the class itself, not one object. Pydantic requires it for validators.

### 5.3 Checking multiple fields together: `@model_validator`
```python
from pydantic import model_validator
from datetime import date

class LeaveRequest(BaseModel):
    start: date
    end: date

    @model_validator(mode="after")
    def end_after_start(self):
        if self.end < self.start:
            raise ValueError("end date must be after start date")
        return self
```
`mode="after"` means "run after the individual fields are validated".

### 5.4 Ready-made special types
```python
from pydantic import EmailStr, HttpUrl   # EmailStr needs: pip install "pydantic[email]"

class Contact(BaseModel):
    email: EmailStr          # must be a valid email
    website: HttpUrl         # must be a valid URL
```

### 5.5 Strict mode & forbidding extra fields
```python
from pydantic import ConfigDict

class StrictUser(BaseModel):
    model_config = ConfigDict(strict=True, extra="forbid")
    name: str
    age: int

StrictUser(name="A", age="28")                 # ❌ strict: "28" is not an int
StrictUser(name="A", age=28, hacker="x")       # ❌ extra field not allowed
```
> 📖 **New terms**
> - **Strict mode:** turn off auto-conversion; the type must already be exactly right.
> - **`extra="forbid"`:** reject any field not defined in the model. Great for stopping the LLM from adding made-up fields.

---

## 6. Structured Output from LLMs

> 📖 **New term: Structured output** = making the LLM return data in an **exact, machine-readable format** (JSON matching your schema) instead of free text.

### 6.1 The 3 ways (from weakest to strongest)

| Way | How | Reliability |
|---|---|---|
| **1. Ask nicely in the prompt** | "Reply ONLY in JSON with keys name, age, city" | 😐 Often works, sometimes breaks |
| **2. Prompt + Pydantic validation** | Ask for JSON, then `Model.model_validate_json(text)`, retry on error | 🙂 Good |
| **3. Native structured output / tool calling** | Send the Pydantic **schema** to the LLM API; the model is constrained to follow it | 😃 Best |

> 📖 **New term: Native structured output** = a feature built into the LLM API itself where the model is **forced** to produce JSON matching your schema.

### 6.2 Example: OpenAI SDK + Pydantic
```python
from pydantic import BaseModel, Field
from openai import OpenAI

class PersonInfo(BaseModel):
    name: str = Field(description="Full name of the person")
    age: int  = Field(description="Age in years, as a number")
    city: str = Field(description="City where the person lives")

client = OpenAI()
completion = client.beta.chat.completions.parse(      # newer SDKs also offer client.chat.completions.parse
    model="gpt-4o-mini",
    messages=[{"role": "user", "content": "Ashish is 28 and lives in Hyderabad"}],
    response_format=PersonInfo,                        # 👈 pass the Pydantic model
)
person = completion.choices[0].message.parsed          # 👈 already a PersonInfo object!
print(person.name, person.age, person.city)            # Ashish 28 Hyderabad
```

### 6.3 Example: LangChain `with_structured_output` (used in Part 2)
```python
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
structured_llm = llm.with_structured_output(PersonInfo)

person = structured_llm.invoke("Ashish is 28 and lives in Hyderabad")
print(person)   # name='Ashish' age=28 city='Hyderabad'
```
> 📖 **New terms**
> - **`temperature`:** controls randomness. `0` = focused and consistent (best for structured data); higher = more creative.
> - **`invoke()`:** LangChain's standard method for "run this with this input".

### 6.4 Real-world example: support-ticket classifier
```python
from typing import Literal
from pydantic import BaseModel, Field

class TicketAnalysis(BaseModel):
    category: Literal["billing", "technical", "account", "other"]
    priority: Literal["low", "medium", "high"]
    sentiment: Literal["positive", "neutral", "negative"]
    summary: str = Field(max_length=200, description="One-line summary of the issue")
    needs_human: bool = Field(description="True if a human agent must handle it")

analyzer = llm.with_structured_output(TicketAnalysis)
result = analyzer.invoke("I was charged twice this month and nobody replies to my emails!!")
# category='billing' priority='high' sentiment='negative'
# summary='Customer double-charged, no response to emails' needs_human=True
```
Now your code can do: `if result.priority == "high": alert_team()`. **No text parsing needed.**

---

## 7. Why Validation Matters in Agents

### 7.1 🧨 Where things go wrong without validation

```
LLM output ──► Tool / API / Database
   │
   ├── wrong type   ("five" instead of 5)          → API rejects / crash
   ├── missing field (no "email")                  → DB insert fails
   ├── made-up value (category="urgent-ish")        → wrong routing
   ├── out of range  (discount=500%)                → 💸 business loss
   └── extra fields  ("admin": true)                → 🔓 security risk
```

> 📖 **New term: Hallucination** = the LLM confidently inventing wrong or made-up information. Validation can't make facts true, but it **catches outputs that break your rules** (wrong types, invented categories, impossible values).

### 7.2 🛡️ Pydantic as a guardrail at 3 places

| Where | What Pydantic checks | Example |
|---|---|---|
| **Tool inputs** | The arguments the LLM wants to pass to a tool | `amount` must be > 0 and ≤ 10,000 |
| **Tool outputs** | Data coming back from external APIs | API returned `price: null` → caught |
| **Final answer** | The agent's final structured result | Must match `TicketAnalysis` |

> 📖 **Reminder: Guardrail** = a safety check that keeps the agent inside safe limits. Pydantic is one of the simplest and strongest code-level guardrails.

### 7.3 🔁 The self-correction loop (pro pattern)
When validation fails, **send the error back to the LLM** and ask it to fix its output.

```python
import json
from pydantic import ValidationError

def get_valid_output(llm, prompt: str, model_cls, max_tries: int = 3):
    messages = [("system", f"Reply ONLY with JSON matching this schema:\n"
                           f"{json.dumps(model_cls.model_json_schema())}"),
                ("user", prompt)]
    for attempt in range(max_tries):
        raw = llm.invoke(messages).content
        try:
            return model_cls.model_validate_json(raw)          # ✅ valid → done
        except ValidationError as e:
            messages.append(("assistant", raw))
            messages.append(("user", f"Your JSON was invalid:\n{e}\nFix it and reply with JSON only."))
    raise ValueError("LLM could not produce valid output")
```
This is exactly what libraries like **Instructor** and **PydanticAI** automate for you.

> 📖 **New terms**
> - **Instructor:** a library that adds Pydantic-based structured output + automatic retries to LLM calls.
> - **PydanticAI:** an agent framework from the Pydantic team, built around typed, validated inputs/outputs.

### 7.4 🔑 One-liner
> **Pydantic turns unpredictable LLM text into trusted, typed data, so tools, APIs, and databases downstream never receive garbage.**

---

# PART 2: BUILDING A SINGLE AI AGENT SYSTEM

## 8. Prerequisites: LangChain Basics

The course says you need a solid understanding of **LangChain**. Here's the minimum you need for this README, explained simply. (For deeper learning, the instructor recommends his separate *"Ultimate LangChain Crash Course"*.)

> 📖 **New term: LangChain** = a popular Python framework that gives ready-made building blocks for LLM apps: a common way to call any LLM, define tools, manage messages, and build agents. It saves you from writing everything from scratch.
>
> 📖 **New term: Framework** = a ready-made structure of code you build on top of, like a house frame you add rooms to.

### 8.1 Install (Windows PowerShell)
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install langchain langchain-openai langgraph pydantic python-dotenv fastapi uvicorn
```

> 📖 **New terms**
> - **Virtual environment (venv):** an isolated folder of Python packages just for this project.
> - **`python-dotenv`:** loads secrets (like API keys) from a `.env` file into environment variables.
> - **LangGraph:** LangChain's companion library for building agents as a **graph** of steps (nodes) with loops and state. Most modern LangChain agents run on it.

### 8.2 Store your API key safely
Create a file named `.env` (and **add it to `.gitignore`**, never push it to GitHub):
```
OPENAI_API_KEY=sk-your-key-here
```

### 8.3 The 5 LangChain building blocks you'll use

| Block | What it is | Code |
|---|---|---|
| **Chat model** | A wrapper around an LLM | `ChatOpenAI(model="gpt-4o-mini")` |
| **Messages** | Typed chat messages | `SystemMessage`, `HumanMessage`, `AIMessage`, `ToolMessage` |
| **Tools** | Python functions the LLM can call | `@tool` decorator |
| **`bind_tools()`** | Tells the LLM which tools exist | `llm.bind_tools([tool1, tool2])` |
| **`with_structured_output()`** | Forces Pydantic-shaped output | `llm.with_structured_output(MyModel)` |

```python
from dotenv import load_dotenv
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage, HumanMessage

load_dotenv()                                   # reads .env
llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)

reply = llm.invoke([
    SystemMessage(content="You are a friendly tutor."),
    HumanMessage(content="Explain an API in one line."),
])
print(reply.content)
```

> 📖 **New terms**
> - **SystemMessage:** the hidden instruction that sets the AI's role and rules.
> - **HumanMessage:** what the user says.
> - **AIMessage:** what the LLM replies (may include tool-call requests).
> - **ToolMessage:** the result of a tool, sent back to the LLM.
>
> 💡 Want a different LLM? Swap `ChatOpenAI` for `ChatAnthropic` (Claude), `AzureChatOpenAI`, `ChatBedrock` (AWS), etc. **The rest of your code stays the same.** That's LangChain's main benefit.

---

## 9. What is a Single AI Agent System?

### 9.1 Definition
> **A single AI agent system** = **one** agent (one LLM brain) that receives a **high-level goal** from the user and **autonomously plans, decides, and executes** a sequence of actions (using tools) until the goal is achieved.

> 📖 **New terms**
> - **High-level goal:** a big-picture request, not detailed steps. ("Plan my Goa trip under ₹20k", not "search flights, then search hotels...")
> - **Autonomously:** on its own, without step-by-step human instructions.
> - **Single vs multi-agent:** *single* = one brain does everything; *multi-agent* = several specialist agents cooperate (a future topic).

### 9.2 🧑‍🔧 Analogy: A skilled freelancer
You hire one freelancer: *"Build me a simple website."*
They **plan** (pages needed) → **decide** (which tools: Figma, VS Code, hosting) → **execute** (design, code, deploy) → **check** (test it) → **deliver**.
One person, many tools, full ownership of the goal. That's a single agent.

### 9.3 The intelligent loop
```
                ┌───────────────────────────────────────────┐
  User goal ──► │ 🧠 LLM: REASON & PLAN                       │
                │   "What do I know? What do I need next?"    │
                └──────────────┬────────────────────────────┘
                               │
                 needs a tool? ├── No ──► ✅ Final answer (validated by Pydantic)
                               │
                              Yes
                               ▼
                ┌───────────────────────────────────────────┐
                │ 🛠️ ACT: execute tool(s)                     │
                │   (validated inputs via Pydantic)           │
                └──────────────┬────────────────────────────┘
                               ▼
                ┌───────────────────────────────────────────┐
                │ 👀 OBSERVE: add tool result to CONTEXT      │
                └──────────────┬────────────────────────────┘
                               └──────► back to REASON & PLAN (loop)
```

### 9.4 Three key abilities (from the course outline)

| Ability | Meaning | How we build it |
|---|---|---|
| **Tool usage** | Integrate external tools, APIs, knowledge sources | `@tool` functions + `bind_tools()` |
| **Context management** | Remember what happened so far and reason across steps | Message history (state) + memory/checkpointer |
| **Adaptation** | Change the plan when new info arrives (errors, surprising results) | Feed tool results/errors back into the loop |

> 📖 **New term: Context management** = deciding **what information the LLM sees** at each step: the goal, past messages, tool results, and memory, while keeping it within the **context window** (the max text the LLM can read at once).

---

## 10. Build It End-to-End (Step by Step)

### 🎯 Our project: "Travel Planner Agent"
**Goal example:** *"Plan a 2-day trip to Goa from Hyderabad for 2 people under ₹20,000."*

The agent will:
1. Check the **weather** (tool)
2. Look up **flight prices** (tool)
3. Look up **hotels** (tool)
4. **Calculate** the total budget (tool)
5. Search a small **knowledge base** of travel tips (tool, mini-RAG)
6. Return a **validated, structured trip plan** (Pydantic)

### 📁 Project structure
```
travel_agent/
├── .env                 # API key (never commit!)
├── .gitignore
├── requirements.txt
├── schemas.py           # Pydantic models
├── tools.py             # Agent tools
├── agent.py             # The agent loop (core)
├── main.py              # Run from terminal
├── api.py               # FastAPI web server (for deployment)
└── Dockerfile           # Container recipe (for deployment)
```

---

### Step 1: `schemas.py`: define data with Pydantic

```python
# schemas.py
from typing import Literal
from pydantic import BaseModel, Field


# ---------- Tool INPUT schemas (validate what the LLM sends to tools) ----------
class WeatherInput(BaseModel):
    city: str = Field(..., min_length=2, description="City name, e.g. 'Goa'")

class FlightInput(BaseModel):
    origin: str = Field(..., description="Departure city")
    destination: str = Field(..., description="Arrival city")
    passengers: int = Field(1, ge=1, le=9, description="Number of passengers (1-9)")

class HotelInput(BaseModel):
    city: str = Field(..., description="City to find hotels in")
    nights: int = Field(..., ge=1, le=30, description="Number of nights")
    max_price_per_night: int = Field(..., gt=0, description="Budget per night in INR")

class CalculatorInput(BaseModel):
    expression: str = Field(..., pattern=r"^[0-9+\-*/(). ]+$",
                            description="Math expression using numbers and + - * / ( ) only")


# ---------- FINAL OUTPUT schema (validate the agent's final answer) ----------
class DayPlan(BaseModel):
    day: int = Field(..., ge=1)
    activities: list[str] = Field(..., min_length=1, max_length=6)

class TripPlan(BaseModel):
    destination: str
    weather_summary: str
    flight_cost: int = Field(..., ge=0, description="Total flight cost in INR")
    hotel_name: str
    hotel_cost: int = Field(..., ge=0, description="Total hotel cost in INR")
    total_cost: int = Field(..., ge=0, description="Flights + hotel, in INR")
    within_budget: bool
    itinerary: list[DayPlan]
    confidence: Literal["low", "medium", "high"]
```

> 💡 Notice the `pattern` on `CalculatorInput`: Pydantic itself blocks anything that isn't math (e.g., `os.remove(...)`). **Validation = guardrail.**

---

### Step 2: `tools.py`: give the agent hands

> 📖 **New terms**
> - **`@tool` decorator:** turns a normal Python function into a LangChain tool. LangChain reads the function name, docstring, and `args_schema` to tell the LLM how to use it.
> - **Docstring:** the text in triple quotes right under a function definition. **The LLM reads it** to understand what the tool does.
> - **`args_schema`:** the Pydantic model that validates the tool's inputs.
> - **Mock / fake data:** pretend data used for learning/testing instead of a real API.

```python
# tools.py
import ast, operator
from langchain_core.tools import tool
from schemas import WeatherInput, FlightInput, HotelInput, CalculatorInput

# 🌦️ Tool 1: weather (mock, swap with a real weather API later)
@tool(args_schema=WeatherInput)
def get_weather(city: str) -> str:
    """Get the current weather forecast for a city."""
    data = {"goa": "31°C, sunny, light breeze", "hyderabad": "29°C, partly cloudy"}
    return data.get(city.lower(), f"No weather data for {city}")

# ✈️ Tool 2: flights (mock)
@tool(args_schema=FlightInput)
def search_flights(origin: str, destination: str, passengers: int = 1) -> str:
    """Search the cheapest round-trip flight price in INR for the given passengers."""
    price_per_person = 6500
    return f"Cheapest round trip {origin}->{destination}: ₹{price_per_person} per person, total ₹{price_per_person * passengers}"

# 🏨 Tool 3: hotels (mock)
@tool(args_schema=HotelInput)
def search_hotels(city: str, nights: int, max_price_per_night: int) -> str:
    """Find a hotel in a city within the per-night budget. Returns name and total cost."""
    hotels = [("Sea Breeze Inn", 2800), ("Palm Grove Resort", 4500), ("Budget Stay", 1500)]
    options = [h for h in hotels if h[1] <= max_price_per_night]
    if not options:
        return "No hotels found within that budget."
    name, price = max(options, key=lambda h: h[1])          # best hotel within budget
    return f"{name}: ₹{price}/night, total ₹{price * nights} for {nights} nights"

# 🧮 Tool 4: safe calculator (no eval!)
_OPS = {ast.Add: operator.add, ast.Sub: operator.sub,
        ast.Mult: operator.mul, ast.Div: operator.truediv}

def _safe_eval(node):
    if isinstance(node, ast.Constant):
        return node.value
    if isinstance(node, ast.BinOp) and type(node.op) in _OPS:
        return _OPS[type(node.op)](_safe_eval(node.left), _safe_eval(node.right))
    raise ValueError("Unsupported expression")

@tool(args_schema=CalculatorInput)
def calculator(expression: str) -> str:
    """Calculate a math expression exactly. Use this for ALL arithmetic."""
    return str(_safe_eval(ast.parse(expression, mode="eval").body))

# 📚 Tool 5: knowledge base (a tiny "RAG" stand-in)
TRAVEL_TIPS = {
    "goa": "Best beaches: Baga, Palolem. Rent a scooter (~₹400/day). Visit Fort Aguada at sunset.",
}

@tool
def travel_knowledge(city: str) -> str:
    """Look up insider travel tips and must-see places for a city from the company knowledge base."""
    return TRAVEL_TIPS.get(city.lower(), "No tips found.")

ALL_TOOLS = [get_weather, search_flights, search_hotels, calculator, travel_knowledge]
```

> 📖 **New terms**
> - **`ast` (Abstract Syntax Tree):** Python's way of reading code as a tree structure. We use it to **safely** calculate math without running arbitrary code.
> - **Knowledge source / knowledge base:** a store of information (docs, FAQs) the agent can search. In real projects this is a **vector database with RAG** (search docs by meaning, then answer).

---

### Step 3: `agent.py`: the brain + loop (built by hand, to truly understand)

```python
# agent.py
import json
from dotenv import load_dotenv
from pydantic import ValidationError
from langchain_openai import ChatOpenAI
from langchain_core.messages import SystemMessage, HumanMessage, ToolMessage
from tools import ALL_TOOLS
from schemas import TripPlan

load_dotenv()

SYSTEM_PROMPT = """You are TripMate, a careful travel-planning agent.
Process:
1. PLAN: think about which information you need.
2. Use tools to gather facts. NEVER guess prices or weather; always use tools.
3. Use the calculator for every sum.
4. If a tool fails or returns nothing, adapt (try other inputs or explain).
5. When you have everything, reply with a short final summary (no tool calls)."""

MAX_STEPS = 8                                       # 🛡️ guardrail: stop runaway loops

llm = ChatOpenAI(model="gpt-4o-mini", temperature=0)
llm_with_tools = llm.bind_tools(ALL_TOOLS)          # 🛠️ tell the LLM about the tools
tools_by_name = {t.name: t for t in ALL_TOOLS}


def run_agent(goal: str, history: list | None = None) -> tuple[TripPlan, list]:
    # 🧠 CONTEXT = system prompt + earlier conversation + new goal
    messages = [SystemMessage(content=SYSTEM_PROMPT)] + (history or []) + [HumanMessage(content=goal)]

    for step in range(1, MAX_STEPS + 1):
        ai_msg = llm_with_tools.invoke(messages)     # 🧠 REASON + PLAN
        messages.append(ai_msg)

        if not ai_msg.tool_calls:                    # ✅ nothing more to do
            print(f"[step {step}] finished reasoning")
            break

        for call in ai_msg.tool_calls:               # 🛠️ ACT
            tool = tools_by_name.get(call["name"])
            try:
                result = tool.invoke(call["args"])   # Pydantic validates args here
            except ValidationError as e:             # 🔁 ADAPT: tell the LLM what was wrong
                result = f"Invalid tool input: {e.errors()[0]['msg']}. Fix the arguments."
            except Exception as e:
                result = f"Tool error: {e}"
            print(f"[step {step}] 🛠️ {call['name']}({call['args']}) → {result}")
            messages.append(ToolMessage(content=str(result), tool_call_id=call["id"]))  # 👀 OBSERVE
    else:
        print("⚠️ Max steps reached")

    # 📦 Turn everything gathered into a VALIDATED structured result
    final_plan = llm.with_structured_output(TripPlan).invoke(
        messages + [HumanMessage(content="Now produce the final trip plan using ONLY facts gathered above.")]
    )
    return final_plan, messages[1:]                  # return plan + updated history (memory)
```

#### 🔍 What each part does

| Code | Agent concept |
|---|---|
| `SYSTEM_PROMPT` | Instructions + planning guidance |
| `messages` list | **Context / short-term memory** |
| `llm_with_tools.invoke(messages)` | **Reason & plan**: decide the next tool or finish |
| `tool.invoke(call["args"])` | **Act** (inputs validated by Pydantic `args_schema`) |
| `except ValidationError` | **Adapt**: errors go back to the LLM to self-correct |
| `ToolMessage(...)` | **Observe**: result added to context |
| `MAX_STEPS` | Guardrail against infinite loops |
| `with_structured_output(TripPlan)` | **Validated final output** |
| returning `messages` | Keeps **conversation memory** for follow-ups |

---

### Step 4: `main.py`: run it

```python
# main.py
from agent import run_agent

goal = "Plan a 2-day trip to Goa from Hyderabad for 2 people under ₹20,000."
plan, history = run_agent(goal)

print("\n===== TRIP PLAN =====")
print(plan.model_dump_json(indent=2))

# 🧠 Follow-up uses the same history (context management!)
plan2, history = run_agent("Make it 3 days instead, same budget.", history)
print(plan2.total_cost, plan2.within_budget)
```

```powershell
python main.py
```

**Example run (your exact wording will vary):**
```
[step 1] 🛠️ get_weather({'city': 'Goa'}) → 31°C, sunny, light breeze
[step 1] 🛠️ search_flights({'origin': 'Hyderabad', 'destination': 'Goa', 'passengers': 2}) → ... total ₹13000
[step 1] 🛠️ travel_knowledge({'city': 'Goa'}) → Best beaches: Baga, Palolem...
[step 2] 🛠️ search_hotels({'city': 'Goa', 'nights': 2, 'max_price_per_night': 3500}) → Sea Breeze Inn: ₹2800/night, total ₹5600
[step 3] 🛠️ calculator({'expression': '13000 + 5600'}) → 18600
[step 4] finished reasoning

===== TRIP PLAN =====
{
  "destination": "Goa",
  "weather_summary": "31°C, sunny, light breeze",
  "flight_cost": 13000,
  "hotel_name": "Sea Breeze Inn",
  "hotel_cost": 5600,
  "total_cost": 18600,
  "within_budget": true,
  "itinerary": [
    {"day": 1, "activities": ["Arrive", "Baga Beach", "Fort Aguada sunset"]},
    {"day": 2, "activities": ["Rent a scooter", "Palolem Beach", "Fly back"]}
  ],
  "confidence": "high"
}
```

See the intelligent loop in action:
- **Step 1:** it called 3 tools at once (parallel tool calls).
- **Step 2:** it **adapted**: flights cost ₹13,000, so it searched hotels with a lower per-night budget to fit ₹20,000.
- **Step 3:** it used the calculator instead of guessing.
- **Final:** clean JSON, guaranteed to match `TripPlan`.

---

### Step 5: The shortcut, LangChain's prebuilt agent

Once you understand the manual loop, LangChain/LangGraph can build the same loop for you in a few lines.

```python
# agent_prebuilt.py
from dotenv import load_dotenv
from langgraph.prebuilt import create_react_agent
from langgraph.checkpoint.memory import MemorySaver
from langchain_openai import ChatOpenAI
from tools import ALL_TOOLS
from schemas import TripPlan

load_dotenv()

agent = create_react_agent(
    model=ChatOpenAI(model="gpt-4o-mini", temperature=0),
    tools=ALL_TOOLS,
    prompt="You are TripMate. Always use tools for facts and the calculator for math.",
    response_format=TripPlan,          # 👈 structured final output
    checkpointer=MemorySaver(),        # 👈 remembers conversations
)

config = {"configurable": {"thread_id": "ashish-trip-1"}}   # one conversation = one thread
result = agent.invoke(
    {"messages": [("user", "Plan a 2-day Goa trip from Hyderabad for 2 under ₹20,000")]},
    config,
)
print(result["structured_response"])
```

> 📖 **New terms**
> - **ReAct agent:** an agent that loops **Re**ason → **Act** → observe (the same loop we built by hand).
> - **Prebuilt:** ready-made by the library, so you don't write the loop yourself.
> - **Checkpointer:** saves the agent's state (messages) after each step, so it can **remember** past turns or resume later. `MemorySaver` keeps it in RAM; production uses a database (e.g., Postgres, Redis).
> - **Thread ID:** an ID for one conversation. Same ID = same memory; new ID = fresh start.
> - **State:** all the data the agent is carrying right now (messages, results, etc.).
>
> ⚠️ **Version note:** LangChain changes fast. In LangChain **v1.x**, the recommended function is `from langchain.agents import create_agent` (similar arguments). If an import fails, check the current LangChain docs for your installed version. **The concepts stay the same.**

| Manual loop (Step 3) | Prebuilt agent (Step 5) |
|---|---|
| You see and control every step | Fewer lines, less control |
| Best for **learning** and custom logic | Best for **speed** and standard agents |
| You manage memory yourself | Checkpointer handles memory |

---

### Step 6: Context management (pro level)

As the conversation grows, the message list gets **too long** for the context window (and expensive). Strategies:

| Strategy | How | Example |
|---|---|---|
| **Sliding window** | Keep only the last N messages | Keep the last 20 messages |
| **Summarization** | Replace old messages with a short summary | "Earlier: user planned Goa trip, budget ₹20k" |
| **Trim tool outputs** | Keep only the key parts of big tool results | Store the top 3 search results, not 50 |
| **Long-term memory store** | Save facts to a DB, retrieve when relevant | "User prefers veg food" → vector DB |
| **Scratchpad / state fields** | Keep structured facts outside messages | `state["budget"] = 20000` |

```python
from langchain_core.messages import trim_messages

trimmed = trim_messages(
    messages,
    max_tokens=3000,          # keep within budget
    strategy="last",          # keep the most recent
    token_counter=llm,        # use the model to count tokens
    include_system=True,      # never drop the system prompt
    start_on="human",
)
```

> 📖 **New terms**
> - **Token:** a small chunk of text (≈ ¾ of a word). LLM limits and pricing are counted in tokens.
> - **Sliding window:** keep only the most recent part of a conversation, like a window that moves forward.
> - **Scratchpad:** a working area where the agent keeps notes/intermediate results.

---

## 11. Deployment Overview (Docker + Cloud)

The course previews this: after building the agent, we **containerize** it and **deploy** it to the cloud for production use.

> 📖 **New terms**
> - **Deployment:** putting your app on a server so real users can use it 24/7.
> - **Production:** the live environment real users use (vs. *development* on your laptop).
> - **Container:** a lightweight, portable box that packages your code + Python + libraries, so it runs **the same everywhere**.
> - **Docker:** the most popular tool for building and running containers.
> - **Image (Docker image):** the recipe/snapshot used to start containers.
> - **Containerize:** package your app into a Docker image.

### 🚚 Analogy: Shipping container
Before shipping containers, every cargo was packed differently. Now everything goes in a **standard box** that fits any ship, train, or truck. Docker does the same for software: **"It works on my machine" → it works everywhere.**

### 11.1 Wrap the agent as a web API (`api.py`)
```python
# api.py
from fastapi import FastAPI
from pydantic import BaseModel, Field
from agent import run_agent
from schemas import TripPlan

app = FastAPI(title="TripMate Agent API")

class PlanRequest(BaseModel):                 # 👈 Pydantic validates incoming requests too!
    goal: str = Field(..., min_length=10, max_length=500)

@app.post("/plan", response_model=TripPlan)   # 👈 and the response
def plan_trip(req: PlanRequest):
    plan, _ = run_agent(req.goal)
    return plan

@app.get("/health")
def health():
    return {"status": "ok"}
```
```powershell
uvicorn api:app --reload
# open http://127.0.0.1:8000/docs → interactive API page (auto-generated from Pydantic!)
```
> 📖 **New terms**
> - **FastAPI:** a Python web framework for building APIs; it uses Pydantic for request/response validation.
> - **Uvicorn:** the server program that runs a FastAPI app.
> - **`/health` endpoint:** a simple URL the cloud uses to check "is the app alive?"

### 11.2 `requirements.txt`
```
langchain
langchain-openai
langgraph
pydantic
python-dotenv
fastapi
uvicorn
```

### 11.3 `Dockerfile`
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8000
CMD ["uvicorn", "api:app", "--host", "0.0.0.0", "--port", "8000"]
```

```powershell
docker build -t tripmate-agent .
docker run -p 8000:8000 --env-file .env tripmate-agent
```
> ⚠️ Add `.env` to a **`.dockerignore`** file so your secret key isn't baked into the image. Pass it at runtime instead (as above), or use a cloud secret manager.

> 📖 **New terms**
> - **Dockerfile:** a text file with step-by-step instructions to build a Docker image.
> - **Port:** a numbered "door" on a computer for network traffic (8000 here).
> - **`.dockerignore`:** files Docker should NOT copy into the image.

### 11.4 Where to deploy (cloud options)

| Cloud | Simple container service | Store secrets in |
|---|---|---|
| **AWS** | ECS on **Fargate**, App Runner (image stored in **ECR**) | Secrets Manager / SSM Parameter Store |
| **Azure** | Azure Container Apps, App Service | Azure Key Vault |
| **GCP** | Cloud Run | Secret Manager |

> 💡 You already know ECS/ECR/Fargate from your AWS learning: push the image to **ECR**, run it on **ECS Fargate** behind a load balancer. Same flow as any container app!

### 11.5 Production checklist ✅
- 🔐 API keys in a **secret manager**, never in code or images
- ⏱️ **Timeouts** and **max steps** on the agent
- 🛡️ **Pydantic validation** on every input/output
- 📊 **Logging + tracing** (LangSmith / Langfuse / CloudWatch)
- 🚦 **Rate limiting** so users can't spam expensive LLM calls
- 💾 **Persistent memory** (Postgres/Redis checkpointer) instead of in-RAM
- ⚡ **Async** endpoints (`async def` + `ainvoke`) for many concurrent users
- 🧪 **Evaluation tests** before each release

---

## 12. System Revision: Architecture Recap

### 🏗️ The complete single-agent architecture
```
                   ┌──────────────────────── USER ────────────────────────┐
                   │             "Plan a Goa trip under ₹20k"             │
                   └──────────────────────────┬───────────────────────────┘
                                              ▼
                         🌐 FastAPI  (Pydantic validates request)
                                              ▼
 ┌────────────────────────────── SINGLE AGENT ───────────────────────────────┐
 │  📜 System prompt (role, rules, process)                                  │
 │  💾 Context: message history + checkpointer memory                        │
 │                                                                           │
 │     ┌─────────► 🧠 LLM: REASON + PLAN ─────────┐                          │
 │     │                                         │ tool calls                │
 │  👀 OBSERVE                                   ▼                           │
 │  (ToolMessage) ◄──── 🛠️ ACT: tools (Pydantic-validated inputs)            │
 │                        weather · flights · hotels · calculator · KB/RAG   │
 │                                                                           │
 │  🛡️ Guardrails: max steps, timeouts, safe calculator, validation          │
 │  📦 Final: with_structured_output(TripPlan) → validated JSON              │
 └───────────────────────────────────────────────────────────────────────────┘
                                              ▼
                         🌐 FastAPI returns TripPlan JSON
                                              ▼
                         🐳 Docker container → ☁️ Cloud (ECS / Container Apps / Cloud Run)
```

### 🔁 The intelligent loop in one line
> **Plan → Reason → Act (tool) → Observe → Adapt → repeat → Validated final answer.**

---

## 📝 Quick Revision Cheat Sheet

| Concept | One-liner |
|---|---|
| **Pydantic** | Library that validates data against Python type-hinted models |
| **`BaseModel`** | Parent class for defining a data model |
| **`Field`** | Adds rules (min/max, pattern), defaults, and descriptions |
| **Coercion** | Auto-convert safe values (`"28"` → `28`) |
| **ValidationError** | Raised when data breaks the model's rules |
| **`model_dump` / `model_validate`** | Model → dict / dict → validated model |
| **`model_json_schema()`** | Generates the JSON Schema the LLM can follow |
| **`Literal` / Enum** | Allow only fixed choices |
| **`field_validator` / `model_validator`** | Custom rules for one field / across fields |
| **Structured output** | LLM returns JSON matching your schema |
| **`with_structured_output()`** | LangChain way to get Pydantic objects from an LLM |
| **Self-correction loop** | Send the validation error back to the LLM to fix |
| **Single agent** | One LLM that plans, decides, and executes with tools toward a goal |
| **`@tool` + `args_schema`** | Define a tool + validate its inputs |
| **`bind_tools()`** | Tell the LLM which tools exist |
| **ToolMessage** | Tool result sent back to the LLM |
| **Context management** | Controlling what the LLM sees (history, trimming, memory) |
| **Checkpointer / thread_id** | Saves conversation state / identifies one conversation |
| **FastAPI** | Serves the agent as a web API (Pydantic built in) |
| **Docker** | Packages the agent to run the same anywhere |

---

## 🎤 Interview Questions

1. **Why is Pydantic important for AI agents?**
   → LLM output is unpredictable; Pydantic enforces exact types, required fields, and value rules before data reaches tools, APIs, or databases.

2. **What's the difference between `BaseModel` and `Field`?**
   → `BaseModel` defines the model (structure); `Field` adds per-field constraints, defaults, and descriptions.

3. **How do you get structured output from an LLM?**
   → Native structured output / tool calling with a Pydantic schema (e.g., `with_structured_output`, OpenAI `parse`), optionally with a validate-and-retry loop.

4. **Does validation prevent hallucination?**
   → It doesn't make facts true, but it catches outputs that break the schema (wrong types, invented categories, out-of-range values) and blocks them from flowing downstream.

5. **Why are field descriptions important?**
   → They're included in the JSON schema the LLM sees, acting as instructions for each field.

6. **How does a tool's `args_schema` help?**
   → It validates the LLM's tool arguments; invalid ones raise errors that can be fed back for self-correction.

7. **What is a single AI agent system?**
   → One LLM-driven agent that takes a high-level goal and autonomously plans, picks tools, executes, observes, and adapts until done.

8. **How does an agent maintain context?**
   → Through the message history (system, human, AI, tool messages) passed each turn, plus checkpointers/memory stores and trimming or summarization.

9. **Manual agent loop vs `create_react_agent`?**
   → Manual gives full control and understanding; prebuilt is faster to build and includes memory and structured output support.

10. **How do you deploy an agent?**
    → Wrap it in an API (FastAPI), containerize with Docker, push to a registry (ECR), and run on a container service (ECS Fargate / Container Apps / Cloud Run) with secrets, logging, and limits.

---

## 13. Glossary: Every Term Explained Simply

| Term | Simple meaning |
|---|---|
| **AIMessage / HumanMessage / SystemMessage / ToolMessage** | LLM reply / user text / hidden instructions / tool result |
| **Alias** | Alternate name for a field in input data |
| **`args_schema`** | Pydantic model validating a tool's inputs |
| **`ast`** | Python module for reading code safely as a tree |
| **Autonomously** | On its own, without step-by-step instructions |
| **`BaseModel`** | Pydantic parent class for data models |
| **`bind_tools()`** | Tells the LLM which tools it can call |
| **Checkpointer** | Saves agent state so it can remember/resume |
| **Class / Inherit** | Blueprint for objects / borrowing abilities from a parent class |
| **Coercion** | Automatic safe type conversion |
| **Constraint** | A rule a value must follow |
| **Container / Containerize** | Portable package of app + dependencies / packing an app into one |
| **Context management** | Controlling what info the LLM sees each step |
| **Context window** | Max text the LLM can read at once |
| **Decorator** | `@something` line adding behaviour to a function |
| **Default / `default_factory`** | Value used if none given / function creating a fresh default |
| **Deployment / Production** | Putting an app live / the live environment |
| **Deserialize / Serialize** | Text → object / object → text |
| **Docker / Dockerfile / Image** | Container tool / build recipe / built snapshot |
| **`.dockerignore` / `.gitignore`** | Files Docker / Git should skip |
| **Docstring** | Function description in triple quotes (LLM reads it for tools) |
| **Downstream service** | System that receives your output next |
| **Enum / `Literal`** | Fixed set of allowed choices |
| **`extra="forbid"`** | Reject unknown fields |
| **FastAPI / Uvicorn** | Python API framework / server that runs it |
| **Field** | Pydantic function to add rules & descriptions |
| **`field_validator` / `model_validator`** | Custom check for one field / whole model |
| **Framework** | Ready-made code structure you build on |
| **Hallucination** | LLM inventing wrong information |
| **High-level goal** | Big-picture request without detailed steps |
| **Instructor / PydanticAI** | Libraries for validated LLM output / typed agents |
| **`invoke()`** | LangChain method to run a model/tool/agent |
| **JSON Schema** | Standard description of a JSON structure |
| **Knowledge base** | Store of information the agent can search |
| **LangChain / LangGraph** | LLM app framework / graph-based agent library |
| **Mock data** | Fake data for learning/testing |
| **Nested model** | Model inside another model |
| **`Optional` / `None`** | Value may be missing / Python's empty value |
| **Parse** | Convert text into a usable structure |
| **Port** | Numbered network "door" on a computer |
| **Prebuilt agent** | Ready-made agent loop from a library |
| **Pydantic** | Python data-validation library |
| **ReAct** | Reason → Act → Observe loop |
| **Regex / `pattern`** | Text-pattern language / Field rule using it |
| **Schema** | Blueprint of data structure |
| **Scratchpad** | Agent's working notes area |
| **Single agent** | One LLM brain handling the whole goal |
| **Sliding window** | Keep only recent messages |
| **State** | All data the agent currently carries |
| **Strict mode** | No auto-conversion; exact types only |
| **Structured output** | LLM output that matches an exact schema |
| **`temperature`** | Randomness control for LLM output |
| **Thread ID** | ID of one conversation's memory |
| **Token** | Small piece of text counted by LLMs |
| **`@tool`** | Decorator turning a function into an agent tool |
| **Type / Type hint** | Kind of value / declaring it in code (`age: int`) |
| **Validation / ValidationError** | Checking data / error when the check fails |
| **Virtual environment** | Isolated Python packages per project |

---

> 🚀 **Next topics you might explore:** LangGraph in depth (nodes, edges, state), Agent Memory, Multi-Agent Systems, MCP, Evaluation & Observability.
>
> *Happy learning! 🎉*
