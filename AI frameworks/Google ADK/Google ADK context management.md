It's <span style="color:rgb(216, 203, 251)">how an agent remembers, stores, and uses information while interacting with a user</span> or other agents.
![[Drawing 2026-02-22 09.26.45.excalidraw|700]]
# Session 
## Short term memory
<span style="color:rgb(216, 203, 251)">Tracks content for a single conversation</span> between a user and an agent.
- Ensures the agent <span style="color:rgb(216, 203, 251)">remembers previous interactions</span>, maintaining continuity.
- By default <span style="color:rgb(216, 203, 251)">agents running locally will use persistent sessions</span> with a SQLite database.

| Service                | Storage/Persistance         | Best use case       |
| ---------------------- | --------------------------- | ------------------- |
| InMemorySessionService | In-memory (lost on restart) | Testing/Development |
| VertexAiSessionService | Google Cloud Vertex AI      | Scalable production |
| DatabaseSessionService | Relational DB               | Persistent storage  |
## Long term memory
Persistent storage that the agent can save and <span style="color:rgb(216, 203, 251)">retrieve across sessions</span> to <span style="color:rgb(216, 203, 251)">remember users</span>, <span style="color:rgb(216, 203, 251)">preferences</span> and <span style="color:rgb(216, 203, 251)">past interactions</span> over time.
- **Generating memories:** Send the session's events to the Memory Bank, which <span style="color:rgb(216, 203, 251)">processes and stores the information as memories</span>.
- **Retrieving memories:** The agent issues queries against the Memory Bank to <span style="color:rgb(216, 203, 251)">retrieve relevant memories from past conversations</span>.

| **Feature**           | **InMemoryMemoryService**                                                                                     | **VertexAiMemoryBankService**                                                                                                                                |
| --------------------- | ------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Persistence**       | None (data is lost on restart)                                                                                | Yes (Manged by Vertex AI)                                                                                                                                    |
| **Primary Use Case**  | <span style="color:rgb(216, 203, 251)">Prototyping</span>, local development, and simple testing              | Building <span style="color:rgb(216, 203, 251)">meaningful, evolving memories</span> from user conversations                                                 |
| **Memory Extraction** | Stores <span style="color:rgb(216, 203, 251)">full conversation</span>                                        | <span style="color:rgb(216, 203, 251)">Extracts meaningful information from conversations</span> and consolidates it with existing memories (powered by LLM) |
| **Search Capability** | Basic <span style="color:rgb(216, 203, 251)">keyword matching</span>                                          | Advanced <span style="color:rgb(216, 203, 251)">semantic search</span>                                                                                       |
| **Setup Complexity**  | None. It's <span style="color:rgb(216, 203, 251)">the default</span>                                          | Low. Requires an <span style="color:rgb(216, 203, 251)">Agent Engine instance in Vertex AI</span>                                                            |
| **Dependencies**      | None                                                                                                          | Google Cloud Project, Vertex AI API                                                                                                                          |
| **When to use it**    | Search across multiple sessions' chat histories for <span style="color:rgb(216, 203, 251)">prototyping</span> | Remember and <span style="color:rgb(216, 203, 251)">learn from past interactions</span>                                                                      |
### Tools for retrieving memories
- `PreloadMemory`: Always <span style="color:rgb(216, 203, 251)">retrieve memory at the beginning of each turn</span> (similar to a callback).
- `LoadMemory`: <span style="color:rgb(216, 203, 251)">Retrieve memory when the agent decides</span> it would be helpful.
### InMemoryMemoryService
`inMemoryMemoryService` <span style="color:rgb(216, 203, 251)">only saves memories while the app is executing</span> NOT persistent.
1. Create a `preload_memory_tool` that <span style="color:rgb(216, 203, 251)">retrieves the agent's memory to provide a response</span>.
```python
from google.adk.tools import preload_memory_tool

preload_memory_tool = preload_memory_tool.PreloadMemoryTool()
```
2. Create an after agent <span style="color:rgb(216, 203, 251)">callback to save the session to memory</span>.
```python
async def auto_save_session_to_memory_callback(callback_context):
    await callback_context._invocation_context.memory_service.add_session_to_memory(callback_context._invocation_context.session)
    print("Session saved to memory.")
```
3. Create the agent.
```python
root_agent = Agent(
    model="gemini-2.5-flash",
    name="root_agent",
    description="A helpful assistant for user questions.",
    instruction="Answer user questions to the best of your knowledge.",
    tools=[preload_memory_tool],
    after_agent_callback=auto_save_session_to_memory_callback
)
```
### VertexAI Memory bank
Service that helps AI agents <span style="color:rgb(216, 203, 251)">remember information about users across different conversations</span>. It requires an agent engine instance.
- <span style="color:rgb(216, 203, 251)">Stores only important facts</span> about each user.
- Remembers <span style="color:rgb(216, 203, 251)">preferences and past interactions</span>.
- <span style="color:rgb(216, 203, 251)">Recalls relevant information</span> when needed.
- Helps the agent give <span style="color:rgb(216, 203, 251)">personalized responses</span>.
![[Pasted image 20260303205930.png|639]]
### Security risks
To <span style="color:rgb(216, 203, 251)">mitigate the risk of memory poisoning</span>, we can do the following:
- **Model armor:** <span style="color:rgb(216, 203, 251)">Inspect prompts being sent to Memory Bank</span> or from the agent.
- **Adversarial testing:** <span style="color:rgb(216, 203, 251)">Test LLM application for prompt injection</span> vulnerabilities by simulating attacks.
- **Sandbox execution:** Agent interactions with external or critical systems should be performed in an <span style="color:rgb(216, 203, 251)">sandboxed environment with strict access control</span>.
# State
In each session (conversation thread), the <span style="color:rgb(216, 203, 251)">state works like the agent's personal notepad</span> for that specific chat.
- `session.events` keeps the <span style="color:rgb(216, 203, 251)">full history of what happened</span>.
- `session.state` is <span style="color:rgb(216, 203, 251)">where the agent saves and updates the important working details it needs</span> during the conversation.
## Use cases
- In multi-agent systems, state <span style="color:rgb(216, 203, 251)">helps sub-agents share information easily</span>.
	- Sub-agent A <span style="color:rgb(216, 203, 251)">can store key data instate</span> for Sub-agent B to use later.
