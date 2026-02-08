# Knowledge Tools: Azure AI Search

---

## Azure AI Search — Foundry Portal (No Code)

### Step 1: Create a Storage Account

1. Go to the [Azure Portal](https://portal.azure.com/)
2. Go to **"Storage center"** service
3. Go to **Object storage > Blob Storage**
4. Click **"+ Create"**
5. Set the following details:
   - **Storage account name:** `azureaicoursestorage`
   - **Preferred storage type:** Azure Blob Storage or Azure Data Lake Storage Gen 2
   - Leave other settings to default
6. Click **"Review + Create"** → **"Create"**

### Step 2: Upload the Document

1. Once created, click **"Go to resource"**
2. Click **"Upload"**
3. Upload the `gpt-4-system-card.pdf` file
4. Click **"Create New"** to create a new container:
   - **Name:** `azureaicoursestoragecontainer`
5. Click **OK** → **Upload**

### Step 3: Create an Azure OpenAI Resource

1. Go to the Azure Portal
2. Click **"Create a resource"**
3. Search for **"Azure OpenAI"** and click **Create > Azure OpenAI**
4. Set the following details:
   - **Name:** `azureaicourseopenai`
   - **Pricing tier:** Standard
5. Click **Next** until you reach the **Create** button, then click it

### Step 4: Create an Embedding Deployment

1. Open the Azure OpenAI resource in Foundry Portal
2. Go to **Model Catalog**
3. Search for `text-embedding-ada-002` and select it
4. Click **"Use this model"**
5. Set the following details:
   - **Deployment name:** Leave default
   - **Deployment type:** Standard
6. Click **"Deploy"**

### Step 5: Create an Azure AI Search Resource

1. Go to the Azure Portal
2. Click **"Create a resource"**
3. Search for **"Azure AI Search"** and click **Create > Azure AI Search**
4. Set the following details:
   - **Name:** `azureaicourseaisearch`
   - **Pricing tier:** Free
5. Click **"Review + create"** → **"Create"**

### Step 6: Create an Index

1. Open your Azure AI Search resource
2. Click **"Import data (new)"**
3. Choose **"Azure Blob Storage"** as a data source
4. **Scenario:** RAG
5. **Connect to your data:**
   - **Storage account:** Select `azureaicoursestorage`
   - **Blob container:** Select `azureaicoursestoragecontainer`
6. Click **"Next"**
7. **Vectorize your text:**
   - **Kind:** Azure OpenAI
   - **Azure OpenAI service:** Select `azureaicourseopenai`
   - **Model deployment:** `text-embedding-ada-002`
   - ✅ Check the acknowledgement
8. Click **Next** until you see **Create**, then click it

### Step 7: Create an Agent

See the [Create an Agent](./azure-ai-foundry-getting-started.md#3-create-an-agent) section, with these settings:

- **Name:** `AzureAISearchAgent`
- **Instructions:** `You are a helpful RAG agent that uses AI Search tool as source of knowledge.`

### Step 8: Add the Azure AI Search Tool

1. Click **+ Add** in the Knowledge tool section
2. Click **"Azure AI Search"**
3. **Azure AI Search resource connection:**
   - Click **"Connect other Azure AI Search resource"**
   - Click **"Add connection"** for `azureaicourseaisearch`
4. **Azure AI Search index:** Select the index created in the resource
5. **Display name:** `aiagentsearchtool`
6. **Search type:** Hybrid
7. Click **"Connect"**

### Step 9: Try Your Agent in the Playground

Prompt:

> What does your AI Search knowledge say about gpt?

---

## Azure AI Search — Python Code

### Prerequisites

- Storage account is created
- Azure AI Search resource created
- Index is created in Azure AI Search (using the storage account)
- Connection between Azure AI Search resource and Foundry project created

### Step 1: Create the File

Duplicate `simple_example.py` and name it `azure_ai_search.py`.

### Step 2: Import the New Dependencies

```python
import os
from dotenv import load_dotenv

from azure.ai.agents.models import AzureAISearchTool  # new
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
```

### Step 3: Get the Connection

```python
conn_list = project_client.connections.list()
conn_id = ""
for conn in conn_list:
    if conn['name'] == 'azureaicourseaisearch':
        conn_id = conn['id']
        break
print(f"Connection ID: {conn_id}")
```

### Step 4: Initialize the Azure AI Search Tool with the Index

```python
# Initialize the AI Search tool with the index name
ai_search = AzureAISearchTool(
    index_connection_id=conn_id,
    index_name="rag-1770502946631",
)
```

> **Note:** You can find the index name by going to the Azure AI Search resource → **Search Management → Indexes**.

### Step 5: Create the Agent with the Tool

```python
agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="ai-search-agent",
    instructions="You are a helpful RAG agent that uses AI Search tool as source of knowledge.",
    tools=ai_search.definitions,
    tool_resources=ai_search.resources,
)
print(f"Created agent, agent ID: {agent.id}")
```

### Step 6: Test the Agent

Update the message content to `"What does your AI Search knowledge say about gpt?"`, then run:

```bash
python azure_ai_search.py
```

---

## Complete Code

```python
import os
from dotenv import load_dotenv

from azure.ai.agents.models import AzureAISearchTool
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

load_dotenv()  # Load .env file

# Main execution block
if __name__ == '__main__':

    project_client = AIProjectClient(
        credential=DefaultAzureCredential(),
        endpoint=os.environ["PROJECT_ENDPOINT"],
    )

    conn_list = project_client.connections.list()
    conn_id = ""
    for conn in conn_list:
        if conn['name'] == 'azureaicourseaisearch':
            conn_id = conn['id']
            break
    print(f"Connection ID: {conn_id}")

    # Initialize the AI Search tool with the index name
    ai_search = AzureAISearchTool(
        index_connection_id=conn_id,
        index_name="rag-1770502946631",
    )

    agent = project_client.agents.create_agent(
        model="gpt-4o-mini",
        name="ai-search-agent",
        instructions="You are a helpful RAG agent that uses AI Search tool as source of knowledge.",
        tools=ai_search.definitions,
        tool_resources=ai_search.resources,
    )
    print(f"Created agent, agent ID: {agent.id}")

    # Create a thread
    thread = project_client.agents.threads.create()
    print(f"Created thread, thread ID: {thread.id}")

    # Create a message
    message = project_client.agents.messages.create(
        thread_id=thread.id,
        role="user",
        content="What does your AI Search knowledge say about gpt?",
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
