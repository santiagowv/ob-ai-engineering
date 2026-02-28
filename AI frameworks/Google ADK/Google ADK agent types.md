![[Pasted image 20260220065031.png]]
# Workflow agents
## Single agent
**Pros:**
- <span style="color:rgb(216, 203, 251)">Simple</span> to implement.
- Easier to <span style="color:rgb(216, 203, 251)">debug</span>.
- Low <span style="color:rgb(216, 203, 251)">latency</span>.
**Cons:**
- <span style="color:rgb(216, 203, 251)">Large system prompt</span> can become unwieldy.
- <span style="color:rgb(216, 203, 251)">Harder to re-use</span> individual components.
- Single point of <span style="color:rgb(216, 203, 251)">failure</span>.
![[Pasted image 20260227191405.png|600]]

## Sequential agents
Each agent uses the <span style="color:rgb(216, 203, 251)">input data provided by the prior agent</span>.
![[Pasted image 20260220065315.png|650]]

```python
from google.adk.agents import SequentialAgent, LlmAgent

# --- Combine into Sequential LLM Pipeline ---
interview_prep_pipeline = SequentialAgent(
    name="InterviewPrepPipeline",
    sub_agents=[open_positions_agent, interview_qa_agent, interview_tips_agent],
    description="Suggests open jobs, interview Q&A, and preparation tips"
)


# root agent
root_agent = interview_prep_pipeline
```
## Parallel agent
Agent execution <span style="color:rgb(216, 203, 251)">doesn't depend on other agents</span>.
![[Pasted image 20260220065652.png|550]]

```python
from google.adk.agents import ParallelAgent, LlmAgent

# Parallel Content Creator Agent
content_creator_agent =  ParallelAgent(
    name="ContentCreatorAgent",
    sub_agents=[blog_agent, youtube_agent, instagram_agent],
    description="Generates blog ideas, YouTube video content, and Instagram reel ideas in parallel."
)

# Root agent (for ADK compatibility)
root_agent = content_creator_agent
```
## Loop agent
It <span style="color:rgb(216, 203, 251)">refines the agent's response</span> until there are no errors.
![[Pasted image 20260220065513.png|600]]
```python
# STEP 2: Refinement Loop Agent
refinement_loop = LoopAgent(
    name="RefinementLoop",
    # Agent order is crucial: Critique first, then Refine/Exit
    sub_agents=[
        critic_agent_in_loop,
        refiner_agent_in_loop,
    ],
    max_iterations=5 # Limit loops
)


# STEP 3: Overall Sequential Pipeline
# For ADK tools compatibility, the root agent must be named `root_agent`
root_agent = SequentialAgent(
    name="IterativeWritingPipeline",
    sub_agents=[
        initial_writer_agent, # Run first to create initial doc
        refinement_loop       # Then run the critique/refine loop
    ],
    description="Iteratively generates and refines a cloud architecture until stable."
)
```