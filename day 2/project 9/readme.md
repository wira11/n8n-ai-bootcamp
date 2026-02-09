# P9 – AI Agent with MCP

This project contains **three workflows** that together demonstrate how to expose n8n functionality as **MCP tools** and use them from an AI agent.

* MCP Server → exposes ticket operations as tools
* Create Ticket Tool → creates GitHub tickets
* MCP Client → AI chatbot that uses those tools

Sources:   

---

# 1. P9 – MCP Server

## Purpose

Expose ticket management functionality (Create, Read, Update) as **MCP tools** that AI agents can call.

This workflow acts as the **tool provider**.

---

## Nodes

### Ticket MCP Server

**Type:** MCP Trigger

| Parameter | Value                                  |
| --------- | -------------------------------------- |
| Path      | `edd5ffc1-e861-4354-85e7-e95ce9195c93` |

---

### Get ticket

**Type:** GitHub Tool

| Parameter        | Value                                          |
| ---------------- | ---------------------------------------------- |
| Tool Description | Load a ticket from GitHub                      |
| Resource         | file                                           |
| Operation        | get                                            |
| Owner            | `tobiaszwingmann-demo`                         |
| Repository       | `n8n-ai-bootcamp`                              |
| File Path        | AI-provided path in `day 2/project 9/tickets/` |

---

### Update ticket

**Type:** GitHub Tool

| Parameter        | Value                      |
| ---------------- | -------------------------- |
| Tool Description | Update a ticket on GitHub  |
| Resource         | file                       |
| Operation        | edit                       |
| Owner            | `tobiaszwingmann-demo`     |
| Repository       | `n8n-ai-bootcamp`          |
| File Path        | AI-provided path           |
| File Content     | AI-generated ticket format |
| Commit Message   | AI-generated               |

---

### Create Ticket

**Type:** Tool Workflow

Calls: **P9 – Create Ticket Tool**

Inputs:

| Field             | Description         |
| ----------------- | ------------------- |
| User Name         | Name of user        |
| Issue Description | Ticket description  |
| Status            | Open / Closed       |
| Prio              | Urgent / Not Urgent |

---

# 2. P9 – Create Ticket Tool

## Purpose

Reusable workflow that **creates a new ticket file in GitHub** when called by another workflow or MCP.

This is the **actual ticket creation logic**.

---

## Nodes

### When Executed by Another Workflow

**Type:** Execute Workflow Trigger

Inputs:

* User Name
* Issue Description
* Status
* Prio

---

### Edit Fields

**Type:** Set

Creates ticket ID.

| Field | Expression                                  |
| ----- | ------------------------------------------- |
| ID    | `={{ $now.ts.toString(36).toUpperCase() }}` |

---

### Create a file

**Type:** GitHub

| Parameter      | Value                                                    |
| -------------- | -------------------------------------------------------- |
| Resource       | file                                                     |
| Owner          | `tobiaszwingmann-demo`                                   |
| Repository     | `n8n-ai-bootcamp`                                        |
| File Path      | `day 2/project 9/tickets/{{ $json.ID }}.txt`             |
| File Content   | Includes User Name, Submitted, Description, Status, Prio |
| Commit Message | `new ticket`                                             |

---

# 3. P9 – MCP Client

## Purpose

AI-powered IT support chatbot that:

* Troubleshoots issues
* Searches documentation
* Creates / retrieves / updates tickets via MCP tools

This workflow is the **end-user interface**.

---

## Nodes

### When chat message received

**Type:** Chat Trigger

| Parameter       | Value                           |
| --------------- | ------------------------------- |
| Public          | true                            |
| Initial Message | Welcome to automated IT support |

---

### AI Agent

**Type:** Agent

Uses system prompt defining:

* IT support behavior
* When to escalate
* How to manage tickets

---

### Google Gemini Chat Model

**Type:** Gemini Chat Model

| Parameter | Value                           |
| --------- | ------------------------------- |
| Model     | `models/gemini-3-flash-preview` |

---

### Memory

**Type:** Memory Buffer Window

| Parameter             | Value |
| --------------------- | ----- |
| Context Window Length | 15    |

---

### MCP Client

**Type:** MCP Client Tool

| Parameter    | Value                                                            |
| ------------ | ---------------------------------------------------------------- |
| Endpoint URL | `http://localhost:5678/mcp/edd5ffc1-e861-4354-85e7-e95ce9195c93` |

Connects to the **MCP Server workflow**.

---

### Search CRM

**Type:** Tool Workflow

Calls document search workflow.

---

# How Everything Works Together

1. User chats with MCP Client
2. AI Agent decides to:

   * Troubleshoot
   * Search documentation
   * Manage tickets
3. Ticket actions call MCP Server
4. MCP Server calls:

   * GitHub tools
   * Create Ticket Tool workflow
5. Ticket stored or updated in GitHub

