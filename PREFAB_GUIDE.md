# Prefab: The Generative UI Framework for Agentic AI — Complete Guide

> **"The generative UI framework that even humans can use."**
>
> Prefab lets you build rich, interactive interfaces in pure Python that AI agents can generate on-the-fly, stream to users, and wire to MCP tools — all without writing a single line of JavaScript.

---

## Table of Contents

1. [What Is Prefab?](#1-what-is-prefab)
2. [Why Prefab for Agentic AI?](#2-why-prefab-for-agentic-ai)
3. [Installation](#3-installation)
4. [Your First App — 60-Second Quickstart](#4-your-first-app--60-second-quickstart)
5. [Core Concepts](#5-core-concepts)
   - 5.1 [Components](#51-components)
   - 5.2 [State Management](#52-state-management)
   - 5.3 [Reactive Expressions](#53-reactive-expressions)
   - 5.4 [Actions](#54-actions)
6. [Running Prefab Apps](#6-running-prefab-apps)
   - 6.1 [Local Preview Server](#61-local-preview-server)
   - 6.2 [REST API / FastAPI Server](#62-rest-api--fastapi-server)
   - 6.3 [MCP App (Claude Desktop, ChatGPT)](#63-mcp-app-claude-desktop-chatgpt)
7. [Component Library Reference](#7-component-library-reference)
   - 7.1 [Layout](#71-layout)
   - 7.2 [Form Controls](#72-form-controls)
   - 7.3 [Data Visualization](#73-data-visualization)
   - 7.4 [Display & Feedback](#74-display--feedback)
   - 7.5 [Control Flow](#75-control-flow)
8. [Actions Deep Dive](#8-actions-deep-dive)
   - 8.1 [Client-Side Actions](#81-client-side-actions)
   - 8.2 [Server / MCP Actions](#82-server--mcp-actions)
   - 8.3 [Action Chaining & Callbacks](#83-action-chaining--callbacks)
9. [Styling & Themes](#9-styling--themes)
10. [Agentic AI Integration Patterns](#10-agentic-ai-integration-patterns)
    - 10.1 [Agent-Generated UIs via MCP](#101-agent-generated-uis-via-mcp)
    - 10.2 [Calling MCP Tools from UI](#102-calling-mcp-tools-from-ui)
    - 10.3 [File Uploads for Agents](#103-file-uploads-for-agents)
    - 10.4 [Polling Long-Running Agent Tasks](#104-polling-long-running-agent-tasks)
    - 10.5 [Silently Updating Model Context](#105-silently-updating-model-context)
    - 10.6 [Sending Messages Back to the Agent](#106-sending-messages-back-to-the-agent)
11. [Complete Examples](#11-complete-examples)
    - 11.1 [Reactive Binding Demo](#111-reactive-binding-demo)
    - 11.2 [Sales Dashboard](#112-sales-dashboard)
    - 11.3 [Deploy Pipeline Tracker](#113-deploy-pipeline-tracker)
    - 11.4 [Expense Tracker with Budget Alert](#114-expense-tracker-with-budget-alert)
    - 11.5 [File Upload + Agent Processing](#115-file-upload--agent-processing)
    - 11.6 [FastAPI Search App](#116-fastapi-search-app)
12. [Best Practices for Agentic AI](#12-best-practices-for-agentic-ai)
13. [Troubleshooting & FAQs](#13-troubleshooting--faqs)

---

## 1. What Is Prefab?

**Prefab** is a Python-first generative UI framework that compiles to a self-contained React renderer. You write UI in Python using context managers — no HTML, no JSX, no JavaScript required.

Key properties:

| Property | Detail |
|---|---|
| **Language** | Pure Python (3.10+) |
| **Components** | 100+ prebuilt (charts, forms, tables, dialogs, etc.) |
| **Reactivity** | Browser-side state store; no full-page reloads |
| **Streaming** | Token-efficient DSL designed for LLM streaming |
| **Backends** | Works with FastAPI, Flask, Django, or standalone |
| **MCP** | First-class support for Claude Desktop, ChatGPT, and other MCP hosts |
| **Output** | Bundled React renderer, self-contained HTML or JSON |

---

## 2. Why Prefab for Agentic AI?

Traditional UI frameworks require a human to write code before a UI exists. Prefab flips this:

- **An AI agent can *generate* a Prefab UI at runtime** by emitting Python/JSON that the framework renders immediately.
- **Token-efficient DSL** — the component protocol is compact enough for LLMs to write inline.
- **MCP-native** — agents running in Claude Desktop or ChatGPT can return a Prefab component tree as a tool result and the host renders it as a rich interactive card.
- **No JavaScript knowledge needed** — agents don't need to know React, CSS, or the DOM.
- **Streaming-compatible** — components can be declared incrementally as the agent generates output.
- **Two-way data flow** — users interact with the rendered UI, which can call back into the agent's tools (`CallTool`) or send messages to the chat (`SendMessage`).

This makes Prefab the natural choice for building **AI-powered dashboards**, **dynamic forms**, **interactive tool results**, and **agent control panels**.

---

## 3. Installation

```bash
pip install prefab-ui
```

**Requirements:** Python 3.10 or later.

For REST API/server mode, install FastAPI and Uvicorn separately:

```bash
pip install fastapi uvicorn
```

Verify installation:

```bash
prefab --version
```

---

## 4. Your First App — 60-Second Quickstart

Create `app.py`:

```python
from prefab_ui.app import PrefabApp
from prefab_ui.components import (
    Badge, Card, CardContent, CardFooter,
    Column, H3, Input, Muted, Row
)
from prefab_ui.rx import Rx

# Reactive state — "name" is the state key, "world" is the default value
name = Rx("name").default("world")

with PrefabApp(css_class="max-w-md mx-auto") as app:
    with Card():
        with CardContent():
            with Column(gap=3):
                H3(f"Hello, {name}!")
                Muted("Type below and watch this update in real time.")
                # name= syncs this input's value to the "name" state key
                Input(name="name", placeholder="Your name...")

        with CardFooter():
            with Row(gap=2):
                Badge(f"Name: {name}", variant="default")
                Badge("Prefab", variant="success")
```

Run the development server:

```bash
prefab serve app.py --reload
```

Open `http://127.0.0.1:5175` in your browser. Type in the input and watch the heading and badge update in real time — no page refresh, no JavaScript.

**What just happened?**

1. `Rx("name")` creates a reactive reference to a browser-side state key called `name`.
2. `Input(name="name")` binds the input's value to that state key automatically.
3. `f"Hello, {name}!"` compiles to `"Hello, {{ name }}!"` — a live template that re-renders when `name` changes.
4. The `--reload` flag hot-reloads the UI when you save `app.py`.

---

## 5. Core Concepts

### 5.1 Components

Every UI element is a Python class. Simple components stand alone; container components accept children via `with` blocks.

```python
from prefab_ui.components import Text, Button, Card, CardContent, Column, Row

# Standalone component
Text("Hello, world!")

# Container with children
with Card():
    with CardContent():
        with Column(gap=4):
            Text("Line one")
            Text("Line two")
```

**Indentation = Visual hierarchy.** Each level of `with` nesting corresponds to a level of visual nesting in the rendered output.

#### Compound Components

Some components are composed of sub-components:

```python
from prefab_ui.components import (
    Card, CardHeader, CardTitle, CardContent, CardFooter,
    Tabs, Tab,
    Accordion, AccordionItem
)

# Card with header, content, footer
with Card():
    with CardHeader():
        CardTitle("My Card")
    with CardContent():
        Text("Content goes here")
    with CardFooter():
        Button("Action")

# Tabbed interface
with Tabs():
    with Tab(label="Overview"):
        Text("Overview content")
    with Tab(label="Details"):
        Text("Details content")
```

#### Build-Time vs Render-Time

```python
# Build-time loop — baked into the component tree at generation time
items = ["Alice", "Bob", "Charlie"]
with Column():
    for item in items:
        Text(item)

# Render-time loop — responds to dynamic state changes
from prefab_ui.components.control_flow import ForEach
with ForEach("users"):   # "users" is a state key holding a list
    Text("{{ $item.name }}")  # $item = current element, $index = position
```

Use Python `for` loops for static data; use `ForEach` when the list comes from state or a server call.

---

### 5.2 State Management

Prefab's state is a centralized **browser-side key-value store**. All components that reference a state key automatically re-render when that key changes.

#### Initializing State

```python
from prefab_ui.app import PrefabApp

# Seed initial state through PrefabApp
with PrefabApp(state={"count": 0, "title": "Dashboard", "items": []}) as app:
    ...
```

#### The `Rx` Class

`Rx` creates a reactive reference to a state key. Operations on `Rx` objects compile to protocol expressions:

```python
from prefab_ui.rx import Rx, STATE

count = Rx("count")            # reference to state key "count"
items = Rx("items")            # reference to state key "items"

Text(count + 10)               # renders as "{{ count + 10 }}"
Text(count.length())           # renders as "{{ count | length }}"

# STATE constant — shorthand without explicit Rx
Text(STATE.count)              # equivalent to Text(Rx("count"))
```

#### Nested State with Dot Notation

```python
Input(name="profile.name")          # binds to nested field
Text("{{ profile.name }}")          # reads nested value
Text("{{ todos.0.done }}")          # accesses first array element's "done" field
```

Missing path segments resolve to `undefined` gracefully (no runtime errors).

#### Reading vs Writing State

| Mechanism | When to Use |
|---|---|
| `Rx("key")` / `STATE.key` | Reading state in templates |
| `Input(name="key")` | Form control auto-sync |
| `SetState("key", value)` | Explicit assignment via action |
| `ToggleState("key")` | Flip a boolean |
| `AppendState("key", item)` | Add to a list |
| `PopState("key", index)` | Remove from a list |

---

### 5.3 Reactive Expressions

Any string prop can contain `{{ }}` template expressions that the renderer evaluates against current state and re-evaluates automatically on state change.

#### Basic Syntax

```python
Text("{{ count }}")                          # direct state reference
Text("{{ count + 10 }}")                     # arithmetic
Progress(value="{{ volume }}")               # drive component props
Text("{{ count > 5 ? 'High' : 'Low' }}")    # ternary
```

#### The `Rx` Python API

The `Rx` class gives you editor auto-complete and type safety:

```python
from prefab_ui.rx import Rx

count = Rx("count")
level = Rx("level")

Text(count + 10)                            # → "{{ count + 10 }}"
Text(count * 2)                             # → "{{ count * 2 }}"
Text((count > 5).then("High", "Low"))       # → "{{ count > 5 ? 'High' : 'Low' }}"
Text(f"Score: {count} / 100")              # → "Score: {{ count }} / 100"
```

#### Supported Operators

```python
# Arithmetic
count + 10      # addition
count - 5       # subtraction
count * 2       # multiplication
count / 4       # division
-count          # negation

# Comparisons
count > 75      # greater than
count >= 50     # greater than or equal
count < 25      # less than
count <= 10     # less than or equal
count == 0      # equality
count != 0      # inequality

# Logical (use bitwise operators — Python's and/or don't work with Rx)
(count > 10) & (count < 90)    # AND
(count < 10) | (count > 90)    # OR
~active                         # NOT

# IMPORTANT: Always wrap comparisons in parentheses with & and |
# because bitwise operators bind tighter than comparisons in Python
```

#### Formatting Pipes

```python
# Number formatting
revenue.currency()           # → "{{ revenue | currency }}"
rate.percent()               # → "{{ rate | percent }}"
count.number()               # → "{{ count | number }}"
big_num.compact()            # → "{{ big_num | compact }}"  (e.g., "1.2K")
value.round()                # → "{{ value | round }}"
value.abs()                  # → "{{ value | abs }}"

# Date/time
timestamp.date()             # → "{{ timestamp | date }}"
timestamp.time()             # → "{{ timestamp | time }}"
timestamp.datetime()         # → "{{ timestamp | datetime }}"

# Strings
name.upper()                 # → "{{ name | upper }}"
name.lower()                 # → "{{ name | lower }}"
name.truncate(20)            # → "{{ name | truncate:20 }}"
count.pluralize("item")      # → "{{ count | pluralize:'item' }}"

# Arrays
items.length()               # → "{{ items | length }}"
tags.join(", ")              # → "{{ tags | join:', ' }}"
items.first()                # → "{{ items | first }}"
items.last()                 # → "{{ items | last }}"

# Chaining pipes
name.lower().truncate(20)    # → "{{ name | lower | truncate:20 }}"
```

#### Component `.rx` Property

Every interactive component exposes an `.rx` property that returns `Rx(component.name)`:

```python
slider = Slider(name="volume", value=50, min=0, max=100)
Text(f"Volume: {slider.rx}%")      # safe forward reference without string duplication
```

#### Special Variables

| Variable | Where Available | Meaning |
|---|---|---|
| `$event` | Action arguments | Value emitted by the triggering component |
| `$result` | `on_success` callbacks | Return value from a successful async action |
| `$error` | `on_error` callbacks | Error message from a failed action |
| `$item` | Inside `ForEach` | Current iteration element |
| `$index` | Inside `ForEach` | Current iteration index (0-based) |

---

### 5.4 Actions

Actions are the mechanism for responding to user interactions — they drive state changes, call servers, and provide UI feedback.

#### Attaching Actions

Actions attach to event handler props on components:

```python
from prefab_ui.actions import SetState, ShowToast

Button("Click me", on_click=SetState("clicked", True))
Input(name="q", on_change=SetState("last_query", "{{ $event }}"))
Slider(name="vol", on_change=ShowToast(f"Volume: {Rx('vol')}"))
```

Common event handler props: `on_click`, `on_change`, `on_submit`, `on_success`, `on_error`.

---

## 6. Running Prefab Apps

### 6.1 Local Preview Server

The built-in server is the fastest way to develop and preview:

```bash
# Basic
prefab serve app.py

# With auto-reload on file save
prefab serve app.py --reload

# Custom port
prefab serve app.py --port 8080
```

The browser opens automatically at `http://127.0.0.1:5175`.

### 6.2 REST API / FastAPI Server

For production apps with a backend, use FastAPI (or any WSGI/ASGI framework):

```python
from fastapi import FastAPI
from fastapi.responses import HTMLResponse
from prefab_ui.app import PrefabApp
from prefab_ui.components import Column, Text

app = FastAPI()

@app.get("/", response_class=HTMLResponse)
def page():
    with Column(gap=4) as view:
        Text("Hello from FastAPI!")
    return HTMLResponse(PrefabApp(title="My App", view=view).html())
```

Run with:

```bash
uvicorn app:app --reload
```

Three types of routes you'll typically define:

| Route Type | Returns | Purpose |
|---|---|---|
| **Page routes** | `HTMLResponse(app.html())` | Full page, user navigates to it |
| **Data routes** | `JSONResponse(data)` | Raw JSON, fetched by `Fetch` actions |
| **Component routes** | `JSONResponse(view.to_json())` | Component trees, rendered by `Slot` |

### 6.3 MCP App (Claude Desktop, ChatGPT)

Prefab has **first-class MCP support**. An agent running in Claude Desktop or ChatGPT can call your MCP server's tools, and those tools can return Prefab component trees that the host renders as interactive cards.

**General pattern:**

1. Expose an MCP server (using `mcp` or `fastmcp` Python packages).
2. Your tool functions build a Prefab component tree and return `.to_json()`.
3. The MCP host (Claude Desktop, ChatGPT) renders the JSON as a rich UI card.
4. The UI can use `CallTool` to call back into your MCP server's other tools.

```python
from mcp.server.fastmcp import FastMCP
from prefab_ui.app import PrefabApp
from prefab_ui.components import Card, CardContent, H3, Text, Column, Badge

mcp = FastMCP("My Agent")

@mcp.tool()
def show_status(service: str) -> dict:
    """Return a Prefab UI card showing service status."""
    with Column(gap=3) as view:
        with Card():
            with CardContent():
                H3(f"Status: {service}")
                Badge("Running", variant="success")
                Text("All systems operational")
    return PrefabApp(view=view).to_json()
```

---

## 7. Component Library Reference

### 7.1 Layout

```python
from prefab_ui.components import (
    Column, Row, Grid, Dashboard, Container, Pages, Page
)

# Vertical stack
with Column(gap=4):
    Text("Item 1")
    Text("Item 2")

# Horizontal row
with Row(gap=2, align="center"):
    Badge("Active", variant="success")
    Text("System is running")

# CSS Grid
with Grid(columns=3, gap=4):
    for i in range(6):
        Text(f"Cell {i+1}")

# 12-column dashboard grid
with Dashboard(columns=12, row_height="auto"):
    with Card(col_span=6):
        Text("Left panel")
    with Card(col_span=6):
        Text("Right panel")
```

### 7.2 Form Controls

```python
from prefab_ui.components import (
    Input, Textarea, Checkbox, Radio, Select, Combobox,
    Slider, Switch, DatePicker, Calendar, Form, Button
)

# Text input — binds value to "username" state key
Input(name="username", placeholder="Enter username...")

# Textarea
Textarea(name="bio", placeholder="Tell us about yourself...", rows=4)

# Slider — writes position to "volume" on every drag
Slider(name="volume", value=50, min=0, max=100, step=1)

# Toggle switch
Switch(name="notifications", label="Enable notifications")

# Checkbox
Checkbox(name="agree", label="I agree to the terms")

# Dropdown select
Select(
    name="country",
    options=[
        {"label": "United States", "value": "us"},
        {"label": "United Kingdom", "value": "uk"},
        {"label": "Canada", "value": "ca"},
    ]
)

# Searchable combobox
Combobox(
    name="framework",
    placeholder="Search frameworks...",
    options=[{"label": "React", "value": "react"}, {"label": "Vue", "value": "vue"}]
)

# Form wrapper — on_submit fires when user submits
from prefab_ui.actions import Fetch
with Form(on_submit=Fetch.post("/api/save", body={"data": "{{ formData }}"})):
    Input(name="formData")
    Button("Submit", type="submit")
```

### 7.3 Data Visualization

```python
from prefab_ui.components import (
    BarChart, LineChart, AreaChart, PieChart, ScatterChart,
    RadarChart, Histogram, Sparkline
)

data = [
    {"month": "Jan", "revenue": 4000, "cost": 2400},
    {"month": "Feb", "revenue": 3000, "cost": 1398},
    {"month": "Mar", "revenue": 5000, "cost": 3200},
]

# Bar chart
BarChart(
    data=data,
    x_key="month",
    bars=[{"key": "revenue", "label": "Revenue"}, {"key": "cost", "label": "Cost"}],
    height=300
)

# Area chart (great for time series)
AreaChart(
    data=data,
    x_key="month",
    areas=[
        {"key": "revenue", "label": "Revenue", "color": "#2563eb"},
        {"key": "cost", "label": "Cost", "color": "#dc2626"},
    ],
    curve="monotone",
    height=250
)

# Pie / Donut chart
PieChart(
    data=[
        {"name": "Compute", "value": 42000},
        {"name": "Storage", "value": 18500},
        {"name": "Network", "value": 12300},
    ],
    name_key="name",
    value_key="value",
    inner_radius=60,   # > 0 makes it a donut chart
    legend=True,
    height=300
)

# Scatter plot
ScatterChart(
    data=data,
    x_key="revenue",
    y_key="cost",
    height=300
)

# Inline sparkline
Sparkline(data=[1, 3, 2, 5, 4, 7, 6], color="#2563eb", height=40)
```

### 7.4 Display & Feedback

```python
from prefab_ui.components import (
    Text, H1, H2, H3, Muted, Badge, Alert, Progress,
    Loader, Tooltip, Popover, Dialog, Markdown, Code,
    DataTable, Image, Icon
)

# Typography
H1("Page Title")
H2("Section Header")
H3("Subsection")
Text("Body text")
Muted("Secondary / helper text")

# Badge
Badge("Active", variant="success")    # variants: default, success, warning, destructive, secondary
Badge("{{ count }} items")            # reactive badge

# Alert
Alert("Budget limit approaching!", variant="warning")
Alert("Deploy complete", variant="success")

# Progress bar
Progress(value="{{ progress }}", max=100)

# Loading spinner
Loader(size="sm")    # sizes: sm, md, lg

# Data table — fully sortable and searchable
DataTable(
    data=[
        {"id": 1, "name": "Alice", "role": "Engineer", "status": "Active"},
        {"id": 2, "name": "Bob", "role": "Designer", "status": "Inactive"},
    ],
    columns=[
        {"key": "id", "label": "ID", "sortable": True},
        {"key": "name", "label": "Name", "sortable": True},
        {"key": "role", "label": "Role"},
        {"key": "status", "label": "Status"},
    ],
    searchable=True,
    page_size=10
)

# Markdown rendering
Markdown("## Hello\n\nThis is **bold** and _italic_ text.")

# Code block
Code("print('hello world')", language="python")

# Modal dialog
from prefab_ui.actions import ToggleState
with Dialog(open="{{ showDialog }}", on_close=ToggleState("showDialog")):
    Text("Dialog content here")
Button("Open Dialog", on_click=ToggleState("showDialog"))
```

### 7.5 Control Flow

```python
from prefab_ui.components.control_flow import ForEach, If, Elif, Else

# Conditional rendering
with If("{{ isLoggedIn }}"):
    Text("Welcome back!")
with Else():
    Button("Login")

# With elif
with If("{{ score > 90 }}"):
    Badge("A+", variant="success")
with Elif("{{ score > 70 }}"):
    Badge("B", variant="default")
with Else():
    Badge("Needs Improvement", variant="warning")

# Dynamic list rendering
with ForEach("users"):               # "users" = state key
    with Row(gap=2):
        Text("{{ $item.name }}")
        Badge("{{ $item.role }}")
        Button("Remove", on_click=PopState("users", "{{ $index }}"))
```

---

## 8. Actions Deep Dive

### 8.1 Client-Side Actions

These execute **instantly in the browser** with no network round-trip:

#### `SetState` — Assign a value

```python
from prefab_ui.actions import SetState

Button("Reset", on_click=SetState("count", 0))
Button("Set Name", on_click=SetState("profile.name", "Alice"))  # nested path
Input(name="q", on_change=SetState("query", "{{ $event }}"))   # capture $event
```

#### `ToggleState` — Flip a boolean

```python
from prefab_ui.actions import ToggleState

Button("Toggle Sidebar", on_click=ToggleState("sidebarOpen"))
Switch(name="darkMode", on_change=ToggleState("darkMode"))
```

#### `AppendState` — Add to a list

```python
from prefab_ui.actions import AppendState

Button("Add Item", on_click=AppendState("todos", {"text": "{{ newTodo }}", "done": False}))
# Insert at position 0 (prepend)
Button("Prepend", on_click=AppendState("logs", "New log entry", index=0))
```

#### `PopState` — Remove from a list

```python
from prefab_ui.actions import PopState

# Inside a ForEach, remove current item
Button("Delete", on_click=PopState("todos", "{{ $index }}"))
# Remove last item
Button("Pop", on_click=PopState("todos", -1))
```

#### `ShowToast` — Display notification

```python
from prefab_ui.actions import ShowToast

Button("Save", on_click=ShowToast("Saved!", variant="success"))
# variants: default, success, error, warning, info
ShowToast("{{ $error }}", variant="error", duration=5000)    # 5 seconds
ShowToast("Done", description="All 42 records were updated", variant="success")
```

#### `OpenLink` — Navigate to URL

```python
from prefab_ui.actions import OpenLink

Button("Open Docs", on_click=OpenLink("https://prefab.prefect.io/docs"))
Button("Profile", on_click=OpenLink("/users/{{ userId }}"))
```

#### `SetInterval` — Scheduled/polling actions

```python
from prefab_ui.actions import SetInterval, SetState, ShowToast
from prefab_ui.components import Button

# Poll every 5 seconds while "polling" state is True
Button("Start Polling",
    on_click=SetInterval(
        duration=5000,
        while_="{{ polling }}",
        on_tick=Fetch.get("/api/status", on_success=SetState("status", RESULT)),
        on_complete=ShowToast("Polling stopped", variant="info")
    )
)

# One-time delayed action (count=1, logic in on_complete)
SetInterval(
    duration=2000,
    count=1,
    on_complete=ShowToast("Delayed toast!", variant="info")
)

# Countdown timer
SetState("countdown", 10)
SetInterval(
    duration=1000,
    while_="{{ countdown > 0 }}",
    on_tick=SetState("countdown", "{{ countdown - 1 }}"),
    on_complete=ShowToast("Time's up!", variant="warning")
)
```

#### `CallHandler` — Run custom JavaScript

```python
from prefab_ui.actions import CallHandler
from prefab_ui.app import PrefabApp

# Register handlers in PrefabApp
js_handlers = {
    "constrainBudget": """
        (ctx) => {
            const infra = ctx.state.infra || 0;
            const dev = ctx.state.dev || 0;
            const total = 100;
            if (infra + dev > total) {
                return { infra: total - dev, dev };
            }
        }
    """,
    "validate": """
        (ctx) => {
            const email = ctx.state.email || '';
            if (!email.includes('@')) throw new Error('Invalid email');
            return true;
        }
    """
}

app = PrefabApp(js_actions=js_handlers, view=view)

# Invoke a handler
Slider(name="infra", on_change=[
    SetState("dirty", True),
    CallHandler("constrainBudget",
        on_success=SetState("budget_valid", "{{ $result }}"),
        on_error=ShowToast("{{ $error }}", variant="error")
    )
])
```

#### `OpenFilePicker` — Browser file picker

```python
from prefab_ui.actions import OpenFilePicker, CallTool

Button("Upload CSV",
    on_click=OpenFilePicker(
        accept=".csv",
        multiple=False,
        max_size=10 * 1024 * 1024,     # 10 MB
        on_success=CallTool(
            "process_csv",
            arguments={"file": "{{ $event }}"}   # $event = FileUpload object
        ),
        on_error=ShowToast("File too large or unreadable", variant="error")
    )
)

# IMPORTANT: OpenFilePicker must be the FIRST action in a chain
# because the browser closes the activation window after any async action
```

---

### 8.2 Server / MCP Actions

These cross the network and return results asynchronously.

#### `CallTool` — Invoke an MCP tool

```python
from prefab_ui.actions import CallTool, SetState, ShowToast
from prefab_ui.rx import RESULT

# Simple tool call
Button("Summarize",
    on_click=CallTool(
        "summarize_text",
        arguments={"text": "{{ inputText }}"},
        on_success=SetState("summary", RESULT),
        on_error=ShowToast("{{ $error }}", variant="error")
    )
)

# With loading state
Button("Analyze",
    on_click=[
        SetState("loading", True),
        CallTool(
            "analyze_data",
            arguments={"data": "{{ dataset }}"},
            on_success=[
                SetState("result", RESULT),
                SetState("loading", False),
                ShowToast("Analysis complete!", variant="success")
            ],
            on_error=[
                SetState("loading", False),
                ShowToast("{{ $error }}", variant="error")
            ]
        )
    ]
)
```

#### `Fetch` — HTTP requests

```python
from prefab_ui.actions import Fetch, SetState, ShowToast, AppendState
from prefab_ui.rx import RESULT

# GET request
Button("Load Users",
    on_click=Fetch.get(
        "/api/users",
        params={"role": "{{ selectedRole }}"},
        on_success=SetState("users", RESULT),
        on_error=ShowToast("Failed to load users", variant="error")
    )
)

# POST request
Button("Create User",
    on_click=[
        SetState("creating", True),
        Fetch.post(
            "/api/users",
            body={"name": "{{ newName }}", "role": "{{ newRole }}"},
            on_success=[
                AppendState("users", RESULT),
                SetState("newName", ""),
                SetState("creating", False),
                ShowToast("User created!", variant="success")
            ],
            on_error=[
                SetState("creating", False),
                ShowToast("{{ $error }}", variant="error")
            ]
        )
    ]
)

# DELETE request, then refresh list
Button("Delete",
    on_click=Fetch.delete(
        "/api/users/{{ userId }}",
        on_success=Fetch.get("/api/users", on_success=SetState("users", RESULT)),
        on_error=ShowToast("Delete failed", variant="error")
    )
)
```

---

### 8.3 Action Chaining & Callbacks

Pass a list of actions to run them sequentially. If one fails, the chain stops.

```python
from prefab_ui.actions import SetState, CallTool, ShowToast, Fetch
from prefab_ui.rx import RESULT

# Sequential execution — loading flag → server call → clear flag + show result
Button("Run Analysis",
    on_click=[
        SetState("loading", True),
        SetState("error", None),
        CallTool(
            "run_ml_analysis",
            arguments={"dataset": "{{ selectedDataset }}"},
            on_success=[
                SetState("results", RESULT),
                SetState("loading", False),
                ShowToast("Analysis complete!", variant="success")
            ],
            on_error=[
                SetState("loading", False),
                SetState("error", "{{ $error }}"),
                ShowToast("Analysis failed: {{ $error }}", variant="error")
            ]
        )
    ]
)

# Dependent server calls — use first result to drive second request
Button("Load Details",
    on_click=Fetch.get(
        "/api/summary",
        on_success=[
            SetState("summary", RESULT),
            Fetch.get(
                "/api/details/{{ $result.id }}",   # use first result
                on_success=SetState("details", RESULT)
            )
        ]
    )
)
```

---

## 9. Styling & Themes

### Tailwind CSS Classes

Every component accepts a `css_class` prop for Tailwind utility classes:

```python
Text("Alert!", css_class="text-red-500 font-bold text-xl")
Button("Submit", css_class="w-full mt-4")
with Card(css_class="shadow-lg border-2 border-blue-500"):
    ...
```

### Built-In Themes

```python
from prefab_ui.app import PrefabApp
from prefab_ui.theme import Theme

# Accent color (Tailwind name, hex, or OKLCH hue 0-360)
app = PrefabApp(accent="blue", view=view)
app = PrefabApp(accent="#2563eb", view=view)
app = PrefabApp(accent=240, view=view)           # OKLCH hue

# Color mode
app = PrefabApp(mode="dark", view=view)
app = PrefabApp(mode="light", view=view)

# Preset themes
app = PrefabApp(theme="presentation", view=view)  # dark slate, Inter font, tall tables
app = PrefabApp(theme="minimal", view=view)        # no defaults, full control

# Custom theme via CSS variables (shadcn/ui compatible)
custom = Theme(
    light_css="""
        --primary: 220 90% 56%;
        --background: 0 0% 100%;
        --card: 0 0% 98%;
        --border: 220 13% 91%;
    """,
    dark_css="""
        --primary: 217 91% 60%;
        --background: 222 84% 5%;
        --card: 222 84% 8%;
        --border: 217 33% 17%;
    """
)
app = PrefabApp(theme=custom, font="Inter", view=view)
```

---

## 10. Agentic AI Integration Patterns

This section covers the patterns most relevant to building AI-powered applications.

### 10.1 Agent-Generated UIs via MCP

The most powerful agentic pattern: your LLM generates a Prefab component tree as a tool result, and the MCP host renders it as a rich interactive card — no HTML, no React, no deployment required.

```python
# mcp_server.py
from mcp.server.fastmcp import FastMCP
from prefab_ui.app import PrefabApp
from prefab_ui.components import (
    Dashboard, Card, CardHeader, CardTitle, CardContent,
    Column, Row, H3, Text, Badge, BarChart, DataTable, Alert
)

mcp = FastMCP("Analytics Agent")

@mcp.tool()
def show_analytics(metric: str, timeframe: str) -> dict:
    """Generate and return an interactive analytics dashboard."""

    data = fetch_data(metric, timeframe)   # your actual data logic

    with Dashboard(columns=12) as view:
        # KPI cards
        with Card(col_span=4):
            with CardContent():
                H3("Total Revenue")
                Text(f"${data['total']:,.0f}", css_class="text-3xl font-bold")
                Badge(f"+{data['growth']}%", variant="success")

        with Card(col_span=4):
            with CardContent():
                H3("Orders")
                Text(str(data['orders']), css_class="text-3xl font-bold")

        with Card(col_span=4):
            with CardContent():
                H3("Avg Order Value")
                Text(f"${data['avg_order']:.2f}", css_class="text-3xl font-bold")

        # Chart
        with Card(col_span=8):
            with CardHeader():
                CardTitle("Revenue Trend")
            with CardContent():
                BarChart(data=data['trend'], x_key="date",
                         bars=[{"key": "revenue", "label": "Revenue"}], height=300)

        # Table
        with Card(col_span=12):
            with CardHeader():
                CardTitle("Recent Transactions")
            with CardContent():
                DataTable(data=data['transactions'], columns=[
                    {"key": "id", "label": "ID", "sortable": True},
                    {"key": "customer", "label": "Customer"},
                    {"key": "amount", "label": "Amount", "sortable": True},
                    {"key": "status", "label": "Status"},
                ], searchable=True, page_size=5)

    return PrefabApp(view=view).to_json()
```

### 10.2 Calling MCP Tools from UI

Users can interact with a rendered Prefab UI to call back into your MCP server's tools — creating a two-way flow between the user interface and the AI backend.

```python
from prefab_ui.actions import CallTool, SetState, ShowToast, AppendState
from prefab_ui.components import Column, Row, Input, Button, Card, CardContent
from prefab_ui.components.control_flow import ForEach, If
from prefab_ui.rx import Rx, RESULT

# Build a UI that lets users run agent tasks interactively
query = Rx("query")
results = Rx("results")
loading = Rx("loading").default(False)

with Column(gap=4) as view:
    with Card():
        with CardContent():
            with Row(gap=2):
                Input(name="query", placeholder="Ask the agent...")
                Button("Run",
                    disabled="{{ loading }}",
                    on_click=[
                        SetState("loading", True),
                        CallTool(
                            "run_agent_query",             # your MCP tool name
                            arguments={"query": "{{ query }}"},
                            on_success=[
                                AppendState("results", RESULT),
                                SetState("loading", False),
                            ],
                            on_error=[
                                SetState("loading", False),
                                ShowToast("{{ $error }}", variant="error")
                            ]
                        )
                    ]
                )

    with If("{{ loading }}"):
        Loader(size="md")

    with ForEach("results"):
        with Card(css_class="mt-2"):
            with CardContent():
                Text("{{ $item.answer }}")
```

### 10.3 File Uploads for Agents

Prefab handles file uploads client-side (as base64), keeping binary data out of the token context window. The `$event` from a `DropZone` or `OpenFilePicker` is a `FileUpload` object with `name`, `type`, `size`, and `data` (base64) fields.

```python
from prefab_ui.actions import OpenFilePicker, CallTool, SetState, ShowToast
from prefab_ui.components import Column, Card, CardContent, CardHeader, CardTitle
from prefab_ui.components import Button, Text, Badge, DropZone
from prefab_ui.components.control_flow import If
from prefab_ui.rx import Rx, RESULT

file_info = Rx("file_info")
analysis = Rx("analysis")
loading = Rx("loading").default(False)

with Column(gap=4) as view:
    with Card():
        with CardHeader():
            CardTitle("Document Analysis")
        with CardContent():
            with Column(gap=3):
                # DropZone — drag-and-drop area
                DropZone(
                    name="uploaded_file",
                    accept=".pdf,.txt,.csv",
                    multiple=False,
                    max_size=10 * 1024 * 1024,
                    on_change=[
                        SetState("file_info", {
                            "name": "{{ $event.name }}",
                            "type": "{{ $event.type }}",
                            "size": "{{ $event.size }}"
                        }),
                        SetState("loading", True),
                        CallTool(
                            "analyze_document",
                            arguments={"file": "{{ $event }}"},
                            on_success=[
                                SetState("analysis", RESULT),
                                SetState("loading", False),
                                ShowToast("Analysis complete!", variant="success")
                            ],
                            on_error=[
                                SetState("loading", False),
                                ShowToast("{{ $error }}", variant="error")
                            ]
                        )
                    ]
                )

                # Or use a button to open the file picker
                Button("Choose File",
                    on_click=OpenFilePicker(
                        accept=".pdf,.csv",
                        on_success=CallTool(
                            "analyze_document",
                            arguments={"file": "{{ $event }}"}
                        )
                    )
                )

    with If("{{ file_info }}"):
        with Card():
            with CardContent():
                Text(f"File: {file_info}.name")
                Badge(f"Type: {file_info}.type")
                Text(f"Size: {file_info}.size bytes")

    with If("{{ analysis }}"):
        with Card():
            with CardContent():
                Text("{{ analysis.summary }}")
```

### 10.4 Polling Long-Running Agent Tasks

Use `SetInterval` to poll a status endpoint while an agent task runs in the background:

```python
from prefab_ui.actions import (
    SetState, SetInterval, Fetch, ShowToast, CallTool
)
from prefab_ui.components import Column, Button, Progress, Text, Card, CardContent
from prefab_ui.components.control_flow import If
from prefab_ui.rx import Rx, RESULT

job_id = Rx("job_id")
progress = Rx("progress").default(0)
status = Rx("status").default("idle")
polling = Rx("polling").default(False)

with Column(gap=4) as view:
    with Card():
        with CardContent():
            Button("Start Agent Task",
                disabled="{{ polling }}",
                on_click=[
                    SetState("status", "starting"),
                    # Start the long-running job
                    CallTool("start_pipeline",
                        arguments={"config": "{{ pipelineConfig }}"},
                        on_success=[
                            SetState("job_id", "{{ $result.job_id }}"),
                            SetState("polling", True),
                            SetState("status", "running"),
                            # Start polling every 3 seconds
                            SetInterval(
                                duration=3000,
                                while_="{{ polling }}",
                                on_tick=Fetch.get(
                                    "/api/jobs/{{ job_id }}/status",
                                    on_success=[
                                        SetState("progress", "{{ $result.progress }}"),
                                        SetState("status", "{{ $result.status }}"),
                                        # Stop polling when done
                                        SetState("polling",
                                            "{{ $result.status != 'complete' and $result.status != 'failed' }}"
                                        )
                                    ]
                                ),
                                on_complete=ShowToast("Job finished!", variant="success")
                            )
                        ]
                    )
                ]
            )

    with If("{{ polling }}"):
        with Column(gap=2):
            Text(f"Status: {status}")
            Progress(value="{{ progress }}", max=100)
            Text(f"{progress}% complete")
            Button("Cancel",
                on_click=[
                    SetState("polling", False),
                    CallTool("cancel_job", arguments={"job_id": "{{ job_id }}"})
                ]
            )
```

### 10.5 Silently Updating Model Context

The `UpdateContext` action silently injects information into the AI model's context without the user seeing it in the chat. This is useful for keeping the agent aware of UI state changes.

```python
from prefab_ui.actions import UpdateContext, SetState

# When the user selects a customer, silently tell the model about it
Select(
    name="customer_id",
    options=customer_options,
    on_change=[
        SetState("customer_id", "{{ $event }}"),
        UpdateContext(
            "The user has selected customer {{ $event }}. "
            "Their current balance is {{ customers[$event].balance }}. "
            "Recent activity: {{ customers[$event].recent_activity }}"
        )
    ]
)
```

### 10.6 Sending Messages Back to the Agent

`SendMessage` sends a message to the chat, as if the user typed it — the agent sees it and responds.

```python
from prefab_ui.actions import SendMessage, SetState

# Button that sends a structured query to the agent
Button("Explain This Chart",
    on_click=SendMessage(
        "Please explain the revenue trend shown in the chart. "
        "Current data: revenue={{ monthly_revenue }}, "
        "growth={{ growth_rate }}%"
    )
)

# After a file upload, prompt the agent
OpenFilePicker(
    accept=".csv",
    on_success=[
        SetState("file_data", "{{ $event }}"),
        SendMessage("I've uploaded a CSV file named {{ $event.name }}. "
                    "Please analyze it and give me a summary.")
    ]
)
```

---

## 11. Complete Examples

### 11.1 Reactive Binding Demo

A slider that drives multiple UI elements simultaneously through reactive expressions:

```python
from prefab_ui.app import PrefabApp
from prefab_ui.components import Column, Card, CardContent, H3, Row, Text, Slider, Progress, Ring
from prefab_ui.rx import Rx

level = Rx("level")
variant = (level > 75).then("destructive", (level > 40).then("warning", "success"))

with PrefabApp(state={"level": 50}) as app:
    with Card(css_class="max-w-lg mx-auto"):
        with CardContent():
            with Column(gap=6):
                H3("Reactive Binding Demo")
                Slider(name="level", min=0, max=100, value=50)

                Ring(value=level, max=100, variant=variant)

                with Column(gap=3):
                    Text("Primary")
                    Progress(value=level, max=100, variant=variant)

                    Text("Inverse")
                    Progress(value=100 - level, max=100)

                    Text("Doubled")
                    Progress(value=level * 2, max=200)

                    Text("Halved")
                    Progress(value=level / 2, max=50)

                Text(
                    (level > 75).then(
                        "Critical level!",
                        (level > 40).then("Moderate level", "Low level")
                    ),
                    css_class="font-semibold text-center"
                )
```

### 11.2 Sales Dashboard

A complete analytics dashboard with KPI cards, charts, and a data table:

```python
from prefab_ui.app import PrefabApp
from prefab_ui.components import (
    Dashboard, Card, CardHeader, CardTitle, CardContent,
    Column, Row, Text, H3, Badge, AreaChart, PieChart, DataTable
)

monthly_data = [
    {"month": "Jan", "revenue": 42000, "cost": 28000},
    {"month": "Feb", "revenue": 38000, "cost": 25000},
    {"month": "Mar", "revenue": 55000, "cost": 32000},
    {"month": "Apr", "revenue": 61000, "cost": 38000},
    {"month": "May", "revenue": 49000, "cost": 30000},
    {"month": "Jun", "revenue": 72000, "cost": 42000},
]

category_data = [
    {"name": "Electronics", "value": 98500},
    {"name": "Clothing", "value": 67200},
    {"name": "Home & Garden", "value": 54100},
    {"name": "Sports", "value": 43800},
    {"name": "Books", "value": 21150},
]

orders = [
    {"id": "#1042", "customer": "Alice Johnson", "product": "Widget Pro", "total": "$284.00", "status": "Shipped"},
    {"id": "#1041", "customer": "Bob Smith", "product": "Gadget Plus", "total": "$119.00", "status": "Processing"},
    {"id": "#1040", "customer": "Carol White", "product": "Super Widget", "total": "$549.00", "status": "Delivered"},
]

with Dashboard(columns=12, row_height="auto") as view:
    # KPI cards
    for label, value, change, up in [
        ("Revenue", "$284,750", "+12.5%", True),
        ("Orders", "1,842", "+8.2%", True),
        ("Avg Order", "$154.59", "-3.1%", False),
        ("Growth", "12.5%", "+2.4pp", True),
    ]:
        with Card(col_span=3):
            with CardContent():
                with Column(gap=1):
                    H3(label)
                    Text(value, css_class="text-2xl font-bold")
                    Badge(change, variant="success" if up else "destructive")

    # Area chart
    with Card(col_span=8):
        with CardHeader():
            CardTitle("Revenue Over Time")
        with CardContent():
            AreaChart(
                data=monthly_data,
                x_key="month",
                areas=[
                    {"key": "revenue", "label": "Revenue", "color": "#2563eb"},
                    {"key": "cost", "label": "Cost", "color": "#dc2626"},
                ],
                curve="monotone",
                height=300
            )

    # Pie chart
    with Card(col_span=4):
        with CardHeader():
            CardTitle("Revenue by Category")
        with CardContent():
            PieChart(
                data=category_data,
                name_key="name",
                value_key="value",
                inner_radius=60,
                legend=True,
                height=300
            )

    # Orders table
    with Card(col_span=12):
        with CardHeader():
            CardTitle("Recent Orders")
        with CardContent():
            DataTable(
                data=orders,
                columns=[
                    {"key": "id", "label": "Order ID", "sortable": True},
                    {"key": "customer", "label": "Customer", "sortable": True},
                    {"key": "product", "label": "Product"},
                    {"key": "total", "label": "Total", "sortable": True},
                    {"key": "status", "label": "Status"},
                ],
                searchable=True,
                page_size=5
            )

app = PrefabApp(title="Sales Dashboard", view=view, accent="blue")
```

### 11.3 Deploy Pipeline Tracker

Monitor CI/CD pipeline status with rich inline components in a data table:

```python
from prefab_ui.app import PrefabApp
from prefab_ui.components import (
    Card, CardHeader, CardTitle, CardContent, DataTable, Badge, Progress, Text
)

services = [
    {"service": "auth-service",  "stage": "Production", "progress": 100, "status": "100/100"},
    {"service": "data-ingest",   "stage": "Staging",    "progress": 72,  "status": "72/100"},
    {"service": "ml-pipeline",   "stage": "Canary",     "progress": 45,  "status": "45/100"},
    {"service": "web-frontend",  "stage": "Rolling",    "progress": 88,  "status": "88/100"},
    {"service": "event-bus",     "stage": "Blocked",    "progress": 12,  "status": "12/100"},
    {"service": "cache-layer",   "stage": "Production", "progress": 100, "status": "100/100"},
    {"service": "search-index",  "stage": "Rolling",    "progress": 63,  "status": "63/100"},
]

stage_variants = {
    "Production": "success",
    "Staging": "default",
    "Canary": "warning",
    "Rolling": "default",
    "Blocked": "destructive",
}

# Enrich data with component cells
for svc in services:
    svc["stage_badge"] = Badge(svc["stage"], variant=stage_variants[svc["stage"]])
    svc["progress_bar"] = Progress(value=svc["progress"], max=100, size="lg")

with Card() as view:
    with CardHeader():
        CardTitle("Deploy Pipeline")
    with CardContent():
        DataTable(
            data=services,
            columns=[
                {"key": "service", "label": "Service", "sortable": True},
                {"key": "stage_badge", "label": "Stage", "sortable": True},
                {"key": "progress_bar", "label": "Progress"},
                {"key": "status", "label": "Status"},
            ],
            searchable=True,
            page_size=10
        )

app = PrefabApp(title="Deploy Tracker", view=view)
```

### 11.4 Expense Tracker with Budget Alert

Expense dashboard with pie chart, data table, and budget warning:

```python
from prefab_ui.app import PrefabApp
from prefab_ui.components import (
    Column, Row, Card, CardContent, H3, Text, Badge, Alert,
    PieChart, DataTable
)

expenses = [
    {"category": "Compute",    "amount": 42000, "pct": 51},
    {"category": "Storage",    "amount": 18500, "pct": 22},
    {"category": "Network",    "amount": 12300, "pct": 15},
    {"category": "Monitoring", "amount": 4200,  "pct": 5},
    {"category": "CI/CD",      "amount": 3800,  "pct": 5},
    {"category": "Support",    "amount": 2100,  "pct": 3},
]

total_spend = sum(e["amount"] for e in expenses)
budget = 95000
budget_pct = int(total_spend / budget * 100)

with Column(gap=4) as view:
    # Budget alert
    if budget_pct > 85:
        Alert(
            f"Approaching Budget Limit — {budget_pct}% utilized",
            variant="warning"
        )

    # Metric cards
    with Row(gap=4):
        with Card():
            with CardContent():
                H3("Total Spend")
                Text(f"${total_spend:,}", css_class="text-3xl font-bold text-red-600")
        with Card():
            with CardContent():
                H3("Budget Remaining")
                Text(f"${budget - total_spend:,}", css_class="text-3xl font-bold text-green-600")

    # Pie chart
    with Card():
        with CardContent():
            PieChart(
                data=expenses,
                name_key="category",
                value_key="amount",
                inner_radius=60,
                legend=True,
                height=300
            )

    # Expense breakdown table
    with Card():
        with CardContent():
            DataTable(
                data=[{**e, "pct_badge": Badge(f"{e['pct']}%", variant="default")} for e in expenses],
                columns=[
                    {"key": "category", "label": "Category", "sortable": True},
                    {"key": "amount",   "label": "Amount ($)", "sortable": True},
                    {"key": "pct_badge","label": "Share"},
                ],
                searchable=True,
                page_size=10
            )

app = PrefabApp(title="Expense Tracker", view=view, accent="rose")
```

### 11.5 File Upload + Agent Processing

A UI for uploading documents and having an agent analyze them:

```python
from prefab_ui.app import PrefabApp
from prefab_ui.components import (
    Column, Card, CardHeader, CardTitle, CardContent,
    Text, H3, Badge, Muted, DropZone, Loader
)
from prefab_ui.components.control_flow import If
from prefab_ui.actions import CallTool, SetState, ShowToast
from prefab_ui.rx import Rx, RESULT

file_name = Rx("file_name")
loading = Rx("loading").default(False)
result = Rx("result")

with Column(gap=4, css_class="max-w-2xl mx-auto") as view:
    with Card():
        with CardHeader():
            CardTitle("Document Analyzer")
        with CardContent():
            with Column(gap=3):
                Muted("Upload a PDF, CSV, or text file for AI analysis.")
                DropZone(
                    name="doc",
                    accept=".pdf,.csv,.txt",
                    multiple=False,
                    max_size=10 * 1024 * 1024,
                    on_change=[
                        SetState("file_name", "{{ $event.name }}"),
                        SetState("result", None),
                        SetState("loading", True),
                        CallTool(
                            "analyze_document",
                            arguments={
                                "file_name": "{{ $event.name }}",
                                "file_type": "{{ $event.type }}",
                                "file_data": "{{ $event.data }}"  # base64
                            },
                            on_success=[
                                SetState("result", RESULT),
                                SetState("loading", False),
                                ShowToast("Analysis complete!", variant="success")
                            ],
                            on_error=[
                                SetState("loading", False),
                                ShowToast("{{ $error }}", variant="error")
                            ]
                        )
                    ]
                )

    with If("{{ loading }}"):
        with Card():
            with CardContent():
                with Column(gap=2, css_class="items-center"):
                    Loader(size="lg")
                    Text(f"Analyzing {file_name}...")

    with If("{{ result }}"):
        with Card():
            with CardHeader():
                CardTitle(f"Analysis: {file_name}")
            with CardContent():
                with Column(gap=3):
                    H3("Summary")
                    Text("{{ result.summary }}")
                    H3("Key Findings")
                    Text("{{ result.findings }}")
                    Badge("{{ result.confidence }}% confidence", variant="success")

app = PrefabApp(title="Document Analyzer", view=view, state={
    "file_name": "", "loading": False, "result": None
})
```

### 11.6 FastAPI Search App

A full-stack search application using Prefab with FastAPI:

```python
# server.py
from fastapi import FastAPI
from fastapi.responses import HTMLResponse, JSONResponse
from prefab_ui.app import PrefabApp
from prefab_ui.components import Column, Row, Card, CardContent, CardHeader, CardTitle
from prefab_ui.components import Input, Button, Text, Badge, Loader
from prefab_ui.components.control_flow import ForEach, If
from prefab_ui.actions import Fetch, SetState, ShowToast
from prefab_ui.rx import Rx, RESULT

app = FastAPI()

# Sample data
PRODUCTS = [
    {"id": 1, "name": "Widget Pro", "category": "Electronics", "price": 99.99, "stock": 45},
    {"id": 2, "name": "Gadget Plus", "category": "Electronics", "price": 149.99, "stock": 12},
    {"id": 3, "name": "Super Slider", "category": "Toys",       "price": 29.99, "stock": 200},
    {"id": 4, "name": "Mega Mixer",   "category": "Kitchen",    "price": 79.99, "stock": 8},
    {"id": 5, "name": "Ultra Widget", "category": "Electronics", "price": 199.99, "stock": 3},
]

@app.get("/api/products")
def search_products(q: str = "", category: str = ""):
    results = PRODUCTS
    if q:
        results = [p for p in results if q.lower() in p["name"].lower()]
    if category:
        results = [p for p in results if p["category"] == category]
    return results

@app.get("/", response_class=HTMLResponse)
def index():
    loading = Rx("loading").default(False)

    with Column(gap=4, css_class="max-w-3xl mx-auto p-4") as view:
        with Card():
            with CardHeader():
                CardTitle("Product Search")
            with CardContent():
                with Row(gap=2):
                    Input(
                        name="query",
                        placeholder="Search products...",
                        on_change=[
                            SetState("loading", True),
                            Fetch.get(
                                "/api/products",
                                params={"q": "{{ $event }}", "category": "{{ category }}"},
                                on_success=[
                                    SetState("products", RESULT),
                                    SetState("loading", False),
                                ],
                                on_error=[
                                    SetState("loading", False),
                                    ShowToast("Search failed", variant="error")
                                ]
                            )
                        ]
                    )
                    Button("Clear",
                        on_click=[
                            SetState("query", ""),
                            SetState("category", ""),
                            Fetch.get("/api/products", on_success=SetState("products", RESULT))
                        ]
                    )

        with If("{{ loading }}"):
            Loader(size="md", css_class="mx-auto")

        with ForEach("products"):
            with Card(css_class="mt-2"):
                with CardContent():
                    with Row(gap=4, css_class="justify-between items-center"):
                        with Column(gap=1):
                            Text("{{ $item.name }}", css_class="font-semibold")
                            Badge("{{ $item.category }}", variant="secondary")
                        with Column(gap=1, css_class="text-right"):
                            Text("${{ $item.price }}", css_class="text-lg font-bold")
                            Text("{{ $item.stock }} in stock",
                                 css_class="text-sm text-muted-foreground")

    initial_products = search_products()
    return HTMLResponse(
        PrefabApp(
            title="Product Search",
            view=view,
            state={"query": "", "category": "", "products": initial_products, "loading": False}
        ).html()
    )
```

Run with:

```bash
uvicorn server:app --reload
```

---

## 12. Best Practices for Agentic AI

### 1. Always Include Loading States

```python
# Bad — no feedback during async operations
Button("Run", on_click=CallTool("run_agent", arguments={"q": "{{ q }}"},
    on_success=SetState("result", RESULT)))

# Good — user sees feedback
Button("Run",
    disabled="{{ loading }}",
    on_click=[
        SetState("loading", True),
        CallTool("run_agent", arguments={"q": "{{ q }}"},
            on_success=[SetState("result", RESULT), SetState("loading", False)],
            on_error=[SetState("loading", False), ShowToast("{{ $error }}", variant="error")]
        )
    ]
)
```

### 2. Handle Errors in Every Async Action

Always provide an `on_error` callback for `CallTool` and `Fetch` — failed requests should clear loading state and inform the user.

### 3. Use `UpdateContext` for State Synchronization

When a user takes a significant action in the UI (selects a record, changes a filter), use `UpdateContext` to silently keep the model aware:

```python
Select(name="region",
    options=region_options,
    on_change=[
        SetState("region", "{{ $event }}"),
        UpdateContext("User selected region: {{ $event }}. Please answer questions in context of this region.")
    ]
)
```

### 4. Keep Binary Data Out of the Chat

Use `DropZone` and `OpenFilePicker` to handle file uploads client-side. Pass the base64 `$event.data` directly to a `CallTool` — never embed it in a `SendMessage`, as it will consume enormous token budget.

### 5. Design for Streaming

Prefab's DSL compiles to compact JSON. When an agent generates a UI, it can emit component declarations incrementally and the host will render each piece as it arrives.

### 6. Use `SetInterval` for Long-Running Agent Tasks

Agents often kick off background jobs (training, indexing, scraping). Use `SetInterval` to poll status and update the UI, stopping when the job completes or fails.

### 7. Prefer `CallTool` Over `Fetch` in MCP Contexts

In MCP hosts (Claude Desktop, ChatGPT), prefer `CallTool` to invoke server-side logic through the model's tool registry. Use `Fetch` for direct HTTP calls when you have a standalone REST backend.

### 8. Scope State Keys Carefully

Use descriptive, namespaced state keys to avoid collisions when multiple components share state:

```python
# Use namespaced keys for complex apps
Rx("search.query")
Rx("search.results")
Rx("upload.status")
Rx("upload.file_name")
```

### 9. Token-Efficient Component Generation

When prompting an LLM to generate Prefab UIs, instruct it to use the JSON protocol directly (more compact than Python) for streaming contexts:

```json
{
  "type": "Column",
  "gap": 4,
  "children": [
    {"type": "H3", "text": "{{ title }}"},
    {"type": "Text", "text": "{{ description }}"}
  ]
}
```

### 10. Export for Sharing

To share a Prefab UI as a standalone HTML file with no server:

```python
# Export to a self-contained HTML file
with open("dashboard.html", "w") as f:
    f.write(app.html())
```

---

## 13. Troubleshooting & FAQs

### `prefab serve` — common issues

| Issue | Fix |
|---|---|
| Port already in use | `prefab serve app.py --port 8081` |
| Changes not reflecting | Ensure `--reload` is set |
| Module not found | `pip install prefab-ui` in the right venv |

### Reactive bindings not updating

- Ensure the state key name in `Rx("key")` exactly matches the `name=` prop on the input.
- Verify the state key is initialized in `PrefabApp(state={...})`.
- Avoid hyphens in state key names — use underscores.

### `$event` is undefined in action arguments

- `$event` is only available inside action arguments for the component that fired the event.
- For cross-component communication, write to state first with `SetState`, then read the state key in subsequent actions.

### `CallTool` on_success not firing

- Ensure your MCP tool returns a valid JSON-serializable value.
- Non-2xx HTTP responses trigger `on_error`, not `on_success`.
- Check the MCP server logs for Python exceptions.

### `OpenFilePicker` not opening

- `OpenFilePicker` must be the first action in the `on_click` chain. Async actions (like `Fetch` or `CallTool`) consume the browser's user-activation window, preventing the picker from opening afterward.

### ForEach not rendering items

- Verify the state key exists and is initialized as an array: `state={"items": []}`.
- Ensure the key name in `ForEach("items")` matches the state key exactly.

### Operator precedence with `Rx`

```python
# Wrong — Python evaluates `a > b & c > d` as `a > (b & c) > d`
(level > 40 & level < 80)

# Correct — always wrap comparisons in parentheses
((level > 40) & (level < 80))
```

---

## Quick Reference Card

```python
# Install
pip install prefab-ui

# Serve
prefab serve app.py --reload

# Skeleton
from prefab_ui.app import PrefabApp
from prefab_ui.components import Column, Card, CardContent, Text, Button
from prefab_ui.actions import SetState, CallTool, ShowToast, Fetch
from prefab_ui.components.control_flow import ForEach, If
from prefab_ui.rx import Rx, STATE, RESULT

# State
count = Rx("count")                          # reactive ref
STATE.count                                   # shorthand
Rx("profile.name")                            # nested path

# Components
with Column(gap=4):                           # container
    Text("{{ count }}")                       # reactive text
    Button("Click", on_click=SetState("count", 0))

# Actions
SetState("key", value)
ToggleState("flag")
AppendState("list", item)
PopState("list", "{{ $index }}")
ShowToast("Message", variant="success")
CallTool("tool_name", arguments={...}, on_success=..., on_error=...)
Fetch.get("/url", params={...}, on_success=SetState("data", RESULT))
Fetch.post("/url", body={...}, on_success=..., on_error=...)
SetInterval(duration=3000, while_="{{ polling }}", on_tick=...)

# Control flow
with ForEach("items"):         # $item, $index
    Text("{{ $item.name }}")
with If("{{ count > 5 }}"):
    Text("High")
with Else():
    Text("Low")

# Run with FastAPI
uvicorn myapp:app --reload

# Export HTML
open("out.html", "w").write(app.html())
```

---

*Generated from the official Prefab documentation at [prefab.prefect.io/docs](https://prefab.prefect.io/docs).*
