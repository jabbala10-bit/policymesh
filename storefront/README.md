# Storefront Agent

Storefront Agent is a lightweight Google Agent Development Kit (ADK) orchestrator for the customer-facing shopping experience. It acts as the front door for users and delegates specialized work to remote A2A agents for shopping and shipping.

This package does not implement product search, cart management, order persistence, shipping calculation, or fulfillment directly. Instead, it connects to remote agent cards and routes customer requests to the right service.

## Features

- Customer-facing storefront orchestrator
- Remote A2A delegation to a shopping agent
- Remote A2A delegation to a shipping agent
- Simple prompt-driven routing behavior
- A2A metadata in `agent.json`
- Minimal dependency footprint

## Project Structure

```text
storefront/
|-- agent.py                 # Root storefront agent and remote A2A wiring
|-- agent.json               # A2A agent metadata
|-- prompts/
|   `-- agent-prompt.txt     # Storefront routing instructions
|-- requirements.txt
`-- README.md
```

## How It Works

`agent.py` defines `root_agent`, the main storefront agent. It uses two `RemoteA2aAgent` instances:

- `shopping_agent` for product search and cart actions.
- `shipping_agent` for shipping costs, tax estimates, order lookup, and fulfillment.

The prompt in `prompts/agent-prompt.txt` instructs the storefront agent to answer general questions directly and delegate specialized tasks to the correct remote agent.

## Remote Agent Dependencies

By default, the storefront agent expects these A2A endpoints to be available locally:

```text
http://localhost:8000/a2a/shopping/.well-known/agent-card.json
http://localhost:8000/a2a/shipping/.well-known/agent-card.json
```

These URLs are currently hard-coded in `agent.py`.

Start the shopping and shipping agents before using the storefront agent for end-to-end workflows.

## Requirements

- Python 3.10+
- Google ADK
- A2A SDK
- A Gemini API key available as `GOOGLE_API_KEY`
- Running shopping and shipping A2A agents

Dependencies are listed in `requirements.txt`:

```text
google-adk>=1.17.0
a2a-sdk>=0.3.6
```

## Setup

From this directory:

```bash
cd storefront
```

Create and activate a virtual environment:

```bash
python -m venv .venv
```

Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

macOS/Linux:

```bash
source .venv/bin/activate
```

Install dependencies:

```bash
pip install -r requirements.txt
```

Set your Gemini API key:

```bash
GOOGLE_API_KEY="your-api-key"
```

## Run

From the parent directory that contains the `storefront/` folder, start the ADK web UI:

```bash
adk web --port 8000
```

Then open:

```text
http://localhost:8000
```

You can also run the agent from the command line:

```bash
adk run storefront
```

## A2A Metadata

`agent.json` describes this agent as an A2A-capable service:

- Name: `storefront`
- URL: `http://localhost:8000/a2a/storefront`
- Input/output mode: `text/plain`
- Function calling: enabled
- Skills: none declared directly

The storefront agent delegates skills to the shopping and shipping agents instead of exposing its own specialized skill list.

## Example Prompts

Try prompts like:

```text
Find me a portable charger and add it to my cart.
```

```text
What is the order ID for my cart?
```

```text
Estimate shipping and taxes for my order to CA.
```

```text
Ship my order to Allen, 123 Main St, Anywhere, CA, 98765.
```

## Development Notes

- This agent is intentionally thin; most business logic lives in the remote shopping and shipping agents.
- Remote A2A URLs are hard-coded for local development. Consider moving them to environment variables for deployment.
- End-to-end behavior depends on the shopping and shipping agents being available and exposing valid A2A agent cards.
- The storefront prompt expects the shopping agent to return or mention an order ID after cart actions so it can pass that ID to shipping workflows.
