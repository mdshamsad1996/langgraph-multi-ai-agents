#### Building Multi AI Agents Chatbots With External Tools With LangGraph

#### create ```wiki_tool``` object of ```WikipediaQueryRun``` 
-   create ```WikipediaAPIWrapper``` object with ```top_k_results=1``` and ```doc_content_chars_max=300```
-   create object of ```WikipediaQueryRun``` to run against ```WikipediaAPIWrapper```
#### creating ```tools``` list using```tools = [wiki_tool]```
### Bind the 'wiki_tool' to the language model (LLM) 
```
llm_with_tools = llm.bind_tools(tools=tools)
```

####  Defining the ```State``` class

```python
class State(TypedDict):
  # Messages have the type "list". The `add_messages` function
  # in the annotation defines how this state key should be updated
  # (in this case, it appends message to list, rather than overwriting them)
  messages: Annotated[list, add_messages]

```
-   This class defines the structure of the State that will be passed throughout the graph.
-   ```messages:``` A list of messages stored in the state, with the function add_messages that appends new messages to the existing list. This ensures that conversation history is maintained and not overwritten.

#### Creating the Graph

```python
graph_builder = StateGraph(State)
```
-   A new ```StateGraph``` object is created, using the defined State class to manage the graph’s state. This state will now include the messages key to store the conversation history.

#### Defining the chatbot function:
```python
def chatbot(state:State):
  return {"messages": [llm_with_tools.invoke(state['messages'])]}

```
-   ```llm_with_tools.invoke(state['messages'])```: This line calls the invoke method on the llm_with_tools instance (the language model that has tools bound to it). It passes the current messages stored in the state.
-   ```Input:``` The ```chatbot``` function takes the current state as input. This state contains the history of messages.
-  ``` Functionality:``` It sends the conversation history ```(state['messages'])``` to the language model (llm) using the invoke method.
-   ```Output:``` The updated state is returned, where the chatbot adds new messages to the messages

#### Adding the Edges and node
-   Adding a chatbot node ```graph_builder.add_node("chatbot", chatbot)```

    ```python
    graph_builder.add_edge(START, "chatbot")
    graph_builder.add_edge("chatbot", END)
    ```
-   ```First Edge:``` This defines a direct flow from START to the chatbot function, which means the chatbot will be the first function called when the workflow starts.```graph_builder.add_edge(START, "chatbot")```
-   creating a tool node ```tool_node = ToolNode(tools=tools)```
-   Add ```tool_node``` node to the graph with the ```tools``` identifier . ```graph_builder.add_node("tools", tool_node)```
-   Adding Conditional Edges: ```graph_builder.add_conditional_edges("chatbot",tools_condition)```
    -   This line establishes conditional edges from the ```chatbot``` node based on the ```tools_condition```
-   Connecting Tool Node to Chatbot: ```graph_builder.add_edge("tools", "chatbot")```
-   Ending the Graph ```graph_builder.add_edge("chatbot", END)```
