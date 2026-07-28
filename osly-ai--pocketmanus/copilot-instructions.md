## pocketmanus

> An agent is a powerful design pattern in which a node can take dynamic actions based on the context it receives. To express an agent, create a Node with a prompt template like:


# Agent

An agent is a powerful design pattern in which a node can take dynamic actions based on the context it receives. To express an agent, create a Node with a prompt template like:

```python
"""
Here is the context: {context}
Now, choose your next actions:
1. Action name, what it is for, and what the parameters are
2. ...

Now decide your action by return:
```yaml
thinking: |
    Based on the context:
action: action_name
```"""
```

Then, connect this agent node with [branching](../core_abstraction/flow.md) to other action nodes.

> The core of building **performant** and **reliable** agents boils down to:
>
> 1. **Context Management:** Provide *clear, relevant context* so agents can understand the problem. For example, rather than dumping an entire chat history or entire files, use a [Workflow](./workflow.md) that filters out and includes only the most relevant information.
>
> 2. **Action Space:** Define *a well-structured, unambiguous, and easy-to-use* set of actions. For instance, avoid creating overlapping actions like `read_databases` and `read_csvs`. Instead, unify data sources (e.g., move CSVs into a database) and design a single action. The action can be parameterized (e.g., with a search string) or be programmable (e.g., through SQL queries).
{: .best-practice }



<div align="center">
  <img src="https://github.com/The-Pocket-World/Pocketflow-Framework-Py/raw/main/assets/agent.png?raw=true" width="250"/>
</div>


### Example: Search Agent

This agent:
1. Decides whether to search or answer
2. If searches, loops back to decide if more search needed
3. Answers when enough context gathered

```python
class DecideAction(Node):
    def prep(self, shared):
        context = shared.get("context", "No previous search")
        query = shared["query"]
        return query, context
        
    def exec(self, inputs):
        query, context = inputs
        prompt = f"""
Given input: {query}
Previous search results: {context}
Should I: 1) Search web for more info 2) Answer with current knowledge
Output in yaml:
```yaml
action: search/answer
reason: why this action
search_term: search phrase if action is search
```"""
        resp = call_llm(prompt)
        yaml_str = resp.split("```yaml")[1].split("```")[0].strip()
        result = yaml.safe_load(yaml_str)
        
        assert isinstance(result, dict)
        assert "action" in result
        assert "reason" in result
        assert result["action"] in ["search", "answer"]
        if result["action"] == "search":
            assert "search_term" in result
        
        return result

    def post(self, shared, prep_res, exec_res):
        if exec_res["action"] == "search":
            shared["search_term"] = exec_res["search_term"]
        return exec_res["action"]

class SearchWeb(Node):
    def prep(self, shared):
        return shared["search_term"]
        
    def exec(self, search_term):
        return search_web(search_term)
    
    def post(self, shared, prep_res, exec_res):
        prev_searches = shared.get("context", [])
        shared["context"] = prev_searches + [
            {"term": shared["search_term"], "result": exec_res}
        ]
        return "decide"
        
class DirectAnswer(Node):
    def prep(self, shared):
        return shared["query"], shared.get("context", "")
        
    def exec(self, inputs):
        query, context = inputs
        return call_llm(f"Context: {context}\nAnswer: {query}")

    def post(self, shared, prep_res, exec_res):
       print(f"Answer: {exec_res}")
       shared["answer"] = exec_res

# Connect nodes
decide = DecideAction()
search = SearchWeb()
answer = DirectAnswer()

decide - "search" >> search
decide - "answer" >> answer
search - "decide" >> decide  # Loop back

flow = Flow(start=decide)
flow.run({"query": "Who won the Nobel Prize in Physics 2024?"})
```

---
> Source: [Osly-AI/PocketManus](https://github.com/Osly-AI/PocketManus) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
