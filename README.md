# Agentic Mail Sender Workflow

## Overview

This n8n workflow is a **Telegram-based AI assistant** that can send structured emails via Gmail and perform web searches on demand. Users interact with it through a Telegram chat — simply ask it to send an email or look something up, and the agent handles the rest.

---

## Architecture

A single pipeline: **Telegram → AI Agent (with tools) → Telegram reply**

The AI Agent is equipped with two tools it can call based on user intent:
- **Gmail Tool** — to compose and send emails
- **SerpAPI Tool** — to search the web

---

## Nodes & Components

| Node | Type | Purpose |
|------|------|---------|
| **Telegram Trigger** | Trigger | Listens for incoming messages from the user |
| **AI Agent** | Agent | Understands the user's intent and calls the right tool |
| **Groq Chat Model** | LLM | Powers the agent using `llama-3.1-8b-instant` via Groq |
| **Simple Memory** | Memory | Tracks conversation context (keyed by `update_id`) |
| **Send a message in Gmail** | Tool | Sends an email — recipient, subject, and body filled by AI |
| **SerpAPI** | Tool | Performs web searches when the user asks a question |
| **Send a text message** | Action | Sends the agent's reply back to the user on Telegram |

---

## How It Works

1. The user sends a message to the Telegram bot (e.g., *"Send an email to john@example.com about the project update"*)
2. The AI Agent interprets the request
3. If it's an email task → the **Gmail Tool** is called; the AI auto-fills the recipient, subject, and a structured message body
4. If it's a question or research task → **SerpAPI** is called to fetch relevant results
5. The agent replies back on Telegram with a brief status (e.g., *"Sent"* or *"Not sent"*)

---

## Email Format

The agent is instructed to always compose emails in a structured format:

```
Subject: <subject line>

<email body>

Thank you
```

- No sender name is included
- Replies to the user are kept minimal (just a status notification)

---

## Configuration & Credentials

| Credential | Used By |
|-----------|---------|
| `Telegram_bot` | Telegram Trigger, Send a text message |
| `Groq account` | Groq Chat Model |
| `Gmail account` | Send a message in Gmail |
| `SerpAPI account` | SerpAPI |

---

## Setup Instructions

1. **Telegram Bot**: Create a bot via [@BotFather](https://t.me/BotFather) and add the token to the `Telegram_bot` credential.
2. **Groq**: Sign up at [groq.com](https://groq.com) and add your API key to the `Groq account` credential.
3. **Gmail**: Authenticate your Google account via OAuth2 in the `Gmail account` credential.
4. **SerpAPI**: Get an API key from [serpapi.com](https://serpapi.com) and add it to the `SerpAPI account` credential.
5. **Activate** the workflow in n8n.

---

## Notes & Limitations

- **Memory is per-update, not per-chat**: The session key is `update_id`, which is unique per message — meaning the agent does **not** retain conversation history across turns. To fix this, change the session key to `$json.message.chat.id` (same approach as the RAG workflow).
- **Workflow is currently inactive** (`"active": false`). Activate it before use.
- The agent uses `llama-3.1-8b-instant` (fast and free-tier friendly), but it may struggle with complex multi-step instructions. Swap to a larger model if needed.
- The Gmail tool dynamically extracts the recipient, subject, and message body from the user's request using AI — no hardcoded values.
