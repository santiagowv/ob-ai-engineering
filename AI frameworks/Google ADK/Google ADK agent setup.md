# Run agent UI
```powershell
adk web
```
# Run agent in CLI
```powershell
adk run <module_name>
```
# Create Agent instance
```python
root_agent = Agent(
    name = "my_first_agent",
    model = "gemini-3-flash-preview",
    description = "An example agent that will answer user queries related to Google Cloud",
    instruction = """
        You are an AI assistant that helps users with Google Cloud related queries based on Google search results.
    """
)
```
