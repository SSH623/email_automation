# Email Automation

Email Automation is one project made of four Workato MCP apps. Together they cover the email campaign workflow, from picking a tactic to building and approving its emails. Tactic Selection and Brief Builder are **v1**, Email Studio is **v2.3**, and the Email Revision App is in progress.

## Workflow

1. **Tactic Selection**: find the tactic to work on.
2. **Brief Builder**: build the email brief for that tactic.
3. **Email Studio**: write, preview, build and approve the tactic's emails.
4. **Email Revision App** (in progress): revise emails after they are built.

## Apps

| App | Purpose | Status |
| --- | --- | --- |
| **Tactic Selection** | Search and filter the available tactic library by status, then send a selected tactic and its context to chat. | v1 |
| **Brief Builder** | Validate a tactic's email support, load its generated brief, choose or edit suggested copy, optionally enhance fields with AI, and send a structured brief to the email composer. | v1 |
| **Email Studio** | Load saved email drafts for a tactic, edit and preview email content, and customize templates, layouts, rich text, calls to action, and banners. | v2.3 |
| **Email Revision App** | Revise emails. Development has started in Sandbox. | In progress |

## Repository Layout

Each app has its own folder with separate `Production` and `Sandbox` folders, and its HTML app entry point in each:

```text
email_automation/
	README.md
	Tactic Selection/
		Production/tactic_selection_app.html
		Sandbox/tactic_selection_app.html
	Brief Builder/
		Production/email_brief_builder.html
		Sandbox/email_brief_builder.html
	Email Studio/
		Production/email_studio.html
		Sandbox/email_studio.html
	Email Revision App/
		Sandbox/email_revision.html
```

Use the `Production` copy for the deployed app and the `Sandbox` copy for development and testing. The Email Revision App has no Production copy yet.

## Runtime

These are MCP apps intended to run inside their host environment, not standalone web pages. They connect to Workato MCP tools to retrieve tactic and email data and to perform brief-generation and enhancement steps. The HTML files load app and editor dependencies from CDNs, so the host needs network access to those dependencies and the configured Workato tools.

## App Setup

Each app's MCP UI resource must allow the external domains the app loads from. The browser blocks any domain that isn't allowed. Blocked scripts stop the app from loading. Blocked images show their alt text instead, for example "Event banner". Allowing these domains is part of the server configuration, not the HTML, so check them every time an app is set up or promoted to Production.

| Domain | Used for | Apps |
| --- | --- | --- |
| `cdn.jsdelivr.net` | MCP Apps SDK (`@modelcontextprotocol/ext-apps`), and the driver.js guided tour (script and stylesheet) | All |
| `esm.sh` | Tiptap rich text editor (`@tiptap/core`, `starter-kit`, `extension-underline`, `extension-link`) | Email Studio, Email Revision App |
| `info.mongodb.com` | MongoDB logo and default banner image in the email preview | Email Studio, Email Revision App |
| Host of the uploaded banner image | The `banner_image_url` returned by `Email_Studio_Utilities` (`process: "banner_upload"`) | Email Studio, Email Revision App |

If an app loads from a new domain, add it to this table and to the app's allowed domains.

## Versions

| App | Version | Notes |
| --- | --- | --- |
| Tactic Selection | v1 | Initial release. Production and Sandbox are identical. |
| Brief Builder | v1 | Initial release. Sandbox adds help text for the event fields (time zone, start time, end time, location, title) that Production does not have yet. |
| Email Studio | v2.3 | Fixed MongoDB logo image in every email preview. See the changelog below. |
| Email Revision App | v1.3 (Sandbox) | Loads one email for revision, with Enhance with AI on body paragraphs. Saves through `Email_Brief_Schema` "Save_Revision". Single-email push only, and the CTA URL keeps its UTM parameters. Audience-secured or within-24-hours emails keep their received send date, and pre-event/post-event ones also keep their template and layout. Not in Production yet. See the changelog below. |

When Sandbox changes are promoted to Production, bump the version here and add an entry to the app's changelog. Production keeps a single file; earlier versions live in the git history. Put the release date under each changelog entry.

## Email Revision App Changelog

Sandbox only for now (`Email Revision App/Sandbox/email_revision.html`, started as a copy of Email Studio v2.3). Each push adds the next version here. Commits made between two pushes all belong to the same version and are listed under it.

