# Multi-Agent Shopping

Multi-Agent Shipping is a small Google Agent Development Kit (ADK) demo for an ecommerce shopping flow. It uses a root shopping orchestrator to route user requests to specialized agents for product search, inventory checks, cart management, shipping, and order lookup.

The project is intentionally lightweight: product data, inventory counts, and orders are stored in local Python dictionaries so the agent behavior is easy to inspect and modify.

## Features

- Shopping orchestrator built with Google ADK
- Product search over a sample electronics catalog
- Exact search and broad keyword search agents behind a simple router
- Inventory lookup with structured output support
- Cart workflow that creates or reuses an order session
- Order placement and shipping-status inquiry tools
- Prompt files separated from Python code for easier iteration

## Project Structure

```text
multi-agent-shipping/
|-- agent.py                  # Root ADK agent and orchestrator wiring
|-- agents/
|   |-- cart.py               # Cart/order session workflow
|   |-- inquiry.py            # Order status lookup agent
|   |-- inventory.py          # Inventory tools and agents
|   |-- order_data.py         # In-memory order store and statuses
|   |-- products.py           # Sample product catalog and stock counts
|   |-- search.py             # Search agents and A/B router
|   `-- shipping.py           # Order placement tool and shipping agent
|-- prompts/                  # Agent instructions
|-- requirements.txt
`-- README.md
```

## How It Works

`agent.py` defines `root_agent`, a shopping orchestrator that delegates to sub-agents:

- `search_agent` finds products by product name or description.
- `inventory_agent` checks whether a product ID is in stock.
- `cart_agent` creates an order session, checks inventory, and adds products to the active cart.

Additional agents are present for shipping and order inquiry:

- `shipping_agent` places an order against a shipping address.
- `inquiry_agent` retrieves cart, address, and status information for an order.

The sample catalog lives in `agents/products.py`; order state lives in `agents/order_data.py`.

## Requirements

- Python 3.10+
- Google ADK
- A Gemini API key available as `GOOGLE_API_KEY`

Dependencies are listed in `requirements.txt`:

```text
a2a-sdk>=0.3.6
google-adk>=1.17.0
toolbox-core>=0.5.0
```

## Setup

Clone the repository and enter the project:

```bash
git clone https://github.com/jabbala10-bit/multi-agent-shipping.git
cd multi-agent-shipping
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

Create a `.env` file or export your Gemini API key:

```bash
GOOGLE_API_KEY="your-api-key"
```

## Run

From the parent directory that contains the `multi-agent-shipping/` folder, start the ADK web UI:

```bash
adk web --port 8000
```

Then open:

```text
http://localhost:8000
```

You can also run the agent from the command line:

```bash
adk run multi-agent-shipping
```

## Example Prompts

Try prompts like:

```text
Find me wireless headphones.
```

```text
Is product P003 in stock?
```

```text
Add the USB-C Cable to my cart.
```

```text
What is the status of ORDER_001?
```

## Development Notes

- This is a demo implementation, not a production ecommerce backend.
- Products, inventory, and orders are stored in memory and reset when the process restarts.
- Inventory is checked before cart insertion by the workflow, but stock counts are not decremented after adding items.
- The root orchestrator currently wires search, inventory, and cart agents. Shipping and inquiry agents exist in the codebase but are not included in the root agent's `sub_agents` list.
- Add automated tests before extending this into a larger workflow.

## License

This project is licensed under the Apache License 2.0. See `LICENSE` for details.
