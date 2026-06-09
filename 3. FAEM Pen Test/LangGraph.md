Think of your agent as a state machine with two nodes:
```
START
  ↓
[llm_node] ←──────────────┐
  ↓                        │
  ├─ made tool calls? ──→ [tool_node]
  │                        │
  └─ no tool calls? ──→  END
```

---
## 3 Concepts:
### 1. State:
What gets passed between the nodes.
```python
from langgraph.graph import MessagesState 
# MessagesState is just {"messages": list[BaseMessage]} 
# LangGraph handles appending automatically
```
### 2. Nodes:
Functions that take in current state and return the updated state. In a way, that is what happens at a node.
```python
def llm_node(state):
    response = llm.invoke(state["messages"])
    return {"messages": [response]}  # LangGraph appends this

def tool_node(state):
    # executes tool calls from last message
    # ToolNode from langchain handles this automatically
```
### 3. Edges:
Decide where to go next.
```python
from langgraph.prebuilt import ToolNode, tools_condition
```

---
## Tool Executer:
LangGraph provides a prebuilt tool executor:
```python
from langgraph.prebuilt import ToolNode, tools_condition
```
- `ToolNode` handles tool execution automatically.
- `tools_condition` checks if the LLM made tool calls and routes accordingly.

---

## Binding Shell Tools to the LLM:
```python
llm_with_tools = llm.bind_tools([shell_tool])
```

This tells the LLM formally "these tools exist and here's their schema." The LLM can then decide to call them by name, and LangChain handles the tool call formatting automatically.

So your `__init__` sequence is:
```python
# 1. initialize LLM
self.llm = ChatOllama(model=..., temperature=...)

# 2. initialize tool
self.shell_tool = ShellTool()

# 3. bind tool to LLM
self.llm_with_tools = self.llm.bind_tools([self.shell_tool])

# 4. build graph using self.llm_with_tools
```

The graph uses `llm_with_tools`, not the raw `llm`.

---
## Graph:
You have all four pieces. For the graph, use this structure:
```python
from langgraph.graph import StateGraph, MessagesState, END
from langgraph.prebuilt import ToolNode, tools_condition

graph = StateGraph(MessagesState)
graph.add_node("llm", ...)
graph.add_node("tools", ToolNode([self.shell_tool]))
graph.set_entry_point("llm")
graph.add_conditional_edges("llm", tools_condition)
graph.add_edge("tools", "llm")
self.graph = graph.compile()
```

---
