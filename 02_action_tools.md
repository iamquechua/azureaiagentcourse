
# Action Tools: Function Calling with Azure AI Agents

> **Note:** Function calling is only available through Python code, not the Foundry Portal.

---

## Step 1: Get a WeatherAPI.com API Key

1. Create an account at [WeatherAPI.com](https://www.weatherapi.com/)
2. Get your API Key
3. Add the `WEATHER_API_KEY` environment variable to your `.env` file

## Step 2: Create the File

Duplicate `simple_example.py` and name it `function_calling.py`.

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

## Step 4: Create the `get_weather` Function

```python
def get_weather(city: str) -> str:
    """
    Retrieves the weather condition from a given location
    :param city: City name
    :type city: str

    :return: Weather details as a JSON string
    """
    print(f"[LOG] get_weather called with city={city}")

    # Get API key from environment variable
    api_key = os.getenv('WEATHER_API_KEY')

    # WeatherAPI.com Endpoint
    base_url = "http://api.weatherapi.com/v1/current.json?"
    complete_url = f"{base_url}key={api_key}&q={city}&aqi=no"

    try:
        # Make the API request
        response = requests.get(complete_url)
        print(f"[LOG] API response status code: {response.status_code}")
        weather_data = response.json()  # Parse JSON response
        print(f"[LOG] API response data: {weather_data}")

        # Check for API errors
        if response.status_code != 200:
            error_msg = weather_data.get("message", "Failed to fetch weather data.")
            print(f"[LOG] API error: {error_msg}")
            return json.dumps({"error": error_msg})

        # Extract weather details (weatherapi.com structure)
        weather_condition = weather_data["current"]["condition"]["text"]
        temperature = weather_data["current"]["temp_c"]

        the_dump = json.dumps({
            "city": city,
            "weather_condition": weather_condition,
            "temperature_celsius": temperature
        }, indent=4)

        print(f"[LOG] Successfully retrieved weather data")
        return the_dump

    except Exception as e:
        print(f"[LOG] Exception occurred: {str(e)}")
        return json.dumps({"error": str(e)})
```

## Step 5: Create a Toolset with Custom Functions

Add the following code right after initializing the project client:

```python
# Collect user functions to be used as tools
user_functions: Set[Callable[..., Any]] = {get_weather}

# Create toolset with custom functions
functions = FunctionTool(user_functions)
toolset = ToolSet()
toolset.add(functions)
project_client.agents.enable_auto_function_calls(toolset)
```

## Step 6: Create an Agent and Pass It the Toolset

Update the agent creation code:

```python
agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="weather-agent",
    instructions="You are a helpful agent that provides weather information using real-time data.",
    toolset=toolset,
)
print(f"Created agent, agent ID: {agent.id}")
```

## Step 7: Test the Agent

Update the message content to `"What is the weather in London today?"`, then run:

```bash
python function_calling.py
```

---

## Complete Code

```python
import json
import os
import requests
from dotenv import load_dotenv
from typing import Any, Callable, Set

load_dotenv()  # Load .env file

from azure.ai.projects import AIProjectClient
from azure.identity import DefaultAzureCredential
from azure.ai.agents.models import FunctionTool, ToolSet


# Function to get weather information
def get_weather(city: str) -> str:
    """
    Retrieves the weather condition from a given location
    :param city: City name
    :type city: str

    :return: Weather details as a JSON string
    """
    print(f"[LOG] get_weather called with city={city}")

    # Get API key from environment variable
    api_key = os.getenv('WEATHER_API_KEY')

    # WeatherAPI.com Endpoint
    base_url = "http://api.weatherapi.com/v1/current.json?"
    complete_url = f"{base_url}key={api_key}&q={city}&aqi=no"

    try:
        # Make the API request
        response = requests.get(complete_url)
        print(f"[LOG] API response status code: {response.status_code}")
        weather_data = response.json()  # Parse JSON response
        print(f"[LOG] API response data: {weather_data}")

        # Check for API errors
        if response.status_code != 200:
            error_msg = weather_data.get("message", "Failed to fetch weather data.")
            print(f"[LOG] API error: {error_msg}")
            return json.dumps({"error": error_msg})

        # Extract weather details (weatherapi.com structure)
        weather_condition = weather_data["current"]["condition"]["text"]
        temperature = weather_data["current"]["temp_c"]

        the_dump = json.dumps({
            "city": city,
            "weather_condition": weather_condition,
            "temperature_celsius": temperature
        }, indent=4)

        print(f"[LOG] Successfully retrieved weather data")
        return the_dump

    except Exception as e:
        print(f"[LOG] Exception occurred: {str(e)}")
        return json.dumps({"error": str(e)})


# Main execution block
if __name__ == '__main__':

    project_client = AIProjectClient(
        credential=DefaultAzureCredential(),
        endpoint=os.environ["PROJECT_ENDPOINT"],
    )

    # Collect user functions to be used as tools
    user_functions: Set[Callable[..., Any]] = {get_weather}
    # Create toolset with custom functions
    functions = FunctionTool(user_functions)
    toolset = ToolSet()
    toolset.add(functions)
    project_client.agents.enable_auto_function_calls(toolset)

    # Create the Azure AI Agent with specified configurations
    agent = project_client.agents.create_agent(
        model="gpt-4o-mini",
        name="weather-agent",
        instructions="You are a helpful agent that provides weather information using real-time data.",
        toolset=toolset,
    )
    print(f"Created agent, agent ID: {agent.id}")

    # Create a thread
    thread = project_client.agents.threads.create()
    print(f"Created thread, thread ID: {thread.id}")

    # Create a message
    message = project_client.agents.messages.create(
        thread_id=thread.id,
        role="user",
        content="What is the weather in London today?",
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
