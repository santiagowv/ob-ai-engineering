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
        "GOOGLE_MAPS_API_KEY": "AIzaSyDETIl3CtIXFU6xZQifX8Mi_aOz_vKiP0o"
      }
    }
  }
}
```