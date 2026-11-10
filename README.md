# Email Automation

This repository contains three Workato MCP apps that support the email campaign workflow. Tactic Selection and Brief Builder are currently **v1**. Email Studio is **v2**.

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

## Versions

| App | Version | Notes |
| --- | --- | --- |
| Tactic Selection | v1 | Initial release. |
| Brief Builder | v1 | Initial release. |
| Email Studio | v2 | Sandbox changes promoted to Production. See the changelog below. |

When Sandbox changes are promoted to Production, bump the version here and add an entry to the app's changelog. The previous Production file is kept in `Production/archive/`.

## Email Studio Changelog

### v2

Promoted from Sandbox to Production. Production is now identical to Sandbox. The v1 file is kept at `Email Studio/Production/archive/email_studio_v1.html`.

**Layout and navigation**
- The app is now a fixed-height panel (max width 760px) that scrolls inside itself. `fitStudioHeight()` sizes it from the progress bar, template and layout box, and email preview.
- The emails list is a horizontal strip at the top instead of a left sidebar.
- The old per-email header (email index, title, stage badge) and the collapsible settings panel are removed. The settings panel is always open and sits above the preview.
- The "Template and layout" selectors now live in their own box.

**Progress tracker and guidance**
- The bottom action row is replaced by a sticky progress tracker: Save draft, Push to brief, Build email, Approve. Completed steps turn green with a check, and the connecting lines fill as you go.
- The tracker shows a status line (for example "Draft saved" or "Unsaved changes") and a "Next:" line. A red "*" line means a required item is missing.
- Pulsing highlights and an arrow chip point to the next section or field to use.
- A floating next-step button follows the visible area once the tracker scrolls out of view. A ring animation plays when a step completes.
- Email status pills in the strip follow the same stages as the tracker and update as you type.
- Built (not yet approved) emails stay editable. Editing one marks it dirty, and saving sends it back through Push and Build. Only approval locks the content and settings.

**Editing**
- Preview text is edited inline in the email, with a floating formatting toolbar and Cancel/Done buttons. Escape cancels and Ctrl/Cmd+Enter finishes. The old popup editor remains as a fallback if the inline editor cannot start.
- Indent, list and formatting commands are shared between both editors through `applyEditorCommand()`. A stray debug `console.log` was removed.
- Each field has a "?" button that shows its help text, and each section has a short description.
- The banner field has a copy button for the `mktg-requests@mongodb.com` sharing address.
- The subject and preheader character count now shows inside the input, and the counter turns red over 50.
- The default "Hi there," salutation was removed. The salutation now comes from the data.

**Email types and rules**
- Reminder emails (any `reminder_*`, including time-based types such as `reminder_2h` or `reminder_1_week`) use a locked design: Operational-Update template and Grey-Card layout. The template and layout box is hidden for them.
- Reminder emails skip "Push to brief" and go straight from a saved draft to Build.
- New confirmation-family email types (cancellation, postponement, confirmation, waitlist, KBYG) use the reminder design and workflow. They keep their original `email_type` when sent to the pipeline.
- New `post_event_no_show` email type is supported as post-event, like registrant and attendee: post-event templates, tomorrow-or-later send date, and sorted with the post-event emails.
- Emails are ordered: confirmation-family, then reminders, then pre-event, then post-event. Order within each group follows sequence.
- Send date minimums depend on type: reminders today or later, post-event tomorrow or later, pre-event at least 7 days out. An early date shows a red field and message.
- Push All skips reminder emails.
- The Pull / Save flow and the `Email_Release_Pipeline` payload are otherwise unchanged.

**Approval**
- The "Recipient time zone" option now sends the chosen hour and minute (`email_schedule_time`) along with `recipient_time_zone: true`. The hour and minute pickers are shared by both tabs.
- For reminder emails the "Recipient time zone" tab is hidden and only "Set time" is offered.
- The approval dialog can open from the floating button.

### v1

Initial release. Sidebar of emails, per-email header, side-by-side preview and settings, popup rich-text editor, and an action row with Save, Push to Brief, Build Email and Approve.
