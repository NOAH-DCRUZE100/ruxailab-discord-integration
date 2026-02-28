# ruxailab-discord-integration
# 🔔 GitHub Actions × Discord Integration — PoC
> **GSoC 2026 Proof of Concept** | RUXAILAB  
> *Integration of GitHub Actions with Discord Role Management*

---

## 📌 What This Does

Every time a **push** is made to any branch of this repository, a
GitHub Actions workflow fires and sends a rich notification card to a
Discord channel via a **Webhook**.  

The card includes:

| Field | Source |
|---|---|
| 👤 Pusher's GitHub username | `github.actor` |
| 🌿 Branch name | `github.ref_name` |
| 🔖 Commit SHA (linked) | `github.sha` |
| 💬 Commit message | `github.event.head_commit.message` |

The webhook URL is stored as a **GitHub Secret** — it is never
exposed in code or logs.

---

## 🗂️ File Structure

```
.github/
└── workflows/
    └── notify.yml   ← The workflow (this PoC)
README.md            ← You are here
```

---

## ⚙️ Setup Guide

### Step 1 — Create a Discord Webhook

1. Open your Discord server and go to the target channel.  
2. Click **Edit Channel → Integrations → Webhooks → New Webhook**.  
3. Give it a name (e.g., `GitHub Notifier`) and copy the **Webhook URL**.

> ⚠️ Treat this URL like a password. Anyone with it can post to your
> channel.

---

### Step 2 — Add the Secret to GitHub

1. Navigate to your repository on GitHub.  
2. Go to **Settings → Secrets and variables → Actions**.  
3. Click **New repository secret**.  
4. Set the name to exactly:

   ```
   DISCORD_WEBHOOK
   ```

5. Paste the Webhook URL you copied as the value, then click **Add secret**.

The workflow references it as `${{ secrets.DISCORD_WEBHOOK }}` — GitHub
masks this value in all logs automatically.

---

### Step 3 — Add the Workflow File

Copy `.github/workflows/notify.yml` from this repository into your own
project, preserving the directory path, then push to GitHub:

```bash
git add .github/workflows/notify.yml
git commit -m "feat: add Discord push notification workflow"
git push
```

The commit you just pushed will itself trigger the workflow — you should
see a notification appear in your Discord channel within seconds.

---

## 🧪 Testing the Integration

### Option A — Push any commit (simplest)
```bash
echo "test" >> test.txt
git add test.txt
git commit -m "test: trigger Discord notification"
git push
```

### Option B — Manually trigger (without pushing)
Add `workflow_dispatch:` under `on:` in the YAML, then go to:

**Actions → 🔔 Discord Push Notification → Run workflow**

### Option C — Validate the payload locally with `curl`
Replace `<YOUR_WEBHOOK_URL>` with your actual URL to test the shape of
the payload before committing:

```bash
curl -X POST "<YOUR_WEBHOOK_URL>" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "👋 Hello World! Manual test.",
    "embeds": [{
      "title": "Local curl test",
      "color": 5793266,
      "fields": [
        { "name": "👤 Pushed by", "value": "your-username", "inline": true },
        { "name": "💬 Commit Message", "value": "Test from curl", "inline": false }
      ]
    }]
  }'
```

A `204 No Content` response means Discord accepted the payload ✅.

---

## 🔒 Security Notes

| Practice | Why it matters |
|---|---|
| Secret stored in GitHub Secrets | URL never appears in source code or CI logs |
| `ubuntu-latest` runner | Clean, minimal environment; no stale dependencies |
| `SystemExit(1)` on HTTP error | Fails the step visibly instead of silently succeeding |
| No third-party Action used | Pure `curl`/Python means zero supply-chain risk |

---

## 🚀 Extending This PoC

This is the foundation for the full GSoC project. Next steps include:

- **Role assignment**: Call the Discord REST API to add/remove roles
  based on the contributor's activity (e.g., first merged PR → assign
  `Contributor` role).
- **Event filtering**: React differently to `pull_request`, `issues`,
  and `release` events.
- **Slash commands**: Let Discord members query GitHub stats directly
  from Discord.

---

## 📄 License

MIT — free to use, modify, and share.
Verification complete. Final PoC test.
