# Shipping Agent

Shipping Agent is a standalone Google Agent Development Kit (ADK) project for order fulfillment. It helps customers set a delivery address, calculate shipping and tax costs, summarize the final order total, and approve an order for fulfillment.

The agent is designed to work with a Toolbox-backed order database. Product prices and static rate tables are kept in local Python modules, while order reads and updates are performed through Toolbox tools.

## Features

- Shipping orchestration with Google ADK
- Order address collection and update flow
- Standard, express, international, and free shipping rates
- State-based tax calculation
- Parallel shipping-cost and tax-cost calculation
- Total order cost computation
- Customer-facing order summary before approval
- Order status update to `placed` after confirmation
- Shipping inquiry agent for order lookup questions
- A2A metadata in `agent.json`

## Project Structure

```text
shipping/
|-- agent.py                         # Root shipping orchestrator
|-- agent.json                       # A2A agent metadata
|-- agents/
|   |-- inquiry.py                   # Shipping/order inquiry agent
|   |-- products.py                  # Product catalog used for pricing
|   |-- rates.py                     # Shipping and tax rate tables
|   `-- shipping.py                  # Fulfillment workflow and tools
|-- prompts/
|   |-- agent-prompt.txt
|   |-- approve-order-prompt.txt
|   |-- compute-order-prompt.txt
|   |-- inquiry-prompt.txt
|   |-- order-summary-prompt.txt
|   |-- place-order-prompt.txt
|   |-- shipping-cost-prompt.txt
|   |-- shipping-prompt.txt
|   `-- taxes-cost-prompt.txt
|-- requirements.txt
`-- README.md
```

## How It Works

`agent.py` defines `root_agent`, the top-level shipping orchestrator. It delegates requests to:

- `shipping_agent` for order fulfillment, cost calculation, summaries, and approval.
- `inquiry_agent` for order lookup and shipping-status questions.

The fulfillment flow is implemented in `agents/shipping.py`:

1. `place_order_agent` identifies the order, resolves the active user when needed, validates that the order is open, and updates the shipping address.
2. `costs_agent` runs shipping and tax calculations in parallel.
3. `compute_order_agent` combines subtotal, shipping, and taxes into the final order total and saves the costs.
4. `order_summary_agent` presents the order total and asks for customer approval.
5. `approve_order_agent` updates the order status to `placed` after the customer confirms.

## External Tools

This project loads database tools from Toolbox at import time. By default it connects to:

```text
http://127.0.0.1:5000
```

Override this with:

```bash
TOOLBOX_URL="http://your-toolbox-host:5000"
```

Expected Toolbox tool names:

- `get-order`
- `get-open-order-for-user`
- `update-order-address`
- `update-order-status`
- `update-order-costs`

Make sure the Toolbox service is running before starting the ADK agent.

## Requirements

- Python 3.10+
- Google ADK
- Toolbox service exposing the required order tools
- A Gemini API key available as `GOOGLE_API_KEY`

Dependencies are listed in `requirements.txt`:

```text
a2a-sdk>=0.3.6
google-adk>=1.17.0
toolbox-core>=0.5.0
```

## Setup

From this directory:

```bash
cd shipping
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

Set environment variables:

```bash
GOOGLE_API_KEY="your-api-key"
TOOLBOX_URL="http://127.0.0.1:5000"
```

## Run

From the parent directory that contains the `shipping/` folder, start the ADK web UI:

```bash
adk web --port 8000
```

Then open:

```text
http://localhost:8000
```

You can also run the agent from the command line:

```bash
adk run shipping
```

## A2A Metadata

`agent.json` describes this agent as an A2A-capable service:

- Name: `shipping`
- URL: `http://localhost:8000/a2a/shipping`
- Skill: `fulfill_order`
- Input/output mode: `text/plain`
- Function calling: enabled

## Example Prompts

Try prompts like:

```text
Ship my open order to Allen, 123 Main St, Anywhere, CA, 98765 using standard shipping.
```

```text
Calculate the total cost for order 1001 with express shipping to NY.
```

```text
Everything looks good. Please place the order.
```

```text
What is the status of order 1001?
```

## Development Notes

- Shipping and tax rates are mock values in `agents/rates.py`.
- Product prices come from `agents/products.py`.
- Order persistence depends on Toolbox tools rather than local in-memory dictionaries.
- The agent assumes the Toolbox schema and tool contracts match the prompt instructions.
- Because Toolbox tools are loaded during module import, missing or unavailable tools can prevent the agent from starting.
