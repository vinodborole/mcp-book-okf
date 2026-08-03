---
type: Web Page
title: Understanding MCP servers - Model Context Protocol
resource: https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts
timestamp: '2026-08-03T09:44:29.575770+00:00'
---

## Core Server Features

Servers provide functionality through three building blocks:
We will use a hypothetical scenario to demonstrate the role of each of these features, and show how they can work together.

### Tools

Tools enable AI models to perform actions. Each tool defines a specific operation with typed inputs and outputs. The model requests tool execution based on context.
#### How Tools Work

Tools are schema-defined interfaces that LLMs can invoke. MCP uses JSON Schema for validation. Each tool performs a single operation with clearly defined inputs and outputs. Tools may require user consent prior to execution, helping to ensure users maintain control over actions taken by a model.
**Protocol operations:**

**Example tool definition:**

#### Example: Travel Booking

Tools enable AI applications to perform actions on behalf of users. In a travel planning scenario, the AI application might use several tools to help book a vacation:
**Flight Search**

**Calendar Blocking**

**Email notification**

#### User Interaction Model

Tools are model-controlled, meaning AI models can discover and invoke them automatically. However, MCP emphasizes human oversight through several mechanisms. For trust and safety, applications can implement user control through various mechanisms, such as:
- Displaying available tools in the UI, enabling users to define whether a tool should be made available in specific interactions
- Approval dialogs for individual tool executions
- Permission settings for pre-approving certain safe operations
- Activity logs that show all tool executions with their results

### Resources

Resources provide structured access to information that the AI application can retrieve and provide to models as context.
#### How Resources Work

Resources expose data from files, APIs, databases, or any other source that an AI needs to understand context. Applications can access this information directly and decide how to use it - whether that’s selecting relevant portions, searching with embeddings, or passing it all to the model. Each resource has a unique URI (e.g.,`file:///path/to/document.md`) and declares its MIME type for appropriate content handling.
Resources support two discovery patterns:
- **Direct Resources** - fixed URIs that point to specific data. Example:`calendar://events/2024` - returns calendar availability for 2024
- **Resource Templates** - dynamic URIs with parameters for flexible queries. Example:
  - `travel://activities/{city}/{category}` - returns activities by city and category
  - `travel://activities/barcelona/museums` - returns all museums in Barcelona

**Protocol operations:**

To watch specific resources for changes, a client sends a 

[request with the resource URIs listed in the](/specification/2026-07-28/basic/patterns/subscriptions)

`subscriptions/listen``resourceSubscriptions` filter. The server delivers `notifications/resources/updated` on the resulting stream whenever a watched resource changes.
#### Example: Getting Travel Planning Context

Continuing with the travel planning example, resources provide the AI application with access to relevant information:
- **Calendar data** (`calendar://events/2024` ) - Checks user availability
- **Travel documents** (`file:///Documents/Travel/passport.pdf` ) - Accesses important documents
- **Previous itineraries** (`trips://history/barcelona-2023` ) - References past trips and preferences

**Resource Template Examples:**

`origin` airport and begins to input “Bar” as the `destination` airport, the system can suggest “Barcelona (BCN)” or “Barbados (BGI)”.
#### Parameter Completion

Dynamic resources support parameter completion. For example:
- Typing “Par” as input for `weather://forecast/{city}` might suggest “Paris” or “Park City”
- Typing “JFK” for `flights://search/{airport}` might suggest “JFK - John F. Kennedy International”

#### User Interaction Model

Resources are application-driven, giving them flexibility in how they retrieve, process, and present available context. Common interaction patterns include:
- Tree or list views for browsing resources in familiar folder-like structures
- Search and filter interfaces for finding specific resources
- Automatic context inclusion or smart suggestions based on heuristics or AI selection
- Manual or bulk selection interfaces for including single or multiple resources

### Prompts

Prompts provide reusable templates. They allow MCP server authors to provide parameterized prompts for a domain, or showcase how to best use the MCP server.
#### How Prompts Work

Prompts are structured templates that define expected inputs and interaction patterns. They are user-controlled, requiring explicit invocation rather than automatic triggering. Prompts can be context-aware, referencing available resources and tools to create comprehensive workflows. Similar to resources, prompts support parameter completion to help users discover valid argument values.
**Protocol operations:**

#### Example: Streamlined Workflows

Prompts provide structured templates for common tasks. In the travel planning context:
**“Plan a vacation” prompt:**

1. Selection of the “Plan a vacation” template
2. Structured input: Barcelona, 7 days, $3000, [“beaches”, “architecture”, “food”]
3. Consistent workflow execution based on the template

#### User Interaction Model

Prompts are user-controlled, requiring explicit invocation. The protocol gives implementers freedom to design interfaces that feel natural within their application. Key principles include:
- Easy discovery of available prompts
- Clear descriptions of what each prompt does
- Natural argument input with validation
- Transparent display of the prompt’s underlying template

- Slash commands (typing ”/” to see available prompts like /plan-vacation)
- Command palettes for searchable access
- Dedicated UI buttons for frequently used prompts
- Context menus that suggest relevant prompts

## Bringing Servers Together

The real power of MCP emerges when multiple servers work together, combining their specialized capabilities through a unified interface.
### Example: Multi-Server Travel Planning

Consider a personalized AI travel planner application, with three connected servers:
- **Travel Server** - Handles flights, hotels, and itineraries
- **Weather Server** - Provides climate data and forecasts
- **Calendar/Email Server** - Manages schedules and communications

#### The Complete Flow

1. 
**User invokes a prompt with parameters:**
2. 
**User selects resources to include:**  - `calendar://my-calendar/June-2024` (from Calendar Server)
  - `travel://preferences/europe` (from Travel Server)
  - `travel://past-trips/Spain-2023` (from Travel Server)
3. 
**AI processes the request using tools:** The AI first reads all selected resources to gather context - identifying available dates from the calendar, learning preferred airlines and hotel types from travel preferences, and discovering previously enjoyed locations from past trips.
Using this context, the AI then executes the prompt provided by the AI application. In our example, the AI application exposes the weather tools from the connected MCP weather server to the model. Because weather can affect travel plans, the AI chooses to call`checkWeather()` when interpreting the prompt.
As a result the AI executes a series of tools:
  - `searchFlights()` - Queries airlines for NYC to Barcelona flights
  - `checkWeather()` - Retrieves climate forecasts for travel dates
 
  - `bookHotel()` - Finds hotels within the specified budget
  - `createCalendarEvent()` - Adds the trip to the user’s calendar
  - `sendEmail()` - Sends confirmation with trip details

**The result:**Through multiple MCP servers, the user researched and booked a Barcelona trip tailored to their schedule. The “Plan a Vacation” prompt guided the AI to combine Resources (calendar availability and travel history) with Tools (searching flights, booking hotels, updating calendars) across different servers—gathering context and executing the booking. A task that could have taken hours was completed in minutes using MCP.

# Citations

1. Source page: https://modelcontextprotocol.io/docs/2026-07-28/learn/server-concepts
