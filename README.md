# Kestrelvale Warehouse Assistant
An AI assistant for a fictional wholesaler, built as a learning project to practise SQL tools and RAG (retrieval-augmented generation) with the Gemini API.

## What it does
- Looks up stock, orders and inbound shipments in an SQLite database through tools (read-only).
- Answers questions about company policies and procedures from a PDF, with source references (RAG).
- Combines both sources in one answer when a question needs it.
- Keeps a limited conversation history.

It uses a Gemini API key, so keep API limits and costs in mind.

### Example questions

> How much basmati rice do we have in stock?

We have 141 cartons of Basmati Rice (12 x 1kg) in stock (SKU: KV-A-0002).

> Which products have less than 80 cartons in stock?

Rolled Porridge Oats (KV-A-0009): 66 cartons, Clear Sandwich & Food Bags (KV-E-0011): 73 cartons, Cellulose Non-Scratch Kitchen Sponges (KV-E-0012): 75 cartons.

> How many deliveries are we expecting?

1 expected shipment with 210 cartons (PO-2026-0012).

> What are the picking wave times?

Wave 1 runs from 08:00 to 11:30, Wave 2 from 12:00 to 15:00 (Source 21).

*Answers depend on the generated sample data and may differ.*

## How it works

**SQL tools**
1. The user's question is sent to Gemini, and the model decides which tool to call.
2. For questions about stock, products, orders or shipments, it calls a Python function that reads from the SQLite database.
3. The model gets the result and answers only from that data.

**RAG (policies)**
1. The policy PDF is split into chunks of 500 characters (with a 100-character overlap), and each chunk is embedded.
2. The question is embedded too, and cosine similarity finds the 3 most similar chunks.
3. The model answers only from those chunks and cites its sources, e.g. (Source 21).

**Safety and limits**
- The bot is read-only: no tool writes to the database.
- Queries are parameterized (`?`), and the model never writes free SQL.
- Inputs are checked: shipment statuses use an allowlist, and an unknown SKU returns "not found" instead of an invented number.

## Data & limitations

- All data is fictional (company, products, orders, policies).
- The bot is read-only and has no tool that returns phone numbers or email addresses.
- It uses a Gemini API key on the free tier, which is only suitable for fictional data. Real customer data would require a paid plan and a review of Google's current terms.
- Answers can be wrong, so important information should always be checked.
- This is a learning project and not production software. Before real use, legal and data protection advice is needed.

## Setup
1. Open the notebook in Google Colab and upload `kestrelvale_warehouse.db` and `kestrelvale_company_base.pdf` (both are in this repository).
2. Add your Gemini API key as a Colab secret named `GEMINI_API_KEY`.
3. Run the first four cells from top to bottom: package install, SQL tools, PDF reading/chunking/embedding, and RAG tools.
4. The fifth cell contains the main bot with all tools.
5. The sixth cell is a list of test questions I used to check how the model calls the tools.
6. The last cell (optional) generates a new `.db` file. It overwrites the existing one.
