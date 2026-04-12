# Sara — Self-hosted AI Voice Agent
**100% local · ₹0 LLM cost · WhatsApp + Telegram + Voice + Bulk calls**

---

## What you get
- **Dashboard** at `http://localhost:8000` — create agents via prompt, view all conversations, test live
- **WhatsApp bot** — customers message your WhatsApp number, Sara replies in Malayalam/English
- **Telegram bot** — easiest to set up, 30 minutes from zero to running
- **Bulk calling** — upload Excel sheet → Sara calls each number via Exotel
- **Fully local LLM** — Ollama runs on your machine, zero API cost

---

## Step 1 — Install Ollama (the local LLM)

```bash
# Linux / Mac:
curl -fsSL https://ollama.com/install.sh | sh

# Pull a Malayalam-capable model (3.8GB download):
ollama pull llama3.1

# Test it works:
ollama run llama3.1 "Say hello in Malayalam"
```

**For Windows:** Download from https://ollama.com/download

---

## Step 2 — Set up Sara

```bash
# Clone / copy the sara folder, then:
cd sara

# Create virtual environment
python -m venv venv
source venv/bin/activate   # Windows: venv\Scripts\activate

# Install dependencies
pip install -r requirements.txt

# Copy environment config
cp .env.example .env
```

---

## Step 3 — Start the server

```bash
uvicorn app.main:app --reload --port 8000
```

Open **http://localhost:8000** — you'll see the Sara dashboard.

Test it immediately: go to Dashboard → Quick test, type a message, click Send.

---

## Step 4 — Telegram bot (30 minutes)

1. Open Telegram → search **@BotFather**
2. Send `/newbot` → follow prompts → copy the **token**
3. Add to `.env`:
   ```
   TELEGRAM_BOT_TOKEN=your_token_here
   ```
4. Expose your local server (for webhook):
   ```bash
   # Install ngrok: https://ngrok.com/download
   ngrok http 8000
   # Copy the https URL, e.g. https://abc123.ngrok.io
   ```
5. Register webhook:
   ```bash
   curl "https://api.telegram.org/bot{YOUR_TOKEN}/setWebhook?url=https://abc123.ngrok.io/webhook/telegram"
   ```
6. Restart Sara, message your bot — Sara replies!

---

## Step 5 — WhatsApp (Meta Cloud API — free)

1. Go to **https://developers.facebook.com** → Create app
2. Add WhatsApp product → API Setup
3. Copy **Phone Number ID** and **Temporary Access Token**
4. Add to `.env`:
   ```
   WHATSAPP_ACCESS_TOKEN=your_token
   WHATSAPP_PHONE_NUMBER_ID=your_phone_id
   WHATSAPP_VERIFY_TOKEN=sara_verify_123
   ```
5. In Meta dashboard → Webhooks:
   - URL: `https://your-ngrok-url.ngrok.io/webhook/whatsapp`
   - Verify token: `sara_verify_123`
   - Subscribe to: `messages`
6. Send a WhatsApp message to your test number — Sara replies!

---

## Step 6 — Bulk calling (Exotel, India)

1. Sign up at **https://exotel.com** (pay-as-you-go, ₹1-2 per minute)
2. Add to `.env`:
   ```
   EXOTEL_SID=your_sid
   EXOTEL_TOKEN=your_token
   EXOTEL_FROM_NUMBER=+910XXXXXXXXXX
   ```
3. Create `calls.xlsx` with columns: `phone, name, notes`
4. Run bulk:
   ```bash
   python scripts/bulk_call.py --file calls.xlsx --mode whatsapp --limit 10
   ```

---

## File structure

```
sara/
├── app/
│   ├── main.py            ← FastAPI server (all routes)
│   ├── ollama_client.py   ← Local LLM calls
│   ├── db.py              ← SQLite database
│   ├── telegram_bot.py    ← Telegram sender
│   ├── whatsapp_client.py ← WhatsApp sender
│   └── templates/
│       └── dashboard.html ← Full web dashboard
├── scripts/
│   └── bulk_call.py       ← Excel bulk caller
├── requirements.txt
├── .env.example
└── README.md
```

---

## API endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/` | Dashboard UI |
| POST | `/chat` | Universal chat (all channels) |
| POST | `/webhook/telegram` | Telegram webhook |
| GET/POST | `/webhook/whatsapp` | WhatsApp webhook |
| GET | `/conversations` | List all chats |
| GET | `/conversations/{phone}` | Chat history |
| POST | `/agent/create` | Save agent config |
| GET | `/agent/list` | List agents |

---

## Customizing Sara's personality

Go to Dashboard → **Create agent**, describe your agent in plain text:

> "Create a polite Malayalam-speaking sales agent named Sara. She should greet customers warmly, explain our telecom offers clearly, handle objections with patience, and always end with 'നന്ദി, ഒരു നല്ല ദിവസം ആശംസിക്കുന്നു'"

Sara generates a full system prompt automatically using Ollama.

---

## Monthly cost estimate

| Item | Cost |
|------|------|
| LLM (Ollama, local) | ₹0 |
| Telegram bot | ₹0 |
| WhatsApp (Meta free tier) | ₹0 |
| Server (your PC / ₹500 VPS) | ₹0–500 |
| Exotel voice calls | ₹1–2 / min |
| **Total (without calls)** | **₹0** |
