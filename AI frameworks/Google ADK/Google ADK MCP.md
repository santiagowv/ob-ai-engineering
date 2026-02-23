**Model Context Protocol** is an <span style="color:rgb(216, 203, 251)">open standard designed to connect AI applications to external systems</span> seamlessly, securely, and in a standarized way.
- MCP operates on a <span style="color:rgb(216, 203, 251)">client-server architecture</span>.
- An <span style="color:rgb(216, 203, 251)">MCP server exposes</span>:
	- Resources.
	- Templates.
	- Tools.
# Model context protocol componentes
- **Model:** any <span style="color:rgb(216, 203, 251)">LLM</span> (Gemini, GPT, Claude).
- **Context:** <span style="color:rgb(216, 203, 251)">external information and resources</span> the LLM requires.
- **Protocol:** standarized <span style="color:rgb(216, 203, 251)">set of rules</span>.
# MCP architecture
![[Pasted image 20260221182526.png|650]]
## MCP host
- AI app <span style="color:rgb(216, 203, 251)">interface that manages clients</span> (vscode, claude, ADK Agent).
- <span style="color:rgb(216, 203, 251)">Controls permissions</span> and approves AI actions.
## MCP server
- Provides <span style="color:rgb(216, 203, 251)">tools and data</span> to AI.
- Examples:
	- Google maps server.
	- Database server.
## MCP client
- <span style="color:rgb(216, 203, 251)">Connects</span> host to servers.
- <span style="color:rgb(216, 203, 251)">Sends requests</span> and gets results.
# MCP communication
![[Drawing 2026-02-21 18.25.59.excalidraw|500]]
# Local vs remote MCP server

