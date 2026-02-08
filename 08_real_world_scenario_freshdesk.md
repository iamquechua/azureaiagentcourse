# Real World Scenario: Freshdesk Agent

---

## Step 1: Get a Freshdesk API Key

1. Go to [https://www.freshworks.com/](https://www.freshworks.com/)
2. Create an account
3. Go to **Profile Settings** to get your API Key
4. Go to the **FreshDesk admin**, scroll down to **HelpDesk Settings** and get the HelpDesk URL
5. Add the following environment variables to your `.env` file:

| Variable | Description |
|----------|-------------|
| `FRESHDESK_API_KEY` | Your Freshdesk API key from Profile Settings |
| `FRESHDESK_DOMAIN` | Your HelpDesk URL |

## Step 2: Create the File

Duplicate `simple_example.py` and name it `freshdesk.py`.

## Step 3: Import the New Dependencies

```python
import json                    # new
import os
import requests                # new
from dotenv import load_dotenv
from typing import Any, Callable, Set  # new

from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.agents.models import FunctionTool, ToolSet  # new
```

## Step 4: Create the `create_freshdesk_ticket` Function

```python
# Custom function to create Freshdesk tickets
def create_freshdesk_ticket(Email: str, Subject: str) -> str:
    """
    Creates a ticket in Freshdesk via its API

    :param Email: The email of the user reporting the issue.
    :param Subject: Subject line of the support ticket.
    :return: Ticket information as a JSON string
    :rtype: str
    """

    # Freshdesk domain and API Key configuration
    FRESHDESK_DOMAIN = os.getenv('FRESHDESK_DOMAIN')
    FRESHDESK_API_KEY = os.getenv('FRESHDESK_API_KEY')

    # Construct the API endpoint URL
    url = f'https://{FRESHDESK_DOMAIN}/api/v2/tickets'

    # Prepare ticket data for API request
    ticket_data = {
        'email': Email,
        'subject': Subject,
        'description': 'This is a ticket created via Azure AI Agent Service.',
        'priority': 2,  # Medium priority
        'status': 2,    # Open status
        'tags': ['API', 'Python']  # Additional ticket metadata
    }

    # Set up request headers
    headers = {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
    }

    # Send POST request to Freshdesk API
    response = requests.post(
        url,
        auth=(FRESHDESK_API_KEY, 'X'),  # Authentication using API Key
        headers=headers,
        data=json.dumps(ticket_data),
    )

    # Process and return API response
    if response.status_code == 201:
        return json.dumps(response.json(), indent=4)
    else:
        return json.dumps({'error': response.text, 'status_code': response.status_code}, indent=4)
```

## Step 5: Create a Toolset with Custom Functions

Add the following code right after initializing the project client:

```python
# Collect user functions to be used as tools
user_functions: Set[Callable[..., Any]] = {create_freshdesk_ticket}

# Create toolset with custom functions
functions = FunctionTool(user_functions)
toolset = ToolSet()
toolset.add(functions)
project_client.agents.enable_auto_function_calls(toolset)
```

## Step 6: Create an Agent and Pass It the Toolset

```python
# Create the Azure AI agent with specified configurations
agent = project_client.agents.create_agent(
    model='gpt-4o-mini',
    name='freshdesk-agent',
    instructions='You are a helpful agent that can create FreshDesk tickets when needed',
    toolset=toolset,
)
print(f'Created agent, agent ID: {agent.id}')
```

## Step 7: Test the Agent

Update the message content to `"I forgot my password for my account. Create a freshdesk ticket and send it to info@54startups.com"`, then run:

```bash
python freshdesk.py
```

---

## Complete Code

```python
import json
import os
import requests
from dotenv import load_dotenv
from typing import Any, Callable, Set

from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.agents.models import FunctionTool, ToolSet

load_dotenv()  # Load .env file


# Custom function to create Freshdesk tickets
def create_freshdesk_ticket(Email: str, Subject: str) -> str:
    """
    Creates a ticket in Freshdesk via its API

    :param Email: The email of the user reporting the issue.
    :param Subject: Subject line of the support ticket.
    :return: Ticket information as a JSON string
    :rtype: str
    """

    # Freshdesk domain and API Key configuration
    FRESHDESK_DOMAIN = os.getenv('FRESHDESK_DOMAIN')
    FRESHDESK_API_KEY = os.getenv('FRESHDESK_API_KEY')

    # Construct the API endpoint URL
    url = f'https://{FRESHDESK_DOMAIN}/api/v2/tickets'

    # Prepare ticket data for API request
    ticket_data = {
        'email': Email,
        'subject': Subject,
        'description': 'This is a ticket created via Azure AI Agent Service.',
        'priority': 2,  # Medium priority
        'status': 2,    # Open status
        'tags': ['API', 'Python']  # Additional ticket metadata
    }

    # Set up request headers
    headers = {
        'Content-Type': 'application/json',
        'Accept': 'application/json'
    }

    # Send POST request to Freshdesk API
    response = requests.post(
        url,
        auth=(FRESHDESK_API_KEY, 'X'),  # Authentication using API Key
        headers=headers,
        data=json.dumps(ticket_data),
    )

    # Process and return API response
    if response.status_code == 201:
        return json.dumps(response.json(), indent=4)
    else:
        return json.dumps({'error': response.text, 'status_code': response.status_code}, indent=4)


# Main execution block
if __name__ == '__main__':

    project_client = AIProjectClient(
        credential=DefaultAzureCredential(),
        endpoint=os.environ["PROJECT_ENDPOINT"],
    )

    # Collect user functions to be used as tools
    user_functions: Set[Callable[..., Any]] = {create_freshdesk_ticket}

    # Create toolset with custom functions
    functions = FunctionTool(user_functions)
    toolset = ToolSet()
    toolset.add(functions)
    project_client.agents.enable_auto_function_calls(toolset)

    # Create the Azure AI agent with specified configurations
    agent = project_client.agents.create_agent(
        model='gpt-4o-mini',
        name='freshdesk-agent',
        instructions='You are a helpful agent that can create FreshDesk tickets when needed',
        toolset=toolset,
    )
    print(f'Created agent, agent ID: {agent.id}')

    # Create a thread
    thread = project_client.agents.threads.create()
    print(f"Created thread, thread ID: {thread.id}")

    # Create a message
    message = project_client.agents.messages.create(
        thread_id=thread.id,
        role="user",
        content="I forgot my password for my account. Create a freshdesk ticket and send it to info@54startups.com",
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
