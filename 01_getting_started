# Getting Started with Azure AI Foundry

## 1. Create a Foundry Resource and Project

1. Go to [https://portal.azure.com/](https://portal.azure.com/)
2. Click on **"Foundry"**
3. Click on **"Create a resource"** to create a Foundry resource
4. Set the following details:
   - **Resource group:** Select a resource group
   - **Name:** `azureaicoursefirstfoundryresource`
   - **Default project name:** `azureaicoursefirstproject`
5. Click **"Review + create"** → **"Create"**
6. Once the resource is created, click **"Go to resource"**
7. Click **"Go to Foundry Portal"** to open the project in Azure AI Foundry

---

## 2. Deploy a Model

1. Open the project in Azure AI Foundry
2. Click on **"Models + endpoints"** under *My assets*
3. Click on **"Deploy model"** → **"Deploy base model"**
4. Search for `gpt-4o-mini` and select it
5. Click **"Confirm"**
6. Set the following details:
   - **Deployment name:** Leave the default value `gpt-4o-mini`
   - **Deployment type:** Standard
7. Click **"Deploy"**

---

## 3. Create an Agent

1. Open the project in Azure AI Foundry
2. Click on **"Agents"**
3. Click **"New agent"**
4. Set the following details:
   - **Agent ID:** Leave default
   - **Agent name:** `MyFirstAgent`
   - **Deployment:** Select the deployment created above
   - **Instructions:** `You are a helpful agent that explain the history of Country flags`
5. Changes are automatically saved

---

## 4. Try Your Agent in the Playground

1. Select the agent in the agent's list
2. Click **"Try in Playground"**
3. In the chat, enter the prompt:

   > Tell me about the history of the flag of Albania.

---

## 5. Creating an Agent with Python

### Step 1: Create the Project Structure

1. Create a directory called `azureaisdk`
2. Inside the directory, create a virtual environment:

   ```bash
   python3 -m venv env
   ```

3. Activate the virtual environment
4. Create a Python file called `simple_example.py`

### Step 2: Install the Packages

1. Create a `requirements.txt` file in the project directory with the following packages:

   ```
   azure-ai-projects
   azure-ai-agents
   azure-identity
   ```

2. Install the packages:

   ```bash
   pip install -r requirements.txt
   ```

### Step 3: Set the Environment Variables

Create a `.env` file in the project directory with the following variables:

| Variable | Description |
|----------|-------------|
| `PROJECT_ENDPOINT` | Found in **Project overview** in the Foundry Portal. Example: `https://ai-agent-course-resource-douno.services.ai.azure.com/api/projects/ai-agent-course` |
| `MODEL_DEPLOYMENT_NAME` | The name of the deployment created earlier. Default: `gpt-4o-mini` |

### Step 4a: Import Packages and Load Environment Variables

```python
import os
from dotenv import load_dotenv
from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

load_dotenv()  # Load .env file

# Main execution block
if __name__ == '__main__':
    # Add code here
    pass
```

### Step 4b: The Main Code

**Initialize the project client:**

```python
project_client = AIProjectClient(
    credential=DefaultAzureCredential(),
    endpoint=os.environ["PROJECT_ENDPOINT"],
)
```

**Create the agent:**

```python
agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="my-agent",
    instructions="You are helpful agent that like to describe country flags.",
)
print(f"Created agent with ID: {agent.id}")
```

**Create a thread:**

```python
thread = project_client.agents.threads.create()
print(f"Created thread with ID: {thread.id}")
```

**Create a message:**

```python
message = project_client.agents.messages.create(
    thread_id=thread.id,
    role="user",
    content="Tell me about the history of the flag of Albania.",
)
print(f"Created message with ID: {message.id}")
```

**Run the agent:**

```python
run = project_client.agents.runs.create_and_process(thread_id=thread.id, agent_id=agent.id)
print(f"Run finished with status: {run.status}")

if run.status == "failed":
    # Check if you got "Rate limit is exceeded.", then you want to get more quota
    print(f"Run failed: {run.last_error}")
```

**Get messages from the thread:**

```python
messages = list(project_client.agents.messages.list(thread_id=thread.id))
agent_response = messages[0].content[0].text.value
print(f"Agent: {agent_response}")
```

**Optionally, delete the agent once done:**

```python
project_client.agents.delete_agent(agent.id)
print("Deleted agent")
```

### Complete Code

```python
import os
from dotenv import load_dotenv

load_dotenv()  # Load .env file

from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential

# Main execution block
if __name__ == '__main__':

    project_client = AIProjectClient(
        credential=DefaultAzureCredential(),
        endpoint=os.environ["PROJECT_ENDPOINT"],
    )

    agent = project_client.agents.create_agent(
        model="gpt-4o-mini",
        name="my-agent",
        instructions="You are helpful agent that like to describe country flags.",
    )
    print(f"Created agent, agent ID: {agent.id}")

    # Create a thread
    thread = project_client.agents.threads.create()
    print(f"Created thread, thread ID: {thread.id}")

    # Create a message
    message = project_client.agents.messages.create(
        thread_id=thread.id,
        role="user",
        content="Tell me about the history of the flag of Albania.",
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

### Step 5: Test the Agent

```bash
python simple_example.py
```

---

## 6. Using the Client Secret Credential

### Step 1: Register an App in Microsoft Entra ID

1. Go to [https://portal.azure.com/](https://portal.azure.com/)
2. Click on the **Microsoft Entra ID** service
3. Go to **Manage > App registrations**
4. Click **"New registration"**
5. Set the following details:
   - **Name:** `azureaicourseappregistration`
   - Leave the other settings to default
6. Click **"Register"**

### Step 2: Assign the Azure AI User Role to the App Registration

1. Go to [https://portal.azure.com/](https://portal.azure.com/)
2. Click on the **Foundry** service → **All resources**
3. Click on `azureaicoursefirstfoundryresource`
4. Click on **Access control (IAM)**
5. Click **"+ Add"** → **"Add role assignment"**
6. Search for the **"Azure AI User"** role, select it, and click **"Next"**
7. In the Members tab, click **"+ Select members"**
8. Search for the app registration created earlier and select it
9. Click **"Review + assign"** (twice)

### Step 3: Import ClientSecretCredential and Update Environment Variables

Add the following environment variables to your `.env` file:

| Variable | How to Find It |
|----------|----------------|
| `CLIENT_ID` | Microsoft Entra ID → Manage → App registrations → All applications → `azureaicourseappregistration` → Copy **"Application (client) ID"** |
| `CLIENT_SECRET` | Microsoft Entra ID → Manage → App registrations → `azureaicourseappregistration` → Manage → **Certificates & secrets** → **New client secret** → Set description: `my client secret`, leave default expiry → Click **"Add"** → Copy the secret value (shown only once) |
| `TENANT_ID` | Microsoft Entra ID → Manage → App registrations → All applications → `azureaicourseappregistration` → Copy **"Directory (tenant) ID"** |

**Update the import:**

```python
from azure.identity import ClientSecretCredential
```

**Configure Azure credentials:**

```python
credential = ClientSecretCredential(
    tenant_id=os.getenv("TENANT_ID"),
    client_id=os.getenv("CLIENT_ID"),
    client_secret=os.getenv("CLIENT_SECRET")
)
```

**Initialize the project client with the ClientSecretCredential:**

```python
project_client = AIProjectClient(
    credential=credential,
    endpoint=os.environ["PROJECT_ENDPOINT"]
)
```

> **Note:** The original document uses `os.getenv["..."]` with square brackets, which is incorrect Python syntax. The correct syntax is `os.getenv("...")` with parentheses (as shown above) or `os.environ["..."]` with square brackets.
