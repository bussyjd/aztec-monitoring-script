# Slack Notifications Setup Guide

This guide explains how to configure the Aztec monitoring script to send notifications to Slack instead of (or in addition to) Telegram.

## Prerequisites

- A Slack workspace where you have permissions to create apps/webhooks
- The Aztec monitoring script installed on your server

## Step 1: Create a Slack Incoming Webhook

1. Go to [Slack API Apps](https://api.slack.com/apps)
2. Click **Create New App**
3. Select **From scratch**
4. Enter an App Name (e.g., "Aztec Monitor") and select your workspace
5. Click **Create App**

### Enable Incoming Webhooks

1. In your app settings, go to **Incoming Webhooks** in the left sidebar
2. Toggle **Activate Incoming Webhooks** to **On**
3. Click **Add New Webhook to Workspace**
4. Select the channel where you want notifications to be posted
5. Click **Allow**
6. Copy the **Webhook URL** - it will look like:
   ```
   https://hooks.slack.com/services/TXXXXX/BXXXXX/XXXXXXXXXX
   ```

## Step 2: Configure the Monitoring Script

When you run the monitoring script and select option **2** (Install node monitoring agent with notifications), you will be prompted to choose your notification channel:

```
Select notification channel:
1. Telegram only
2. Slack only
3. Both Telegram and Slack
```

### Option 1: Slack Only

1. Select option **2** (Slack only)
2. When prompted, paste your Slack Webhook URL:
   ```
   Enter Slack Webhook URL:
   > https://hooks.slack.com/services/TXXXXX/BXXXXX/XXXXXXXXXX
   ```
3. The script will send a test message to verify the webhook works
4. Continue with the rest of the setup

### Option 2: Both Telegram and Slack

1. Select option **3** (Both Telegram and Slack)
2. First, enter your Telegram Bot Token and Chat ID as usual
3. Then enter your Slack Webhook URL
4. Both channels will receive notifications

## Environment Variables

The script stores the following variables in `~/.env-aztec-agent`:

| Variable | Description | Example |
|----------|-------------|---------|
| `NOTIFICATION_CHANNEL` | Notification destination | `1` (Telegram), `2` (Slack), `3` (Both) |
| `SLACK_WEBHOOK_URL` | Your Slack webhook URL | `https://hooks.slack.com/services/...` |
| `TELEGRAM_BOT_TOKEN` | Telegram bot token (if using Telegram) | `123456789:ABC...` |
| `TELEGRAM_CHAT_ID` | Telegram chat ID (if using Telegram) | `-1001234567890` |

## Manual Configuration

If you need to change your notification settings after initial setup, you can edit the environment file directly:

```bash
nano ~/.env-aztec-agent
```

Update the relevant variables:

```bash
# For Slack only
NOTIFICATION_CHANNEL="2"
SLACK_WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"

# For both Telegram and Slack
NOTIFICATION_CHANNEL="3"
TELEGRAM_BOT_TOKEN="your_telegram_bot_token"
TELEGRAM_CHAT_ID="your_telegram_chat_id"
SLACK_WEBHOOK_URL="https://hooks.slack.com/services/YOUR/WEBHOOK/URL"
```

After editing, reinstall the monitoring agent (option 2 in the menu) or restart the systemd service:

```bash
sudo systemctl restart aztec-agent.timer
```

## Notification Types

All notification types work with Slack:

- **Critical errors** - Node container issues, sync problems
- **Block sync status** - Node falling behind
- **Committee participation** - When your validator is selected
- **Validator queue updates** - Position changes in the queue
- **Publisher balance warnings** - Low balance alerts

## Slack Message Formatting

Notifications in Slack will appear with:
- Emoji indicators for status
- Monospace formatting for addresses and technical data
- Server IP and timestamp information

Example notification:
```
🚨 Critical error detected
🌐 Server: 123.45.67.89
ERROR: Connection timeout
Solution:
Check your RPC endpoint and network connectivity
🕒 2025-01-07 12:34:56
```

## Troubleshooting

### Webhook Validation Failed

If you see "Invalid Slack webhook URL":

1. Verify the URL starts with `https://hooks.slack.com/services/`
2. Check that the webhook is still active in your Slack app settings
3. Ensure your server can reach `hooks.slack.com` (check firewall rules)

### No Notifications Received

1. Check the channel selected for the webhook in Slack
2. Verify the webhook hasn't been revoked
3. Check the agent logs:
   ```bash
   cat ~/aztec-monitoring/agent.log
   ```

### Test Your Webhook Manually

```bash
curl -X POST -H 'Content-Type: application/json' \
  -d '{"text":"Test message from Aztec Monitor"}' \
  YOUR_WEBHOOK_URL
```

## Security Considerations

- Keep your webhook URL confidential - anyone with it can post to your channel
- The webhook URL is stored in `~/.env-aztec-agent` with standard file permissions
- Consider using a dedicated channel for monitoring alerts
- Regularly rotate webhooks if you suspect they've been compromised

## Switching Between Channels

To switch from Telegram to Slack (or vice versa):

1. Edit `~/.env-aztec-agent`
2. Update `NOTIFICATION_CHANNEL` to your desired value
3. Add the required credentials for your new channel
4. Restart the monitoring service

## Support

For issues or questions:
- Open an issue on the GitHub repository
- Contact the developer via Telegram: https://t.me/+zEaCtoXYYwIyZjQ0
