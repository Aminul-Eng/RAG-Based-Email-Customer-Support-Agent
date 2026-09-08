# RAG-Based AI Customer Support Agent

An automated **AI-powered email customer support system** built with **n8n, OpenAI, Pinecone, and Gmail API**.

The system uses **Retrieval-Augmented Generation (RAG)** to retrieve relevant information from a company's knowledge base before generating a response. This allows the AI agent to answer customer inquiries using organization-specific information such as policies, FAQs, shipping details, refund guidelines, and product documentation.

---

## Project Overview

Traditional LLM-based customer support systems may rely primarily on the model's general knowledge.

This project improves that approach by connecting the language model to a **company-specific knowledge base** using Retrieval-Augmented Generation (RAG).

When a customer sends an email, the workflow automatically:

1. Captures the incoming email through Gmail.
2. Processes the customer inquiry.
3. Searches the Pinecone Vector Store for relevant company knowledge.
4. Supplies the retrieved context to the OpenAI Chat Model.
5. Generates a context-aware response.
6. Replies automatically within the original Gmail thread.

The result is an automated support workflow that combines:

**Email Automation + LLM + Embeddings + Vector Search + RAG**

---

## System Workflow

### 1. Capture Customer Email

The **Gmail Trigger** monitors the configured support inbox and starts the workflow when a new customer email is received.

Relevant email information can include:

- Sender
- Subject
- Message body
- Message ID
- Thread ID

These values are passed to the AI workflow for processing.

---

### 2. Process the Customer Inquiry

The incoming email content is prepared for the AI agent.

The customer's message becomes the query used to determine what information should be retrieved from the company knowledge base.

---

### 3. Retrieve Relevant Knowledge

The AI Agent uses the **Pinecone Vector Store** as a retrieval tool.

The customer's inquiry is compared against vectorized company knowledge using semantic similarity search.

Relevant information may include:

- Company policies
- Frequently Asked Questions
- Shipping information
- Refund and return policies
- Product information
- Support documentation

The most relevant context is retrieved and made available to the language model.

---

### 4. Generate a Context-Aware Response

The **OpenAI Chat Model** generates the customer response using both:

- The customer's original inquiry
- Relevant information retrieved from Pinecone

**Simple Memory** is used to maintain conversational context where applicable.

This RAG-based approach helps the system produce responses that are better grounded in the organization's own knowledge instead of relying only on the LLM's general knowledge.

---

### 5. Reply to the Customer

After the response is generated, the **Gmail Reply** node sends it back using the original message/thread information.

This keeps the response inside the existing customer email conversation.

---

## Workflow Architecture

```text
                         ┌───────────────────────┐
                         │ OpenAI Chat Model     │
                         └───────────┬───────────┘
                                     │
                                     ▼
┌───────────────┐            ┌───────────────┐
│ Gmail Trigger │───────────►│   AI Agent    │
└───────────────┘            └───────┬───────┘
                                     │
                     ┌───────────────┼───────────────┐
                     │               │               │
                     ▼               ▼               ▼
              ┌─────────────┐ ┌─────────────┐ ┌──────────────────┐
              │Simple Memory│ │  Pinecone   │ │ Retrieved Context│
              └─────────────┘ │Vector Store │ └──────────────────┘
                              └──────┬──────┘
                                     │
                                     ▼
                              ┌─────────────┐
                              │ Embeddings  │
                              │   OpenAI    │
                              └─────────────┘

                                     │
                                     ▼
                            ┌──────────────────┐
                            │ Generated Reply  │
                            └────────┬─────────┘
                                     │
                                     ▼
                            ┌──────────────────┐
                            │   Gmail Reply    │
                            └────────┬─────────┘
                                     │
                                     ▼
                            ┌──────────────────┐
                            │ Customer Inbox   │
                            └──────────────────┘
```

---

## Main Workflow

![RAG-Based Email Customer Support Agent](workflow.png)

The main workflow handles:

```text
Customer Email
      │
      ▼
Gmail Trigger
      │
      ▼
AI Agent
      │
      ├──── OpenAI Chat Model
      │
      ├──── Simple Memory
      │
      └──── Pinecone Vector Store
                    │
                    ▼
             Relevant Context
      │
      ▼
Generate Response
      │
      ▼
Gmail Reply
      │
      ▼
Customer
```