### v1.3

Committed 2026-11-15.

**Restricted emails keep their template and layout**
- Applies to pre-event and post-event emails when any of these flags from `Email_Editor_Utility` is true: `restrictions_audience_secured`, `audience_secured` (always sent together with the first), `restrictions_send_within_24_hours`. The audience is already secured or the email sends within 24 hours, so only the copy (subject, preheader, body, CTA text) can still change.
- The whole "Template and layout" box is hidden. The content preview shows only the template and layout the email was received with.
- The "Saved draft" picker is hidden too, so earlier drafts can't be seen or loaded. Picking a saved draft is also blocked in code.
- If the received template or layout is missing or unknown, the preview falls back to `Promotional-Text-Header + CTA + Banner` (pre-event, shows every token field) or `Post-Event-Follow-Up` (post-event), with `Grey-Card-Layout`.
- The "Select your template and layout" next-step guidance is skipped, and the help tour leaves out the Template and Layout steps because their sections are hidden.
- Unrestricted pre-event/post-event emails, reminders and confirmation-family emails (confirmation, waitlist, cancellation, postponement, KBYG) work as before, tour included.
- The flags are still sent back unchanged on every save.

**Restricted emails keep their send date**
- When any of the same three flags is true, the send date the email was received with (`due_date`) is fixed. The app keeps it and sends it unchanged on every save (Save_Revision) and every Email_Release_Pipeline call.
- The Send date field is hidden, and Reply-to and From name share the row as two columns, as for confirmation and waitlist emails.
- Applies to every email type that has the flags, not only pre-event and post-event. Confirmation and waitlist emails are unchanged: their field was already hidden and they still send today's date.
- The held date is not checked against the send date rules (minimum days ahead, no weekends), so it never shows a "Set a send date" warning or blocks Save, Push, Build or Approve. It may be within 24 hours, and there is no field to fix it.
- If a restricted email arrives with no `due_date`, the app sets one on load with the calendar's rule: the earliest allowed day (pre-event 7 days from today, reminder and post-event 3 days, confirmation family today), moved forward past any Saturday or Sunday. That date is then held like a received one. A received date is never changed.

### v1.2

Committed 2026-11-14.

**Save through Save_Revision**
- Every save now calls `Email_Brief_Schema` with `path: "Save_Revision"` instead of "Save". This covers Save draft and the background saves after Push to brief (`briefed`), Build (`build`) and Approve (`approved`). The email is stored as one document in `AI_Email_Editor`.
- Request: top-level `tactic_link` plus `revision_email[]` with the one email being saved. `tactic[]`, `emails[]` and `brief_document_id` are no longer sent.
- `subject_line_1` and `preheader_text_1` carry whatever is in the subject and preheader boxes. Options 2 and 3 and the selected-index fields are no longer sent.
- Sends the required `restrictions_*` flags, `email_template_found` and `email_layout_found` (always true/false), plus the optional fields (`email_schedule`, `audience_secured`, `send_datetime` and the other restrictions).
- `draft_datetime` is set to the save time. The recipe sets its own value.
- Task fields and UTMs are optional, so confirmation and waitlist emails without a task save normally.
- From name defaults to `MongoDB` when it is empty, on load and on save.

**Result and retry**
- Only `process_flag` is read from the response. The app does not need the email, brief document or brief tabs back; those come from `Email_Editor_Utility` "get_email_draft".
- A save succeeds when `process_flag` is `true` (a boolean or the string `"true"`). Otherwise the save is retried once automatically.
- If the retry also fails, the app shows "Save failed, please try again." This replaces the old "contact MOPs" save message.

**Push to brief is single-email only**
- The app always edits one email, so the "Push all to brief" option is removed from both the progress bar and the floating next-step button.
- Push to brief in the progress bar is now a plain button like Build email; the ▾ toggle and its menu are gone, along with their code and styles.
- A push always sends just the current email with `process_name: "brief"`. `"brief all"` is no longer sent.
- The "Pushing all…", "Push all to brief" and "push all emails" wording is removed from the labels and hints.

