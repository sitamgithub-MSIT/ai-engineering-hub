# Conversational Customer Support Agent

A conversational customer-support assistant built on CrewAI's conversational flows. An LLM router sends each incoming message to the route that fits it — a deterministic order lookup, a RAG answer over the return policy, or a live web-search crew — and conversation state carries across turns so follow-ups and pronouns resolve. The running flow streams to a CopilotKit chat UI over the AG-UI protocol.

We use:

- [CrewAI](https://docs.crewai.com/) for multi-agent orchestration — a conversational Flow with an LLM router and a Crew per route
- [AG-UI](https://docs.ag-ui.com/) for streaming the running flow to the browser
- [CopilotKit](https://docs.copilotkit.ai/) for the React chat runtime and UI
- [OpenRouter](https://openrouter.ai/) as the LLM provider (`gpt-4o` for conversation, `gpt-4o-mini` for routing)
- [Exa](https://exa.ai/) for live web search in the research route
- [Next.js](https://nextjs.org/) for the frontend
- [FastAPI](https://fastapi.tiangolo.com/) + [Uvicorn](https://www.uvicorn.org/) for the agent server

## How It Works

1. Open the chat UI and send a message about an order, a return, or a shipping question
2. A router LLM (`gpt-4o-mini`) classifies the message into one of the flow's routes — the route catalog is built automatically from each handler's docstring
3. The matched route handles the turn:
   - **ORDER_LOOKUP** — deterministic lookup against the order database (status, carrier, tracking, delivery date); no agent
   - **RETURN_POLICY** — a single-agent Crew runs real RAG over the return-policy knowledge base and answers in plain language
   - **RESEARCH** — a two-agent Crew (researcher + communicator) uses Exa web search for live conditions (carrier delays, weather) and rewrites the findings for the customer
   - **converse** — a plain conversational reply for everything else, answered from history when possible
4. Conversation state (customer id, last order id, message history) persists across turns within a session, so pronouns and follow-ups resolve ("Has that arrived yet?", "What was the order number again?")
5. The flow streams over AG-UI to the CopilotKit chat UI in the browser

## Set Up

### Create .env File

Create a `.env` file in the root directory with the following content:

```env
OPENROUTER_API_KEY=<your_openrouter_api_key>
EXA_API_KEY=<your_exa_api_key>
```

### Install Dependencies

Backend (Python 3.12, via [uv](https://docs.astral.sh/uv/)):

```bash
uv sync
source .venv/bin/activate
```

On Windows (PowerShell):

```powershell
uv sync
.venv\Scripts\activate.ps1
```

Frontend (Node.js):

```bash
cd frontend
npm install
```

## Run the App

Start the backend (serves the flow at `http://127.0.0.1:8000/conversation`):

```bash
uvicorn server:app --port 8000
```

In a second terminal, start the frontend:

```bash
cd frontend
npm run dev
```

Open the URL shown in the terminal (e.g. `http://localhost:3000`). Ask about an order (try order `4471`), a return, or a shipping delay to trigger each route.

## 📬 Stay Updated with Our Newsletter!

**Get a FREE Data Science eBook** 📖 with 150+ essential lessons in Data Science when you subscribe to our newsletter! Stay in the loop with the latest tutorials, insights, and exclusive resources. [Subscribe now!](https://join.dailydoseofds.com)

[![Daily Dose of Data Science Newsletter](https://github.com/patchy631/ai-engineering/blob/main/resources/js.png)](https://join.dailydoseofds.com)

## Contribution

Contributions are welcome! Feel free to fork this repository and submit pull requests with your improvements.