- State can <span style="color:rgb(216, 203, 251)">hold user metadata</span> like user ID, role, and department.
- Agents can <span style="color:rgb(216, 203, 251)">apply conditional logic</span> based on user details.
	- Only <span style="color:rgb(216, 203, 251)">Admin users can access certain agent features</span>.
	- <span style="color:rgb(216, 203, 251)">Non-admin users receive an unauthorized</span> access message.

| Prefix Type                   | Scope (where it applies)                                                                                               | Persistance                                                    | Common use cases                                                    | Example key                           |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------- | ------------------------------------------------------------------- | ------------------------------------- |
| **No Prefix** (Session State) | Only within the <span style="color:rgb(216, 203, 251)">current session</span> (`session.id`)                           | Persists only if `SessionService` is persistent (DB/Vertex AI) | Task progress, conversation step tracking, temporary session flags. | `current_intent = "book_flight"`      |
| `user:` (User State)          | <span style="color:rgb(216, 203, 251)">Shared across all sessions for the same user</span> (`user.id`)                 | Persistent with DB/VertexAI (lost in InMemory restart)         | User preferences, profile details, personalization.                 | `user:preferred_language = "fr"`      |
| `app:` (App State)            | <span style="color:rgb(216, 203, 251)">Shared across all users</span> and sessions in the same app (`app_name`)        | Persistent with DB/VertexAI (lost InMemory restart)            | Global configuration, shared templates, app_wide settings           | `app:global_discount_code = "SAVE10"` |
| `temp:` (Invocation State)    | <span style="color:rgb(216, 203, 251)">Only during the current agent invocation</span> (single request-response cycle) | Not persistent, discarded after completion                     | Intermediate tools outputs, temporary calculations, internal flags. | `temp:already_validated = True`       |
