---
docs: https://google.github.io/adk-docs/deploy/
---
# Agent engine
![[Pasted image 20260220195537.png|550]]
A fully <span style="color:rgb(216, 203, 251)">managed runtime within Google Cloud's Vertex AI platform</span>. Designed to help developers:
- Deploy.
- Manage.
- Scale AI agents in production environments.
```bash
PROJECT_ID=my-project-id
LOCATION_ID=us-central1

adk deploy agent_engine \
        --project=$PROJECT_ID \
        --region=$LOCATION_ID \
        --display_name="My First Agent" \
        multi_tool_agent
```
# Cloud run
```bash
adk deploy cloud_run \
--project=$GOOGLE_CLOUD_PROJECT \
--region=$GOOGLE_CLOUD_LOCATION \
--service_name=$SERVICE_NAME \
--app_name=$APP_NAME \
--with_ui \
$AGENT_PATH
```