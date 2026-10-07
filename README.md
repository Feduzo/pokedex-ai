# Pokédex AI

![CI](https://github.com/Feduzo/pokedex-ai/actions/workflows/ci.yml/badge.svg)

A Pokédex for the first two generations with an AI assistant: pick a Pokémon, check its data and chat with **Professor Oak** (Professor Carvalho) about types, weaknesses and strategy. The chat answers with the selected Pokémon as context.

This project started as the technical challenge of a software engineering internship selection process, which I passed. Afterwards I kept improving it so it runs on any OS without needing an API key.

![Pokédex main screen](docs/screenshots/home.png)

![Chat with Professor Oak](docs/screenshots/chat.png)

> The chat screenshot was generated with the `mistral` model running locally on Ollama. The professor answers in Brazilian Portuguese.

## Features

- All 251 Pokémon from Generations 1 and 2, with search by name and batch loading.
- Details with official artwork, types, stats, height, weight, ability and cry.
- Chat with Professor Oak, who receives the Pokémon's types, abilities and stats as context.
- Configurable LLM: **local Ollama** (no key, no cost) or **OpenRouter** (optional).
- The backend proxies every LLM call, so no key ever reaches the browser.

## Tech stack

| Layer | Technologies |
| --- | --- |
| Frontend | React 19, Vite |
| Backend | FastAPI, httpx, python-dotenv |
| Data | [PokeAPI](https://pokeapi.co) |
| LLM | Ollama (local) or OpenRouter |
| Quality | ESLint, pytest, GitHub Actions |

## How it works

```text
Browser ──► React (Vite)
               ├──► PokeAPI          (Pokémon list and details)
               └──► FastAPI /chat/   ──► Ollama (default)
                                     └─► OpenRouter (if a key is set)
```

In production (`npm start`), FastAPI also serves the frontend build, so everything runs on a single port.

## Running it

Requirements: **Node.js 18+** and **Python 3.10+**. Works on Windows, macOS and Linux.

```bash
git clone https://github.com/Feduzo/pokedex-ai.git
cd pokedex-ai
npm start
```

Open **http://localhost:8000**.

On the first run, `scripts/run.mjs` takes care of everything:

1. creates the virtual environment and installs backend and frontend dependencies;
2. asks for an OpenRouter key. Press **Enter** to use **local Ollama** instead: it installs Ollama (after confirmation), starts the service and pulls the `llama3.2` model;
3. builds the frontend and starts the server.

Your choice is saved in `backend/.env`, which is ignored by Git.

### Choosing the LLM

| Option | How to use |
| --- | --- |
| **Local Ollama** (default) | Nothing to configure. To set it up by hand, install [Ollama](https://ollama.com) and run `ollama pull llama3.2`. |
| **OpenRouter** | Create a free key at [openrouter.ai/keys](https://openrouter.ai/keys) and put it in `backend/.env`. When a key is set it takes priority over Ollama and uses a free (`:free`) model. |

```bash
# backend/.env
OPENROUTER_API_KEY=your_key_here
```

Optional variables: `OPENROUTER_MODEL`, `OLLAMA_URL` and `OLLAMA_MODEL` (see `backend/.env.example`).

> Never commit your `.env`. The key stays on your machine.

### Development mode

```bash
npm run dev
```

Backend with reload on `http://localhost:8000` and frontend with hot reload on `http://localhost:5173`.

## Tests and quality

```bash
npm test        # backend tests (pytest)
npm run lint    # ESLint on the frontend
npm run build   # production build
```

CI (GitHub Actions) runs lint, build and tests on every push.

## Technical decisions

- **Backend as the LLM proxy:** keeps keys out of the frontend and allows switching providers without touching the UI.
- **Provider with fallback:** without a key the app uses local Ollama, so anyone can run it with no sign-up and no cost.
- **Config read at call time:** environment variables are resolved inside the function, not at import time, so they respect the `.env` loaded by `load_dotenv()`.
- **Batch loading:** avoids a frozen screen while the 251 Pokémon arrive.
- **Prompt with context:** types, abilities and stats are sent along with the question for more accurate answers.
- **Single port in production:** FastAPI serves the frontend build, which makes it simpler to run and demo.

## What was hard

Getting the LLM to work inside the app took the most time. The challenge provided an API key, but wiring it in and sending the selected Pokémon as context to the model took several iterations. After that, the visual design was the next big hurdle.

## What I learned

- How to integrate an LLM into a real application, from the prompt to the API layer.
- How to structure a project in layers (frontend, backend, services) and justify each decision, which improved my software architecture skills a lot.

## Next steps

- Expand the backend with more endpoints and more data per Pokémon.
- Polish the frontend with animations and more sounds.
- Add more generations, aiming for a complete Pokédex.

## License

[MIT](LICENSE)
