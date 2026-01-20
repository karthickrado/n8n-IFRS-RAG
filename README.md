## Agentic IFRS S1 & S2 Assistant

**Overview**
This project is an autonomous AI Agent designed to assist financial analysts with IFRS compliance and general accounting queries. It employs a **Hybrid RAG (Retrieval-Augmented Generation)** architecture:
1.  **For IFRS S1 & S2:** It strictly retrieves and cites official legal text from a vector database.
2.  **For General Finance:** It leverages the LLM's broader training to answer questions about other standards (e.g., IFRS 1, IAS 1) without requiring database lookups.

**How to Run**
1. **Retrieve the Data:**
   - [IFRS S1 (General Requirements)](https://drive.google.com/uc?export=download&id=15fBhR0L7Nu_ibXupozuF8FsNSqqFQYC2)
   - [IFRS S2 (Climate-related Disclosures)](https://drive.google.com/uc?export=download&id=1VpKPZLsCiB-Jatv4-mr6G1fcDHTuxk0L)
   *(Note: You will need to host these files or update the HTTP Request nodes in the workflow with the direct download links).*
2. **Import Workflows:** Import the two JSON files into [n8n](https://n8n.io/):
   - `IFRS_Ingestion_Pipeline.json` (Workflow 1)
   - `IFRS_Chat_Agent.json` (Workflow 2)
3. **Setup Credentials:** Add your **Google Gemini** and **Pinecone** API keys in n8n.
4. **Ingest Data:** Run the **Ingestion Pipeline** once manually to process the PDFs and populate the vector database.
5. **Start Chatting:** Open the **Chat Agent** workflow and click "Chat" to query the standards.

**Setup**
1. **Model Flexibility:**
   - This workflow is configured for **Google Gemini** (`gemini-1.5-flash` + `text-embedding-004`) for a balance of speed and cost.
   - *Customisation:* You can swap these nodes for OpenAI (GPT-4) or Anthropic (Claude) if preferred.
   - **Important:** If you change the embedding model (e.g., to `text-embedding-3-small` from OpenAI), you **must** recreate your Pinecone Index with the matching dimensions (1536 for OpenAI vs. 768 for Gemini).
2. **Pinecone Index:**
   - Create an index named `ifrs-bot`.
   - Dimensions: **768** (Default for Gemini).
   - Metric: Cosine.

## Architecture & Logic

**Ingestion Pipeline (Workflow 1)**
- **MIME Type Correction:** Includes a custom JavaScript node to force `application/pdf` recognition on raw Google Drive binary streams - API returns generic `octet-stream` by default, which causes the data loader to fail. 
- **Chunking Strategy:** Splits legal text into 500-character chunks with a 50-character overlap. This ensures context (e.g., paragraph numbers) is preserved across boundaries.

**Agentic Retrieval (Workflow 2)**
- **Tool Use:** The Agent is configured as a 'Tools Agent', meaning it autonomously decides whether to query the Vector Store (for S1/S2 deep dives) or rely on internal knowledge (for general IFRS/IAS definitions).
- **Citations:** The system prompt (which can be customised) is engineered to strictly require paragraph-level citations (e.g., *IFRS S2, Para 29(a)*) when the tool is used.
- **Guardrails:** Domain restrictions prevent the agent from answering off-topic queries (e.g., general knowledge or advice unrelated to finance).

**Scalability & Production Notes**
- **Knowledge Base Expansion:** The current architecture is designed for modular growth. To improve accuracy or cover new standards, you can add more PDF URLs (direct download links - not view/edit links) to the ingestion workflow by adding additional HTTP Request nodes. The Vector Store will index them, instantly making them searchable by the agent.
- **Input/Output Channels:** While this demo uses the native n8n Web Chat, the "Headless" agent architecture allows you to swap the trigger for **Slack**, **Telegram**, or **Microsoft Teams** for corporate deployment.
- **File Handling:** The HTTP Request node is used here for easy sharing and testing. For production, replace the HTTP Request node with the **Google Drive Node** (OAuth) to monitor a specific folder for auto-ingestion of new standards.
- **State Management:** This prototype uses Simple Memory. For high-volume deployment, the memory backend should be upgraded to Redis or PostgreSQL to handle concurrent user sessions.
