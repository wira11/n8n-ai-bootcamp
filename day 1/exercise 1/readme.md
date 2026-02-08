# Build your first basic automation

## Workflow Overview
**Purpose:** Collect IT support requests, generate a unique ticket ID, and create a corresponding text file in GitHub.

**Trigger:** Form submission (`Form Trigger`)

**Nodes:**

1. On form submission
2. Edit Fields
3. Create a file

---

## Node 1: On form submission
**Type:** `Form Trigger (n8n-nodes-base.formTrigger)`

**Purpose:** Collect user input through an IT Service Request form.

| Parameter            | Value                                                              |
| -------------------- | ------------------------------------------------------------------ |
| **Form Title**       | IT Service Request                                                 |
| **Form Description** | Submit your issue here                                             |
| **Form Fields**      | - Issue description (textarea, required)<br>- Your Name (required) |
| **Submit Message**   | IT support will be in touch shortly!                               |

---

## Node 2: Edit Fields

**Type:** `Set (n8n-nodes-base.set)`

**Purpose:** Generate a unique ticket ID from the form submission timestamp.

| Parameter      | Value                                                                 |
| -------------- | --------------------------------------------------------------------- |
| **Field Name** | ID                                                                    |
| **Expression** | `={{ $json.submittedAt.toDateTime().ts.toString(36).toUpperCase() }}` |
| **Type**       | string                                                                |

---

## Node 3: Create a file

**Type:** `GitHub (n8n-nodes-base.github)`

**Purpose:** Save a new ticket file to a GitHub repository.

| Parameter          | Value                                                                                                                                                                                                           |
| ------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Authentication** | oAuth2                                                                                                                                                                                                          |
| **Owner**          | your Github username                                                                                                                                                                  |
| **Repository**     | your Github repo                                                                                                                                                                  |
| **File Path**      | `project 1/tickets/{{ $json.ID }}.txt`                                                                                                                                                                          |
| **File Content**   | Name: `{{ $('On form submission').item.json['Your Name'] }}`<br>Submitted: `{{ $('On form submission').item.json.submittedAt }}`<br>Description: `{{ $('On form submission').item.json['Issue description'] }}` |
| **Commit Message** | new ticket                                                                                                                                                                                                      |