**CTA URL keeps its UTM parameters**
- Editing `utm_source`, `utm_medium`, `utm_campaign` or `utm_content` in the CTA URL updates that email's UTM value. The new value is what Save_Revision and the release pipeline send, and what gets restored later.
- When the person leaves the CTA URL field, any of those four that was removed or left empty (e.g. `utm_campaign=`) is put back with its latest known value, and the email is marked as having unsaved changes.
- A short note under the field ("UTM parameters restored: …") shows for about 5 seconds.
- Nothing is restored while the URL is not a valid `https://` URL, and a UTM with no known value is not added. Other query parameters are kept.

**Wording for one email**
- The note under the preview now reads "This email has been pushed to the brief." (was "All emails in this brief have been confirmed and pushed to the brief."). It shows only after this email is pushed, and never for reminder or confirmation-family emails, which skip the brief (before, it showed for them as soon as they loaded).
- The push error now reads "Push failed. This email was not pushed to the brief." (was "No emails were confirmed.").
- The unreachable "Select your email" guidance step is removed, and code comments that described several emails now describe the one email being revised.

**Help tour only shows what is visible**
- The "Your emails" step is removed.
- Each time the tour opens, a step is kept only if its section exists and is actually shown, so reminder and confirmation-family emails skip the Template and Layout steps.
- The "Your progress" step leaves out "push it to the brief" when the Push to brief step is hidden.

**Bullets without an inner paragraph**
- The editor stores each bullet as `<li><p>text</p></li>`. Email clients give that inner `<p>` its default margin, so lists looked double-spaced in the sent email but tight in the preview.
- `normalizeBodyFragment` now removes `<p>` wrappers directly inside a `<li>` (`<li><p>text</p></li>` becomes `<li>text</li>`). Several paragraphs in one bullet are joined with `<br>`, and a `dir` attribute moves to the `<li>`. Nested lists are left as they are.
- Applies to `body_para_1`, `2` and `3` everywhere they are sent (Save_Revision and every Email_Release_Pipeline call), and to drafts on load and AI suggestions. The preview looks the same.

**After approval: no pop-up, plain confirmation**
- The "Email approved" notice that opened after a pre-event or post-event approval (added in v1.0) is removed entirely: its markup, styles, open/close code and Esc handler.
- The progress bar's next-step line for an approved email no longer says "Next: Update audience in your brief and approve it in your tactic". It now reads "This email has been approved. No further action is needed." as plain status text, without "Next:".

**Confirmation and waitlist emails: no Send date field**
- These emails are sent automatically on registration, so they have no send date to choose. For `registration_confirmation`, `confirmation` and `waitlist` the Send date field is hidden, and Reply-to and From name share the row as two columns (narrow screens still stack).
- A send date is still required by the tools, so `due_date` is always today's date for these emails, on every save (Save_Revision) and every Email_Release_Pipeline call. It is worked out when the request is sent, so it never goes stale, and a saved draft's old date is replaced by today's when it loads.
- The send date rule always passes for these emails: no "Set a send date" warning, and it never blocks Save, Build or Approve.
- KBYG, cancellation and postponement emails keep their Send date field.

**Link and AI instruction boxes use their full width**
- In the body paragraph toolbar, the Link menu's URL box and the Enhance with AI instruction box reused the `choice-input` style, which keeps 104px free on the right for the subject/preheader character count and ✦ button. In the narrow link menu that hid about half the box: text stopped midway and the right side stayed blank.
- Both boxes now have normal right padding (9px, same as the left), so text runs the full width. The Subject line and Preheader fields are unchanged.

**Brief tabs read fresh on every Push to brief**
- Push to brief now reads the brief's current tab names with `Email_Editor_Utility` (`process_name: "get_brief_tabs_name"`, `document_id`) every time, so tabs added during a revision are picked up. The lock overlay shows "Accessing email brief…" while it reads.
- More than one tab opens the tab picker (the earlier choice is pre-selected if it still exists). One tab is used directly.
- If the tabs can't be read (tool error, failed flag or no `brief_document_id`), the reason goes to the console, the status line shows "Could not access the email brief, please try again." and nothing is pushed.
- The background tab reads on load and after save are removed. Build for pre-event and post-event emails reuses the tab chosen at push. Reminder Build and Approve still pick a tab silently.

