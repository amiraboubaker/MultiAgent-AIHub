# MultiAgent AI Hub

MultiAgent AI Hub is a collection of focused AI assistants. Each assistant is a
standalone full-stack application with a React/Vite frontend and a FastAPI
backend. Choose the specialist that matches your task instead of using one
general-purpose prompt for every type of work.

## Available agents

| Agent | Best for | Docker frontend | Docker backend |
| --- | --- | ---: | ---: |
| `ai-machine-learning-agent` | ML, RAG, prompts, and evaluation | 5173 | 8000 |
| `cybersecurity-agent` | Defensive security and threat modeling | 5174 | 8001 |
| `data-science-agent` | Data quality, analysis, statistics, and modeling | 5174 | 8001 |
| `devops-agent` | Infrastructure, deployment, monitoring, and troubleshooting | 5173 | 8000 |
| `mobile-application-agent` | React Native, Expo, Flutter, Android, and iOS | 5174 | 8004 |
| `product-management-agent` | Product discovery, prioritization, and planning | 5173 | 8000 |
| `software-development-agent` | Coding, debugging, architecture, and testing | 5173 | 8000 |
| `ui-ux-design-agent` | UX research, flows, UI design, and accessibility | 5174 | 8004 |
| `web-development-agent` | Web requirements, frontend, backend, and deployment | 5173 | 8000 |

Each agent directory contains its own README with domain-specific details.

## How it works

1. The frontend sends a user message to the selected backend's `/api/chat`
   endpoint.
2. The backend classifies the request into a specialist task type.
3. The backend adds conversation history and the task type to an
   OpenAI-compatible chat-completions request.
4. The response is returned as Markdown, rendered in the frontend, and saved
   in the agent's SQLite database.
5. Conversations can be listed and continued through the API.

The applications display generated guidance but do not execute generated code,
commands, or analysis automatically. API keys stay in the backend environment.

## Prerequisites

- Docker Desktop (recommended), or:
  - Python 3.12+
  - Node.js 20+
  - npm 10+
- An OpenAI-compatible provider. Ollama is the default local option; a hosted
  provider can be configured through the same API shape.

## Run an agent with Docker

From the repository root, replace the directory name with the agent you want:

```powershell
cd software-development-agent
Copy-Item .env.example .env
docker compose up --build
```

Open the frontend URL from the table above. For example, the software
development agent is available at <http://localhost:5173>.

To stop the agent:

```powershell
docker compose down
```

The backend stores conversation data in a named Docker volume, so stopping the
containers does not remove existing conversations.

## Configure the LLM provider

Edit the selected agent's `.env` file:

```dotenv
LLM_API_URL=http://localhost:11434/v1/chat/completions
LLM_API_KEY=
LLM_MODEL=llama3.2
```

For Ollama, start Ollama and pull the model before launching the agent:

```powershell
ollama pull llama3.2
```

The Docker configuration uses `host.docker.internal` to reach a provider
running on the host. For a hosted provider, set `LLM_API_URL`, `LLM_API_KEY`,
and `LLM_MODEL` as required by that provider. Never commit `.env` files or
credentials.

## Run without Docker

The following starts the backend and frontend separately. Run both terminals
from the selected agent directory:

**Terminal 1 - backend**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r backend\requirements.txt
Copy-Item .env.example .env
uvicorn app.main:app --reload --port 8000
```

Use the backend port listed for the agent if it is not `8000`.

**Terminal 2 - frontend**

```powershell
cd frontend
npm install
npm run dev
```

If the frontend does not use the default backend URL, set `VITE_API_URL` to
the backend API URL, for example `http://localhost:8000/api`.

## Project structure

```text
MultiAgent-AIHub/
├── <agent-name>/
│   ├── frontend/
│   │   └── src/
│   │       ├── components/   # Chat UI pieces
│   │       ├── hooks/        # Frontend state and API orchestration
│   │       ├── services/     # Backend API client
│   │       └── types/        # TypeScript types
│   ├── backend/
│   │   ├── app/
│   │   │   ├── agents/       # Task classification
│   │   │   ├── api/          # FastAPI routes
│   │   │   ├── core/         # Settings and cross-cutting controls
│   │   │   ├── models/       # Schemas and SQLite access
│   │   │   └── services/     # LLM and conversation services
│   │   └── tests/            # Backend tests
│   ├── .env.example          # Local configuration template
│   └── docker-compose.yml    # Agent-specific container setup
└── package-lock.json         # Root placeholder; dependencies are per agent
```

The agents are intentionally isolated. This keeps domain prompts, classifiers,
ports, databases, and dependencies independent, making it easier to run or
change one specialist without affecting the others.

## Useful checks

From an agent directory:

```powershell
# Backend health
Invoke-RestMethod http://localhost:<backend-port>/api/health

# Frontend production build
cd frontend
npm run build

# Frontend tests
npm test
```

## Further documentation

For implementation notes and domain behavior, see the README inside the
specific agent directory you are running.
