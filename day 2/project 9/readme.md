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

| Parameter      | Value  |
| -------------- | ------ |
| Authentication | `None` |

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
| File Path        | Defined automatically by the model             |
| File Path Descr. | ```Tickets are stored in the directory `day 2/project 9/tickets/` Example path: "day 2/project 9/tickets/MHGPYF9K.txt"```|

---

### Update ticket

**Type:** GitHub Tool

#### Parameters
**Tool Description**
`Update a ticket on GitHub`

**Resource**
`file`

**Operation**
`edit`

**Owner**
`your-github-username`

**Repository**
`n8n-ai-bootcamp` (forked)

**File Path**
*Defined automatically by the model*

**File Path Descrription** 
```
Tickets are stored in the directory `day 2/project 9/tickets/`.
Example path: "day 2/project 9/tickets/MHGPYF9K.txt"
```

**File Content**
*Defined automatically by the model*

**File Content Description**
```
Required fields: "User Name", "Submitted", "Description", "Status", "Prio". 
Optional fields: "Activity Log"

## Example Ticket:
User Name: Tobias

Submitted: 2025-11-01T23:15:54.110+01:00

Description:
User Tobias cannot sign into Windows on company laptop. Device is domain-joined. User reports no access to desktop and requests immediate assistance. Unable to authenticate at login screen; needs urgent account or device support.

Status: Closed
Prio: Urgent

## Optional – Activity Log:
When making updates, append an activity log like so:

Activity Log:
2025-11-01T23:15:54.110+01:00 - Ticket created (Urgent). Escalated to human IT technician for account/machine support.
2025-11-01T23:40:00.000+01:00 - User reported: "I can log in now! it works". User regained access to Windows. No further action required at this time. Ticket closed.
```

**Commit Message**
*Defined automatically by the model*

---

### Create Ticket
**Type:** Tool Workflow

#### Description
```
Call this tool to create a new ticket. Status can be "Open" or "Closed". Prio can be "Urgent" or "Not Urgent".
```
#### Source
```
Database
```

#### Workflow
```
From list: P9 - Create Ticket Tool
```

#### Workflow Inputs

##### User Name
*Defined automatically by the model*
**Description**
```
Name of the user. Required to follow up later on.
```

##### Issue Description
*Defined automatically by the model*
**Description**
```
Description of the problem. Required.
```

##### Status
*Defined automatically by the model*
**Description**
```
Current status of the ticket. Required. Allowed values: "Open", "Closed"
```

##### Prio
*Defined automatically by the model*
**Description**
```
Time criticality of the ticket. Required. Allowed values: "Urgent", "Not Urgent"
```
---

# 2. P9 – Create Ticket Tool

## Purpose

Reusable workflow that **creates a new ticket file in GitHub** when called by another workflow or MCP.

This is the **actual ticket creation logic**.

---

## Nodes

### When Executed by Another Workflow

| Parameter | Type   |
| ----- | ---------- |
| User Name | String` |
| Issue Description | String` |
| Status | String` |
| Prio | String` |

---

### Edit Fields
Creates ticket ID.

| Field | Expression                                  |
| ----- | ------------------------------------------- |
| ID    | `={{ $now.ts.toString(36).toUpperCase() }}` |

---

### Create a file
**Type:** GitHub
Creates the ticket file on Github

| Parameter      | Value                                                    |
| -------------- | -------------------------------------------------------- |
| Resource       | File                                                     |
| Operation      | Create                                                   |
| Owner          | `tobiaszwingmann-demo`                                   |
| Repository     | `n8n-ai-bootcamp`                                        |

#### File Path
```
day 2/project 9/tickets/{{ $json.ID }}.txt`
```

#### File Content
```
User Name: {{ $('When Executed by Another Workflow').item.json['User Name'] }}

Submitted: {{ $now }}

Description:
{{ $('When Executed by Another Workflow').item.json['Issue Description'] }}

Status: {{ $('When Executed by Another Workflow').item.json.Status }}
Prio: {{ $('When Executed by Another Workflow').item.json.Prio }}
```

#### Commit Message 
```
new ticket
```

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

#### System Message
```
### **Role**

You are an **IT Support Chatbot** for a corporate helpdesk.
You assist employees with **NovaCRM**, **Windows** and **Microsoft Office** issues, and manage **support tickets**.

You can:

* Troubleshoot common technical issues.
* Answer questions about the internal NovaCRM tool-
* Decide when to **escalate** to a human IT technician.
* **Create**, **update**, and **retrieve** tickets using your tools.

---

### **Conversation Flow**

1. **Start** by asking whether the user already has a ticket ID.
   * If yes → retrieve the ticket.
   * If no → ask for the user’s **name** and **problem description**.

2. **Classify the issue:**

   * **Automated** → Routine or self-service problem (Windows, Office, login, Teams, etc.).
   * **Route to IT** → Contains keywords like *hardware, repair, network down, outage, admin rights, security, virus, data loss, urgent*, or when unsure.

3. **Respond:**

   * If **Automated**, give clear, numbered troubleshooting steps.
   * If **Route to IT**, say:

     > “This looks like it needs a human IT technician. I’ll escalate it for you.”
     > Then **create a ticket** and share the ID.

4. **Ticket Management:**

   * **Create** → when logging or escalating new issues.
   * **Update** → when adding progress or notes.
   * **Retrieve** → when a ticket ID is provided.

---

### **Details**
* Whether a ticket is urgent or not is classified by *YOU*, not the user

### **Response Style**

* Short, direct, step-by-step answers.
* Quote commands exactly (e.g., `excel /safe`).
* Confirm after each step.
* If unable to help, escalate politely.

---

### **Example**

**User:** “I can’t open Excel files.”
**Chatbot:**

> Do you already have a ticket ID?
>
> Thanks, [Name]. Try this:
>
> 1. Close all Excel windows.
> 2. Press **Windows + R**, type `excel /safe`, and press **Enter`.
> 3. Try reopening the file.
>
> Did that work? If not, I can create a ticket for escalation.
```

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

| Parameter    | Value                     |                                   
| ------------ | ------------------------- |
| Endpoint URL | `your-mcp-production-url` |

Connects to the **MCP Server workflow**.

---

### Search CRM

**Type:** Tool Workflow

Calls document search workflow.

| Parameter    | Value                     |                                   
| ------------ | ------------------------- |
| Description | `Call this tool to search the user manual of the Nova CRM software.` |
| Source | `Database` |
| Workflow | From list: `P8 - Document Search` |

#### Workflow Inputs
**query**
*Defined automatically by the model*

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

