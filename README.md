# n8n AI-Powered Customer Feedback Automation

## 1. Project Overview

This n8n workflow automates the process of collecting, understanding, categorizing, storing, and responding to customer feedback.

### Main workflow

**Customer → Feedback Form → n8n Trigger → AI Agent → Merge → Switch → Category-specific action → Airtable → Slack → Gmail**

The workflow classifies customer feedback into three categories:

1. **Complaint**
2. **Compliment**
3. **Feature Addition Request**

The AI Agent uses Google Gemini to determine the category. The Switch node then sends the data to the appropriate branch.

---

# 2. What is n8n?

**n8n** is a workflow automation platform.

It allows us to connect different applications and services together and automate tasks between them.

For example:

**Form submission**
↓
**AI analyzes feedback**
↓
**Database stores feedback**
↓
**Slack receives notification**
↓
**Customer receives email**

Instead of performing all these tasks manually, n8n executes them automatically.

---

# 3. What is a Node?

A **node** is an individual step in an n8n workflow.

Each node performs a particular task.

For example:

* Form Trigger → receives data
* AI Agent → analyzes data
* Merge → combines data
* Switch → decides which path to follow
* Airtable → stores data
* Slack → sends notification
* Gmail → sends email

A workflow is basically a collection of connected nodes.

---

# 4. Complete Workflow Architecture

The workflow can be understood like this:

```text
                CUSTOMER
                   │
                   ▼
          ┌─────────────────┐
          │  Feedback Form  │
          └────────┬────────┘
                   │
                   ▼
          ┌─────────────────┐
          │  Form Trigger   │
          └────────┬────────┘
                   │
          ┌────────┴──────────┐
          │                   │
          ▼                   ▼
   ┌─────────────┐      ┌─────────────┐
   │  AI Agent   │      │ Form Data   │
   └──────┬──────┘      └──────┬──────┘
          │                    │
          │   Google Gemini    │
          └────────┬───────────┘
                   ▼
             ┌──────────┐
             │  Merge   │
             └────┬─────┘
                  ▼
             ┌──────────┐
             │  Switch  │
             └────┬─────┘
                  │
       ┌──────────┼───────────┐
       ▼          ▼           ▼
  Complaint   Compliment   Feature Request
       │          │           │
       ▼          ▼           ▼
   Airtable    Airtable    Airtable
       │                      │
       ▼                      ▼
     Slack                  Slack
       │                      │
       ▼                      ▼
     Gmail                  Gmail
```

The exact workflow configuration shows the three classification outputs and their corresponding Switch branches.

---

# 5. Node 1 — On Form Submission

## What is it?

This is the **Trigger Node**.

A trigger node is the node that starts a workflow when a particular event occurs.

In this project, the event is:

> A customer submits the feedback form.

The workflow uses an n8n Form Trigger.

---

## What does the form collect?

The form contains four fields:

* Full Name
* Email
* Mobile No.
* Feedback

The form configuration defines these fields and their placeholders.

For example, a customer could submit:

```text
Full Name:
Prasen Nimje

Email:
example@gmail.com

Mobile No.:
9876543210

Feedback:
The application is very slow when I try to open it.
```

Once the customer presses **Submit**, the trigger fires.

---

# 6. What does "Trigger" mean?

This is very important in n8n.

A **trigger** is an event that tells n8n:

> "Start executing this workflow now."

Examples of triggers in n8n can include:

* Form submission
* Webhook request
* New Gmail email
* Scheduled time
* New database record
* New Slack event

In this project:

**Trigger = Customer submits the feedback form**

Therefore:

```text
Customer clicks Submit
        ↓
Form Trigger activates
        ↓
Workflow execution starts
```

---

# 7. Data Received by the Trigger

After submission, n8n receives the form data.

Conceptually, the data looks like:

```json
{
  "Full Name": "Prasen Nimje",
  "Email": "example@gmail.com",
  "Mobile No.": "9876543210",
  "Feedback": "The application is very slow."
}
```

This data becomes available to the next nodes.

---

# 8. Node 2 — AI Agent

The next important node is the **AI Agent**.

Its purpose is to understand the customer's feedback and classify it.

The workflow explicitly tells the AI Agent to determine whether the feedback is:

* Complaint
* Compliment
* Feature Addition Request

and return only one of those three values.

---

# 9. How the AI Agent Works

The AI Agent receives the customer's feedback.

For example:

```text
Feedback:
"The app keeps crashing whenever I try to upload a file."
```

The AI analyzes the meaning.

It understands that the customer is reporting a problem.

Therefore:

```text
AI Output:
Complaint
```

Another example:

```text
"The application is very easy to use. Great work!"
```

AI output:

```text
Compliment
```

Another:

```text
"Please add dark mode to the application."
```

AI output:

```text
Feature Addition Request
```

---

# 10. AI Agent Prompt

The workflow uses a prompt similar to:

```text
Your role is to determine if the feedback filled by
the customers is the complaint or an Compliment or
an Feature adding request.

The Feedback is {{ $json.Feedback }}

Your response should be only one from below:

Complaint
Compliment
Feature Addition Request
```

The important part here is:

```text
{{ $json.Feedback }}
```

This is an n8n expression.

It dynamically takes the customer's feedback from the incoming data.

---

# 11. What is $json?

In n8n, `$json` refers to the JSON data currently available to the node.

For example:

```json
{
  "Full Name": "Prasen Nimje",
  "Email": "example@gmail.com",
  "Feedback": "The application is slow."
}
```

Then:

```text
$json.Feedback
```

means:

```text
"The application is slow."
```

Similarly:

```text
$json.Email
```

returns:

```text
example@gmail.com
```

This concept is extremely important when working with n8n.

---

# 12. Node 3 — Google Gemini Chat Model

The AI Agent needs an actual AI model to generate its response.

For this workflow, the AI Agent is connected to:

**Google Gemini Chat Model**

The JSON configuration shows the Gemini Chat Model connected to the AI Agent as its language model.

So the relationship is:

```text
AI Agent
    │
    ▼
Google Gemini
    │
    ▼
Classification
```

The AI Agent defines **what task needs to be performed**, while Gemini provides the language-model capability used to perform that task.

---

# 13. Why Use Gemini?

Gemini is useful here because the customer can write feedback in natural language.

For example:

```text
"Honestly, I really like the application but it would
be much better if you could add dark mode."
```

A simple keyword-based system might have difficulty determining the category.

An AI model can understand that the main request is:

**Feature Addition Request**

---

# 14. Node 4 — Merge

After the AI Agent produces the classification, the workflow uses a **Merge** node.

The Merge node is configured to combine data based on position.

This is important because we need both:

### Original customer information

```text
Full Name
Email
Mobile No.
Feedback
```

AND

### AI classification

```text
Complaint
```

The Merge node combines these pieces of information.

---

# 15. Why is Merge Necessary?

Imagine the original form data is:

```json
{
  "Full Name": "Prasen Nimje",
  "Email": "example@gmail.com",
  "Feedback": "The app is crashing."
}
```

The AI Agent produces:

```json
{
  "output": "Complaint"
}
```

We need both.

So after merging, conceptually we want:

```json
{
  "Full Name": "Prasen Nimje",
  "Email": "example@gmail.com",
  "Feedback": "The app is crashing.",
  "output": "Complaint"
}
```

Now the Switch node knows:

1. Who submitted the feedback
2. Their email
3. Their feedback
4. What category the AI assigned

---

# 16. Node 5 — Switch

The **Switch** node is responsible for routing the data.

Think of it as a traffic controller.

It asks:

> "What did the AI classify this feedback as?"

The workflow has three rules:

