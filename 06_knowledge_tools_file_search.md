# Knowledge Tools: File Search

---

## File Search — Foundry Portal (No Code)

### Step 1: Create an Agent

See the [Create an Agent](./azure-ai-foundry-getting-started.md#3-create-an-agent) section, with these settings:

- **Name:** `FileSearchAgent`
- **Instructions:** `You are a helpful assistant that can search and summarize documents. Use the file search tool to answer questions based on the uploaded documents.`

### Step 2: Add the File Search Tool

1. Click **+ Add** in the Knowledge tool section
2. Click **"Files"**
3. Upload the PDF file (e.g., `gpt-4-system-card.pdf`)

### Step 3: Try Your Agent in the Playground

Example prompts:

- `What are the main topics covered in the uploaded file?`
- `Tell me more about Future Work section`

---

## File Search — Python Code

### Step 1: Create the File

Duplicate `simple_example.py` and name it `file_search.py`.

### Step 2: Import the New Dependencies

```python
import os
from dotenv import load_dotenv

from azure.ai.agents.models import FileSearchTool, FilePurpose  # new
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
```

### Step 3: Upload the File to the Vector Store

```python
# Upload a file for vector store creation
file = project_client.agents.files.upload_and_poll(
    file_path="test_document.txt",
    purpose=FilePurpose.AGENTS,
)
```

> **Note:** Ensure that `test_document.txt` is in your project directory.

### Step 4: Create a Vector Store

```python
# Create a vector store
vector_store = project_client.agents.vector_stores.create_and_poll(
    file_ids=[file.id],
    name="my_vectorstore",
)
print(f"Created vector store, vector store ID: {vector_store.id}")
```

### Step 5: Create the File Search Tool

```python
# Create a file search tool
file_search_tool = FileSearchTool(vector_store_ids=[vector_store.id])
```

### Step 6: Create a New Agent and Attach the Tool

```python
# Create a new agent and attach the tool to it
agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="file-search-agent",
    instructions="You are a helpful assistant that can search and summarize documents. Use the file search tool to answer questions based on the uploaded documents",
    tools=file_search_tool.definitions,
    tool_resources=file_search_tool.resources,
)
print(f"Created agent, agent ID: {agent.id}")
```

### Step 7: Test the Agent

Update the message content to `"What are the key concepts in this document?"`, then run:

```bash
python file_search.py
```

---

## Complete Code

```python
import os
from dotenv import load_dotenv

from azure.ai.agents.models import FileSearchTool, FilePurpose
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

load_dotenv()  # Load .env file

# Main execution block
if __name__ == '__main__':

    project_client = AIProjectClient(
        credential=DefaultAzureCredential(),
        endpoint=os.environ["PROJECT_ENDPOINT"],
    )

    # Upload a file for vector store creation
    file = project_client.agents.files.upload_and_poll(
        file_path="test_document.txt",
        purpose=FilePurpose.AGENTS,
    )

    # Create a vector store
    vector_store = project_client.agents.vector_stores.create_and_poll(
        file_ids=[file.id],
        name="my_vectorstore",
    )
    print(f"Created vector store, vector store ID: {vector_store.id}")

    # Create a file search tool
    file_search_tool = FileSearchTool(vector_store_ids=[vector_store.id])

    # Create a new agent and attach the tool to it
    agent = project_client.agents.create_agent(
        model="gpt-4o-mini",
        name="file-search-agent",
        instructions="You are a helpful assistant that can search and summarize documents. Use the file search tool to answer questions based on the uploaded documents",
        tools=file_search_tool.definitions,
        tool_resources=file_search_tool.resources,
    )
    print(f"Created agent, agent ID: {agent.id}")

    # Create a thread
    thread = project_client.agents.threads.create()
    print(f"Created thread, thread ID: {thread.id}")

    # Create a message
    message = project_client.agents.messages.create(
        thread_id=thread.id,
        role="user",
        content="What are the key concepts in this document?",
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
# project_client.agents.delete_agent(agent.id)
# print("Deleted agent")
```
