# OpenAPI 3.0 Tool with Azure AI Agents

---

## OpenAPI 3.0 Tool — Foundry Portal

### Step 1: Create a RapidAPI Account

1. Create an account on [rapidapi.com](https://rapidapi.com)
2. Go to the [IMDB API page](https://rapidapi.com/rapidapi-org1-rapidapi-org-default/api/imdb236)
3. Click **"Subscribe to Test"** and subscribe
4. Copy the `x-rapidapi-key` value

### Step 2: Create a Connection

1. Open your project in Foundry Portal
2. Click on **"Management center"**
3. Click on **"Connected resources"**
4. Click on **"Custom keys"**
5. Click **"+ Add key value pairs"**
   - **Key:** `x-rapidapi-key`
   - **Value:** Paste the value copied earlier
   - **Check:** ✅ Is secret
   - **Connection name:** `imdbconnection`
   - **Access:** This project only
6. Click **"Add Connection"**

### Step 3: Create an Agent

See the [Create an Agent](./azure-ai-foundry-getting-started.md#3-create-an-agent) section, with these settings:

- **Name:** `MovieAgent`
- **Instructions:** `You are an agent that get the list of movies an actor was in using the IMDB API.`

### Step 4: Add the OpenAPI Tool

1. Click **+ Add** in the Actions tool section
2. Click **"OpenAPI 3.0 specified tool"**
3. Set the tool details:
   - **Name:** `openapitool`
4. Click **"Next"**
5. Define schema:
   - **Authentication method:** Select connection
   - **Choose a connection:** `imdbconnection`
   - **Schema:** (paste the JSON below)

<details>
<summary>Click to expand the OpenAPI schema</summary>

```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "IMDB236 Cast Titles API",
    "description": "API to fetch movie and TV show titles for a given cast member from IMDb.",
    "version": "1.0.0"
  },
  "servers": [
    {
      "url": "https://imdb236.p.rapidapi.com/api/imdb"
    }
  ],
  "paths": {
    "/cast/{castId}/titles": {
      "get": {
        "operationId": "getCastTitles",
        "summary": "Get titles by cast member",
        "description": "Fetches all movie and TV show titles associated with a given cast member (actor, director, etc.) identified by their IMDb name ID.",
        "parameters": [
          {
            "name": "castId",
            "in": "path",
            "required": true,
            "schema": {
              "type": "string"
            },
            "description": "IMDb name ID of the cast member (e.g., nm0000190 for Matthew McConaughey)",
            "example": "nm0000190"
          }
        ],
        "responses": {
          "200": {
            "description": "Successful response",
            "content": {
              "application/json": {
                "schema": {
                  "type": "array",
                  "items": {
                    "type": "object",
                    "properties": {
                      "id": {
                        "type": "string",
                        "description": "IMDb title ID (e.g., tt1234567)"
                      },
                      "url": {
                        "type": "string",
                        "description": "URL to the IMDb title page"
                      },
                      "primaryTitle": {
                        "type": "string",
                        "description": "Primary display title of the movie or show"
                      },
                      "originalTitle": {
                        "type": "string",
                        "description": "Original title of the movie or show"
                      },
                      "type": {
                        "type": "string",
                        "description": "Type of title (e.g., movie, tvSeries, tvEpisode)"
                      },
                      "genres": {
                        "type": "array",
                        "items": { "type": "string" },
                        "description": "List of genres associated with the title"
                      },
                      "isAdult": {
                        "type": "boolean",
                        "description": "Whether the title is adult content"
                      },
                      "startYear": {
                        "type": "integer",
                        "description": "Year the title was released or started airing"
                      },
                      "endYear": {
                        "type": "integer",
                        "nullable": true,
                        "description": "Year the title ended (null if still ongoing or a movie)"
                      },
                      "runtimeMinutes": {
                        "type": "integer",
                        "nullable": true,
                        "description": "Runtime in minutes (null if not available)"
                      },
                      "averageRating": {
                        "type": "number",
                        "description": "IMDb average user rating (0-10)"
                      },
                      "numVotes": {
                        "type": "integer",
                        "description": "Number of user votes on IMDb"
                      },
                      "directors": {
                        "type": "array",
                        "items": { "$ref": "#/components/schemas/Person" },
                        "description": "List of directors for the title"
                      },
                      "writers": {
                        "type": "array",
                        "items": { "$ref": "#/components/schemas/Person" },
                        "description": "List of writers for the title"
                      },
                      "totalSeasons": {
                        "type": "integer",
                        "description": "Total number of seasons (for TV series)"
                      },
                      "totalEpisodes": {
                        "type": "integer",
                        "description": "Total number of episodes (for TV series)"
                      },
                      "episodes": {
                        "type": "array",
                        "items": { "type": "object" },
                        "description": "List of episode objects (for TV series)"
                      }
                    }
                  }
                }
              }
            }
          },
          "400": { "description": "Bad request due to missing or incorrect parameters" },
          "401": { "description": "Unauthorized - Invalid or missing API key" },
          "404": { "description": "Cast member not found" },
          "500": { "description": "Internal server error" }
        },
        "security": [
          {
            "apiKeyAuth": [],
            "apiHostAuth": []
          }
        ]
      }
    }
  },
  "components": {
    "schemas": {
      "Person": {
        "type": "object",
        "properties": {
          "id": {
            "type": "string",
            "description": "IMDb name ID (e.g., nm0000190)"
          },
          "url": {
            "type": "string",
            "description": "URL to the person's IMDb page"
          },
          "fullName": {
            "type": "string",
            "description": "Full name of the person"
          }
        }
      }
    },
    "securitySchemes": {
      "apiKeyAuth": {
        "type": "apiKey",
        "in": "header",
        "name": "x-rapidapi-key",
        "description": "RapidAPI subscription key"
      },
      "apiHostAuth": {
        "type": "apiKey",
        "in": "header",
        "name": "x-rapidapi-host",
        "description": "RapidAPI host identifier (imdb236.p.rapidapi.com)"
      }
    }
  }
}
```

</details>

6. Click **"Next"** → **"Create Tool"**

### Step 5: Try Your Agent in the Playground

Prompt:

> Tom Hanks

---

## OpenAPI 3.0 Tool — Python Code

### Step 1: Create the File

Duplicate `simple_example.py` and name it `openapi.py`.

### Step 2: Import the New Dependencies

```python
import jsonref  # new
import os
from dotenv import load_dotenv

from azure.ai.projects import AIProjectClient
from azure.ai.agents.models import (  # new
    OpenApiTool,
    OpenApiConnectionAuthDetails,
    OpenApiConnectionSecurityScheme,
)
from azure.identity import DefaultAzureCredential
```

### Step 3: Load the OpenAPI Spec for the IMDB API

Create `openapi-imdb.json` in your project directory with the schema from above.

Then load the specification immediately after initializing the project client:

```python
# Load OpenAPI Spec
with open('./openapi-imdb.json', 'r') as f:
    openapi_spec = jsonref.loads(f.read())
```

### Step 4: Get the Connection Created in the Foundry Portal

```python
# Extract the connection list
conn_list = project_client.connections.list()
imdb_conn_id = [conn["id"] for conn in conn_list if conn.get("name") == "imdbconnection"][0]
print(f"The ID of the connection is: {imdb_conn_id}")
```

### Step 5: Create an Auth Object for the OpenAPI Tool

```python
# Create Auth object for the OpenAPI Tool
auth = OpenApiConnectionAuthDetails(
    security_scheme=OpenApiConnectionSecurityScheme(
        connection_id=f"{imdb_conn_id}"
    )
)
```

### Step 6: Initialize the OpenAPI Tool with the IMDB API

```python
# Initialize OpenAPI tool with IMDB API
openapi_tool = OpenApiTool(
    name="IMDBTool",
    spec=openapi_spec,
    description="Fetches movies of various actors from the IMDB API.",
    auth=auth,
)
```

### Step 7: Create the Agent and Pass It the OpenAPI Tool

```python
# Create a new AI assistant and attach the tool to it
agent = project_client.agents.create_agent(
    model="gpt-4o-mini",
    name="imdb-agent",
    instructions="You are an agent that get the list of movies an actor was in using the IMDB API.",
    tools=openapi_tool.definitions,
)
print(f"Created agent, agent ID: {agent.id}")
```

### Step 8: Test the Agent

Update the message content to `"Tom Hanks"`, then run:

```bash
python openapi.py
```

---

## Complete Code

```python
import jsonref
import os
from dotenv import load_dotenv

from azure.ai.projects import AIProjectClient
from azure.ai.agents.models import (
    OpenApiTool,
    OpenApiConnectionAuthDetails,
    OpenApiConnectionSecurityScheme,
)
from azure.identity import DefaultAzureCredential

load_dotenv()  # Load .env file

# Main execution block
if __name__ == '__main__':

    project_client = AIProjectClient(
        credential=DefaultAzureCredential(),
        endpoint=os.environ["PROJECT_ENDPOINT"],
    )

    # Load OpenAPI Spec
    with open('./openapi-imdb.json', 'r') as f:
        openapi_spec = jsonref.loads(f.read())

    # Extract the connection list
    conn_list = project_client.connections.list()
    imdb_conn_id = [conn["id"] for conn in conn_list if conn.get("name") == "imdbconnection"][0]
    print(f"The ID of the connection is: {imdb_conn_id}")

    # Create Auth object for the OpenAPI Tool
    auth = OpenApiConnectionAuthDetails(
        security_scheme=OpenApiConnectionSecurityScheme(
            connection_id=f"{imdb_conn_id}"
        )
    )

    # Initialize OpenAPI tool with IMDB API
    openapi_tool = OpenApiTool(
        name="IMDBTool",
        spec=openapi_spec,
        description="Fetches movies of various actors from the IMDB API.",
        auth=auth,
    )

    # Create a new AI assistant and attach the tool to it
    agent = project_client.agents.create_agent(
        model="gpt-4o-mini",
        name="imdb-agent",
        instructions="You are an agent that get the list of movies an actor was in using the IMDB API.",
        tools=openapi_tool.definitions,
    )
    print(f"Created agent, agent ID: {agent.id}")

    # Create a thread
    thread = project_client.agents.threads.create()
    print(f"Created thread, thread ID: {thread.id}")

    # Create a message
    message = project_client.agents.messages.create(
        thread_id=thread.id,
        role="user",
        content="Tom Hanks",
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