| Feature       | Local MCP Server                                                                                                                               | Remote MCP Server                                                                                                                           |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Location      | Running on the <span style="color:rgb(216, 203, 251)">same machine as the Agent/Client</span>                                                  | Running on a <span style="color:rgb(216, 203, 251)">different machine or cloud infrastructure</span>                                        |
| Communication | Uses local channels like <span style="color:rgb(216, 203, 251)">Standard Input/Output (stdio)</span> or local sockets                          | Uses <span style="color:rgb(216, 203, 251)">network protocols</span> like HTTP (e.g., SSE, Streamable HTTP) over an IP address/URL          |
| Deployment    | Often used for <span style="color:rgb(216, 203, 251)">development, testing and debugging</span>                                                | Used for <span style="color:rgb(216, 203, 251)">production, team collaboration, and scalability</span> (e.g., deployed on Google Cloud Run) |
| Access        | Primarily accessible <span style="color:rgb(216, 203, 251)">only to the agent on the machine</span>                                            | Accessible to <span style="color:rgb(216, 203, 251)">multiple agents and clients</span> across a network                                    |
| Example       | An ADK agent on your laptop using an MCP server <span style="color:rgb(216, 203, 251)">running via a python command in a local terminal</span> | An ADK agent on your laptop connecting to an MCP server URL hosted on <span style="color:rgb(216, 203, 251)">Cloud Run</span>               |
# Create local MCP server
Create `mcp.json` file inside `.vscode` in root folder.
## Google maps
```json
{
  "servers": {
    "google-maps": {
      "command": "docker",
      "args": [
        "run",
        "-i",
        "--rm",
        "-e",
        "GOOGLE_MAPS_API_KEY",
        "mcp/google-maps"
      ],
      "env": {
        "GOOGLE_MAPS_API_KEY": ""
      }
    }
  }
}
```
## Google BigQuery
```json
{
  "servers": {
    "bigquery": {
        "command": "./mcp_toolbox_database/toolbox.exe",
        "args": ["--prebuilt","bigquery","--stdio"],
        "env": {
        "BIGQUERY_PROJECT": "main-analog-487920-c9"
      }
    }
  }
}
```
# MCP with ADK
<span style="color:rgb(216, 203, 251)">Integrate external MCP tools into the ADK</span> agents using the `MCPToolset` class.
## MCPToolset class
- The agent uses `MCPToolset` to <span style="color:rgb(216, 203, 251)">integrate an external MCP server</span>.
- <span style="color:rgb(216, 203, 251)">Runs an MCP server locally</span> using Node.js (via npx).
- The command runs <span style="color:rgb(216, 203, 251)">@modelcontextprotocol/server-google-maps</span>.
- <span style="color:rgb(216, 203, 251)">Passes the Google Maps API key to the server</span> through environment variables.
- Optionally, a `tool_filter` can be added to <span style="color:rgb(216, 203, 251)">expose only specific tools</span>.
```python
from google.adk.agents.llm_agent import Agent
from google.adk.tools.mcp_tool.mcp_toolset import MCPToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StdioConnectionParams
from mcp import StdioServerParameters
import os

google_maps_api_key = os.environ.get("GOOGLE_MAPS_API_KEY", "")

root_agent = Agent(
    model="gemini-3-flash-preview",
    name='root_agent',
    description='A helpful assistant for user questions related to locations,maps & directions',
    instruction='Answer user questions to the best of your knowledge ',
    tools=[
        MCPToolset(
            connection_params=StdioConnectionParams(
                server_params = StdioServerParameters(
                    command='npx',
                    args=["-y", "@modelcontextprotocol/server-google-maps"],
                    # Pass the API key as an environment variable to the npx process
                    # This is how the MCP server for Google Maps expects the key.
                    env={
                        "GOOGLE_MAPS_API_KEY": google_maps_api_key
                    }
                ),
            ),
            # You can filter for specific Maps tools if needed:
            # tool_filter=['get_directions', 'find_place_by_id']
        )
    ],
)
```
# MCP toolbox for database
![[Pasted image 20260222061428.png|400]]
Open source <span style="color:rgb(216, 203, 251)">MCP server for databases</span>.
- Handles complexities such as <span style="color:rgb(216, 203, 251)">connection pooling, authentication, and more</span>.
## Key features
- **Secure access:** <span style="color:rgb(216, 203, 251)">Safely query</span> and retrieve data from databases.
- **Pre-built tools:** <span style="color:rgb(216, 203, 251)">Production-ready functions</span> like `query_table`, `get_summary`, and analytical operations.
- **Connectors library:** Supports a <span style="color:rgb(216, 203, 251)">wide array of database types</span> (SQL, NoSQL, cloud databases).
- **Gen AI Integration:** Lets <span style="color:rgb(216, 203, 251)">AI agents analyze and interpret data efficiently</span>.
## How it works
1. MCP Toolbox runs as a <span style="color:rgb(216, 203, 251)">server exposing database tools</span>.
2. The <span style="color:rgb(216, 203, 251)">AI agent calls these tools</span> to fetch or analyze data.
3. The server <span style="color:rgb(216, 203, 251)">handles all database interactions securely</span>.
4. <span style="color:rgb(216, 203, 251)">Results are returned to the agent</span> for further processing.
## How to use it
1. <span style="color:rgb(216, 203, 251)">Install MCP</span> Toolbox for Databases.
2. <span style="color:rgb(216, 203, 251)">Configure MCP</span> Toolbox for Databases.
3. <span style="color:rgb(216, 203, 251)">Run MCP Server</span> testing the MCP Toolbox for Databases.
4. <span style="color:rgb(216, 203, 251)">Test the tools</span> via MCP Toolbox for Databases: `./toolbox.exe --tools-file "tools.yaml"`
5. <span style="color:rgb(216, 203, 251)">Test the tools via MCP inspector</span> (interactive developer tool for testing and debugging MCP servers).
6. <span style="color:rgb(216, 203, 251)">Write ADK Agent</span> and connect it with MCP.
7. <span style="color:rgb(216, 203, 251)">Start</span> agent.
8. <span style="color:rgb(216, 203, 251)">Test</span> agent.