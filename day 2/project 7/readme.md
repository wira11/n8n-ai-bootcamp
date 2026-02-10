# P7 – Build a Simple Agent With Custom Tools

## Part 1: Simple Agent Workflow

**Purpose:** Create an interactive IT support chatbot agent that can chat with users, troubleshoot common problems, and manage support tickets (create, update, and retrieve).

### 1. Create New Workflow
- n8n home --> New workflow
- Name: `P7 - Simple AI Agent`

### 2. Add `AI Agent` node
- New node `AI Agent`
Parameters:
- **Source**: Chat Connected Chat Trigger Node
- **System Message**
```
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

**Settings**
- Retry On Fail: `On`
- Max. Tries: `3`

### 3. Add `Gemini Chat Model` node
- Pick `gemini-3-flash-preview` or `gemini-2.5-flash`

### 4. Add `Simple Memory` node
- Context Window Length: `10`

### 5. Add Tools

#### 5.1 Add `Github Get File` Tool
**Tool Description**
```
Load a ticket from GitHub
```

**Resource:**: `file`

**Operation**: `get`

**Owner**: `your-github-username`

**Repository**: `n8n-ai-bootcamp` (forked)

**File Path**: *Defined automatically by the model*

**File Path Descrription** 
```
Tickets are stored in the directory `day 2/project 7/tickets/`.
Example path: "day 2/project 7/tickets/MHGPYF9K.txt"
```

**Binary**: `False`

#### 5.2 Add `Github Edit File` Tool
**Tool Description**
```
Update a ticket on GitHub
```

**Resource**: `file`

**Operation**: `edit`

**Owner**: `your-github-username`

**Repository**: `n8n-ai-bootcamp` (forked)

**File Path**: *Defined automatically by the model*

**File Path Descrription** 
```
Tickets are stored in the directory `day 2/project 7/tickets/`.
Example path: "day 2/project 7/tickets/MHGPYF9K.txt"
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

**Commit Message**: *Defined automatically by the model*


### 6. Try it out!
- Try fetching the ticket ID `MLFNSMXT` using the Chatbot and update it.

---

## Part 2: Building a Custom Tool (`Create Ticket` Tool)

You will build a custom n8n workflow which can be added to the AI agent as a tool. In this example, the workflow will generate a new ticket ID and create a new ticket.

### 1. Create New Workflow
- n8n home --> New workflow

### 2. Add Trigger Node `When Executed by Another Workflow`
#### Input data mode: `Define using fields below`

| Name | Type   |
| ----- | ---------- |
| `User Name` | String |
| `Issue Description` | String |
| `Status` | String |
| `Prio` | String |

**Pin Test Data**:
- User Name: `Tobias`
- Issue Description: `My laptop fell down and is broken now`
- Status: `Open`
- Prio: `Urgent`

### 3. Add `Edit Fields` Node
**Manual Mapping**
 Name | Type | Value |
| --- | ---- | ----- |
| ID | String | `{{ $now.ts.toString(36).toUpperCase() }}` |


### 4. Add `Github Create File` Node

**Resource**: `File`

**Operation**: `Create`

**Repository Owner**: `your-github-username`

**Repository Name**: `n8n-ai-bootcamp` (forked)

**File Path**
```
day 2/project 7/tickets/{{ $json.ID }}.txt
```

**File Content**
```
User Name: {{ $('When Executed by Another Workflow').item.json['User Name'] }}

Submitted: {{ $now }}

Description:
{{ $('When Executed by Another Workflow').item.json['Issue Description'] }}

Status: {{ $('When Executed by Another Workflow').item.json.Status }}
Prio: {{ $('When Executed by Another Workflow').item.json.Prio }}
```

**Commit Message**
```
new ticket
```

### 5. Try it out!
- Run a test execution

### 6. Publish
- Everything work?
- Click **Publish** to publish your workflow

---

## Part 3: Add the Custom Tool to your AI Agent

Let's give your AI Agent access to the custom tool

### 1. Open AI Agent Workflow from Part 1
- n8n home --> Open `Simple AI Agent` Workflow

### 2. Add Custom Tool
- Click the `+` icon under AI Agent Tool
- Select `Call n8n Workflow Tool`
  
#### Custom Tool Settings
**Description**
```
Call this tool to create a new ticket. Status can be "Open" or "Closed". Prio can be "Urgent" or "Not Urgent".
```

**Source**: `Database`

**Workflow**: `From List` --> `Select the ticket workflow you just created`

#### Workflow Inputs
- **User Name**: *Defined automatically by the model*
- **Description**
```
Name of the user. Required to follow up later on.
```

- **Issue Description**: *Defined automatically by the model*
- **Description**
```
Description of the problem. Required.
```

- **Status**: *Defined automatically by the model*
- **Description**
```
Current status of the ticket. Required. Allowed values: "Open", "Closed"
```

- **Prio**: *Defined automatically by the model*
- **Description**
```
Time criticality of the ticket. Required. Allowed values: "Urgent", "Not Urgent"
```

### 3. Try it out!
- Create a new ticket through your agent
- Check the ticket in Github
