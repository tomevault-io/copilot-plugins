## agent-protocol

> Name | Type | Description | Notes

# Agent


## Properties

Name | Type | Description | Notes
------------ | ------------- | ------------- | -------------
**agent_id** | **str** | The ID of the agent. | 
**name** | **str** | The name of the agent | 
**description** | **str** | The description of the agent. | [optional] 
**metadata** | **object** | The agent metadata. | [optional] 
**capabilities** | [**AgentCapabilities**](AgentCapabilities.md) |  | 

## Example

```python
from ap_client.models.agent import Agent

# TODO update the JSON string below
json = "{}"
# create an instance of Agent from a JSON string
agent_instance = Agent.from_json(json)
# print the JSON string representation of the object
print(Agent.to_json())

# convert the object into a dict
agent_dict = agent_instance.to_dict()
# create an instance of Agent from a dict
agent_from_dict = Agent.from_dict(agent_dict)
```
[[Back to Model list]](../README.md#documentation-for-models) [[Back to API list]](../README.md#documentation-for-api-endpoints) [[Back to README]](../README.md)

---
> Source: [langchain-ai/agent-protocol](https://github.com/langchain-ai/agent-protocol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