---

## Knowledge Base & Data Loader

The project also contains a separate workflow for loading company knowledge into the vector database.

![Pinecone Data Loader](Data%20Loader.png)

The ingestion pipeline conceptually follows:

```text
Company Knowledge
       │
       ▼
Document Processing
       │
       ▼
Text Chunking
       │
       ▼
OpenAI Embeddings
       │
       ▼
Pinecone Vector Store
```

### How It Works

Company documentation is divided into smaller text chunks.

Each chunk is converted into a numerical vector representation using an **OpenAI Embedding Model**.

These vectors are stored inside **Pinecone**.

When a customer asks a question, the system converts the query into a compatible vector representation and performs semantic similarity search to find relevant knowledge.

---

## Technology Stack

| Technology | Purpose |
|---|---|
| **n8n** | Workflow orchestration and automation |
| **Gmail API** | Capturing incoming emails and sending replies |
| **OpenAI Chat Model** | Understanding inquiries and generating responses |
| **OpenAI Embeddings** | Converting text into vector representations |
| **Pinecone** | Vector storage and semantic similarity search |
| **Simple Memory** | Maintaining conversational context |

---

## Key Features

- Automated customer email processing
- Retrieval-Augmented Generation (RAG)
- Company-specific knowledge retrieval
- Semantic vector search
- Context-aware AI response generation
- Automatic Gmail replies
- Same-thread email communication
- Conversation context using memory
- Separate knowledge ingestion workflow
- Modular n8n workflow architecture

---

## How RAG Works in This Project

Without RAG:

```text
Customer Question
       │
       ▼
      LLM
       │
       ▼
    Response
```

With RAG:

```text
Customer Question
       │
       ▼
Semantic Search
       │
       ▼
Pinecone Vector Store
       │
       ▼
Relevant Company Knowledge
       │
       ▼
Customer Question + Retrieved Context
       │
       ▼
OpenAI Chat Model
       │
       ▼
Context-Aware Response
```

The important difference is that the LLM receives relevant company information **before generating the final response**.

This makes the architecture more suitable for business applications where answers should be grounded in organization-specific information.

---

## Real-World Use Cases

### E-commerce Customer Support

The system can assist with inquiries related to:

- Shipping policies
- Return procedures
- Refund policies
- Product information
- Order-related FAQs

### SaaS Customer Support

It can help answer:

- Product usage questions
- Feature documentation
- Common troubleshooting questions
- Account-related FAQs
- Technical documentation queries

### Educational Platforms

The same architecture can support:

- Course information
- Enrollment procedures
- Fee information
- Class schedules
- Student FAQs

### Internal Knowledge Assistant

The RAG architecture can also be adapted for:

- HR policies
- Internal documentation
- Employee FAQs
- IT support
- Standard operating procedures

---

## Challenges & Limitations

### Hallucination Risk

RAG reduces reliance on the model's general knowledge, but it does not completely eliminate hallucinations.

The LLM may still generate unsupported or inaccurate information if retrieval quality is poor or the instructions are insufficient.

### Knowledge Base Freshness

The quality of the responses depends heavily on the information stored in Pinecone.

If company policies or documentation change without updating the vector database, the system may retrieve outdated information.

### Retrieval Quality

Poor document chunking, embedding selection, or retrieval configuration can result in irrelevant context being supplied to the LLM.

### Complex Customer Queries

Highly nuanced, sensitive, multi-part, or emotionally charged customer issues may require human intervention.

### API Cost

Operating costs can increase with:

- Email volume
- LLM token usage
- Embedding generation
- Vector database usage

### Production Reliability

A demonstration workflow is not automatically production-ready.

Production deployments should include additional monitoring, security controls, error handling, evaluation, and human escalation mechanisms.

---

## Security Considerations

Before using this workflow in a production environment, consider:

- Secure credential management
- OAuth security
- API key protection
- Input validation
- Prompt-injection defenses
- Customer data privacy
- Access control
- Logging and monitoring
- Rate limiting
- Error handling
- Human escalation for sensitive requests