```text
If output = Complaint
        ↓
Complaint branch

If output = Compliment
        ↓
Compliment branch

If output = Feature Addition Request
        ↓
Feature Request branch
```

These three rules are explicitly configured in the Switch node.

---

# 17. Why use Switch instead of AI directly connecting to Airtable?

The AI should only make the decision.

The Switch should control the routing.

This gives the workflow a clean architecture:

```text
AI
↓
Classification
↓
Switch
↓
Correct business process
```

This separation makes the automation easier to understand and modify.

---

# 18. Branch 1 — Complaint

If:

```text
AI Output = Complaint
```

the workflow goes to:

**Complaint Record**

---

# 19. Complaint Record — Airtable

The Complaint Record node creates a record in the Airtable **Complaint** table.

It stores:

* Full Name
* Email
* Mobile No.
* Feedback

The workflow maps these fields into Airtable.

Conceptually:

```text
Customer Feedback
        ↓
Complaint
        ↓
Airtable Complaint Table
```

This gives the support team a centralized record of customer complaints.

---

# 20. Complaint → Slack

After the complaint is stored in Airtable, the workflow sends a message to Slack.

The Slack node sends:

```text
Full Name
Email
Feedback
```

to the customer-support channel.

So:

```text
Complaint
   ↓
Airtable
   ↓
Slack #customer-support
```

This allows the support team to become aware of the complaint automatically.

---

# 21. Complaint → Gmail

After Slack notification, Gmail sends an automatic response to the customer.

The email subject is:

```text
We Have Received Your Complaint
```

The message acknowledges that the complaint has been received and tells the customer that the team is investigating it.

Therefore the complete complaint flow is:

```text
Customer
   ↓
Form
   ↓
AI
   ↓
Complaint
   ↓
Airtable
   ↓
Slack
   ↓
Gmail
```

---

# 22. Branch 2 — Compliment

If:

```text
AI Output = Compliment
```

the Switch routes the data to:

**Compliment Record**

The Compliment Record creates a record in the Airtable **Compliment** table.

It stores:

```text
Full Name
Email
Mobile No.
Feedback
```

In the current workflow configuration, the **Compliment branch ends after the Airtable record**. The exported workflow does not show a Slack or Gmail node connected after `Compliment Record`.

So the current compliment flow is:

```text
Customer
   ↓
Form
   ↓
AI
   ↓
Compliment
   ↓
Airtable
```

---

# 23. Branch 3 — Feature Addition Request

If:

```text
AI Output = Feature Addition Request
```

the workflow sends the data to:

**Feature Addition Request**

This node creates a record in the Airtable **Feature Addition Request** table.

Again, it stores:

```text
Full Name
Email
Mobile No.
Feedback
```

---

# 24. Feature Request → Slack

After Airtable, the feature request is sent to Slack.

The Slack node posts the customer's information and feedback to the feature-adding-request channel.

So:

```text
Feature Request
       ↓
Airtable
       ↓
Slack #feature-adding-request
```

This means the product/development team can see feature requests without manually checking the form.

---

# 25. Feature Request → Gmail

After the Slack notification, Gmail automatically sends an email to the customer.

The email subject is:

```text
Thank You for Your Feature Suggestion
```

The email explains that the request has been received and forwarded to the product team for review.

Therefore:

```text
Feature Request
       ↓
Airtable
       ↓
Slack
       ↓
Gmail
```

---

# 26. Complete Node-by-Node Explanation

| Node                         | Purpose                                                |
| ---------------------------- | ------------------------------------------------------ |
| **On form submission**       | Starts the workflow when a customer submits feedback   |
| **AI Agent**                 | Determines the feedback category                       |
| **Google Gemini Chat Model** | Provides the AI model for classification               |
| **Merge**                    | Combines original customer data with AI classification |
| **Switch**                   | Routes the data according to classification            |
| **Complaint Record**         | Stores complaints in Airtable                          |
| **Compliment Record**        | Stores compliments in Airtable                         |
| **Feature Addition Request** | Stores feature requests in Airtable                    |
| **Send a message**           | Sends complaint notification to Slack                  |
| **Send a message1**          | Sends feature request notification to Slack            |
| **Send a message2**          | Sends automated complaint email                        |
| **Send a message3**          | Sends automated feature-request email                  |

