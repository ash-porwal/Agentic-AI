# ⚡ Asynchronous Programming for AI Agents (Beginner → Pro)

> **Who this is for:** Beginners who know basic Python (functions, loops, `print`) but have never used `async` / `await`.
> **Goal:** Understand **why** async matters for AI agents, **how** Python's `asyncio` works, and **how to use it safely** (error handling + debugging) when your agent calls many LLMs and tools at once.
>
> 📖 Every new term is explained in a **"📖 New term"** box, and all terms are collected in the **[Glossary](#11-glossary--every-term-explained-simply)** at the end.

---

## 📚 Table of Contents

1. [Why Should an AI Agent Builder Care About Async?](#1-why-should-an-ai-agent-builder-care-about-async)
2. [Asynchronous Programming Basics](#2-asynchronous-programming-basics)
   - 2.1 Synchronous (blocking) code
   - 2.2 Asynchronous (non-blocking) code
   - 2.3 I/O-bound vs CPU-bound work
   - 2.4 The Event Loop
3. [Python's `asyncio` Library](#3-pythons-asyncio-library)
   - 3.1 `async` and `await`
   - 3.2 Coroutines (pause & resume)
   - 3.3 `asyncio.run()`
   - 3.4 Tasks & `create_task()`
   - 3.5 `asyncio.gather()`
   - 3.6 `TaskGroup` (modern way)
   - 3.7 `as_completed()`: handle results as they arrive
   - 3.8 Timeouts
   - 3.9 Semaphore: limit how many run at once
   - 3.10 Running blocking code safely (`to_thread`)
4. [Concurrency vs Parallelism](#4-concurrency-vs-parallelism)
5. [Async in AI Agents (Practical)](#5-async-in-ai-agents-practical)
6. [Error Handling in Async](#6-error-handling-in-async)
7. [Debugging Async Code](#7-debugging-async-code)
8. [Best Practices & Common Mistakes](#8-best-practices--common-mistakes)
9. [Quick Revision Cheat Sheet](#9-quick-revision-cheat-sheet)
10. [Interview Questions](#10-interview-questions)
11. [Glossary: Every Term Explained Simply](#11-glossary--every-term-explained-simply)

---

## 1. Why Should an AI Agent Builder Care About Async?

An AI agent spends most of its time **waiting**:
- ⏳ Waiting for the **LLM** to reply (1–10 seconds)
- ⏳ Waiting for a **web search** API
- ⏳ Waiting for a **database** query
- ⏳ Waiting for a **file download** or a **web page** to load

If your agent waits for each one **one after another**, it's slow. If it can **start them all together and wait for all at once**, it's fast.

```
❌ One by one (synchronous):
   Search (2s) ──► Scrape page (3s) ──► Query DB (1s)     = 6 seconds total

✅ All together (asynchronous):
   Search (2s)       ─┐
   Scrape page (3s)  ─┼─► all done                      = ~3 seconds total
   Query DB (1s)     ─┘     (time of the slowest one)
```

> 🔑 **One-liner:** Async lets your agent **do other useful work while waiting**, instead of sitting idle.

> 📖 **New terms**
> - **API (Application Programming Interface):** a way for your code to talk to another service over the internet (e.g., the OpenAI API, a weather API).
> - **Latency:** the waiting time between asking for something and getting the answer.

---

## 2. Asynchronous Programming Basics

### 2.1 🐢 Synchronous (Blocking) Code

**Synchronous** = things happen **one after another, in order**. Each step must **finish** before the next one starts.

> 📖 **New term: Blocking** = a line of code that **stops everything else** until it finishes. While it's blocking, your program can't do anything else.

#### ☕ Analogy: A coffee shop with one slow barista
1. Takes Customer A's order
2. Makes A's coffee (**stands and waits 3 minutes** for the machine)
3. Only then takes Customer B's order

Everyone waits in a long line, even though the barista is just standing there while the machine works.

#### Code example
```python
import time

def fetch_data(name, seconds):
    print(f"Start {name}")
    time.sleep(seconds)          # 😴 pretend we are waiting for an API
    print(f"Done  {name}")
    return f"{name} result"

start = time.perf_counter()

fetch_data("Search", 2)
fetch_data("Scrape", 3)
fetch_data("Database", 1)

print(f"Total: {time.perf_counter() - start:.1f}s")
```

**Output:**
```
Start Search
Done  Search
Start Scrape
Done  Scrape
Start Database
Done  Database
Total: 6.0s      ← 2 + 3 + 1
```

> 📖 **New terms**
> - **`time.sleep(n)`:** pauses the **whole program** for `n` seconds. It's **blocking**.
> - **`time.perf_counter()`:** a precise stopwatch in Python, used to measure how long code takes.

---

### 2.2 🚀 Asynchronous (Non-Blocking) Code

**Asynchronous** = when a task has to **wait**, the program **switches to another task** instead of standing idle. When the waiting is over, it **comes back** and continues.

> 📖 **New term: Non-blocking** = code that **doesn't freeze** the program while waiting, so other work can happen meanwhile.

#### ☕ Analogy: A smart barista
1. Takes A's order → starts A's coffee machine
2. **While A's coffee brews**, takes B's order → starts B's machine
3. Takes C's order...
4. A's coffee is ready → hands it over

Same one barista, but **no idle time**. That's async.

#### Code example (same task, async)
```python
import asyncio
import time

async def fetch_data(name, seconds):
    print(f"Start {name}")
    await asyncio.sleep(seconds)   # 😴 non-blocking wait: others can run meanwhile
    print(f"Done  {name}")
    return f"{name} result"

async def main():
    start = time.perf_counter()
    results = await asyncio.gather(
        fetch_data("Search", 2),
        fetch_data("Scrape", 3),
        fetch_data("Database", 1),
    )
    print(results)
    print(f"Total: {time.perf_counter() - start:.1f}s")

asyncio.run(main())
```

**Output:**
```
Start Search
Start Scrape
Start Database
Done  Database
Done  Search
Done  Scrape
['Search result', 'Scrape result', 'Database result']
Total: 3.0s      ← only as long as the slowest task!
```

Notice:
- All 3 **started immediately**.
- They **finished in order of speed** (Database 1s → Search 2s → Scrape 3s).
- `gather` still returns results **in the order you passed them in**.
- **6s → 3s** 🎉

Don't worry about `async`, `await`, `gather`, and `asyncio.run` yet. They're explained one by one in [Section 3](#3-pythons-asyncio-library).

---

### 2.3 ⚖️ I/O-bound vs CPU-bound Work (VERY important)

Async only helps with **one kind** of slowness.

> 📖 **New terms**
> - **I/O (Input/Output):** talking to something **outside** your program: network, internet, disk, database. Your code mostly **waits**.
> - **I/O-bound:** a task that is slow because it's **waiting** for I/O.
> - **CPU-bound:** a task that is slow because the processor is **busy calculating** (heavy math, image processing, training a model).
> - **CPU (Central Processing Unit):** the "brain" chip of your computer that runs calculations.

| | I/O-bound | CPU-bound |
|---|---|---|
| Why slow? | **Waiting** for something external | **Busy computing** |
| Examples | LLM API call, web search, DB query, downloading files, reading S3 | Resizing 10,000 images, heavy pandas math, training ML models |
| Does async help? | ✅ **Yes, a lot!** | ❌ **No.** Use multiprocessing instead |
| Analogy | Waiting for a pizza delivery 🍕 | Cooking the pizza yourself 👨‍🍳 |

> 💡 **Good news:** AI agents are mostly **I/O-bound** (LLM calls + tool calls = waiting on the network). That's why async is so useful for agents.

---

### 2.4 🔄 The Event Loop

The **event loop** is the **heart of async**. It's the **manager** that decides which task runs next.

> 📖 **New term: Event loop** = a loop that keeps checking: *"Which task is ready to continue right now?"* It runs that task until the task says *"I'm waiting now"*, then switches to another ready task.

#### 🍽️ Analogy: One waiter in a restaurant
- The waiter (**event loop**) serves many tables (**tasks**).
- Table 1 orders → the waiter sends the order to the kitchen → **doesn't stand there waiting** → goes to Table 2.
- The kitchen rings the bell ("Table 1's food is ready!") → the waiter takes it to Table 1.
- **One waiter, many tables, nobody ignored.**

#### How it works (simplified)
```
┌──────────────────────── EVENT LOOP ────────────────────────┐
│                                                            │
│   Ready queue:  [ Task A ]  [ Task B ]  [ Task C ]          │
│                                                            │
│   1. Pick Task A → run it until it hits `await` (waiting)  │
│   2. Park Task A in the "waiting" area                     │
│   3. Pick Task B → run until `await` → park it             │
│   4. Pick Task C → run until `await` → park it             │
│   5. "Task A's network reply arrived!" → A is ready again  │
│   6. Resume Task A from where it paused ...                │
│   7. Repeat until all tasks are done                       │
└────────────────────────────────────────────────────────────┘
```

#### 🔑 Key facts about the event loop
1. **It runs on a single thread.** Only **one** piece of your Python code runs at any exact moment.
2. **Tasks give up control voluntarily** at `await`. This is called **cooperative multitasking**.
3. If a task **never awaits** (e.g., uses `time.sleep()` or heavy math), it **blocks the whole loop** and every other task freezes. ⚠️

> 📖 **New terms**
> - **Thread:** a single line of execution inside a program, like one worker doing one thing at a time.
> - **Cooperative multitasking:** tasks **politely take turns**: each one says "I'm waiting, someone else can go" (via `await`). Nobody is forced to stop.
> - **Queue:** a waiting line: first in, first out.

---

# 3. Python's `asyncio` Library

> 📖 **New term: `asyncio`** = Python's **built-in library** (no install needed) for writing async code. It gives you the event loop, tasks, `gather`, timeouts, and more.
> - **Library:** a collection of ready-made code you can import and use.

---

### 3.1 🔑 `async` and `await` Keywords

| Keyword | Where you use it | Meaning |
|---|---|---|
| `async def` | In front of a function definition | "This function is **asynchronous**: it can pause and resume." |
| `await` | In front of a call inside an `async def` | "**Pause me here** until this finishes; meanwhile let other tasks run." |

```python
import asyncio

async def make_tea():                 # async def → defines an async function
    print("Boiling water...")
    await asyncio.sleep(2)            # await → pause here, let others run
    print("Tea ready ☕")
    return "tea"
```

#### 📏 Rules to remember
1. You can only use `await` **inside** an `async def` function (except in Jupyter notebooks, see 3.3).
2. You can only `await` things that are **awaitable**: coroutines, tasks, and futures.
3. Calling an `async def` function **does NOT run it**. It just creates a coroutine object (see 3.2).

> 📖 **New terms**
> - **Keyword:** a reserved word with special meaning in Python (`def`, `if`, `async`, `await`...).
> - **Awaitable:** anything you are allowed to put after `await`.
> - **Future:** a low-level object meaning "a result that will be available later". You rarely create these yourself.

---

### 3.2 🔁 Coroutines: Functions That Can Pause & Resume

> 📖 **New term: Coroutine** = a special function (made with `async def`) that can **pause in the middle** (at `await`) and **resume later from the exact same spot**, remembering all its variables.

#### 📺 Analogy: Pausing a Netflix movie
A normal function is like a **movie theatre**: once it starts, it plays till the end.
A coroutine is like **Netflix**: you can **pause**, go do something else, and **resume exactly where you left off**.

#### The big beginner mistake: forgetting `await`
```python
async def get_answer():
    return 42

result = get_answer()          # ❌ forgot await / asyncio.run
print(result)
```
**Output:**
```
<coroutine object get_answer at 0x0000020B...>
RuntimeWarning: coroutine 'get_answer' was never awaited
```
Calling it only created a **coroutine object** (a paused "recipe"), and **nothing actually ran**.

✅ Correct:
```python
import asyncio

async def get_answer():
    return 42

async def main():
    result = await get_answer()   # ✅ actually runs it
    print(result)                 # 42

asyncio.run(main())
```

#### Why coroutines matter for LLM calls
```python
async def ask_llm(question):
    print(f"Sending: {question}")
    answer = await call_llm_api(question)   # ⏸️ pauses here for ~3s
    # while paused, the event loop runs OTHER coroutines
    print(f"Got answer for: {question}")    # ▶️ resumes here with 'answer'
    return answer
```
While waiting for the LLM, your agent can send **other** LLM requests, call tools, and so on.

> 📖 **New term: Object** = a "thing" in Python that holds data and behaviour. Here, a coroutine object is a not-yet-run piece of async work.

---

### 3.3 ▶️ `asyncio.run()`: Starting the Event Loop

`asyncio.run(main())` is the **entry point**:
1. Creates a **new event loop**
2. Runs your `main()` coroutine until it finishes
3. Cleans up and closes the loop

```python
import asyncio

async def main():
    print("Hello async world!")

asyncio.run(main())   # ✅ call it ONCE, at the very top of your program
```

> 📖 **New term: Entry point** = the place where your program starts running.

#### ⚠️ Jupyter Notebook / VS Code notebook gotcha
Notebooks **already have an event loop running**, so `asyncio.run()` fails with:
```
RuntimeError: asyncio.run() cannot be called from a running event loop
```
✅ In a notebook, just use `await` directly in a cell:
```python
await main()
```

---

### 3.4 📋 Tasks & `asyncio.create_task()`

> 📖 **New term: Task** = a coroutine that has been **handed to the event loop to start running in the background right away**.

#### Coroutine vs Task

| | Coroutine | Task |
|---|---|---|
| Created by | Calling an `async def` function | `asyncio.create_task(coro)` |
| Starts running? | ❌ Only when awaited | ✅ **Immediately scheduled** in the background |
| Analogy | A recipe written on paper | The recipe handed to a chef who starts cooking |

#### ❌ Common mistake: awaiting one by one (still sequential!)
```python
async def main():
    a = await fetch_data("Search", 2)    # waits 2s
    b = await fetch_data("Scrape", 3)    # THEN waits 3s
    # Total 5s: no speedup! Each await waits before the next one even starts.
```

#### ✅ Create tasks first, then await
```python
async def main():
    task_a = asyncio.create_task(fetch_data("Search", 2))   # starts now
    task_b = asyncio.create_task(fetch_data("Scrape", 3))   # starts now

    # Both are already running in the background...
    print("I can do other things here!")

    a = await task_a       # collect results
    b = await task_b
    # Total ≈ 3s ✅
```

> ⚠️ **Pro tip:** Keep a reference to every task you create (store it in a variable or list). Python can **garbage-collect** a task that nothing refers to, and it may disappear mid-run.
>
> 📖 **New term: Garbage collection** = Python automatically deleting objects nobody is using anymore, to free memory.

---

### 3.5 🧺 `asyncio.gather()`: Run Many, Collect All

The most common tool for running things together.

```python
results = await asyncio.gather(coro1, coro2, coro3)
```

- Runs all of them **concurrently**
- Waits until **all** finish
- Returns a **list of results in the same order you passed them** (not the finishing order)

```python
import asyncio

async def ask(city, delay):
    await asyncio.sleep(delay)
    return f"Weather in {city}: sunny"

async def main():
    cities = ["Hyderabad", "Delhi", "Mumbai", "Pune"]
    results = await asyncio.gather(*(ask(c, 1) for c in cities))
    for r in results:
        print(r)

asyncio.run(main())   # ~1s total, not 4s
```

> 📖 **New term: `*` (unpacking)** = `*list` spreads a list into separate arguments. `gather(*[a, b, c])` is the same as `gather(a, b, c)`.

---

### 3.6 🧑‍🤝‍🧑 `asyncio.TaskGroup`: The Modern, Safer Way (Python 3.11+)

`TaskGroup` is a newer, **safer** alternative to `gather`.

```python
import asyncio

async def main():
    async with asyncio.TaskGroup() as tg:
        t1 = tg.create_task(fetch_data("Search", 2))
        t2 = tg.create_task(fetch_data("Scrape", 3))
    # ⬆️ When the 'async with' block ends, ALL tasks are guaranteed finished
    print(t1.result(), t2.result())

asyncio.run(main())
```

**Why it's safer:** if **any** task fails, TaskGroup **automatically cancels the other tasks** and raises the errors together (as an `ExceptionGroup`, see Section 6). No tasks are left running by accident.

> 📖 **New terms**
> - **`async with`:** an async version of `with`. It sets something up at the start and **always cleans up** at the end, even if errors happen.
> - **Cancel:** tell a running task to stop early.

| | `gather()` | `TaskGroup` |
|---|---|---|
| Python version | All modern versions | 3.11+ |
| If one task fails | By default the error is raised, but **other tasks keep running** | **Cancels the others** and raises all errors together |
| Can collect errors as values? | ✅ `return_exceptions=True` | ❌ (handle errors inside each task instead) |
| Best for | "Get everything I can, even if some fail" | "All-or-nothing" jobs, clean structured code |

> 📖 **New term: Structured concurrency** = the idea (used by TaskGroup) that tasks started inside a block must **all finish before the block ends**. No forgotten background tasks.

---

### 3.7 🏁 `asyncio.as_completed()`: Handle Results as They Arrive

`gather` waits for **all**. Sometimes you want to **use each result as soon as it's ready** (e.g., show the fastest search result to the user first).

```python
import asyncio

async def search(engine, delay):
    await asyncio.sleep(delay)
    return f"{engine} answered in {delay}s"

async def main():
    coros = [search("Bing", 2), search("Wikipedia", 1), search("ArXiv", 3)]
    for next_done in asyncio.as_completed(coros):
        result = await next_done
        print("✅", result)          # printed in FINISHING order

asyncio.run(main())
```
**Output:**
```
✅ Wikipedia answered in 1s
✅ Bing answered in 2s
✅ ArXiv answered in 3s
```

---

### 3.8 ⏱️ Timeouts: Don't Wait Forever

An API might **hang**. Never let your agent wait forever.

```python
import asyncio

async def slow_tool():
    await asyncio.sleep(10)
    return "finally!"

async def main():
    try:
        result = await asyncio.wait_for(slow_tool(), timeout=3)   # all Python versions
    except asyncio.TimeoutError:
        result = "⏱️ Tool took too long, skipping it."
    print(result)

asyncio.run(main())
```

Python 3.11+ also has a block style:
```python
async def main():
    try:
        async with asyncio.timeout(3):
            result = await slow_tool()
    except TimeoutError:
        result = "⏱️ Timed out"
```

> 📖 **New term: Timeout** = a maximum waiting time. If it's exceeded, the task is cancelled and a `TimeoutError` is raised.

---

### 3.9 🚦 Semaphore: Limit How Many Run at Once

If you fire **500 LLM calls at once**, the API will reply with **"429 Too Many Requests"** (rate limit). A **semaphore** lets only N tasks run at the same time.

> 📖 **New terms**
> - **Semaphore:** a counter that allows only a fixed number of tasks inside a section at once, like a **parking lot with 5 spaces**: car 6 waits until someone leaves.
> - **Rate limit:** the maximum number of requests an API allows per minute/second.
> - **HTTP 429:** the error code an API returns when you send too many requests too fast.

```python
import asyncio

sem = asyncio.Semaphore(5)          # max 5 at the same time

async def call_llm(i):
    async with sem:                 # 🚦 waits here if 5 are already running
        await asyncio.sleep(1)      # pretend LLM call
        return f"answer {i}"

async def main():
    results = await asyncio.gather(*(call_llm(i) for i in range(20)))
    print(len(results), "answers")  # 20 answers in ~4s (20 / 5 batches × 1s)

asyncio.run(main())
```

---

### 3.10 🧱 Running Blocking Code Safely: `asyncio.to_thread()`

Some libraries are **not async** (e.g., `requests`, `boto3`, or `time.sleep`). Calling them directly **blocks the event loop**. Wrap them with `asyncio.to_thread()` (Python 3.9+) to run them in a **separate thread**:

```python
import asyncio
import time

def old_blocking_function(x):      # normal (sync) function
    time.sleep(2)
    return x * 2

async def main():
    results = await asyncio.gather(
        asyncio.to_thread(old_blocking_function, 1),
        asyncio.to_thread(old_blocking_function, 2),
    )
    print(results)                 # [2, 4] in ~2s, loop not blocked

asyncio.run(main())
```

#### Sync library → Async alternative

| Sync (blocks) | Async alternative |
|---|---|
| `time.sleep()` | `await asyncio.sleep()` |
| `requests.get()` | `httpx.AsyncClient` or `aiohttp` |
| `openai.OpenAI()` | `openai.AsyncOpenAI()` |
| `anthropic.Anthropic()` | `anthropic.AsyncAnthropic()` |
| `open().read()` (files) | `aiofiles`, or `asyncio.to_thread(...)` |
| `boto3` (AWS) | `aioboto3`, or `asyncio.to_thread(...)` |

> 📖 **New terms**
> - **`httpx` / `aiohttp`:** Python libraries for making web (HTTP) requests that support async.
> - **HTTP:** the language browsers and APIs use to send requests and responses over the internet.

---

# 4. Concurrency vs Parallelism

These two words sound the same but mean **different things**.

> 📖 **New terms**
> - **Concurrency:** **managing** many tasks at once by **switching between them**. They make progress during overlapping time periods, but only one runs at any exact instant.
> - **Parallelism:** **literally running** many tasks **at the exact same instant**, on multiple CPU cores.
> - **CPU core:** one independent processor inside your CPU. A laptop may have 8 cores, so it can truly do 8 things at the same moment.

### 👨‍🍳 Analogy

| Concurrency | Parallelism |
|---|---|
| **One chef** cooking 3 dishes: stirs the curry, then while rice boils, chops vegetables, then checks the curry... | **Three chefs**, each cooking one dish at the same time |
| One worker, smart switching | Many workers, true simultaneous work |

```
CONCURRENCY (1 thread, switching):
Thread 1: [A]..[B]..[C]..[A]..[B]..[C]   ← interleaved, tasks overlap in time

PARALLELISM (multiple cores):
Core 1:   [A A A A A A]
Core 2:   [B B B B B B]                 ← truly at the same instant
Core 3:   [C C C C C C]
```

### 🐍 Python's 3 options

| Tool | Type | Best for | Example |
|---|---|---|---|
| **`asyncio`** | Concurrency (1 thread) | **Many I/O waits** (thousands of API calls) | LLM calls, web search, scraping |
| **`threading`** | Concurrency (multiple threads) | I/O with **sync-only** libraries | Old `requests`/`boto3` code |
| **`multiprocessing`** | **Parallelism** (multiple processes) | **CPU-heavy** work | Image processing, heavy math |

> 📖 **New terms**
> - **Process:** a fully separate running program with its own memory. Multiple processes can run on different cores in parallel.
> - **GIL (Global Interpreter Lock):** a lock in standard Python that lets only **one thread run Python code at a time**. That's why threads don't speed up CPU-heavy work in Python, but they're still fine for I/O waiting. (Newer Python versions are experimenting with removing it.)

### 🎯 Which one should an AI agent use?

```
Is the slow part WAITING (API, LLM, DB, network)?
   ├── Yes → Does the library support async?
   │          ├── Yes → asyncio ✅ (best choice for agents)
   │          └── No  → asyncio.to_thread() or threading
   └── No, it's heavy COMPUTING → multiprocessing
```

> 🔑 **One-liner:** *Concurrency is about **dealing with** lots of things at once; parallelism is about **doing** lots of things at once.* Agents mostly need **concurrency**.

---

# 5. Async in AI Agents (Practical)

### 5.1 🧠 Where async helps inside an agent

| Situation | Without async | With async |
|---|---|---|
| Agent calls 3 tools (search, scrape, DB) | 2s + 3s + 1s = **6s** | max(2,3,1) = **3s** |
| Summarize 50 documents with an LLM | 50 × 3s = **150s** | ~**15s** (with a semaphore of 10) |
| Ask 3 different LLMs and compare | 3 × 4s = **12s** | **~4s** |
| Chat server with 100 users | Users wait in line | All served concurrently |
| Streaming LLM tokens to the UI | Blocks the app | Smooth, word-by-word output |

> 📖 **New terms**
> - **Bottleneck:** the slowest part of a system that holds everything else back, like the narrow neck of a bottle.
> - **Streaming:** receiving the LLM's answer **piece by piece** as it's generated, instead of waiting for the full answer (that's why ChatGPT "types").

### 5.2 🛠️ Example: Agent calls multiple tools at the same time

When an LLM decides it needs several tools at once (called **parallel tool calls**), run them concurrently.

> 📖 **New term: Parallel tool calls** = the LLM asks for **several tools in one reply** (e.g., "search the web AND read this file"). Your code should run them together, not one by one.

```python
# agent_tools_async.py
import asyncio
import time

# 🛠️ Async tools (simulated with sleep; in real life they'd call APIs)
async def web_search(query: str) -> str:
    await asyncio.sleep(2)
    return f"[search] Top results for '{query}'"

async def scrape_page(url: str) -> str:
    await asyncio.sleep(3)
    return f"[scrape] Text content of {url}"

async def query_db(sql: str) -> str:
    await asyncio.sleep(1)
    return f"[db] 42 rows for: {sql}"

TOOLS = {"web_search": web_search, "scrape_page": scrape_page, "query_db": query_db}

async def run_tool_calls(tool_calls: list[dict]) -> list[str]:
    """Run all tool calls requested by the LLM concurrently."""
    coros = [TOOLS[c["name"]](**c["args"]) for c in tool_calls]
    return await asyncio.gather(*coros)

async def main():
    # Pretend the LLM asked for these 3 tools in one turn
    tool_calls = [
        {"name": "web_search",  "args": {"query": "latest AI agent frameworks"}},
        {"name": "scrape_page", "args": {"url": "https://example.com/blog"}},
        {"name": "query_db",    "args": {"sql": "SELECT * FROM sales"}},
    ]
    start = time.perf_counter()
    results = await run_tool_calls(tool_calls)
    for r in results:
        print(r)
    print(f"⏱️ {time.perf_counter() - start:.1f}s (instead of 6s)")

asyncio.run(main())
```

**Output:**
```
[search] Top results for 'latest AI agent frameworks'
[scrape] Text content of https://example.com/blog
[db] 42 rows for: SELECT * FROM sales
⏱️ 3.0s (instead of 6s)
```

### 5.3 🤖 Example: Many real LLM calls concurrently (OpenAI async client)

```python
# summarize_many.py
import asyncio
from openai import AsyncOpenAI          # 👈 the async client

client = AsyncOpenAI()                  # needs OPENAI_API_KEY
sem = asyncio.Semaphore(5)              # 🚦 max 5 requests at a time (rate-limit safety)

async def summarize(text: str) -> str:
    async with sem:
        resp = await client.chat.completions.create(     # ⏸️ non-blocking wait
            model="gpt-4o-mini",
            messages=[{"role": "user", "content": f"Summarize in 1 line: {text}"}],
        )
        return resp.choices[0].message.content

async def main():
    docs = [
        "AWS Glue is a serverless ETL service...",
        "Terraform is an infrastructure-as-code tool...",
        "PySpark is the Python API for Apache Spark...",
    ]
    summaries = await asyncio.gather(*(summarize(d) for d in docs))
    for s in summaries:
        print("•", s)

asyncio.run(main())
```

> 📖 **New term: Client** = an object from a library that handles talking to a service (here, the OpenAI API). `AsyncOpenAI` is the async version.

### 5.4 🌊 Example: Streaming LLM output asynchronously

```python
import asyncio
from openai import AsyncOpenAI

client = AsyncOpenAI()

async def stream_answer(question: str):
    stream = await client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": question}],
        stream=True,
    )
    async for chunk in stream:                 # 👈 async for: get pieces as they arrive
        piece = chunk.choices[0].delta.content or ""
        print(piece, end="", flush=True)
    print()

asyncio.run(stream_answer("Explain async in one paragraph."))
```

> 📖 **New terms**
> - **`async for`:** a loop that **awaits** each next item, so other tasks can run while waiting for the next chunk.
> - **Chunk:** one small piece of a streamed response.

### 5.5 🧩 Example: Full mini async agent loop

```python
# async_agent.py: LLM decides tools → tools run concurrently → loop
import asyncio, json
from openai import AsyncOpenAI

client = AsyncOpenAI()
MODEL = "gpt-4o-mini"

async def get_weather(city: str) -> str:
    await asyncio.sleep(1)
    return f"{city}: 29°C, partly cloudy"

async def get_news(topic: str) -> str:
    await asyncio.sleep(2)
    return f"Top news on {topic}: New agent frameworks released."

TOOLS = {"get_weather": get_weather, "get_news": get_news}
SCHEMA = [
    {"type": "function", "function": {"name": "get_weather", "description": "Current weather for a city",
     "parameters": {"type": "object", "properties": {"city": {"type": "string"}}, "required": ["city"]}}},
    {"type": "function", "function": {"name": "get_news", "description": "Latest news on a topic",
     "parameters": {"type": "object", "properties": {"topic": {"type": "string"}}, "required": ["topic"]}}},
]

async def run_one_tool(call) -> dict:
    try:
        args = json.loads(call.function.arguments)
        result = await asyncio.wait_for(TOOLS[call.function.name](**args), timeout=10)
    except Exception as e:                       # 🛡️ one bad tool won't crash the agent
        result = f"Tool error: {type(e).__name__}: {e}"
    return {"role": "tool", "tool_call_id": call.id, "content": result}

async def run_agent(question: str, max_steps: int = 5) -> str:
    messages = [{"role": "user", "content": question}]
    for _ in range(max_steps):
        resp = await client.chat.completions.create(model=MODEL, messages=messages, tools=SCHEMA)
        msg = resp.choices[0].message
        messages.append(msg)
        if not msg.tool_calls:
            return msg.content
        # ⚡ run ALL requested tools concurrently
        tool_msgs = await asyncio.gather(*(run_one_tool(c) for c in msg.tool_calls))
        messages.extend(tool_msgs)
    return "Stopped: max steps reached."

print(asyncio.run(run_agent("What's the weather in Hyderabad and the latest AI news?")))
```

Weather (1s) and news (2s) run **together** → ~2s instead of 3s. With 5–10 tools, the savings are big.

### 5.6 🧰 Async in popular agent frameworks
Most modern frameworks are **async-first** or support async:
- **LangChain / LangGraph:** `ainvoke()`, `astream()` (the `a` prefix = async version)
- **OpenAI Agents SDK:** `await Runner.run(...)`
- **AutoGen, CrewAI, Semantic Kernel, Claude Agent SDK:** async APIs
- **FastAPI:** `async def` endpoints for serving agents as web APIs
- **MCP (Model Context Protocol) servers** (e.g., FastMCP) often use `async def` tools

> 📖 **New terms**
> - **Async-first:** designed mainly to be used with `async`/`await`.
> - **Endpoint:** a URL in a web API that does one job, e.g., `POST /chat`.
> - **FastAPI:** a popular Python framework for building web APIs, with built-in async support.

---

# 6. Error Handling in Async

**Goal:** One failed task (e.g., a website is down) must **not crash the whole agent**.

> 📖 **New terms**
> - **Exception:** Python's way of signalling an error (e.g., `ValueError`, `TimeoutError`). If it isn't handled, the program crashes.
> - **`try / except`:** catch an exception and handle it gracefully instead of crashing.
> - **Graceful degradation:** when something fails, keep working with what you have instead of failing completely.

### 6.1 ❌ The problem: default `gather` behaviour

```python
import asyncio

async def ok_tool():
    await asyncio.sleep(1); return "✅ ok"

async def broken_tool():
    await asyncio.sleep(0.5); raise ValueError("website down")

async def main():
    results = await asyncio.gather(ok_tool(), broken_tool())   # 💥
    print(results)

asyncio.run(main())
# ValueError: website down  → whole program crashes, the good result is lost
```

### 6.2 ✅ Fix 1: `return_exceptions=True`

Errors are **returned as values** in the results list instead of being raised.

```python
async def main():
    results = await asyncio.gather(ok_tool(), broken_tool(), return_exceptions=True)
    for r in results:
        if isinstance(r, Exception):
            print("⚠️ Failed:", r)
        else:
            print("Result:", r)

asyncio.run(main())
```
**Output:**
```
Result: ✅ ok
⚠️ Failed: website down
```

> 📖 **New term: `isinstance(x, Type)`** = checks whether `x` is of a certain type, e.g. "is this result an Exception?"

### 6.3 ✅ Fix 2: wrap each task with its own try/except (most common in agents)

```python
async def safe(coro, name: str):
    try:
        return await coro
    except Exception as e:
        return f"[{name} failed: {type(e).__name__}: {e}]"   # tell the LLM it failed

async def main():
    results = await asyncio.gather(
        safe(ok_tool(), "ok_tool"),
        safe(broken_tool(), "broken_tool"),
    )
    print(results)
```
The failure message goes back to the LLM, which can then **decide** to retry, use another tool, or tell the user. That's agent adaptability.

### 6.4 ✅ Fix 3: TaskGroup + `except*` (Python 3.11+)

```python
async def main():
    try:
        async with asyncio.TaskGroup() as tg:
            tg.create_task(ok_tool())
            tg.create_task(broken_tool())
    except* ValueError as eg:
        for err in eg.exceptions:
            print("Handled:", err)
```
Here the other tasks are **cancelled** when one fails, which is good for all-or-nothing jobs.

> 📖 **New terms**
> - **ExceptionGroup:** a bundle of several exceptions raised together (when several tasks fail).
> - **`except*`:** a special `except` (Python 3.11+) that catches matching errors **out of** an ExceptionGroup.

### 6.5 🔁 Retries with backoff

Network errors are often **temporary**. Retry a few times, waiting longer each time.

```python
import asyncio, random

async def with_retry(func, *args, retries=3, base_delay=1):
    for attempt in range(1, retries + 1):
        try:
            return await func(*args)
        except Exception as e:
            if attempt == retries:
                raise                                   # give up after last try
            wait = base_delay * 2 ** (attempt - 1) + random.uniform(0, 0.5)
            print(f"Attempt {attempt} failed ({e}); retrying in {wait:.1f}s")
            await asyncio.sleep(wait)                   # ✅ non-blocking wait
```

> 📖 **New terms**
> - **Retry:** trying the same operation again after it fails.
> - **Exponential backoff:** waiting 1s, then 2s, then 4s... between retries, so you don't hammer a struggling server.
> - **Jitter:** a small random extra delay so many clients don't all retry at the exact same moment.
>
> 💡 Libraries like **`tenacity`** give you ready-made retry decorators, and the OpenAI/Anthropic SDKs have **built-in retries** (`max_retries`).

### 6.6 🧹 Cancellation and cleanup

When a task is cancelled (timeout, TaskGroup failure, user stops the agent), Python raises `asyncio.CancelledError` **inside** it. Use `finally` to clean up.

```python
async def tool_with_cleanup():
    conn = "db-connection"
    try:
        await asyncio.sleep(10)
    finally:
        print("🧹 closing", conn)       # always runs, even when cancelled
```

> ⚠️ Don't swallow `CancelledError` with a bare `except:`. If you catch it, **re-raise it**, or cancellation stops working properly. (In Python 3.8+, `except Exception` does **not** catch it, which is good.)

> 📖 **New term: `finally`** = a block that **always runs** at the end of `try`, whether there was an error or not. Perfect for cleanup.

### 6.7 🛡️ Error-handling checklist for async agents
- ✅ Every tool call has a **timeout**
- ✅ Every tool call is wrapped so errors become **messages to the LLM**, not crashes
- ✅ **Retries with backoff** for network calls
- ✅ **Semaphore** to avoid 429 rate-limit errors
- ✅ `finally` / `async with` for cleanup
- ✅ **Log** every failure (see Section 7)

---

# 7. Debugging Async Code

Async code is harder to debug because things **don't happen in a straight line**: tasks pause, interleave, and finish in different orders.

> 📖 **New terms**
> - **Debugging:** finding and fixing problems in code.
> - **Execution path:** the order in which lines of code actually run.
> - **Interleave:** mixing steps of different tasks together (A1, B1, A2, C1, B2...).

### 7.1 🪵 Log with timestamps and task names

Plain `print` makes it hard to tell which task printed what. Use `logging` with time and task names.

```python
import asyncio, logging

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s.%(msecs)03d [%(taskName)s] %(message)s",   # taskName: Python 3.12+
    datefmt="%H:%M:%S",
)
log = logging.getLogger("agent")

async def tool(name, delay):
    log.info(f"start {name}")
    await asyncio.sleep(delay)
    log.info(f"end   {name}")

async def main():
    async with asyncio.TaskGroup() as tg:
        tg.create_task(tool("search", 2), name="SearchTask")
        tg.create_task(tool("db", 1),     name="DBTask")

asyncio.run(main())
```
**Output:**
```
19:02:01.001 [SearchTask] start search
19:02:01.001 [DBTask] start db
19:02:02.003 [DBTask] end   db
19:02:03.002 [SearchTask] end   search
```
Now you can **see the real order** of events.

> 📖 **New terms**
> - **`logging`:** Python's built-in library for writing log messages with time, level (INFO/ERROR), and more.
> - **Timestamp:** the exact time something happened.
> - On Python older than 3.12, drop `%(taskName)s` and include the name in the message instead, or use `asyncio.current_task().get_name()`.

### 7.2 🐞 Turn on asyncio debug mode

Debug mode warns you about **coroutines never awaited** and **slow callbacks that block the loop**.

```python
asyncio.run(main(), debug=True)
```
or from the terminal (Windows PowerShell):
```powershell
$env:PYTHONASYNCIODEBUG = "1"
python -X dev my_agent.py
```
Example warning:
```
Executing <Task ...> took 2.003 seconds
```
That means something **blocked the loop for 2 seconds**. You probably used `time.sleep()` or a sync library by mistake!

> 📖 **New terms**
> - **Debug mode:** an extra-careful mode that shows more warnings to help find bugs.
> - **`-X dev`:** Python "development mode": turns on extra checks and warnings.
> - **Callback:** a function that's scheduled to run later when something happens.

### 7.3 🔍 See what tasks are running

```python
for t in asyncio.all_tasks():
    print(t.get_name(), "done" if t.done() else "running")
    t.print_stack()          # where is this task paused right now?
```
Useful when your agent **hangs** and you don't know which task is stuck.

### 7.4 🧭 Common async bugs and how to spot them

| Symptom | Likely cause | Fix |
|---|---|---|
| `RuntimeWarning: coroutine '...' was never awaited` | Forgot `await` | Add `await` (or `create_task`) |
| Prints `<coroutine object ...>` | Called an async function without `await` | `await func()` |
| No speedup at all | Awaiting one by one, or using `time.sleep` / sync libraries | Use `gather`/TaskGroup and async libraries |
| Whole app freezes | Blocking call inside async (`requests`, `time.sleep`, heavy CPU) | `asyncio.to_thread()` or an async library |
| `asyncio.run() cannot be called from a running event loop` | Running in Jupyter | Use `await main()` directly |
| Program hangs forever | A task waits on something that never completes | Add timeouts; inspect `all_tasks()` |
| Random **429** errors | Too many concurrent API calls | Semaphore + retries with backoff |
| `Task exception was never retrieved` | A background task failed and nobody awaited it | Always await tasks / use TaskGroup |

### 7.5 🔭 Tracing tools for agents
For bigger agents, use **tracing tools** that show every LLM call and tool call on a timeline, even when they run concurrently: **LangSmith, Langfuse, Arize Phoenix, OpenTelemetry**, and Azure AI Foundry tracing.

> 📖 **New term: Tracing** = recording the full step-by-step story of a request (each LLM/tool call, its timing, inputs and outputs), shown as a timeline.

---

# 8. Best Practices & Common Mistakes

### ✅ Best Practices
1. **Use async for I/O-bound work** (LLMs, APIs, DBs). Use multiprocessing for CPU-heavy work.
2. **Go async all the way:** use async clients (`AsyncOpenAI`, `httpx.AsyncClient`).
3. **Never block the loop:** no `time.sleep`, no `requests` inside `async def`. Use `to_thread()` when you must.
4. **Run independent work together** with `gather` / `TaskGroup`.
5. **Limit concurrency** with a `Semaphore` to respect rate limits.
6. **Always set timeouts** on external calls.
7. **Handle errors per task** so one failure doesn't kill the agent.
8. **Retry with exponential backoff + jitter** for temporary errors.
9. **Clean up** with `finally` / `async with`.
10. **Name your tasks and log with timestamps** for easier debugging.
11. **Call `asyncio.run()` only once**, at the program's entry point.

### ❌ Common Mistakes

| Mistake | Why it's bad |
|---|---|
| `time.sleep()` inside `async def` | Freezes **every** task |
| `await a(); await b()` for independent work | Still sequential: no speedup |
| Firing 1,000 requests with no limit | Rate limits, memory spikes, bans |
| No timeouts | Agent hangs forever on one slow tool |
| Bare `except:` everywhere | Hides bugs and breaks cancellation |
| Using async for CPU-heavy math | No benefit; the loop gets blocked |
| Creating tasks and not keeping a reference | Task may vanish or its errors go unseen |

---

## 9. Quick Revision Cheat Sheet

| Concept | One-liner |
|---|---|
| **Synchronous** | One task at a time; each waits for the previous one |
| **Asynchronous** | While waiting, switch to other tasks |
| **Blocking** | Freezes the program until done |
| **Non-blocking** | Lets other work continue while waiting |
| **I/O-bound** | Slow because of waiting (network/disk), so async helps |
| **CPU-bound** | Slow because of computing, so use multiprocessing |
| **Event loop** | Manager that runs ready tasks and switches at `await` |
| **`async def`** | Defines a coroutine function |
| **`await`** | Pause here; let other tasks run until this finishes |
| **Coroutine** | A function that can pause and resume |
| **Task** | A coroutine scheduled to run in the background |
| **`asyncio.run()`** | Starts the event loop and runs `main()` |
| **`gather()`** | Run many, get all results in input order |
| **`TaskGroup`** | Safer gather; cancels the rest if one fails (3.11+) |
| **`as_completed()`** | Process results in the order they finish |
| **`wait_for()` / `timeout()`** | Stop waiting after N seconds |
| **Semaphore** | Allow only N tasks at a time |
| **`to_thread()`** | Run blocking code without freezing the loop |
| **Concurrency** | Managing many tasks by switching (1 worker) |
| **Parallelism** | Running tasks at the exact same instant (many cores) |
| **`return_exceptions=True`** | Errors returned as values, not raised |
| **Backoff** | Wait longer between each retry |

### 🗺️ Mini decision map
```
Need many API/LLM/tool calls at once?        → asyncio.gather / TaskGroup
Need results as soon as each is ready?       → asyncio.as_completed
Worried about rate limits?                   → asyncio.Semaphore
A call might hang?                           → asyncio.wait_for / asyncio.timeout
Library is sync-only?                        → asyncio.to_thread
Some tasks may fail, keep the rest?          → return_exceptions=True or a per-task try/except
All-or-nothing job?                          → TaskGroup
Heavy CPU work?                              → multiprocessing (not asyncio)
```

---

## 10. Interview Questions

1. **What is the difference between synchronous and asynchronous code?**
   → Sync runs tasks one after another, blocking while waiting; async lets the program switch to other tasks during waits.

2. **What is the event loop?**
   → The asyncio scheduler that runs ready coroutines on a single thread and switches between them at `await` points.

3. **What is a coroutine?**
   → A function defined with `async def` that can pause at `await` and resume later; calling it returns a coroutine object that must be awaited.

4. **Difference between a coroutine and a task?**
   → A coroutine runs only when awaited; a task wraps a coroutine and schedules it immediately on the event loop.

5. **Concurrency vs parallelism?**
   → Concurrency interleaves many tasks on one worker; parallelism runs them at the same instant on multiple cores.

6. **Why is async a good fit for AI agents?**
   → Agents are I/O-bound (LLM and tool calls), so async lets them run multiple calls concurrently and cut total latency.

7. **Will asyncio speed up CPU-heavy code?**
   → No. It's single-threaded; use multiprocessing for CPU-bound work.

8. **What happens if you use `time.sleep()` inside an async function?**
   → It blocks the entire event loop, freezing all tasks. Use `await asyncio.sleep()`.

9. **How do you stop one failed tool from crashing the agent?**
   → Per-task try/except, `gather(..., return_exceptions=True)`, timeouts, and retries.

10. **`gather` vs `TaskGroup`?**
    → `gather` can return exceptions as values and leaves other tasks running on failure; `TaskGroup` (3.11+) cancels sibling tasks and raises an ExceptionGroup.

11. **How do you avoid rate-limit (429) errors with many concurrent LLM calls?**
    → Limit concurrency with `asyncio.Semaphore`, and retry with exponential backoff.

12. **How do you debug async code?**
    → Logging with timestamps and task names, `asyncio.run(debug=True)` / `PYTHONASYNCIODEBUG=1`, inspecting `asyncio.all_tasks()`, and tracing tools.

---

## 11. Glossary: Every Term Explained Simply

| Term | Simple meaning |
|---|---|
| **API** | A way for code to talk to another service |
| **`async def`** | Defines a function that can pause and resume |
| **`async for`** | Loop that awaits each next item |
| **`async with`** | Async setup + guaranteed cleanup block |
| **Async-first** | Designed mainly for async use |
| **Asynchronous** | Switch to other work while waiting |
| **`asyncio`** | Python's built-in async library |
| **`await`** | Pause here until this finishes; others can run |
| **Awaitable** | Anything allowed after `await` |
| **Backoff (exponential)** | Wait 1s, 2s, 4s... between retries |
| **Blocking** | Freezes everything until it finishes |
| **Bottleneck** | Slowest part holding everything back |
| **Callback** | Function scheduled to run later |
| **Cancel / CancelledError** | Stop a task early / the error raised inside it |
| **Chunk** | One small piece of a streamed response |
| **Client** | Library object that talks to a service |
| **Concurrency** | Handling many tasks by switching between them |
| **Cooperative multitasking** | Tasks voluntarily take turns at `await` |
| **Coroutine** | Pausable/resumable async function |
| **CPU / Core** | Computer's processor / one independent processor unit |
| **CPU-bound** | Slow because of heavy computing |
| **Debug mode** | Extra warnings to find bugs |
| **Endpoint** | A URL in a web API that does one job |
| **Entry point** | Where the program starts |
| **Event loop** | Manager that runs and switches between tasks |
| **Exception / ExceptionGroup** | An error / several errors bundled together |
| **`except*`** | Catches errors from an ExceptionGroup |
| **Execution path** | The actual order code runs in |
| **FastAPI** | Python web-API framework with async support |
| **`finally`** | Block that always runs (cleanup) |
| **Future** | Low-level "result available later" object |
| **`gather()`** | Run many awaitables, collect all results |
| **Garbage collection** | Python auto-deleting unused objects |
| **GIL** | Lock allowing one thread to run Python at a time |
| **Graceful degradation** | Keep working partially when something fails |
| **HTTP / 429** | Web request protocol / "too many requests" error |
| **httpx / aiohttp** | Async-capable HTTP libraries |
| **I/O** | Input/Output: network, disk, database |
| **I/O-bound** | Slow because of waiting on I/O |
| **Interleave** | Mixing steps from different tasks |
| **Jitter** | Small random delay added to retries |
| **Latency** | Waiting time for a response |
| **Library** | Ready-made code you import |
| **Logging** | Recording messages with time and level |
| **Multiprocessing** | Using multiple processes for true parallelism |
| **Non-blocking** | Doesn't freeze; others can run |
| **Parallel tool calls** | LLM asking for several tools in one turn |
| **Parallelism** | Truly running at the same instant on many cores |
| **Process** | A separate running program with its own memory |
| **Queue** | First-in, first-out waiting line |
| **Rate limit** | Max requests allowed per time period |
| **Retry** | Trying a failed operation again |
| **Semaphore** | Allows only N tasks in a section at once |
| **Streaming** | Receiving output piece by piece |
| **Structured concurrency** | All tasks in a block finish before the block ends |
| **Synchronous** | One after another, waiting each time |
| **Task** | Coroutine scheduled on the event loop |
| **TaskGroup** | Safe way to run and manage a group of tasks |
| **Thread** | One line of execution inside a program |
| **Timeout** | Maximum allowed waiting time |
| **Timestamp** | Exact time something happened |
| **`to_thread()`** | Run blocking code in a separate thread |
| **Tracing** | Timeline record of every step in a request |

---

> 🚀 **Next topics you might explore:** Pydantic for structured outputs, FastAPI for serving agents, LangGraph, Multi-Agent Systems, MCP.
>
> *Happy learning! 🎉*
