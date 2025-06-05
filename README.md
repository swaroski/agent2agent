# A2A with MCP as Registry

Leveraging **Model Context Protocol (MCP)** as a standardized mechanism for discovering and retrieving **Google A2A Agent Cards**, enabling dynamic agent interaction using A2A.

---

## Table of Contents

- [Objective](#objective)
- [Background](#background)
  - [A2A Protocol](#a2a-protocol)
  - [Model Context Protocol (MCP)](#model-context-protocol-mcp)
- [Core Proposal](#core-proposal)
  - [Storing Agent Cards](#storing-agent-cards)
  - [Discovering Agents via MCP](#discovering-agents-via-mcp)
  - [Retrieving Agent Cards](#retrieving-agent-cards)
  - [Finding an Agent for a Task](#finding-an-agent-for-a-task)
  - [Initiating A2A Communication](#initiating-a2a-communication)
- [Use Case: Orchestrated Task Execution](#use-case-orchestrated-task-execution)
- [Core Concepts](#core-concepts)
- [Architectural Components](#architectural-components)
- [Example Flow: Travel Agent](#example-flow-travel-agent)
- [Steps to Execute the Example](#steps-to-execute-the-example)
- [File/Directory Descriptions](#filedirectory-descriptions)

---

## Objective

To leverage **Model Context Protocol (MCP)** as a standardized mechanism for discovering and retrieving **Google A2A Agent Cards**. This enables dynamic agent interaction, particularly for planning and orchestration agents that utilize the A2A protocol.

---

## Background

### A2A Protocol

The Agent-to-Agent (A2A) protocol standardizes runtime communication between agents. It defines:

- **Agent Card**: A JSON schema describing an agent's identity, capabilities (actions/functions), and interaction endpoints.
- **Message Formats & Flows**: e.g., `ExecuteTask` for direct agent-to-agent interaction.

### Model Context Protocol (MCP)

MCP defines a standard way for applications (including AI models) to discover, access, and utilize contextual information—called "tools", "resources", etc.

---

## Core Proposal

Use MCP as a centralized, queryable registry for A2A Agent Cards.

### Storing Agent Cards

- Each A2A Agent Card is stored as a JSON file.
- The MCP server exposes these cards as retrievable resources.
- Backend can be a file system, database, or vector store.

### Discovering Agents via MCP

- Clients query `list_resources` on the MCP server to discover available agent cards.
- Optional: filter using metadata (e.g., tags, capabilities).

### Retrieving Agent Cards

- Use the resource URI to fetch full JSON agent card via MCP.

### Finding an Agent for a Task

- Requesting agents use MCP tools to find the most relevant agent for the job.

### Initiating A2A Communication

- Retrieved agent cards are passed to an `A2AClient`.
- Direct A2A protocol begins. MCP is not involved after discovery.

---

## Use Case: Orchestrated Task Execution

Enable workflows where specialized agents collaborate dynamically for complex task execution.

---

## Core Concepts

- **Orchestration**: Planner & Executor agents manage the task flow.
- **Specialization**: Task agents are domain experts.
- **Dynamic Discovery**: MCP registry enables plug-and-play agent architecture.
- **Standardized Communication**: A2A protocol ensures interoperability.

---

## Architectural Components

- **UI / Application Gateway**: User entry point.

- **Orchestrator Agent**:
  - Manages tasks based on Planner's plan.
  - Queries MCP, communicates with Task Agents using A2A.

- **Planner Agent**:
  - Converts raw user input into a structured task plan.

- **MCP Server**:
  - Hosts Agent Cards, acts as registry & discovery endpoint.

- **Task Agents**:
  - Independent A2A-compatible agents (e.g., Booking, Search).

- **A2A Protocol Layer**:
  - Handles inter-agent communication.

---

## Example Flow: Travel Agent

1. User requests a trip plan.
2. Orchestrator gets Planner Agent card via MCP.
3. Planner Agent breaks down the task.
4. For each step:
   - Orchestrator queries MCP to find a matching agent.
   - Sends task via A2A to that agent:
     - **Air Tickets**: queries SQLite via MCP tool.
     - **Car Rental**: similar DB query.
5. Results aggregated and sent to user.
6. On error (e.g., budget mismatch), replanning is triggered.

---

## Steps to Execute the Example

> ⚠️ Export your Google API key before proceeding:
```bash
export GOOGLE_API_KEY=your_key_here
```


## 1. Start MCP Server
```bash

cd a2a_mcp
uv venv
source .venv/bin/activate
uv run a2a-mcp --run mcp-server --transport sse
```

## 2. Start Orchestrator Agent
```bash


source .venv/bin/activate
uv run agents/ --agent-card agent_cards/orchestrator_agent.json --port 10101
or
PYTHONPATH=. uv run a2a_mcp/agents --agent-card a2a_mcp/agent_cards/orchestrator_agent.json --port 10101
```

## 3. Start Planner Agent
```bash

cd a2a_mcp
source .venv/bin/activate
uv run agents/ --agent-card agent_cards/planner_agent.json --port 10102
```

## 4. Start Airline Ticketing Agent
```bash

cd a2a_mcp
source .venv/bin/activate
uv run agents/ --agent-card agent_cards/air_ticketing_agent.json --port 10103
```

## 5. Start Car Rental Agent
```bash

cd a2a_mcp
source .venv/bin/activate
uv run agents/ --agent-card agent_cards/car_rental_agent.json --port 10105
```

## 6. Start CLI Terminal
```bash

cd cli
uv run . --agent http://localhost:10101
```

## File/Directory Descriptions
```
a2a_mcp/agent_cards/
```
- Stores JSON Agent Cards describing each agent's identity, capabilities, and endpoint.

- ```*_agent.json```: Each represents a specific agent (e.g., air_ticketing_agent.json).

```
a2a_mcp/
```
- Core Python source for this system.
```
agents/
```
- Implementation of A2A agents.

- ``` __main__.py```: Script to launch agents

- ``` adk_travel_agent.py```: ADK-based travel agent logic

- ``` langgraph_planner_agent.py```: LangGraph-based planner

- ``` orchestrator_agent.py```: Orchestration logic
```
common/
```
- ```agent_executor.py```: Task flow manager

- ```agent_runner.py```: Utility to launch agents

- ```base_agent.py```: Base class for agents

- ```prompts.py```: Prompt templates

- ```types.py```: Data models and types

- ```utils.py```: Misc utilities

- ```workflow.py```: Manages workflows
```
mcp/
```
- MCP logic

- ```client.py```: Client utility (test use)

- ```server.py```: MCP registry server
```
cli/
```
- CLI to interact with orchestrator
```
travel_agency.db
```
SQLite database with sample travel booking data