**Confirmation and waitlist emails: their own Build process**
- Build for `confirmation`, `registration_confirmation` and `waitlist` calls `Email_Release_Pipeline` with `process_name` `confirmation` or `waitlist` (a registration confirmation is sent as `confirmation`). It no longer reads brief tabs.
- The request carries only what that process takes: `process_name`, `marketo_program_id`, `confirmation_waitlist_email` (one item: `email_type` matching `process_name`, `from_name`, `reply_to_email_address`, `subject_line_1`, `preheader_text_1`, `body_para_1`, `2`, `3`) and `emails` (one item, the usual email payload).
- Success is still `flag: true`; a failure shows the returned `msg`. Approve is unchanged for now.
- The process skips blank values, which would leave old text in Marketo, so for these emails From name, Reply-to address, Subject line, Preheader and all three body paragraphs are now required, like the event fields. A blank one shows "… cannot be blank." and blocks Save, Build and Approve. A blank paragraph is outlined red in the preview with an "Add paragraph N" hint instead of sample text.
- Subject and preheader lengths (60/80) are not enforced; they are sent as they are.

### v1.1

Committed 2026-11-14.

**Enhance with AI for body paragraphs**
- Clicking body paragraph 1, 2 or 3 in the content preview opens the editing toolbar with a ✦ (Enhance with AI) button at the end. Other fields do not get it.
- ✦ opens a small panel under the paragraph with an optional instruction box (for example "make it shorter") and a ✦ button that runs the enhancement. Enter in the box also runs it.
- Uses `Email_Studio_Utilities` (`process: "enhance_with_ai"`). The prompt includes what the paragraph is for (opening, supporting detail, or closing), asks to improve the wording without changing its intent, and adds the typed instruction when there is one.
- The prompt tells the AI, in bold caps, to keep every HTML tag, attribute and link exactly as they are and to return HTML only.
- The suggestion is shown under the paragraph with an "AI generated" label and Discard / Use this buttons. Nothing changes until "Use this" is clicked, and Done still has to be clicked to keep it (Cancel undoes it).
- Plain-text replies are cleaned up: surrounding blank lines are removed and each block becomes its own paragraph. A tool reply whose JSON is broken by raw line breaks is repaired before reading.
- A red warning shows when the suggestion drops a link from the original, or when the original had formatting (bold, italic, lists, links) and the reply has none.
- Escape closes the AI panel first, then the editor.

**Language**
- All three Enhance with AI prompts (subject line, preheader, body paragraphs) now tell the AI, in bold caps, to reply in the same language as the original text and not to translate it.

**Setup**
- New shared [App Setup](#app-setup) section listing the external domains each app must allow (`cdn.jsdelivr.net`, `esm.sh`, `info.mongodb.com`, and the uploaded banner host).

### v1

Committed 2026-11-13.

- **Launch tool:** `Email_Revision` replaces `Tactic_Email_Studio`. The app reads `process_flag`, `msg`, `tactic_link` and `task_link` from it.
- **Load and Refresh:** `Email_Editor_Utility` (`process_name: "get_email_draft"`) replaces `Email_Brief_Schema` "Pull". It loads the one email linked to `task_link`, shows the newest draft and lists all drafts in the saved-draft picker.
- **Errors:** the tool's `msg` is shown as-is on the loading screen.
- **Header:** the email strip is replaced by "Editing: <email label>".
- **Email task:** read-only, and hidden for confirmation and waitlist emails. The task dropdown and the "Create email task" button (`add_task`) are removed.
- **Subject and preheader:** plain text boxes with an "Enhance with AI" (✦) button (`Email_Studio_Utilities`, `process: "enhance_with_ai"`). The numbered Suggestions picker is removed. Each field has help text under it.
- **Restrictions:** the `restrictions_*` flags are stored but not enforced yet.
- **Setup:** the banner preview only loads once the image domains are allowed in the app's resource settings. See [App Setup](#app-setup).

## Email Studio Changelog

### v2.3

Released 2026-11-10. Promoted from Sandbox to Production. Production is identical to Sandbox.

**Email logo**
- The "MONGODB" text at the top of every email preview is now the MongoDB logo image (`https://info.mongodb.com/rs/149-CBJ-136/images/mongodb-logo-112x28-2x.png`), in all three templates and in both card and simple layouts.
- It is fixed: it has no edit target, so it cannot be clicked, edited or removed, and it ignores pointer events and cannot be selected or dragged.
- It is shown 20px tall (about the height the text took) with the width following the image's proportions. The logo is defined once (`EMAIL_LOGO_URL` and `emailLogoMarkup()`).

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