---

# 27. Example: Complaint Execution

Suppose a customer submits:

```text
Name:
Rahul

Email:
rahul@gmail.com

Mobile:
9876543210

Feedback:
The application keeps crashing when I upload a file.
```

### Step 1 — Trigger

The customer submits the form.

```text
On form submission
```

fires.

---

### Step 2 — AI Agent

The feedback is sent to Gemini.

Gemini understands:

```text
Application crashing
=
Problem/issue
```

Output:

```text
Complaint
```

---

### Step 3 — Merge

The original data and AI output are combined.

```text
Name       → Rahul
Email      → rahul@gmail.com
Mobile     → 9876543210
Feedback   → Application keeps crashing
Output     → Complaint
```

---

### Step 4 — Switch

The Switch checks:

```text
Is output = Complaint?
```

Yes.

Therefore it chooses the Complaint branch.

---

### Step 5 — Airtable

The complaint is saved.

---

### Step 6 — Slack

The support channel receives the complaint.

---

### Step 7 — Gmail

The customer automatically receives:

```text
We Have Received Your Complaint
```

The entire process happens automatically.

---

# 28. Example: Feature Request Execution

Customer submits:

```text
Feedback:
Please add dark mode to the application.
```

AI determines:

```text
Feature Addition Request
```

Then:

```text
Form
 ↓
AI Agent
 ↓
Gemini
 ↓
Merge
 ↓
Switch
 ↓
Feature Addition Request
 ↓
Airtable
 ↓
Slack
 ↓
Gmail
```

The product team gets notified and the customer gets an acknowledgment automatically.

---

# 29. Example: Compliment Execution

Customer submits:

```text
Feedback:
The application is very easy to use. Great work!
```

AI determines:

```text
Compliment
```

Then:

```text
Form
 ↓
AI
 ↓
Merge
 ↓
Switch
 ↓
Compliment Record
 ↓
Airtable
```

The current workflow stops at the Compliment Airtable record.

---

# 30. Understanding Connections in n8n

The lines between nodes represent **data flow**.

For example:

```text
A → B
```

means:

> The output of node A becomes input for node B.

Your workflow starts with:

```text
On form submission
```

and sends the form information into the workflow.

The form data is also passed toward the Merge node, while another path sends the information through the AI Agent for classification. The AI result is then merged back with the original form data.

---

# 31. What is an Execution?

An **execution** is one run of the workflow.

For example:

```text
Customer A submits form
```

→ Execution #1

```text
Customer B submits form
```

→ Execution #2

```text
Customer C submits form
```

→ Execution #3

Each submission creates a new workflow execution.

---

# 32. Input and Output

Every n8n node generally follows the concept:

```text
INPUT
  ↓
PROCESS
  ↓
OUTPUT
```

For example:

### Form Trigger

```text
Input:
Customer submits form

Output:
Customer data
```

### AI Agent

```text
Input:
Customer feedback

Process:
AI analysis

Output:
Complaint / Compliment / Feature Addition Request
```

### Switch

```text
Input:
Classification

Process:
Check rules

Output:
Correct branch
```

### Airtable

```text
Input:
Customer data

Process:
Create record

Output:
Created Airtable record
```

---

# 33. Why This Workflow is Useful

Without automation, a person might need to:

```text
1. Check feedback form
2. Read feedback
3. Decide its category
4. Open Airtable
5. Enter the feedback
6. Open Slack
7. Notify the appropriate team
8. Write an email
9. Send response to customer
```

With this workflow:

```text
Customer submits feedback
          ↓
       AUTOMATION
          ↓
AI classification
          ↓
Database storage
          ↓
Team notification
          ↓
Customer response
```

Most of the repetitive work is removed.

---

# 34. Technologies Used

### n8n

Workflow automation and orchestration.

### Google Gemini

AI-powered feedback classification.

### Airtable

Stores categorized customer feedback.

### Slack

Notifies the appropriate internal team.

### Gmail

Sends automated responses to customers.

### n8n Forms

Collects customer feedback.

---

# 35. Important n8n Concepts Learned From This Project

This project demonstrates several important n8n concepts:

### 1. Trigger

Starts the workflow when an event occurs.

### 2. Expressions

Example:

```text
{{ $json.Feedback }}
```

Used to dynamically access data.

### 3. AI Agent

Uses an AI model to make a classification decision.

### 4. AI Model Connection

The Gemini Chat Model provides the model used by the AI Agent.

### 5. Merge

Combines data coming from different paths.

### 6. Switch

Creates conditional branches.

### 7. Integrations

Connects n8n with:

```text
Airtable
Slack
Gmail
Google Gemini
```

### 8. Data Mapping

Maps fields such as:

```text
Full Name → Full Name
Email → Email
Mobile No. → Mobile No.
Feedback → Feedback
```

### 9. Automation

Multiple actions happen automatically after a single trigger.

---

# 36. The Most Important Concept to Remember

If you need to explain this project in an interview, remember this simple sentence:

> "I built an n8n-based customer feedback automation workflow where a form submission triggers an AI Agent powered by Google Gemini to classify feedback as a complaint, compliment, or feature request. A Switch node routes the feedback to the appropriate Airtable table, while complaints and feature requests are additionally sent to Slack and acknowledged through automated Gmail responses."

That is the **30-second explanation** of your project.

---

# 37. One-Line Explanation of Every Node

```text
On Form Submission
→ Starts the workflow.

AI Agent
→ Understands and classifies the feedback.

Google Gemini Chat Model
→ Provides the AI intelligence.

Merge
→ Combines customer information with AI classification.

Switch
→ Decides which category branch should execute.

Complaint Record
→ Saves complaints in Airtable.

Compliment Record
→ Saves compliments in Airtable.

Feature Addition Request
→ Saves feature requests in Airtable.

Send a message
→ Sends complaint information to Slack.

Send a message1
→ Sends feature requests to Slack.

Send a message2
→ Sends complaint acknowledgment through Gmail.

Send a message3
→ Sends feature-request acknowledgment through Gmail.
```

---

# 38. Final Architecture to Memorize

```text
                    CUSTOMER
                       │
                       ▼
                ┌─────────────┐
                │ FORM TRIGGER│
                └──────┬──────┘
                       │
             ┌─────────┴─────────┐
             │                   │
             ▼                   ▼
        ORIGINAL DATA         AI AGENT
                                 │
                                 ▼
                           GOOGLE GEMINI
                                 │
                                 ▼
                          CLASSIFICATION
                                 │
             ┌───────────────────┘
             ▼
           MERGE
             │
             ▼
          SWITCH
             │
      ┌──────┼─────────┐
      ▼      ▼         ▼
  COMPLAINT COMPLIMENT FEATURE
      │      │         │
      ▼      ▼         ▼
   AIRTABLE AIRTABLE AIRTABLE
      │                │
      ▼                ▼
    SLACK             SLACK
      │                │
      ▼                ▼
    GMAIL             GMAIL
```

## Key takeaway

The workflow follows a very common automation architecture:

**Trigger → Process → Decision → Action**

In your project:

**Trigger** = Form Submission
**Process** = AI Classification
**Decision** = Switch
**Action** = Airtable + Slack + Gmail

Once you understand this pattern, you can build many other n8n automations by simply changing the trigger, processing logic, decision rules, and final actions.
