## Structure of a MCP server

- #1 Importar FastMCP: ver https://modelcontextprotocol.io/docs/2026-07-28/getting-started/intro

- #2 Initialize MCP server
FastMCP(name="server-name", version"1.0.0")

- #3 Tools - Functions that DO things -> Add tool definition using @mcp.tool()
- #4 Resources - Data endpoints -> Add resource definition using @mcp.resource()
- #5 Prompts - AI assistance templates -> adding prompt definitions using @mcp.prompt()
- #6 Run server -> mcp.run(transport="stdio") o http



### Examples with a flight booker mcp server

##### Resources

- airports
- flight_status
- seat_map
- weather
- booking_info
- gates
- policies
- loyalty

###### Code snippet

```python
@mcp.resource("flight://airports/{code}")
async def get_airport_info(code: String):
    "Get airports details like timezone, termianles, etc."
    return {
        "code": code.upper(),
        "name": "Airport name mock",
        "city": "City name mock",
        "timezone": "America/Buenos-Aires",
        "terminals": ["1","2", "3", "International"]
    }
``` 

##### Tools

- search_flights
- get_flight_details 
- get_booking
- create_booking
- cancel_booking
- modify_booking
- check_in
- select_seats
- add_baggage
- add_services
- track_flight
- price_alert
- upgrade_seat

###### Code snippet

```python
@mcp.tool()
async def search_flights(
    origin: str,
    destination: str,
    departure_date: str,
    passengers: int = 1
) -> dict: """Search for flights between airports"""

    flgihts=search_flights(origin, destination, departure_date)
    return {
        "success": True,
        "flights": flights,
        "total_results": len(flights)
    }
```

##### Prompts

- find_best_flight
- plan_multi:city
- budget_optimizer
- handle_disruption
- accessibility_help
- loyalty_optimizer

###### Code snippet

```python
@mcp.prompt()
async def find_best_flight(
    travel_details: str,
    preferences: str = "best value"
) -> str:
    """AI-assisted flight search with smart recommendations"""
    return f"""
Based on your travel details: "{travel_details}"
And your preferences: "{preferences}"

I'll help you find the perfect flight by:

1. **Analyzing our need:**
    - Extracting cities, dates, passenger count
    - Understanding your priorities (price vs time vs confort)
    
2. **Smart recommendations:**
    -Best value options if budget-focused
    -Fastest routes if time-sensitive
    -Premium options if comfort-focused
3. **Pro tips:**
    -Alternatives airports nearby
    -Best days to fly for savings
    -Optiomal bookign timing

Let me search for flights that match your criteria and suggest the best options!
"""
```

### Server options

##### Stateless vs statefull servers
`mcp = FastMCP("StatefulServer")`

`mcp = FastMCP("StatelessServer, stateless_http=True`)

#### Types of transport
`mcp.run(transport="stdio")`

`mcp.run(transport="http", host="localhost", port=8000)`

`mcp.run(transport="streamable-http", host="localhost", port=8000)`


## MCP Inspector
npx @modelcontextprotocol/inspector

## MCP Client

To build my own AI agent from scratch...

```python
from mcp.client.session import ClientSession
import asyncio

async def client:
    # List what's available 
    tools = await client.list_tools()
    
    # Use tools
    flight = await client.call_tool("search_flight", {
        "origin": "SFO",
        "destination": "JFK"
    })
    
    # Read resources
    status = await client.read_resource("flight://status/UA123")
    
    # Get prompts 
    advice = await client.get_prompt("find_flight", {
        "details": "SFO to JFK"
    })
    
asyncio.run(client())

```
### Contexts

```python
@mcp.tool()
async def long_running_task(task_name: str, ctx: Context, steps: int = 5) -> str:
    """Execute a task with progress updates."""
    await ctx.info(f"Executing task {task_name}")

    for i in range(steps):
        progress = (i + 1) / steps
        await ctx.report_progress(
            progress=progress,
            total=1.0,
            message=f"Step {i + 1} / {steps}",
        )
        await ctx.debug(f"Completed Step {i + 1}")

    return f"Task '{task_name}' completed."
```

### Roots

Folders on the client machine that the MCP server that is allowed to see and interacted with.

MCP tools might need some folders 

