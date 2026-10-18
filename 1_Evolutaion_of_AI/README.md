# 🤖 Agentic AI:

## 📚 Table of Contents

1. [What is Agentic AI?](#1-what-is-agentic-ai)
2. [Generative AI vs Agentic AI](#2-generative-ai-vs-agentic-ai)
3. [LLMs, Knowledge Cutoff, RAG, Fine-Tuning & Semantic Search](#3-llms-knowledge-cutoff-rag-fine-tuning--semantic-search)
4. [Problems with RAG and Why Agents Came](#4-problems-with-rag--why-agents-came)
5. [How Early Agents Worked (The Search Tool Example)](#5-how-early-agents-worked--the-search-tool-example)
6. [Part 1 of an Agent: Planning](#6-first-part-of-an-agent--planning)
7. [Pro Level: Putting It All Together](#7-pro-level--putting-it-all-together)
8. [Quick Revision Cheat Sheet](#8-quick-revision-cheat-sheet)
9. [Interview Questions](#9-interview-questions)

---

## 1. What is Agentic AI?

### 🧒 The Simple Definition

**Agentic AI** is AI that can **think, plan, and take actions on its own** to finish a goal, instead of only answering a question.

The word **"Agent"** means *someone who acts on your behalf*.
- A **travel agent** books your flights and hotels for you.
- A **real estate agent** finds houses and talks to sellers for you.
- An **AI agent** does tasks for you: it searches, calculates, sends emails, writes files, calls APIs, and so on.

### 🍕 Real-Life Example

Imagine two friends:

| Friend A (Normal AI) | Friend B (Agentic AI) |
|---|---|
| You: "I'm hungry, what should I eat?" | You: "I'm hungry, get me food." |
| Friend A: "You could try pizza from Domino's." | Friend B: checks your favourite food → opens Swiggy → compares prices → orders pizza → tells you "Pizza arriving in 30 mins 🍕" |
| **Gives advice only** | **Actually does the work** |

Friend B is an **agent**.

### 🧩 The 4 Superpowers of an AI Agent

| Superpower | Meaning | Example |
|---|---|---|
| 🧠 **Reasoning** | Thinks about the problem | "User wants weather, so I need live data" |
| 📋 **Planning** | Breaks the goal into steps | Step 1: search, Step 2: summarize |
| 🛠️ **Tool Use** | Uses outside tools | Google search, calculator, database, email |
| 💾 **Memory** | Remembers past info | "User likes vegetarian food" |

Some people add a 5th one:
| 🔁 **Reflection / Loop** | Checks its own work and retries if wrong | "Result looks incomplete, let me search again" |

### 🔑 One-Liner
> **Agentic AI = an LLM brain + planning + tools + memory, working in a loop until the goal is complete.**

---

## 2. Generative AI vs Agentic AI

### 🎨 What is Generative AI (GenAI)?

**Generative AI** is AI that **generates (creates) new content**: text, images, code, music, or video, based on what you ask.

Examples:
- ChatGPT writing an email ✉️
- DALL·E / Midjourney creating an image 🖼️
- GitHub Copilot writing code 💻

It works like this:

```
You (Prompt)  ──►  GenAI Model  ──►  Output (text / image / code)
```

**One question → one answer.** That's it. It does not go out and *do* anything.

### 🤖 What is Agentic AI?

Agentic AI **uses** Generative AI as its *brain*, but adds the ability to **plan and take actions**.

```
You (Goal) ──► Agent ──► Plan ──► Use Tool 1 ──► Use Tool 2 ──► Check result ──► Final Answer / Action done
                  ▲                                                   │
                  └───────────────── retry if needed ─────────────────┘
```

### ⚖️ Side-by-Side Comparison

| Feature | Generative AI | Agentic AI |
|---|---|---|
| **Main job** | Create content | Complete a goal |
| **Works how?** | Single step: prompt → response | Multi-step: plan → act → observe → repeat |
| **Uses tools?** | ❌ No (by itself) | ✅ Yes (search, APIs, DB, email...) |
| **Live / new data?** | ❌ Only what it learned in training | ✅ Can fetch fresh data |
| **Takes actions?** | ❌ Only talks | ✅ Books, sends, writes, updates |
| **Autonomy** | Low: waits for every instruction | High: decides next steps by itself |
| **Memory** | Usually only the current chat | Can have short-term + long-term memory |
| **Example** | "Write a leave application" | "Apply leave for me in Workday for Friday and inform my manager" |

### 🏠 Easy Analogy

- **GenAI = a very smart encyclopedia that can talk.** It knows a lot, but it just sits on the shelf.
- **Agentic AI = a smart personal assistant** who reads that encyclopedia **and** can pick up the phone, browse the internet, and get things done.

### 🔑 One-Liner
> **GenAI creates. Agentic AI acts.** Agentic AI is built *on top of* GenAI.

---

## 3. LLMs, Knowledge Cutoff, RAG, Fine-Tuning & Semantic Search

### 3.1 🧠 What is an LLM?

**LLM = Large Language Model**

Let's break the name down:
- **Large** → trained on a HUGE amount of text (books, websites, code, Wikipedia...) and has billions of parameters (internal "knobs").
- **Language** → it understands and produces human language.
- **Model** → a mathematical program that learned patterns from data.

#### How does it work (super simple)?
An LLM is basically a **super-powerful "next word predictor"**.

```
Input:  "The capital of India is"
LLM thinks: which word most likely comes next?
Output: "New Delhi"
```

It does this one word (actually one *token*) at a time, very fast, which is how it writes full paragraphs.

#### Basic flow

```
 Prompt  ───────►  [   LLM   ]  ───────►  Response
                       │
                       └──► has a "Knowledge Cutoff"
```

#### What can an LLM do?

| Task | Meaning | Example |
|---|---|---|
| 🌐 **Translation** | Convert one language to another | "Hello" → "नमस्ते" |
| 📝 **Summarization** | Make long text short | 10-page report → 5 bullet points |
| 💬 **Chat** | Have a conversation | ChatGPT, Copilot, Claude |
| 🏷️ **NER** (Named Entity Recognition) | Find names, places, dates, orgs in text | "Ashish works at Accenture in Hyderabad" → Person: Ashish, Org: Accenture, Place: Hyderabad |
| 🖼️ **Image Generation** | Create images from text (via multimodal / image models) | "A cat wearing sunglasses" → 🐱😎 |
| 💻 **Code Generation** | Write code | "Write a Python function to add 2 numbers" |
| ❓ **Question Answering** | Answer questions | "What is Terraform?" |

Popular LLMs: **GPT (OpenAI)**, **Claude (Anthropic)**, **Gemini (Google)**, **Llama (Meta)**, **Mistral**.

---

### 3.2 📅 What is Knowledge Cutoff?

An LLM learns from data **up to a certain date**. After that date, it knows **nothing**. That date is called the **Knowledge Cutoff**.

#### 🧊 Analogy: The person who slept for 2 years
Imagine your friend was in a deep sleep for 2 years. When he wakes up:
- He knows everything up to 2 years ago ✅
- He has **no idea** who won the latest cricket match, the latest iPhone, today's petrol price ❌
- Worse, he might **confidently guess wrong**!

That's exactly an LLM after its knowledge cutoff.

#### Example
```
Model trained till: Jan 2024
You ask (in 2026):  "Who won IPL 2025?"
LLM: "I don't know" OR it makes up a wrong answer 😬
```

#### 👻 Related Problem: Hallucination
When an LLM doesn't know something but **still answers confidently with made-up info**, it's called a **hallucination**.

#### The 3 big problems with a plain LLM

| Problem | Meaning |
|---|---|
| 📅 **Outdated knowledge** | Knows nothing after cutoff |
| 🔒 **No private data** | Doesn't know your company documents, your HR policy, your project code |
| 👻 **Hallucination** | Makes up answers when unsure |

So people asked: **"How do we give the LLM new or private knowledge?"**

Two main solutions came up:
1. **Fine-Tuning** → teach the model again
2. **RAG** → give the model the info at the time of the question

---

### 3.3 🎓 What is Fine-Tuning?

**Fine-tuning** = taking an already-trained LLM and **training it a little more** on your own specific data so it becomes an expert in that area.

#### 🩺 Analogy
- A **general MBBS doctor** = base LLM (knows a bit of everything)
- Send them for **3 more years of cardiology training** = fine-tuning
- Now they're a **heart specialist** = fine-tuned LLM

#### Example
A bank fine-tunes an LLM on 50,000 past customer support chats. Now the model talks in the bank's style and knows its products well.

#### Fine-tuning flow
```
Base LLM  +  Your training data (Q&A pairs, examples)  ──►  Training (GPUs, hours/days)  ──►  Fine-tuned LLM
```

#### Example training data (JSONL format)
```json
{"prompt": "What is the minimum balance for a savings account?", "response": "The minimum balance is ₹5,000 for metro branches."}
{"prompt": "How do I block my debit card?", "response": "Call 1800-XXX or use the mobile app > Cards > Block."}
```

#### ✅ Pros and ❌ Cons

| ✅ Good at | ❌ Bad at |
|---|---|
| Teaching **style, tone, format** | **Expensive** (GPUs, time, ML expertise) |
| Teaching a **specific skill or domain behaviour** | Knowledge gets **outdated again**: new data means retraining |
| Faster answers (no extra lookup) | Can't easily tell you **where** the answer came from |
| | Risk of the model "forgetting" old general knowledge |

> 💡 **Rule of thumb:** Fine-tuning teaches the model **HOW to behave**, not really **new fast-changing facts**.

---

### 3.4 🔍 What is Semantic Search? (needed to understand RAG)

#### Normal (Keyword) Search
Matches **exact words**.
```
Search: "car repair"
Finds:  documents with the words "car" and "repair"
Misses: "automobile servicing" ❌ (different words, same meaning!)
```

#### Semantic Search
Matches **meaning**, not just words.
```
Search: "car repair"
Finds:  "automobile servicing" ✅, "vehicle maintenance" ✅, "fix my bike engine" (somewhat) ✅
```

#### How does it understand meaning? → **Embeddings**

An **embedding** is a list of numbers that represents the **meaning** of text.

```
"dog"    → [0.21, 0.80, 0.11, ...]
"puppy"  → [0.23, 0.79, 0.13, ...]   ← very close to "dog"
"laptop" → [0.91, 0.02, 0.67, ...]   ← far away
```

Think of it like a **map of meanings** 🗺️: similar words and sentences sit **close together**, unrelated ones are **far apart**.

```
        🐶 dog   🐕 puppy
             🐱 cat

                                 💻 laptop
                                 🖥️ computer
```

When you search, your question is also turned into an embedding, and we find the documents whose embeddings are **nearest** to it. (This "nearness" is usually measured with **cosine similarity**.)

#### Where are embeddings stored? → **Vector Database**
A special database built to store embeddings and find "nearest" ones quickly.
Examples: **Pinecone, ChromaDB, FAISS, Weaviate, Qdrant, pgvector, Azure AI Search, Amazon OpenSearch**.

#### Tiny Python example (conceptual)
```python
from sentence_transformers import SentenceTransformer, util

model = SentenceTransformer("all-MiniLM-L6-v2")

docs = ["How to reset my password",
        "Office cafeteria menu",
        "Steps to change login credentials"]

query = "I forgot my password"

doc_emb   = model.encode(docs)
query_emb = model.encode(query)

scores = util.cos_sim(query_emb, doc_emb)
print(scores)
# Highest scores → "How to reset my password" and "Steps to change login credentials"
# Notice: the 2nd match shares NO exact word "password", but the meaning matches. That's semantic search!
```

---

### 3.5 📖 What is RAG (Retrieval-Augmented Generation)?

Let's split the name:
- **Retrieval** → *fetch* relevant information (using semantic search)
- **Augmented** → *add* that information to the prompt
- **Generation** → the LLM *generates* the answer using it

#### 📝 Analogy: Open-Book Exam
- **Plain LLM** = closed-book exam → answers only from memory (may be outdated or wrong)
- **RAG** = open-book exam → looks at the right page of the textbook **first**, then answers ✅

#### RAG works in 2 phases

**Phase 1: Indexing (done once, in advance)**
```
Your documents (PDFs, Word, wiki, policies)
        │
        ▼
  Split into small "chunks" (e.g. 500 words each)
        │
        ▼
  Convert each chunk into an embedding
        │
        ▼
  Store in a Vector Database 🗄️
```

**Phase 2: Querying (every time a user asks)**
```
User question: "How many leaves do I get per year?"
        │
        ▼
  Convert question → embedding
        │
        ▼
  Semantic search in Vector DB → top 3 matching chunks
        │
        ▼
  Build prompt:
   "Answer using ONLY this context:
    <chunk 1: Leave policy says 24 days...>
    <chunk 2: ...>
    Question: How many leaves do I get per year?"
        │
        ▼
  LLM → "You get 24 leave days per year (as per HR Leave Policy, section 3)." ✅
```

#### Simple Python-style pseudo-code
```python
def rag_answer(question):
    # 1. Retrieve
    q_vec  = embed(question)
    chunks = vector_db.search(q_vec, top_k=3)

    # 2. Augment
    context = "\n".join(chunks)
    prompt  = f"""Use only the context below to answer.
Context:
{context}

Question: {question}"""

    # 3. Generate
    return llm(prompt)
```

#### ✅ Why RAG is great
| Benefit | Why |
|---|---|
| 🔒 Uses **private data** | Your company docs, no retraining |
| 💰 **Cheaper** than fine-tuning | Just update the vector DB |
| 📌 **Citations** | Can show which document the answer came from |
| 👻 **Fewer hallucinations** | The model answers from real text |

---

### 3.6 ⚖️ Fine-Tuning vs RAG

| Point | Fine-Tuning | RAG |
|---|---|---|
| Analogy | Sending a doctor for specialist training | Giving a student an open book |
| Changes the model? | ✅ Yes (model weights change) | ❌ No (model stays the same) |
| Best for | Style, tone, format, special skills | Facts, documents, private knowledge |
| Updating knowledge | Retrain 😩 | Add new docs to the DB 😊 |
| Cost | High | Lower |
| Shows source? | ❌ | ✅ |

> 💡 **Pro tip:** In real projects, teams often use **both**: fine-tune for behaviour, RAG for knowledge.

---

## 4. Problems with RAG & Why Agents Came

RAG was a big improvement, but it still had limits.

### ❌ Problem 1: Data keeps changing
RAG only knows what is **already stored** in the vector DB.

- Stock prices change **every second** 📈
- Weather changes **every hour** 🌦️
- News, cricket scores, flight status, your inbox 📬

You'd have to **keep re-indexing** (re-embedding and re-storing) data all the time. That's slow, costly, and still never truly "live."

```
10:00 AM → You index the stock price: TCS = ₹4,000
10:05 AM → Real price: ₹4,050
10:05 AM → User asks → RAG says ₹4,000 ❌ (stale!)
```

### ❌ Problem 2: RAG can only *read*, not *do*
RAG can answer "What's the leave policy?"
But it **cannot** actually **apply the leave**, **send an email**, or **book a meeting**.

### ❌ Problem 3: Fixed, one-shot pipeline
RAG always does the same thing: retrieve → answer. It doesn't **decide**:
- "Do I even need to search for this?"
- "Should I search the web, the database, or both?"
- "My first result was bad, so let me search again with different words."

### ❌ Problem 4: Can't handle multi-step tasks
"Compare our Q3 sales with our competitor's latest public results and email a summary to my manager."
That needs: internal DB → web search → calculation → write → send email. One RAG lookup can't do that.

### ✅ The Solution: AI Agents
People thought: *"What if the LLM itself could **decide** what to do and **call tools** when needed?"*

```
       Plain LLM  ──►   RAG    ──►    AI Agent
   (memory only)    (reads docs)   (thinks + uses tools + acts)
```

| Problem | How an Agent solves it |
|---|---|
| Data keeps changing | Calls a **live tool/API** (web search, stock API) at the moment of the question |
| Can't take action | Uses **action tools** (send email, create ticket, run code) |
| Fixed pipeline | **Decides** which tool to use, or no tool at all |
| Multi-step tasks | **Plans** and runs steps in a **loop** |

> 💡 Note: RAG didn't die! In agents, **RAG often becomes just one of the tools** ("search company docs").

---

## 5. How Early Agents Worked (The Search Tool Example)

### 🧠 The core idea: the LLM decides "Do I know this, or do I need a tool?"

#### Case A: The LLM already knows → answers directly
```
You:  "What is the capital of France?"
LLM thinks: "I know this. It's general knowledge from before my cutoff."
LLM:  "Paris." ✅  (no tool used)
```

#### Case B: The LLM doesn't know (new/live info) → uses the Search Tool
```
You:  "What is the weather in Hyderabad right now?"
LLM thinks: "This is live data. I can't know it from training. I should use the web_search tool."
   │
   ▼
Tool call: web_search("current weather Hyderabad")
   │
   ▼
Tool result: "Hyderabad: 29°C, partly cloudy, humidity 70%"
   │
   ▼
LLM: "It's currently 29°C and partly cloudy in Hyderabad." ✅
```

This is exactly what ChatGPT / Copilot / Claude do: for normal questions they **answer directly**, but for recent or live questions you'll see something like **"Searching the web..."**. That's the **agent using a search tool** internally.

### 🔁 The famous ReAct pattern (Reason + Act)

Early agents followed a simple loop called **ReAct**:

```
Thought      → what should I do?
Action       → call a tool
Observation  → see the tool's result
... repeat ...
Final Answer
```

#### Full example
```
Question: Who is the current CEO of Microsoft and what is his age?

Thought 1: I need the current CEO. This could change, so I'll search.
Action 1: web_search("current CEO of Microsoft")
Observation 1: Satya Nadella is the CEO of Microsoft.

Thought 2: Now I need his age. I'll search his birth date.
Action 2: web_search("Satya Nadella date of birth")
Observation 2: Born 19 August 1967.

Thought 3: I need to calculate his age today. I'll use the calculator.
Action 3: calculator(2026 - 1967)
Observation 3: 59

Thought 4: I have everything.
Final Answer: Satya Nadella is Microsoft's CEO and is 59 years old.
```

See how the agent **chose** different tools in a chain? That's agent behaviour.

### 🛠️ How does the LLM "know" about tools? → Tool / Function Calling

We **describe** our tools to the LLM (name + what it does + inputs). The LLM then replies with a **structured request** like *"call this tool with these inputs"*. **Our code** runs the tool and sends the result back.

> ⚠️ Important beginner point: **The LLM never runs the tool itself.** It only *asks* for it. Your program does the actual work.

#### Simple Python example (OpenAI-style tool calling)
```python
import json
from openai import OpenAI

client = OpenAI()

# 1️⃣ Our actual tool (normal Python function)
def get_weather(city: str):
    # In real life: call a weather API here
    return f"{city}: 29°C, partly cloudy"

# 2️⃣ Describe the tool to the LLM
tools = [{
    "type": "function",
    "function": {
        "name": "get_weather",
        "description": "Get current weather of a city",
        "parameters": {
            "type": "object",
            "properties": {"city": {"type": "string"}},
            "required": ["city"]
        }
    }
}]

messages = [{"role": "user", "content": "What's the weather in Hyderabad now?"}]

# 3️⃣ Ask the LLM; it decides whether to use the tool
response = client.chat.completions.create(model="gpt-4o-mini", messages=messages, tools=tools)
msg = response.choices[0].message

if msg.tool_calls:                               # LLM wants a tool
    call = msg.tool_calls[0]
    args = json.loads(call.function.arguments)   # {"city": "Hyderabad"}
    result = get_weather(**args)                 # 4️⃣ WE run the tool

    # 5️⃣ Send the result back so the LLM can write the final answer
    messages.append(msg)
    messages.append({"role": "tool", "tool_call_id": call.id, "content": result})
    final = client.chat.completions.create(model="gpt-4o-mini", messages=messages)
    print(final.choices[0].message.content)
else:
    print(msg.content)                           # LLM answered directly
```

#### Flow diagram
```
User ──► LLM ──"I need get_weather(Hyderabad)"──► Your Code ──► Weather API
                                                     │
LLM ◄────────────── "29°C, partly cloudy" ───────────┘
 │
 └──► "It's 29°C and partly cloudy in Hyderabad." ──► User
```

---

## 6. First Part of an Agent: Planning

### 🗺️ What is Planning?

When you give an agent a **goal**, the first thing a good agent does is **make a plan** before acting.

It answers 3 questions:
1. **What is the requirement?** → What exactly does the user want?
2. **How will we achieve it?** → What steps, in what order?
3. **What tools will we need?** → Search? Calculator? Email? Database?

#### 🧳 Analogy: Planning a trip
You don't just run to the airport. You first think:
- **Requirement:** Go to Goa for 3 days under ₹20,000
- **Steps:** Check dates → book flight → book hotel → plan sightseeing
- **Tools:** MakeMyTrip, Google Maps, bank app

An agent does the same thing.

### 📋 Example: Planning in action

**User goal:** *"Find the top 3 trending AI news today and email a short summary to my manager."*

**Agent's plan:**
```
🎯 Requirement:
   - Today's trending AI news (live data)
   - Pick the top 3
   - Write a short summary
   - Email it to the manager

🪜 Steps:
   1. Search the web for today's AI news
   2. Pick the 3 most important stories
   3. Summarize each in 2 lines
   4. Find the manager's email address
   5. Draft and send the email

🛠️ Tools needed:
   - web_search        → step 1
   - LLM itself        → steps 2 & 3 (no tool needed, it's good at summarizing)
   - directory_lookup  → step 4
   - send_email        → step 5
```

Then it **executes** step by step, **checks** each result, and **re-plans** if something fails (e.g., the manager's email isn't found → ask the user).

### 🧩 Common Planning Styles (Pro level)

| Style | How it works | When to use |
|---|---|---|
| **ReAct** | Think → act → observe, one step at a time (plans as it goes) | Simple tasks, unknown paths |
| **Plan-and-Execute** | Make the **full plan first**, then run each step | Longer, clear multi-step tasks |
| **Task Decomposition** | Break a big goal into small sub-goals | Complex projects |
| **Reflection / Self-critique** | After acting, review: "Is this good? Fix it if not" | When quality matters |
| **Tree of Thoughts** | Explore several possible plans, pick the best | Puzzles, tricky decisions |

### 💻 Simple planning prompt example
```python
planner_prompt = """
You are a planning assistant. For the user's goal:
1. State the requirement clearly.
2. List the steps in order.
3. For each step, name the tool needed (or 'none').
Available tools: web_search, calculator, send_email, read_file

Goal: {goal}
Return your answer as JSON.
"""
```

**Possible output:**
```json
{
  "requirement": "Get today's top 3 AI news and email a summary to the manager",
  "steps": [
    {"step": 1, "task": "Search today's AI news",  "tool": "web_search"},
    {"step": 2, "task": "Select top 3 and summarize", "tool": "none"},
    {"step": 3, "task": "Email the summary to the manager", "tool": "send_email"}
  ]
}
```

---

## 7. Pro Level: Putting It All Together

### 🏗️ Anatomy of a Modern AI Agent

```
                    ┌──────────────────────────┐
     User Goal ───► │     🧠 LLM (the Brain)    │
                    └────────────┬─────────────┘
                                 │
         ┌───────────────┬───────┴───────┬────────────────┐
         ▼               ▼               ▼                ▼
   📋 Planning      🛠️ Tools         💾 Memory       🔁 Reflection
  (break goal     (search, RAG,    (short-term:     (check output,
   into steps)     APIs, email,     this chat;       retry/fix)
                   code, DB)        long-term:
                                    vector DB)
                                 │
                                 ▼
                       ✅ Final Answer / Action
```

### 🔁 The Agent Loop
```
while goal not complete:
    think   → what's next?
    act     → call a tool (or answer)
    observe → read the result
    reflect → good enough? if not, adjust the plan
return final answer
```

### 🧭 The Evolution Story (remember this!)

| Stage | What it could do | Limitation |
|---|---|---|
| 1️⃣ **Plain LLM** | Answer from memory | Knowledge cutoff, hallucination, no private data |
| 2️⃣ **Fine-tuned LLM** | Domain expert style | Expensive, still gets outdated |
| 3️⃣ **RAG** | Answer from your documents | Data must be pre-indexed; read-only; one fixed step |
| 4️⃣ **Tool-using LLM** | Fetch live data (search) | Mostly single tool calls |
| 5️⃣ **AI Agent** | Plan + many tools + memory + loop | Needs guardrails, cost control, testing |
| 6️⃣ **Multi-Agent Systems** | Many specialist agents working together | More complex to design (future topic!) |

### ⚠️ Real-World Challenges of Agents (pro awareness)
- **Infinite loops** → set a max number of steps
- **Wrong tool choice** → write clear tool descriptions
- **Cost & speed** → every step is an LLM call 💸
- **Safety** → don't let an agent delete or send things without human approval (**human-in-the-loop**)
- **Hallucinated tool inputs** → validate inputs before running tools

### 🧰 Popular frameworks you'll meet in your course
**LangChain, LangGraph, CrewAI, AutoGen, Microsoft Semantic Kernel, OpenAI Agents SDK, Claude Agent SDK, LlamaIndex**, plus **MCP (Model Context Protocol)**, a standard way to connect tools to agents.

---

## 8. Quick Revision Cheat Sheet

| Term | One-line meaning |
|---|---|
| **AI Agent** | AI that plans and uses tools to complete a goal on your behalf |
| **Agentic AI** | The field/approach of building such autonomous, goal-driven AI |
| **Generative AI** | AI that creates new content (text, image, code) |
| **LLM** | A huge next-word-prediction model trained on lots of text |
| **Token** | A small piece of text (word or part of a word) the LLM processes |
| **Knowledge Cutoff** | The date after which the LLM knows nothing |
| **Hallucination** | The LLM confidently making up wrong information |
| **Fine-Tuning** | Extra training of an LLM on your data to change its behaviour |
| **Embedding** | Numbers that represent the meaning of text |
| **Vector DB** | Database that stores embeddings and finds similar ones |
| **Semantic Search** | Search by meaning, not exact words |
| **RAG** | Retrieve relevant docs → add to prompt → LLM answers |
| **Tool / Function Calling** | The LLM asks your code to run a function and return the result |
| **ReAct** | Thought → Action → Observation loop |
| **Planning** | Breaking a goal into steps + choosing tools before acting |
| **Human-in-the-loop** | A human approves risky agent actions |

---

## 9. Interview Questions

1. **What is the difference between GenAI and Agentic AI?**
   → GenAI generates content in one step; Agentic AI uses GenAI as a brain to plan, use tools, and take actions over multiple steps.

2. **What is a knowledge cutoff and why is it a problem?**
   → It's the last date of the model's training data; the model can't know newer facts and may hallucinate about them.

3. **RAG vs Fine-tuning, when would you use each?**
   → RAG for changing or private facts with citations; fine-tuning for style, tone, format, or special skills.

4. **What is semantic search?**
   → Searching by meaning using embeddings and vector similarity instead of exact keywords.

5. **Why did agents come after RAG?**
   → RAG needs pre-indexed data, can't fetch live info, can't take actions, and follows a fixed pipeline. Agents decide dynamically and use live tools.

6. **Does the LLM execute the tool itself?**
   → No. It outputs a structured tool-call request; the application runs the tool and returns the result.

7. **What is ReAct?**
   → A prompting pattern where the model alternates Reasoning (Thought) and Acting (tool call), observing results until it reaches an answer.

8. **What does the planning component of an agent do?**
   → It understands the requirement, breaks it into ordered steps, and picks the tools needed for each step.

---

> 🚀 **Next topics to explore:** Tools in depth, Agent Memory, Multi-Agent Systems, LangGraph, MCP, Guardrails & Evaluation.
>
> *Happy learning! 🎉*
