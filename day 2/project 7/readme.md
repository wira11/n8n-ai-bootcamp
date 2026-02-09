# P7 – Simple Agent

## Workflow (Part 1)

**Purpose:** Create an interactive IT support chatbot agent that can chat with users, troubleshoot common problems, and manage support tickets (create, update, and retrieve).

**Trigger:** Chat message (`Chat Trigger`)

**Nodes:**

1. When chat message received
2. AI Agent

   2.1 Gemini Chat Model

   2.2 Simple Memory

   2.3 Get ticket

   2.4 Update ticket

---

#### Node 1: When chat message received

**Type:** `Chat Trigger (@n8n/n8n-nodes-langchain.chatTrigger)`

| Parameter             | Value                                                           |
| --------------------- | --------------------------------------------------------------- |
| **Public**            | true                                                            |
| **Authentication**    | n8nUserAuth                                                     |
| **Initial Message**   | Welcome to our automated IT support. How can we help you today? |
| **Input Placeholder** | Type your name..                                                |

---

#### Node 2: AI Agent

**Type:** `Agent (@n8n/n8n-nodes-langchain.agent)`

| Parameter         | Value                                                       |
| ----------------- | ----------------------------------------------------------- |
| **Prompt Type**   | define                                                      |
| **Text Input**    | `{{ $('When chat message received').item.json.chatInput }}` |
| **System Prompt** |                                                             |

```markdown
### **Role**

You are an **IT Support Chatbot** for a corporate helpdesk.
You assist employees with **Windows** and **Microsoft Office** issues, and manage **support tickets**.

You can:

* Troubleshoot common technical issues.
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

#### Subnode 2.1: Gemini Chat Model

| Parameter            | Value          |
| -------------------- | -------------- |
| **Model**            | gemini-3-flash |
| **Reasoning Effort** | low            |
| **Credential**       | Gemini account |

---

#### Subnode 2.2: Simple Memory

**Purpose:** Maintain short-term chat history for contextual responses.

| Parameter                 | Value |
| ------------------------- | ----- |
| **Context Window Length** | 10    |

---

#### Subnode 2.3: Get ticket

| Parameter            | Value                                          |
| -------------------- | ---------------------------------------------- |
| **Operation**        | get                                            |
| **Resource**         | file                                           |
| **Authentication**   | oAuth2                                         |
| **Owner**            | tobiaszwingmann                                |
| **Repository**       | demo-building-ai-workflows-and-agents-with-n8n |
| **File Path**        | **Defined automatically by the model**.        |
| **Tool Description** | Load a ticket from GitHub for reference.       |

**File Path Description** 
```markdown 
Tickets are stored in the directory `day 2/project 7/tickets/`.
Example path: "day 2/project 7/tickets/MHGPYF9K.txt"`
``` 

---

#### Subnode 2.4: Update ticket

**Type:** `GitHub Tool (n8n-nodes-base.githubTool)`
**Purpose:** Update an existing ticket on GitHub.

| Parameter            | Value                                               |
| -------------------- | --------------------------------------------------- |
| **Operation**        | edit                                                |
| **Resource**         | file                                                |
| **Authentication**   | oAuth2                                              |
| **Owner**            | tobiaszwingmann                                     |
| **Repository**       | demo-building-ai-workflows-and-agents-with-n8n      |
| **File Path**        | **Defined automatically by the model**              |
| **File Content**     | **Defined automatically by the model**              |
| **Commit Message**   | **Defined automatically by the model**              |
| **Tool Description** | Update a ticket on GitHub.                          |

**File Path Description** 
```markdown
Tickets are stored in the directory `day 2/project 7/tickets/`.
Example path: "day 2/project 7/tickets/MHGPYF9K.txt"
```
**File Content Description** 
```markdown
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

---

### Sample Queries

Use the following queries to test the chatbot’s capabilities:

* “I can’t log in to my laptop”
* “I already have a ticket: MHGUCSN1”
* “My computer is broken”
* “Can you update my ticket?”

---

## Workflow 2: Create Ticket Tool
**This workflow is triggered when executed by another workflow (e.g., the Simple Agent).**

**Tip:** Copy the "Create Ticket" workflow from P3

**Trigger:** Executed by another workflow (`Execute Workflow Trigger`)

**Nodes:**
1. When Executed by Another Workflow
2. Edit Fields
3. Create a file

---

## Node 1: When Executed by Another Workflow

| Parameter           | Value                                                    |
| ------------------- | -------------------------------------------------------- |
| **Workflow Inputs** | - User Name<br>- Issue Description<br>- Status<br>- Prio |

**Sample Input:**

| Field                 | Value                                 |
| --------------------- | ------------------------------------- |
| **User Name**         | Tobias                                |
| **Issue Description** | My laptop fell down and is broken now |
| **Status**            | Open                                  |
| **Prio**              | Urgent                                |

---

## Node 2: Edit Fields

| Parameter      | Value                                       |
| -------------- | ------------------------------------------- |
| **Field Name** | ID                                          |
| **Expression** | `={{ $now.ts.toString(36).toUpperCase() }}` |
| **Type**       | string                                      |

---

## Node 3: Create a file

| Parameter          | Value                                          |
| ------------------ | ---------------------------------------------- |
| **Resource**       | File                                           |
| **Operation**      | Create                                         |
| **Repository**     | demo-building-ai-workflows-and-agents-with-n8n |
| **File Path**      | `day 2/project 7/tickets/{{ $json.ID }}.txt`         |

**File Content**:

```markdown
User Name: {{ $('When Executed by Another Workflow').item.json['User Name'] }}

Submitted: {{ $now }}

Description:
{{ $('When Executed by Another Workflow').item.json['Issue Description'] }}

Status: {{ $('When Executed by Another Workflow').item.json.Status }}
Prio: {{ $('When Executed by Another Workflow').item.json.Prio }}
```

**Commit Message**: `new ticket`

---

## Workflow (Part 2)

Add the **Create Ticket Tool** to our agent.

**Tool Description** 
```markdown
Call this tool to create a new ticket. Status can be "Open" or "Closed". Prio can be "Urgent" or "Not Urgent".
```

**Source**: Database

**Workflow** Create Ticket Tool

**User Name: Defined by the model**
```
Name of the user. Required to follow up later on.
```

**Issue: Defined by the model**
```
Description of the problem. Required.
```

**Status**: **Defined by the model**
```
Current status of the ticket. Required. Allowed values: "Open", "Closed"
```

**Prio: Defined by the model**
```
Time criticality of the ticket. Required. Allowed values: "Urgent", "Not Urgent"
```
