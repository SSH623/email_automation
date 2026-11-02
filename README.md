# Email Automation

This repository contains three Workato MCP apps that support the email campaign workflow. All three apps are currently **v1**.

## Apps

| App | Purpose |
| --- | --- |
| **Tactic Selection** | Search and filter the available tactic library by status, then send a selected tactic and its context to chat. |
| **Brief Builder** | Validate a tactic's email support, load its generated brief, choose or edit suggested copy, optionally enhance fields with AI, and send a structured brief to the email composer. |
| **Email Studio** | Load saved email drafts for a tactic, edit and preview email content, and customize templates, layouts, rich text, calls to action, and banners. |

## Repository Layout

Each app has separate `Production` and `Sandbox` folders, with its HTML app entry point in each:

```text
Brief Builder/
	Production/email_brief_builder.html
	Sandbox/email_brief_builder.html
Email Studio/
	Production/email_studio.html
	Sandbox/email_studio.html
Tactic Selection/
	Production/tactic_selection_app.html
	Sandbox/tactic_selection_app.html
```

Use the `Production` copy for the deployed app and the `Sandbox` copy for development and testing.

## Runtime

These are MCP apps intended to run inside their host environment, not standalone web pages. They connect to Workato MCP tools to retrieve tactic and email data and to perform brief-generation and enhancement steps. The HTML files load app and editor dependencies from CDNs, so the host needs network access to those dependencies and the configured Workato tools.
