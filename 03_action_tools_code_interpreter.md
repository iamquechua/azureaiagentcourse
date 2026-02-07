# Code Interpreter with Azure AI Agents

---

## Code Interpreter — Foundry Portal

### Step 1: Create an Agent

See the [Create an Agent](./azure-ai-foundry-getting-started.md#3-create-an-agent) section, with these settings:

- **Name:** `CodeInterpreterAgent`
- **Instructions:** `You are a helpful agent that performs data analytics and visualization.`

### Step 2: Add the Code Interpreter Tool

1. Click on **+ Add** in the Actions tool section
2. Click on **"Code interpreter"**
3. Upload a CSV data file (e.g., `sales_data.csv`)

### Step 3: Try Your Agent in the Playground

Example prompts:

- `Tell me about the data`
- `Can you create a line chart of revenue over time`

---

## Code Interpreter — Python Code

### Step 1: Create the File

Duplicate `simple_example.py` and name it `code_interpreter.py`.

### Step 2: Import the New Dependencies

```python
import os
from dotenv import load_dotenv
from pathlib import Path                                      # new
from azure.ai.agents.models import CodeInterpreterTool, FilePurpose  # new
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
```

### Step 3: Upload the `sales_data.csv` File

Add the following code after the project client initialization:

```python
# Upload a file
file = project_client.agents.files.upload_and_poll(
    file_path="sales_data.csv", purpose=FilePurpose.AGENTS
)
print(f"Uploaded a file, file ID: {file.id}")
```

> **Note:** Ensure that `sales_data.csv` is in your project directory.

### Step 4: Create the Agent

```python
agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="my-agent-for-code-interpreter",
    instructions="You are a helpful agent that performs data analytics and visualization.",
    tools=CodeInterpreterTool(file_ids=[file.id]).definitions,
    tool_resources=CodeInterpreterTool(file_ids=[file.id]).resources,
)
print(f"Created agent, agent ID: {agent.id}")
```

### Step 5: Update the Message Retrieval to Get the Generated File

```python
# Retrieve and display agent's response
messages = list(project_client.agents.messages.list(thread_id=thread.id))
latest_message = messages[0]

if latest_message.content[0].type == 'text':
    agent_response = latest_message.content[0].text.value
    print(f"Agent: {agent_response}")

if latest_message.content[0].type == 'image_file':
    agent_response = latest_message.content[1].text.value
    file_name = f"{latest_message.content[0].image_file.file_id}_image_file.png"
    project_client.agents.files.save(
        file_id=latest_message.content[0].image_file.file_id,
        file_name=file_name,
    )
    print(f"Agent: {agent_response}")
    print(f"Saved image file to: {Path.cwd() / file_name}")
```

### Step 6: Test the Agent

Update the message content to `"Can you generate a line chart for revenue over time"`, then run:

```bash
python code_interpreter.py
```

---

## Complete Code

```python
import os
from dotenv import load_dotenv
from pathlib import Path
from azure.ai.agents.models import CodeInterpreterTool, FilePurpose
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

load_dotenv()  # Load .env file

# Main execution block
if __name__ == '__main__':

    project_client = AIProjectClient(
        credential=DefaultAzureCredential(),
        endpoint=os.environ["PROJECT_ENDPOINT"],
    )

    # Upload a file
    file = project_client.agents.files.upload_and_poll(
        file_path="sales_data.csv", purpose=FilePurpose.AGENTS
    )
    print(f"Uploaded a file, file ID: {file.id}")

    agent = project_client.agents.create_agent(
        model="gpt-4o-mini",
        name="my-agent-for-code-interpreter",
        instructions="You are a helpful agent that performs data analytics and visualization.",
        tools=CodeInterpreterTool(file_ids=[file.id]).definitions,
        tool_resources=CodeInterpreterTool(file_ids=[file.id]).resources,
    )
    print(f"Created agent, agent ID: {agent.id}")

    # Create a thread
    thread = project_client.agents.threads.create()
    print(f"Created thread, thread ID: {thread.id}")

    # Create a message
    message = project_client.agents.messages.create(
        thread_id=thread.id,
        role="user",
        content="Can you generate a line chart for revenue over time",
    )
    print(f"Created message, message ID: {message.id}")

    # Run the agent
    run = project_client.agents.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    print(f"Run finished with status: {run.status}")

    if run.status == "failed":
        # Check if you got "Rate limit is exceeded.", then you want to get more quota
        print(f"Run failed: {run.last_error}")

    # Retrieve and display agent's response
    messages = list(project_client.agents.messages.list(thread_id=thread.id))
    latest_message = messages[0]

    if latest_message.content[0].type == 'text':
        agent_response = latest_message.content[0].text.value
        print(f"Agent: {agent_response}")

    if latest_message.content[0].type == 'image_file':
        agent_response = latest_message.content[1].text.value
        file_name = f"{latest_message.content[0].image_file.file_id}_image_file.png"
        project_client.agents.files.save(
            file_id=latest_message.content[0].image_file.file_id,
            file_name=file_name,
        )
        print(f"Agent: {agent_response}")
        print(f"Saved image file to: {Path.cwd() / file_name}")

# Delete the agent once done
project_client.agents.delete_agent(agent.id)
print("Deleted agent")
```
