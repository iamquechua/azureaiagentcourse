# Knowledge Tools: Grounding with Bing Search

---

## Grounding with Bing Search — Foundry Portal (No Code)

### Step 1: Create a Grounding with Bing Search Resource

1. Go to the [Azure Portal](https://portal.azure.com/)
2. Click **"Create a resource"**
3. Search for **"Grounding with Bing Search"** and click **Create > Grounding with Bing Search**
4. Configure the resource:
   - **Name:** `azureaicoursebingsearchresource`
   - **Region:** Global
   - **Pricing tier:** Select the only available one
   - ✅ Check terms of use
5. Click **"Review + create"** → **"Create"**

### Step 2: Set Up a Connection to Your Foundry Project

1. Go to Azure Foundry
2. Go to the **Management Center** for your project
3. Click on **"Connected resources"**
4. Click **"New connection"**
5. Scroll down to **"Grounding with Bing Search"** and click on it
6. Your Bing Search resource should be selected — click **"Add connection"**
7. Click **"Close"**

### Step 3: Create an Agent

See the [Create an Agent](./azure-ai-foundry-getting-started.md#3-create-an-agent) section, with these settings:

- **Name:** `BingSearchAgent`
- **Instructions:** `You are an intelligent assistant that is grounding using Bing Search.`

### Step 4: Add the Bing Search Tool

1. Click **+ Add** in the Knowledge tool section
2. Click **"Grounding with Bing Search"**
3. Select the newly created connection, click **"Next"**, then **"Connect"**

Your agent is ready for testing.

### Step 5: Try Your Agent in the Playground

Prompt:

> What's the latest AI news today?

---

## Grounding with Bing Search — Python Code

### Prerequisites

- Grounding with Bing Search resource created
- Connection between Bing Search resource and Foundry project created

### Step 1: Create the File

Duplicate `simple_example.py` and name it `bing_search.py`.

### Step 2: Import the New Dependencies

```python
import os
from dotenv import load_dotenv

from azure.ai.agents.models import BingGroundingTool  # new
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
```

### Step 3: Retrieve the Connection and Get Its ID

```python
# Get Bing search resource connection ID
bing_connection = project_client.connections.get(
    name='azureaicoursebingsearchresource'
)
conn_id = bing_connection.id
print(f"Connection ID: {conn_id}")
```

### Step 4: Initialize the Bing Grounding Tool

```python
# Initialize Bing grounding tool
bing = BingGroundingTool(connection_id=conn_id)
```

### Step 5: Create the Agent and Attach the Tool

```python
# Create a new AI assistant and attach the tool to it
agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="bing-search-agent",
    instructions="You are an intelligent assistant that is grounding using Bing Search",
    tools=bing.definitions,
    headers={"x-ms-enable-preview": "true"},
)
print(f"Created agent, agent ID: {agent.id}")
```

### Step 6: Test the Agent

Update the message content to `"What's the latest AI news today?"`, then run:

```bash
python bing_search.py
```

---

## Complete Code

```python
import os
from dotenv import load_dotenv

from azure.ai.agents.models import BingGroundingTool
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

load_dotenv()  # Load .env file

# Main execution block
if __name__ == '__main__':

    project_client = AIProjectClient(
        credential=DefaultAzureCredential(),
        endpoint=os.environ["PROJECT_ENDPOINT"],
    )

    # Get Bing search resource connection ID
    bing_connection = project_client.connections.get(
        name='azureaicoursebingsearchresource'
    )
    conn_id = bing_connection.id
    print(f"Connection ID: {conn_id}")

    # Initialize Bing grounding tool
    bing = BingGroundingTool(connection_id=conn_id)

    # Create a new AI assistant and attach the tool to it
    agent = project_client.agents.create_agent(
        model="gpt-4o-mini",
        name="bing-search-agent",
        instructions="You are an intelligent assistant that is grounding using Bing Search",
        tools=bing.definitions,
        headers={"x-ms-enable-preview": "true"},
    )
    print(f"Created agent, agent ID: {agent.id}")

    # Create a thread
    thread = project_client.agents.threads.create()
    print(f"Created thread, thread ID: {thread.id}")

    # Create a message
    message = project_client.agents.messages.create(
        thread_id=thread.id,
        role="user",
        content="What's the latest AI news today?",
    )
    print(f"Created message, message ID: {message.id}")

    # Run the agent
    run = project_client.agents.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
    print(f"Run finished with status: {run.status}")

    if run.status == "failed":
        # Check if you got "Rate limit is exceeded.", then you want to get more quota
        print(f"Run failed: {run.last_error}")

    # Get messages from the thread
    messages = list(project_client.agents.messages.list(thread_id=thread.id))
    agent_response = messages[0].content[0].text.value
    print(f"Agent: {agent_response}")

# Delete the agent once done
project_client.agents.delete_agent(agent.id)
print("Deleted agent")
```
