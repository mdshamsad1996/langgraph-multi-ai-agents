#### Build a stateful chatbot workflow using ```StateGraph```

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
def chatbot(state: State):
  return {"messages": llm.invoke(state['messages'])}

```
-   ```Input:``` The ```chatbot``` function takes the current state as input. This state contains the history of messages.
-  ``` Functionality:``` It sends the conversation history ```(state['messages'])``` to the language model (llm) using the invoke method.
-   ```Output:``` The updated state is returned, where the chatbot adds new messages to the messages

#### Adding the Edges
```python
graph_builder.add_edge(START, "chatbot")
graph_builder.add_edge("chatbot", END)
```
-   ```First Edge:``` This defines a direct flow from START to the chatbot function, which means the chatbot will be the first function called when the workflow starts.
-   ```Second Edge:``` After the chatbot function finishes, the flow proceeds to END, marking the conclusion of the workflow
