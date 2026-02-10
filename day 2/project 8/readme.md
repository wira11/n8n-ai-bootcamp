# P8 – Advanced Agent with RAG
In this project, we'll enhance our AI Agent with access to internal data sources.

## Part 1: Document ingestion

**Purpose:** Load internal data sources into a vector data store so the AI Agent can perform a semantic search and retrieve relevant pieces of information during the conversation.

### 1. Create New Workflow
- n8n home --> New workflow
- Name the workflow `P8 - Document Ingestion`

### 2. Add `Manual Trigger` node

### 3. Add `Github List File` Node

**Resource**: `file`

**Operation**: `list`

**Owner**: `your-github-username`

**Repository**: `n8n-ai-bootcamp` (forked)

**Path** 
```
day 2/project 8/documents/
```

### 3. Add `Github Get File` Node

**Resource**: `file`

**Operation**: `Get`

**Owner**: `your-github-username`

**Repository**: `n8n-ai-bootcamp` (forked)

**File Path** 
```
{{ $json.path }}
```

**As Binary Property**: `True`

**Put Output File in Field**: `data`

### 4. Add `Pinecone Vector Stors` Node

#### 4.1 Add Pinecone credential
- Go to the [Pinecone website](https://www.pinecone.io/)
- Click **Start building** / **Sign up for free**
- Enter your Email --> Confirm code
- Copy the **API Key**
- Paste the API Key into **n8n Pinecone Credentials**

#### 4.2 Create Pinecone index
- In Pinecone, click **Create index**
- Index name: `documents`
- Choose **Manual Configuration**
- For **Gemini Embeddings**, choose:
  - Vector type: `Dense`
  - Dimension size: `3072`
  - Metric: `cosine`
- Click **Create Index**

### 5. Configure `Pinecone Vector Stors` Node
- **Operation Mode**: `Insert Documents
- **Pinecone Index**: `documents`
- **Embedding Batch Size**: `200`
Options
- **Pinecone Namespace**: `it`
- **Clear Namespace**: `true`

#### 5.1 Subnodes
Model
- **Embeddings**: Add `Embeddings Google Gemini` node
- **Model**: `gemini-embedding-001`

Data Loader
- Add **Default Data Loader** Node
- **Type of Data**: `Binary`
- **Mode**: `Load All Input Data`
- **Data Format**: `Automatically Detect by Mime Type`
- **Text Splitting**: `Simple`
Options:
- **Split Pages in PDF**: `True`
- **Metadata**
   - Name: `file_name`
   - Value: `{{ $json.name }}`

### 5. Try it out!
- Run the workflow
- Check the embedded documents in Pinecone
---

## Part 2: Building a Custom Tool (`Create Ticket` Tool)

You will build a custom n8n workflow which can be added to the AI agent as a tool. In this example, the workflow will generate a new ticket ID and create a new ticket.

### 1. Create New Workflow
- n8n home --> New workflow

### 2. Add Trigger Node `When Executed by Another Workflow`
#### Input data mode: `Define using fields below`

| Parameter | Type   |
| ----- | ---------- |
| User Name | String` |
| Issue Description | String` |
| Status | String` |
| Prio | String` |

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





# P8 – Advanced Agent

This project combines:

- AI support chatbot
- Knowledge base search (RAG)
- Ticket management
- GitHub storage

It consists of four workflows that work together.

## Part 1: Document Ingestion

**Purpose:**
Convert all CRM documentation into vector embeddings for searchable access by the agent.

**Trigger:** Manual (`When clicking 'Test workflow'`)

**Nodes:**

1. When clicking ‘Test workflow’
2. List files
3. Get a file
4. Pinecone Vector Store
   4.1 Embeddings Gemini
   4.2 Default Data Loader
      4.2.1 Recursive Character Text Splitter
8. Pinecone Vector Store

---

#### Node 1: When clicking ‘Test workflow’

**Trigger:** Manual execution of the ingestion process.

---

#### Node 2: List files

| Parameter     | Value                        |
| ------------- | ---------------------------- |
| **Resource**  | file                   	   |
| **Operation** | list                   	   |
| **File Path** | `day 2/project 8/documents/` |

---

#### Node 3: Get a file

| Parameter     | Value               |
| ------------- | ------------------- |
| **Resource**  | file                |
| **Operation** | get                 |
| **File Path** | `={{ $json.path }}` |

---

#### Node 4: Pinecone Vector Store

| Parameter           | Value   |
| ------------------- | ------- |
| **Operation Mode**  | `insert`  |
| **Index**           | `documents` |
| **Namespace**       | `it`      |
| **Clear Namespace** | `false`   |

---

#### Node 4.1: Embeddings Gemini

| Parameter      | Value                    |
| -------------- | ------------------------ |
| **Model**      | `gemini-embedding-001` |
| **Dimensions** | 3072                     |

---

#### Node 4.2: Default Data Loader

| Parameter       | Value                           |
| --------------- | ------------------------------- |
| **Data Type**   | binary                          |
| **Split Pages** | true                            |
| **Metadata**    | file_name = `={{ $json.name }}` |

---

#### Node 4.2.1: Recursive Character Text Splitter

| Parameter         | Value               |
| ----------------- | ------------------- |
| **Splitter Type** | Recursive Character |
| **Options**       | default             |

---

## Part 2: Document Search

**Purpose**: Perform a semantic document search against the vector database and prepare structured results for the chatbot agent.

**Trigger:** Executed by another workflow (`Execute Workflow Trigger`)

**Nodes:**

1. When Executed by Another Workflow

2. Pinecone Vector Store

     2.1 Embeddings (Gemini)

3. Relevant Data for Chatbot Response

4. Aggregate

5. Create Response

---

#### Node 1: When Executed by Another Workflow

| Parameter           | Value   |
| ------------------- | ------- |
| **Workflow Inputs** | `query` |

**Sample Input:**

| Field     | Value                                             |
| --------- | ------------------------------------------------- |
| **query** | Email templates aren’t displaying dynamic fields. |

---

#### Node 2: Pinecone Vector Store

| Parameter          | Value                |
| ------------------ | -------------------- |
| **Operation Mode** | load                 |
| **Index**          | support              |
| **Namespace**      | it                   |
| **Prompt**         | `={{ $json.query }}` |

##### Subnode 2.1: Embeddings (Gemini)

| Parameter      | Value                    |
| -------------- | ------------------------ |
| **Model**      | `gemini-embedding-001` |
| **Dimensions** | 3,072                     |

---

#### Node 3: Relevant Data for Chatbot Response

| Parameter       | Value                                              |
| --------------- | -------------------------------------------------- |
| **Assignments** | Extract fields from retrieved documents            |
| **Content**     | `={{ $json.document.pageContent }}`                |
| **Source File** | `={{ $json.document.metadata.file_name }}`         |
| **Page**        | `={{ $json.document.metadata['loc.pageNumber'] }}` |

---

#### Node 4: Aggregate

| Parameter          | Value                   |
| ------------------ | ----------------------- |
| **Aggregate Mode** | Aggregate All Item Data |

---

#### Node 5: Create Response

| Parameter      | Value                         |
| -------------- | ----------------------------- |
| **Field Name** | `response`                    |
| **Value**      | `={{ $json.toJsonString() }}` |

---

## Part 3: Create Ticket Tool (Updated File Path)

**Trigger:** Executed by another workflow (`Execute Workflow Trigger`)

**Nodes:**
1. When Executed by Another Workflow
2. Edit Fields
3. Create a file

---

#### Node 1: When Executed by Another Workflow

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

#### Node 2: Edit Fields

| Parameter      | Value                                       |
| -------------- | ------------------------------------------- |
| **Field Name** | ID                                          |
| **Expression** | `={{ $now.ts.toString(36).toUpperCase() }}` |
| **Type**       | string                                      |

---

#### Node 3: Create a file

| Parameter          | Value                                          |
| ------------------ | ---------------------------------------------- |
| **Resource**       | File                                           |
| **Operation**      | Create                                         |
| **Repository**     | n8n-ai-bootcamp								  |
| **File Path**      | `day 2/project 8/tickets/{{ $json.ID }}.txt`   |

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

## Part 4: Update the Agent

**Start by copying the workflow from “P4 – Simple Agent.”**

We’ll extend it with new tools that allow the agent to search CRM manuals and knowledge documents, alongside its existing ticket management capabilities.

**Trigger:** Chat message (`Chat Trigger`)

**Nodes:**

1. When chat message received
2. AI Agent

   2.1 Gemini Chat Model

   2.2 Simple Memory

   2.3 Get ticket

   2.4 Update ticket

   2.5 Create Ticket Tool

   **2.6 Document Search**

---

### Node 1: When chat message received
(No changes)

---

### Node 2: AI Agent
(No changes)

---

#### Subnode 2.1: Gemini Chat Model
(No changes)

---

#### Subnode 2.2: Simple Memory
(No changes)

---

#### Subnode 2.3: Get ticket

| Parameter            | Value                                    |
| -------------------- | ---------------------------------------- |
| **Operation**        | get                                      |
| **Resource**         | file                                     |
| **Authentication**   | oAuth2                                   |
| **File Path**        | Defined automatically by the model       |
| **Tool Description** | Load a ticket from GitHub for reference. |

**File Path Description**

```markdown
Tickets are stored in the directory `day 2/project 8/tickets/`.
Example path: "day 2/project 8/tickets/MHGPYF9K.txt"
```

---

#### Subnode 2.4: Update ticket

| Parameter            | Value                              |
| -------------------- | ---------------------------------- |
| **Operation**        | edit                               |
| **Resource**         | file                               |
| **Authentication**   | oAuth2                             |
| **File Path**        | Defined automatically by the model |
| **File Content**     | Defined automatically by the model |
| **Commit Message**   | Defined automatically by the model |
| **Tool Description** | Update a ticket on GitHub.         |

**File Path Description**

```markdown
Tickets are stored in the directory `day 2/project 8/tickets/`.
Example path: "day 2/project 8/tickets/MHGPYF9K.txt"
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

#### Subnode 2.5: Create Ticket Tool

| Parameter            | Value                          |
| -------------------- | ------------------------------ |
| **Workflow**         | P8 – Create Ticket Tool        |

**Tool Description** 
```markdown
Call this tool to create a new ticket. Status can be "Open" or "Closed". Prio can be "Urgent" or "Not Urgent".
```

**Source**: Database

**Workflow** P8 Create Ticket Tool

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

---

#### Subnode 2.6: Document Search

| Parameter            | Value                                                              |
| -------------------- | ------------------------------------------------------------------ |
| **Workflow**         | P8 – Document Search                                               |
| **Tool Description** | Call this tool to search the user manual of the Nova CRM software. |

**Input Field**
**Query: Defined automatically by the model**
```

---

### Sample Queries

Use the following to test the agent’s capabilities:

* “How do I reset a CRM user’s password?”
* “Email templates aren’t displaying dynamic fields.”
* “Create a ticket for this CRM sync error.”
* “I already have a ticket: MHGPYF9K.”
* “Update my CRM issue ticket.”



