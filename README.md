# Email Automation

This repository contains three Workato MCP apps that support the email campaign workflow. Tactic Selection and Brief Builder are currently **v1**. Email Studio is **v2.2**.

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
| Email Studio | v2.2 | Saving and pushing overlays, approval notice, approval button fix. See the changelog below. |

When Sandbox changes are promoted to Production, bump the version here and add an entry to the app's changelog. Production keeps a single file; earlier versions live in the git history. Put the release date under each changelog entry.

## Email Studio Changelog

### v2.2

Released 2026-11-10. Promoted from Sandbox to Production. Production is now identical to Sandbox and holds a single file (the archive folder was removed).

**Blocking overlays**
- Saving now shows the same translucent overlay as Build, reading "Saving changes…". It also covers the save that runs when a task is selected or created. Nothing underneath can be clicked while it is showing.
- Push to brief shows the overlay with "Pushing content to your tactic brief…".
- Build keeps "Building email…".

**After approval**
- Once a pre-event or post-event email is approved successfully, a notice appears saying the email asset is built and the requester still needs to update the audience in the brief and approve the brief under the tactic. It has a Close button (Esc also closes it). Reminder and confirmation-family emails do not get it.
- The progress bar's next-step line for an approved email now reads "Next: Update audience in your brief and approve it in your tactic" (it used to say "All done, this email is approved").

**Bug fix**
- After approving one email, the Approve button inside the approval dialog stayed disabled for every other email until the app was reloaded. The dialog is shared by all emails and its button was only re-enabled after a failed approval. It is now re-enabled after a successful one and whenever the dialog opens.

### v2.1

Released 2026-11-10. Promoted from Sandbox to Production.

**Send date rules**
- Post-event emails (registrant, attendee, no-show) and reminder emails (all `reminder_*` types): the send date must be at least 3 days from today (the next two days are blocked) and cannot fall on a Saturday or Sunday.
- Confirmation-family emails (event cancellation, event postponement, confirmation, waitlist, KBYG): any date from today onwards, weekends allowed. These previously followed the reminder rule.
- Pre-event emails: unchanged, at least 7 days from today.
- Dates before the minimum are greyed out in the calendar. The browser's date picker cannot grey out individual weekend days, so a weekend date can still be picked. The Send date field then turns red with a message, and Save draft, Push to brief and Build stay blocked until a valid date is set.
- The help text under Send date now shows the rule for that email's family.

### v2

Released 2026-11-10. Promoted from Sandbox to Production. Production is now identical to Sandbox.

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

Released 2026-11-02. Initial release. Sidebar of emails, per-email header, side-by-side preview and settings, popup rich-text editor, and an action row with Save, Push to Brief, Build Email and Approve.
