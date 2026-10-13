# mcp-mail

Superhuman Mail MCP server. Connects an AI assistant to a Superhuman Mail account so it can search mail and calendar, draft and send in your voice, and handle inbox and calendar work from the chat.

Requires a Business or Enterprise plan with Ask AI enabled. The first connection opens an OAuth sign-in.

Add to your MCP client configuration:

```json
{
  "mcpServers": {
    "Superhuman Mail": {
      "command": "npx",
      "args": ["-y", "@superhuman/mcp-mail"]
    }
  }
}
```

## What it can do

- Search email and calendar in natural language, or filter threads by sender, label, and Split Inbox.
- Read a thread, a message, or an attachment, and see who opened mail you sent.
- Draft and revise email in your writing style, then send it, schedule it, or undo it inside the send window.
- Find a meeting time and create or update the event.
- Set reminders, mark threads done or read, star them, change labels, unsubscribe, or move a thread to trash or spam.
- Link more than one Superhuman account and choose which one is the default.

## Tools

- **Search and read:** `query_email_and_calendar`, `list_threads`, `get_thread`, `get_message`, `get_attachment`, `list_labels`, `list_splits`, `get_read_status_feed`
- **Drafts and snippets:** `create_or_update_draft`, `list_drafts`, `get_draft`, `send_draft`, `undo_send`, `get_draft_send_status`, `discard_draft`, `list_snippets`, `get_snippet`, `create_or_update_snippet`, `discard_snippet`, `update_personalization`
- **Calendar and reminders:** `get_availability`, `create_or_update_event`, `create_or_update_reminder`, `list_reminders`, `discard_reminder`
- **Inbox:** `update_thread`, `trash_thread`, `mark_spam`, `unsubscribe`
- **Accounts:** `list_accounts`, `add_account`, `remove_account`, `set_default_account`

## Hosted MCP server

This package connects to `https://mcp.mail.superhuman.com/mcp`. Clients that support a remote HTTP server can use that URL directly. OAuth still runs on first connection, and the tools above are available only for the signed-in account.