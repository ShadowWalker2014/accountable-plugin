# Accountable plugins

Accountable keeps double-entry books for founders who run one or more companies. This repository holds the plugins that connect Claude and ChatGPT to Accountable, so they can read and keep those books: reports, cash and runway, transactions, bills and invoices, and the month's close.

- [claude-plugin](claude-plugin): the Claude plugin. It adds Accountable's MCP server (`https://accountable.im/mcp`) as a connector, and skills that guide Claude through bookkeeping work.
- [chatgpt-plugin](chatgpt-plugin): the ChatGPT plugin. It connects to `https://accountable.im/mcp/chatgpt` and bundles the skills that fit its tools.

Both plugins only connect to Accountable's own server, over OAuth with your Accountable account. They run nothing on your computer, have no hooks or scripts, and never move money. Every change is logged and can be undone.

Help: https://accountable.im/help/agents/agents-overview. Privacy: https://accountable.im/legal/privacy. Terms: https://accountable.im/legal/terms.

Built from Accountable's main repository; changes here are overwritten by the next copy.
