![[Drawing 2026-02-19 16.15.17.excalidraw|700]]
Special <span style="color:rgb(216, 203, 251)">ability given to an AI agent</span>.
- It lets the agent <span style="color:rgb(216, 203, 251)">do actions</span> and <span style="color:rgb(216, 203, 251)">interact with the world</span>.
- Tool types:
	- Small code unit.
	- Class method.
	- Another agent.
- Each tool is <span style="color:rgb(216, 203, 251)">built to do one clear task</span>.
- Often involves <span style="color:rgb(216, 203, 251)">working with external systems or data</span>.
	- Querying databases.
	- Making API requests.
	- Searching the web.
	- Executing code snippets.
# Built-in tools
<span style="color:rgb(216, 203, 251)">Ready-to-use tools provided by the framework</span> for common tasks. Examples:
- Google Search.
- Code Execution.
- Retrieval-Augmented Generation (RAG).
- BigQuery.
```python
from google.adk.tools import google_search

root_agent = Agent(
    name = "my_first_agent",
    model = "gemini-3-flash-preview",
    description = "An example agent that will answer user queries related to Google Cloud",
    instruction = """
        First ask user a name and start converstaion by greeting based on users Greet.
        You are an AI assistant that helps users with Google Cloud related queries based on Google search results.
    """,
    tools = [google_search]

)
```
# Function tools
Tools tailored to out specific application's needs.
- **Functions/Methods:** Define standard <span style="color:rgb(216, 203, 251)">synchronous functions or methods</span> in the code.
- **Agents-as-tools:** Use another, potentially specialized, <span style="color:rgb(216, 203, 251)">agent as a tool for a parent agent</span>.
- **Long running function tools:** Support for <span style="color:rgb(216, 203, 251)">tools that perform asynchronous operations</span> or take significant time to complete.
```python
from google.adk.agents import Agent
from google.adk.tools import google_search

def morning_greet(name: str) -> str:
    return f"Good morning, {name}! How can I assist you today? My Mood is amazing"

def evening_greet(name: str) -> str:
    return f"Good evening, {name}! My Mood is a bit low. How can i Assit you today?"

root_agent = Agent(
    name = "my_first_agent",
    model = "gemini-3-flash-preview",
    description = "An example agent that will answer user queries related to Google Cloud",
    instruction = """
        First ask user a name and start converstaion by greeting based on users Greet.
        If user says Good Morning, use morning_greet tool to greet user.
        If user says Good Evening, use evening_greet tool to greet user.
        You are an AI assistant that helps users with Google Cloud related queries based on Google search results.
    """,
    tools = [evening_greet, morning_greet]
)
```
# Third-Party tools
Integrate tools seamlessly from <span style="color:rgb(216, 203, 251)">popular external libraries</span>.
- LangChain tools.
- CrewAI Tools.