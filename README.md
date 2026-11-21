# DistributorOS

> **Status: closed.** This was a working prototype, and I've stopped building it. It was never launched and never had paying users. I'm keeping the code and documentation here as a record of what I built, how I thought about it, and why I decided to stop. The full reasoning is in [`docs/DistributorOS_Lessons_Learned.pdf`](docs/DistributorOS_Lessons_Learned.pdf).

## What I was trying to build

DistributorOS was my attempt at an operations layer for independent retailers to manage their suppliers, from the purchase order all the way to payment.

The idea came from a simple picture: a supermarket orders goods from a supplier, the delivery doesn't quite match the order, the invoice doesn't match either, and somewhere between WhatsApp messages, paper bills and the accounting software, nobody is sure what's actually owed. I wanted one place where the purchase order, the goods received and the supplier invoice could be checked against each other *before* anyone paid.

I also wanted to try something different with the interface. Instead of making people learn another set of screens, the retailer could use their own Claude subscription to talk to the system ("record that 95 of the 100 bags were accepted") over MCP. DistributorOS would stay the system of record and apply the business rules.

## What it does

- Supplier setup
- Purchase order creation
- Goods received, including accepted vs rejected quantities
- Invoice registration
- **Three-way matching** of PO, accepted goods and invoice. If there's a mismatch, the invoice is put on hold instead of being marked ready to pay
- Payment tracking and due dates (it doesn't move money or give credit)
- Supplier ledger
- Dashboard summary

## How it's built

```
Claude (customer's own subscription)
        │  natural language → picks an MCP tool
        ▼
MCP server (Streamable HTTP, bearer-token protected)
        │  7 tools
        ▼
Application / backend  ──►  Supabase (Postgres)
        │
        ▼
Web app (review and edit every record)
```

- **No AI calls inside the app.** All the reasoning happens in the customer's Claude. The app only validates, applies rules and stores data, so there are no API keys and no per-request AI costs.
- **MCP tools:** `create_supplier`, `create_purchase_order`, `record_goods_received`, `register_invoice`, `record_payment`, `get_supplier_ledger`, `get_dashboard_summary`.
- **Deterministic rules:** matching and the hold decision are plain code, not model output, so the same input always gives the same result.
- **Hosting:** app and MCP service on Render, database on Supabase.

## The demo workflow

1. Through Claude: create a supplier and a PO for 100 bags of basmati rice at ₹1,200 each (₹1,20,000).
2. Record the delivery: 100 bags arrive, 5 are damaged and rejected, 95 are accepted.
3. Register the supplier's invoice for all 100 bags.
4. DistributorOS compares the PO, the accepted goods and the invoice, finds a **₹6,000 discrepancy**, and puts the invoice on hold.
5. Open the web app. The records Claude created are all there and editable, and the invoice stays on hold until the difference is resolved.

The point of the demo was to show that this works in both directions: Claude creates real records, and a person can inspect and manage them in the app.

## Running it / connecting Claude

- Live app (may be asleep or taken down): https://distributoros.onrender.com
- MCP endpoint: `https://distributoros.onrender.com/mcp`
- In Claude, add a custom connector using Streamable HTTP and the endpoint above, with the header `Authorization: Bearer <MCP_TOKEN>`.

Tokens and demo logins are **not** in this repo, on purpose. Environment variables are listed in `.env.example`.

## What's in this repo

| Path | What it is |
| --- | --- |
| `/` (source) | Backend, MCP server and web app |
| `docs/DistributorOS_Business_Strategy.pdf` | My original strategy: problem, customer, pricing, go-to-market |
| `docs/DistributorOS_Product_Execution.pdf` | Prototype, architecture and Claude integration write-up |
| `docs/DistributorOS_Lessons_Learned.pdf` | Why I stopped, what the research showed, and what I learned |
| `docs/screenshots/` | Screens from the demo workflow |

## Why I stopped (short version)

When I finally researched the market properly, after I'd already built the prototype, I found that the core workflow already exists in software Indian retailers already use, at prices of a few thousand rupees a year. The retailers who'd actually need formal three-way matching are a small group that's already served. The smaller shops I'd pictured don't buy this way at all. I also had no real way to reach customers that incumbents don't already own.

The biggest lesson was about order: I started with a solution and went looking for a problem afterwards. Next time I'll start by watching the problem happen.

The full write-up, including my original assumptions next to what I found, is in [`docs/DistributorOS_Lessons_Learned.pdf`](docs/DistributorOS_Lessons_Learned.pdf).

---

*Jui Karkhele*