> Never store API keys, OAuth tokens, passwords, or other production secrets directly inside workflow files committed to GitHub.

---

## Repository Structure

```text
.
├── Assignment requirement.txt
├── Data Loader.png
├── Pinecone Data Loader.json
├── RAG-Based Email Customer Support Agent.json
├── README.md
└── workflow.png
```

### Main Files

#### `RAG-Based Email Customer Support Agent.json`

The primary n8n workflow responsible for:

- Capturing customer emails
- Calling the AI Agent
- Retrieving company knowledge
- Generating responses
- Replying through Gmail

#### `Pinecone Data Loader.json`

The knowledge-ingestion workflow responsible for preparing company information and storing embeddings inside Pinecone.

#### `workflow.png`

Screenshot of the main customer-support workflow.

#### `Data Loader.png`

Screenshot of the knowledge-base ingestion workflow.

#### `Assignment requirement.txt`

Original project requirements.

---

## Setup Requirements

To run this project, you will need:

- An n8n instance
- Google/Gmail OAuth credentials
- OpenAI API credentials
- Pinecone account
- Pinecone index
- Company knowledge-base content

---

## Setup Guide

### 1. Clone the Repository

```bash
git clone <your-repository-url>
cd <repository-name>
```

### 2. Import the Workflows

Import the following JSON files into n8n:

```text
Pinecone Data Loader.json
RAG-Based Email Customer Support Agent.json
```

### 3. Configure Credentials

Configure the required credentials inside n8n:

```text
Gmail OAuth2
OpenAI API
Pinecone API
```

Do not hard-code credentials directly inside workflow nodes.

### 4. Configure the Vector Store

Create or select a Pinecone index and configure the corresponding Pinecone nodes.

The embedding configuration used during ingestion and retrieval must remain compatible.

### 5. Load the Knowledge Base

Run the **Pinecone Data Loader** workflow to process the company documentation and store its embeddings in Pinecone.

### 6. Configure Gmail

Connect the Gmail account that will receive customer support inquiries.

Configure the trigger and reply nodes with the appropriate Gmail credentials.

### 7. Test the Workflow

Send a test customer inquiry to the configured Gmail inbox.

Verify that:

```text
Email Received
      ↓
Workflow Triggered
      ↓
Relevant Knowledge Retrieved
      ↓
AI Response Generated
      ↓
Reply Sent to Original Thread
```

### 8. Activate the Workflow

After successful testing, activate the n8n workflow.

---

## Production Improvements

Several features could make the system more robust for real-world deployment:

- Human-in-the-loop approval
- Confidence-based response handling
- Automatic escalation to human agents
- Retrieval relevance thresholds
- Source citations in responses
- Prompt-injection protection
- Customer intent classification
- CRM integration
- Support ticket creation
- Response quality evaluation
- Observability and monitoring
- Retry and failure handling
- Conversation analytics
- Multi-channel support
- Advanced document ingestion
- Automated knowledge-base synchronization

A production architecture could use a decision layer such as:

```text
Customer Email
      │
      ▼
RAG Retrieval
      │
      ▼
AI Response
      │
      ▼
Confidence / Validation
      │
      ├── High Confidence ──► Automatic Reply
      │
      └── Low Confidence ───► Human Review
```

---

## Key Learning

The most important takeaway from this project is that the value of RAG is not simply about giving an LLM more information.

The real value comes from connecting an LLM with **proprietary business knowledge** and retrieving the relevant information when it is needed.

This project demonstrates how:

```text
LLMs
  +
Embeddings
  +
Vector Databases
  +
Retrieval-Augmented Generation
  +
Workflow Automation
```

can work together to build practical AI systems for real business workflows.

---

## Author

**Aminul Islam**

AI Automation Specialist | Oracle APEX & Database Developer

Building practical solutions with:

`n8n` `AI Agents` `RAG` `Vector Databases` `LLM Integration` `REST APIs` `Oracle APEX` `SQL` `PL/SQL`

---

## Disclaimer

This project was developed for **learning, demonstration, and portfolio purposes**.

For production deployment, additional security, privacy, monitoring, evaluation, error handling, and human-review mechanisms should be implemented.