`Client.py`
```python
from mcp.client.session import ClientSession

# Client-side roots
allowed_roots = [
    "file:///home/projects/flygpt"
]

client = Client("http//localhost:8080/mcp", roots=allowed_roots)
```

`server.py`
```python
from mcp.server.fastmcp import FastMCP

mcp = FastMCP("flight-server")

@mcp.tool()
aysnc def get_client_roots(ctx: Context):
    roots_response = await ctx.session.list_roots()
    return roots_response
```

### Sampling

The server might want to interact with the LLM, like sending resources to summarize them

The server cannot interact with the LLM directly because it might be used by other clients. 
The client has to have access and control over the LLM integrations, token limits and model selection.

The server takes care of
- Logic
- Tool definition
- Resources
- Is lightweight
- Is decoupled from AI

`client.py`
```python
async def my_llm_handler(messages, params, context):
    # 1. Extract prompt from messages
    prompt = messages[0].context.text
    # 2. generate response (call your LLM)
    response = call_my_llm_service(prompt)
    # 3. Return generated text
    return response

client = Client("http://localhost:8080/mcp", sampling_handler=my_llm_handler)
```

`server.py`
```python
@mcp.toll()
async def genereate_content(topic:str, ctx: Context):
    # 1. Create prompt
    prompt = f"Write something about {topic}"
    
    # 2. Request LLM generation
    result = await ctx.session.create_message([
        SamplingMessage(role="user", content=TextContent(text=prompt))
    ])
    # 3. Return generated content
    return result.content.text
```

## Elicitation

The server might need some additional information from the user

`server.py`

```python
from mcp.server.fastmcp import FastMCP, Context 
from pydantic import BaseModel

mcp = FastMCP("elicitation-demo")

class UserInfo(BaseModel):
    name: str
    age: int

@mcp.tool()
aysnc def ask_user_info(ctx: Context) -> str:
    # Request structured input from user
    result = await ctx.elicit(
        message="Please provide your information",
        schema=UserInfo
    )

    # Use validated data
    user_data = result.data
    return f"Hello {user_data.name}, {user_data.age}"
```

`Client.py`

```python
from mcp.client.session import ClientSession
from mcp.client.streamable_http import streamable_http_client
from mcp.types import ElicitRequestParams, ElicitRequest

aysnc def handle_elicitation(context, params: ElicitRequestParams) -> ElicitResult:
    print(f"Server requests: {params.message}")
    
    name = input("Enter your name: ")
    age = int(input("Enter your Age: "))
    
    user_response = {"name": name, "age": age}

    return ElicitRequest(action="accept", content=user_response)

async def main():
    # Connect to server
    async with streamable_http_client("http://localhost:8080") as (read,write, _):
        async with ClientSession(read, write,
            elicitation_callback=handle_elicitation) as session:
            await session.initialize()
            
            result = await session.call_tool("ask_user_info", {})
            print(f"Result: {result.content[0].text}")
```


## Building Production MCP Clients

##### Learn the key considerations for building robust, production-ready MCP clients.

    Error handling and connection retry logic
    Proper async/await patterns and resource cleanup
    Security considerations for roots and callbacks
    Testing strategies for MCP client applications

##### 🏗️ Best practices covered:

    Connection management and error recovery
    Callback implementation patterns
    Integration with existing Python applications
    Monitoring and logging for MCP operations


## Kubernetes mcp server

Before we configure the MCP server, let's understand what the k8s-mcp-server provides.

##### Key Features:

    22 Kubernetes Tools - Complete cluster management capabilities
    Pod Operations - List, describe, get logs, delete pods
    Node Management - Get node information and metrics
    Resource Operations - Create, update, and manage Kubernetes resources
    Helm Integration - Install, upgrade, and manage Helm charts
    Event Monitoring - Get cluster events and troubleshooting information

##### Benefits:

    Natural language cluster management
    AI-powered troubleshooting
    Automated resource management
    Seamless integration with Roo-Code

```json
{
  "mcpServers": {
    "k8s-mcp-server": {
      "command": "sudo",
      "args": [
        "docker",
        "run",
        "-i",
        "--rm",
        "-v",
        "/home/lab-user/.kube/config:/home/appuser/.kube/config:ro",
        "ginnux/k8s-mcp-server:latest",
        "--mode",
        "stdio"
      ]
    }
  }
}

```