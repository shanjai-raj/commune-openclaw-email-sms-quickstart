# Setup Guide — Commune + OpenClaw Integration

Step-by-step instructions for connecting your OpenClaw agent to Commune email.

---

## Step 1: Get Your Commune API Key

1. Sign up at [commune.email](https://commune.email)
2. Go to your dashboard → API Keys
3. Create a new key — it will start with `comm_`
4. Copy it somewhere safe. You won't see it again after this screen.

---

## Step 2: Install the Skills

```bash
# Clone the repo
git clone https://github.com/commune-dev/commune-openclaw-email-quickstart
cd commune-openclaw-email-quickstart

# Copy skills into your OpenClaw workspace
cp -r skills/commune-email ~/.openclaw/workspace/skills/

# Make the helper scripts executable
chmod +x ~/.openclaw/workspace/skills/commune-email/commune.js
```

Verify the skill appeared:

```bash
openclaw skill list | grep commune
# commune-email
```

---

## Step 3: Set Environment Variables

Add these to your shell profile (`~/.zshrc` or `~/.bashrc`) or your server's environment config. Fill in the values as you complete subsequent steps.

```bash
# Required
export COMMUNE_API_KEY=comm_your_key_here

# Set after Step 4 (inbox creation)
export COMMUNE_INBOX_ID=inbox_xxx
export COMMUNE_INBOX_ADDRESS=assistant@yourdomain.commune.email
```

After editing your shell profile, reload it:

```bash
source ~/.zshrc   # or source ~/.bashrc
```

---

## Step 4: Create Your Inbox

Either ask your agent directly:

```
You: Create a Commune inbox called "assistant"
Agent: Created inbox: assistant@yourdomain.commune.email | ID: inbox_xxx
```

Or via the CLI helper:

```bash
node ~/.openclaw/workspace/skills/commune-email/commune.js create-inbox assistant
```

Output:
```
Inbox created successfully.
Address: assistant@yourdomain.commune.email
Inbox ID: inbox_xxx

Add these to your environment:
  export COMMUNE_INBOX_ID=inbox_xxx
  export COMMUNE_INBOX_ADDRESS=assistant@yourdomain.commune.email
```

Copy the values and add them to your environment (Step 3).

**Note:** The inbox address is permanent. Choose a local part (`assistant`, `support`, `me`) that makes sense for your use case.

---

## Step 5: Verify the Integration

Test that everything is connected:

```bash
# Test email — create a scratch inbox
node ~/.openclaw/workspace/skills/commune-email/commune.js create-inbox test

# Test list threads (should return empty or your threads)
node ~/.openclaw/workspace/skills/commune-email/commune.js list-threads $COMMUNE_INBOX_ID
```

Send a test email to your new inbox from any email client, wait 30 seconds, then:

```bash
node ~/.openclaw/workspace/skills/commune-email/commune.js list-threads
# [WAITING REPLY] thread_xxx | 1 msg | Test email
```

If you see your test email listed, the integration is working.

---

## Step 6: Tell Your Agent

Add your inbox details to `~/.openclaw/workspace/USER.md` so the agent knows about them:

```markdown
## Communication

### Email
My Commune inbox: assistant@yourdomain.commune.email
Inbox ID: inbox_xxx

When checking email:
- Prioritize threads with last_direction: inbound (waiting for reply)
- Skip newsletter/automated notifications unless I ask
- Reply in my voice — casual but professional
```

For a company agent, update the agent's `SOUL.md` with its inbox and responsibilities. See [../use-cases/company-assistant/README.md](../use-cases/company-assistant/README.md) for a full template.

---

## Step 7: Test With Your Agent

Restart OpenClaw to pick up the new skills and environment variables, then test:

```
You: Check my Commune email inbox
Agent: [lists your threads]

You: Send a test email to yourself@gmail.com with subject "Hello from my agent"
Agent: [sends via Commune]
```

---

## Troubleshooting

### "Agent doesn't seem to use Commune"

1. Verify the skill files are in the right place:
   ```bash
   ls ~/.openclaw/workspace/skills/commune-email/
   # SKILL.md  commune.js
   ```
2. Check that `COMMUNE_API_KEY` is actually set in the environment OpenClaw runs in:
   ```bash
   echo $COMMUNE_API_KEY
   ```
3. Restart OpenClaw after setting environment variables — it reads env on startup.
4. Explicitly mention Commune in your request: "Use the Commune email skill to check my inbox."

---

### "Authentication error (401)"

Your API key is invalid or expired.

1. Go to [commune.email/dashboard](https://commune.email/dashboard) → API Keys
2. Verify the key matches exactly what's in your environment (no trailing spaces)
3. If needed, rotate the key and update `COMMUNE_API_KEY`

---

### "No threads showing"

1. Check that `COMMUNE_INBOX_ID` is set and correct:
   ```bash
   echo $COMMUNE_INBOX_ID
   ```
2. Send a test email to your inbox address and wait 30 seconds
3. Confirm the inbox address is correct:
   ```bash
   node ~/.openclaw/workspace/skills/commune-email/commune.js create-inbox check
   # If you get a "already exists" error, the address is correct
   ```
4. Try listing with the inbox ID explicitly:
   ```bash
   node ~/.openclaw/workspace/skills/commune-email/commune.js list-threads inbox_xxx
   ```

---

### "Replies are starting new threads instead of replying"

The `thread_id` is not being passed in the send request. Make sure the agent:
1. Reads the thread first to get the `thread_id`
2. Passes `thread_id` in the body of `POST /v1/messages/send`

In SKILL.md this is documented as critical — if the agent is skipping it, add an explicit note to your `SOUL.md` or `USER.md`: "Always include thread_id when replying to email."

---

### "Search isn't finding emails I know exist"

1. Vector indexing takes a short time after new emails arrive — wait 1-2 minutes after a new email arrives before searching
2. Try rephrasing — semantic search responds to meaning, not just keywords
3. For exact matches, use `list-threads` and scan subject lines instead

---

## Environment Variable Reference

| Variable | Required | Description | Where to find it |
|----------|----------|-------------|-----------------|
| `COMMUNE_API_KEY` | Yes | Your Commune API key | commune.email/dashboard → API Keys |
| `COMMUNE_INBOX_ID` | Recommended | Default inbox ID | Response from `create-inbox` |
| `COMMUNE_INBOX_ADDRESS` | Recommended | Full inbox address | Response from `create-inbox` |

---

## What's Next

- [Personal assistant use case](../use-cases/personal-assistant/README.md) — prompts and setup for personal email management
- [Company agent use case](../use-cases/company-assistant/README.md) — deployment patterns for customer-facing agents
- [commune-email SKILL.md](../skills/commune-email/SKILL.md) — full email API reference for your agent
