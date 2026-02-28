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
| **Search Capability** | Basic <span style="color:rgb(216, 203, 251)">keyword matchin</span>                                           | Advanced <span style="color:rgb(216, 203, 251)">semantic search</span>                                                                                       |
| **Setup Complexity**  | None. It's <span style="color:rgb(216, 203, 251)">the default</span>                                          | Low. Requires an <span style="color:rgb(216, 203, 251)">Agent Engine instance in Vertex AI</span>                                                            |
| **Dependencies**      | None                                                                                                          | Google Cloud Project, Vertex AI API                                                                                                                          |
| **When to use it**    | Search across multiple sessions' chat histories for <span style="color:rgb(216, 203, 251)">prototyping</span> | Remember and <span style="color:rgb(216, 203, 251)">learn from past interactions</span>                                                                      |
### Tools for retrieving memories
- `PreloadMemory`: Always <span style="color:rgb(216, 203, 251)">retrieve memory at the beginning of each turn</span> (similar to a callback).
- `LoadMemory`: <span style="color:rgb(216, 203, 251)">Retrieve memory when the agent decides</span> it would be helpful.
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
