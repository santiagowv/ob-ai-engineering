Powerful mechanism to <span style="color:rgb(216, 203, 251)">hook into an agent's execution process</span>. 
- Callbacks are special functions that let us o<span style="color:rgb(216, 203, 251)">bserve, customize, or control</span> how the agent behaves without changing the ADK code.
![[Pasted image 20260221103430.png|594]]
# Why they are important
- They act like <span style="color:rgb(216, 203, 251)">checkpoints during the agent's workflow</span>.
- <span style="color:rgb(216, 203, 251)">Monitor what's happening</span> behind the scenes.
- Help <span style="color:rgb(216, 203, 251)">tweak or stop the process</span> when need.
- <span style="color:rgb(216, 203, 251)">Extend agent behavior</span> safely and flexibly.
# How callbacks work
- We simply define our own <span style="color:rgb(216, 203, 251)">functions and attach them to an agent</span>.
- ADK will <span style="color:rgb(216, 203, 251)">trigger them at key stages of execution</span>.
# Types of callbacks

| Callback scope  | Callback type | When it runs                                                 | Purpose                                                                          | Return behavior                                                                           |
| --------------- | ------------- | ------------------------------------------------------------ | -------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| Agent lifecycle | Before Agent  | Before agent's main logic (_run_async_impl / _run_live_impl) | - Validate user access.<br>- Validate session state.<br>- Setup resources        | - None -> continue normally; returning Content -> skip agent execution                    |
| Agent lifecycle | After agent   | After agent completes main logic                             | - Cleanup.<br>- Log completition.<br>- Modify or augment output.                 | - None -> keep original output; returning Content -> replace output                       |
| Model lifecycle | Before model  | Before sending a request to the LLM                          | - Modify prompt.<br>- Inject few-shot exmaples.<br>- Apply filters.<br>- Caching | - None -> call LLM normally; returnning LlmResponse -> skip LLM and use provided response |
| Model lifecycle | After model   | After receiving LLM response                                 | - Inspect.<br>- Format.<br>- Censor.<br>- Parse output.                          | - None -> use original LLM response; returning LlmResponse -> replace output              |
| Tool lifecycle  | Before tool   | Before tool's run_async executes                             | - Modify tool args.<br>- Authorization.<br>- Log usage.<br>- Caching.            | - None -> execute tool normally; returning dict -> skip tool and use returned dict        |
| Tool lifecycle  | After tool    | After tool completes successfully                            | - Post-process results.                                                          | - None -> keep original tool result; returning dict -> replace output                     |
# Agent callbacks
## Imports
```python
from google.adk.agents.llm_agent import Agent
from google.adk.tools import ToolContext
from typing import Dict, Any, Optional
from google.adk.agents.callback_context import CallbackContext
from google.genai import types
from google.adk.tools import google_search
```
## Before agent callback
```python
def check_access(callback_context:CallbackContext) -> Optional[types.Content]:
    """
    Checks if the user_id is in an allowed list.
    If yes -> allow agent/LLM execution (return None)
    If no -> skip LLM and return a message
    """

    session = callback_context._invocation_context.session
    user_id = session.user_id
    allowed_users = ["user_123", "user_abc", "user"]

    print(f"\n[Callback] Checking user: {user_id}")

    if user_id in allowed_users:
        print(f"[Callback] User allowed: {user_id} - proceeding with agent/LLM call.")
        return None # Continue normal
    else:
        print(f"[Callback] User not allowed: {user_id} - skipping LLM call.")
        return types.Content(
            parts=[types.Part(text=f"Access denied for user: {user_id}. LLM call skipped.")],
            role="model" # Optional: role can be 'system' or 'model'
        )
```
## After agent callback
```python
def log_completion(callback_context: CallbackContext) -> Optional[types.Content]:
    current_state = callback_context.state.to_dict()
    print(f"\n[Callback] state: {current_state}")
    print(f"\n[Callback] agent execution completed. Logging results.")
```
## Agent setup
```python
root_agent = Agent(
    model = "gemini-3-flash-preview",
    name = "root_agent",
    description = "A helpful assistant for answering user questions.",
    instruction = "Answer user questions",
    before_agent_callback = check_access,
    after_agent_callback = log_completion,
    tools = [google_search],
    output_key = "result"
)
```
# Model callbacks
## Imports
```python
from google.adk import Agent
from google.adk.tools import ToolContext
from typing import Dict, Any, Optional
from google.adk.agents.callback_context import CallbackContext
from google.genai import types
from google.adk.models import llm_request, llm_response
from google.adk.tools import google_search
```
## Before model callback
```python
def before_model(callback_context: CallbackContext, llm_request: llm_request.LlmRequest) -> Optional[llm_response.LlmResponse]:
    """
    Appends instructions to the last user message before sending to LLM.
    """

    agent_name = callback_context.agent_name
    
    # Find the last user message
    last_user_message = ""
    if llm_request.contents and len(llm_request.contents) > 0:
        for content in reversed(llm_request.contents):
            if content.role == "user" and content.parts and len(content.parts) > 0:
                if hasattr(content.parts[0], "text") and content.parts[0].text:
                    last_user_message = content.parts[0].text
                    break

    print("===MODEL REQUEST STARTED===")
    print(f"Agent: {agent_name}")

    if last_user_message:
        print(f"User message: {last_user_message[:100]}...")
        # Append instructions to the last user message
        content_part = content.parts[0]
        content_part.text += "\nPlease respond politely and keep the answer under 50 words."
        print(f"Modified user message: {content_part.text[:150]}...")
        print("===MODEL REQUEST COMPLETED===")
```
## After model callback
```python
def after_model(callback_context: CallbackContext, llm_response: llm_response.LlmResponse) -> llm_response.LlmResponse:
    """
    Converts all text in the LLM response to uppercase.
    """
    if not llm_response or not llm_response.content or not llm_response.content.parts:
        return None
    for part in llm_response.content.parts:
        if hasattr(part, "text") and part.text:
            part.text = part.text.upper() # convert to uppercase
    return llm_response
```
## Agent setup
```python
root_agent = Agent(
    model="gemini-3-flash-preview",
    name="root_agent",
    description="A helpful assistant for answering user questions.",
    instruction="Answer user questions",
    before_model_callback=before_model,
    after_model_callback=after_model,
    output_key="result",
    tools=[google_search]

)
```
# Tool callbacks
## Imports
```python
from typing import Any, Dict, Optional
from cohere import ToolContent
from google.adk.agents.llm_agent import Agent
from langchain_community.tools import BaseTool
from requests import session
from toolbox_core import ToolboxSyncClient

# Connect to MCP tool server
toolbox = ToolboxSyncClient("http://127.0.0.1:5000")

# Load BigQuery HR toolset
tools = toolbox.load_toolset("gcp_bq_employees")
```
## Before tool callback
```python
def before_tool_callback(
        tool:BaseTool, args: Dict[str, Any], tool_context: ToolContent
) -> Optional[Dict]:
    """
    Converts the 'Country' argument to Camel Case before the tool call.
    """

    tool_name = tool.name
    print(f"[Callback] Before tool call for '{tool_name}'")
    print(f"[Callback] Original args: {args}")

    # Only modify args for this specific tool
    if tool_name == "search_employees_by_country":
        country = args.get("Country", "")
        if country:
            # Convert to Camel Case
            args["Country"] = " ".join(word.capitalize() for word in country.split())
            print(f"[Callback] Converted Country to Camel Case: {args['Country']}")

    print("[Callback] Proceeding with normal too call")
    return None
```
## After tool callback
```python
def after_tool_callback(tool, args, tool_context, tool_response: str) -> list:
    try:
        user_id = session.user_id.strip()
    except AttributeError:
        user_id = "user"
    USER_DEPARTMENT_MAP = {
        "vishal": "Finance",
        "alice": "Marketing",
        "user": "IT",
    }
    
    department = USER_DEPARTMENT_MAP.get(user_id)

    if not department:
        return [{"Department": None, "message": "You do not have access to department-specific data."}]
```
## Agent setup
```python
root_agent = Agent(
    model='gemini-2.5-flash',
    name='root_agent',
    description='A helpful assistant for user questions.',
    instruction=(
        "You are an HR Data Analyst.\n"
        "Use the tools to answer questions about employees, countries, and workforce stats.\n\n"
        "→ Use `search_users_by_id` for EmployeeID.\n"
        "→ Use `search_users_by_country` for country-based queries.\n"
        "Respond clearly and concisely — no SQL, no tool names, just insights."
    ),

    tools=tools,
    before_tool_callback=before_tool_callback,
    after_tool_callback=after_tool_callback
)
```