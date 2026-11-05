# AI-Powered Automation with n8n: E-commerce Order Tracker

An n8n workflow that reads online-shopping order confirmation emails from Gmail, uses **Google Gemini AI** to extract the order details, and logs one row per order in **Google Sheets**. It works automatically, every minute.

**Built by:** Christine Massey · **Tools:** n8n, Gmail, Google Gemini, Google Sheets

## What it does

When an email like "Your order is confirmed" arrives from Amazon, Flipkart, Myntra, Nykaa or Swiggy, the workflow pulls out:

| Field | Example |
| --- | --- |
| Store | Myntra |
| Order ID | 1234-5678-90 |
| Items | Cotton kurta, Leggings |
| Amount and currency | 1499 INR |
| Order date and delivery date | 2026-10-22, 2026-10-26 |
| Category | Clothing (one of Electronics, Clothing, Groceries, Beauty, Books, Home, Food, Other) |
| AI reasoning | Why the AI chose that category, and which details were missing |

Each order becomes a new row in a Google Sheet, so all purchases are tracked in one place without any typing.

## How it works

```
Gmail Trigger → Filter → Prepare Email → Aggregate → AI Agent (Gemini + Output Parser) → Split Out → Format Row → Google Sheets → Mark as Read
```

1. **Gmail Trigger:** checks every minute for unread emails with "order confirmed", "order placed" or "payment successful" in the subject.
2. **Filter Order Keywords:** double-checks the subject line.
3. **Prepare Email for AI:** keeps only the useful parts of each email (sender, subject, date, body).
4. **Aggregate All Emails:** bundles every email from the run into one list. This is the key idea: 5 emails make **1 AI call instead of 5**, which is cheaper, faster and avoids rate limits.
5. **AI Order Extraction Agent:** a Google Gemini model reads the whole batch, and a backup Gemini model takes over if the first fails.
6. **Order Output Parser:** forces the AI to return clean, structured data in a fixed format. Missing details are left empty, never invented.
7. **Split Into One Order Per Item:** turns the batch back into one item per order.
8. **Format Sheet Row:** arranges the fields to match the sheet's columns.
9. **Log Order to Google Sheets:** adds one row per order.
10. **Mark Email as Read:** stops the same email from being logged twice.

## Files

| File | What it is |
| --- | --- |
| `ecommerce-order-tracker.workflow.json` | The complete n8n workflow, ready to import |
| `sheet-template.csv` | The column headers for the Google Sheet |
| `README.md` | This description |

## How to set it up

1. Create a Google Sheet and paste the headers from `sheet-template.csv` into row 1.
2. In n8n, choose **⋯ → Import from file** and select `ecommerce-order-tracker.workflow.json`.
3. Connect your own **Gmail**, **Google Sheets** and **Google Gemini** accounts on the nodes that need them. No passwords, API keys or account IDs are stored in this file; they show as `YOUR_..._ID` placeholders.
4. In **Log Order to Google Sheets**, select your own Google Sheet.
5. Turn the workflow on, and send yourself a test email with the subject "Order confirmed".

## Key concepts shown

- **Batching with the Aggregate node** to cut the number of AI calls
- **Structured output parsing**, so AI answers are reliable, consistent data
- **A fallback AI model** for resilience
- **Duplicate prevention** by marking processed emails as read
