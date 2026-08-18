---
name: journal-entry
description: Appends a properly formatted entry to today's trading journal at memory/journal/<YYYY-MM-DD>.md, following the template in memory/README.md. Use this any time a routine or session in this repo needs to record portfolio, watchlist, scan, or trade activity — a daily check, weekly scan, earnings watch, trade proposal, or any ad hoc trading decision. Always go through this skill instead of hand-editing journal files directly, so the timestamp format, section order, and file/no-overwrite rules stay consistent across every routine that writes to the journal.
---

# Journal Entry

The journal in `memory/journal/` is the durable record of everything this repo's routines and sessions do. Its value comes entirely from being consistent — every entry in the same shape, nothing ever overwritten. This skill is the one place that logic lives, so every routine (daily-check, weekly-scan, earnings-watch, propose-trade, or an ad hoc session) gets the same behavior for free instead of re-implementing it slightly differently each time.

## Steps

1. **Find today's file.** `memory/journal/<YYYY-MM-DD>.md`, using the current date. If it doesn't exist yet, create it — it starts empty, with no header of its own; entries are self-contained.

2. **Never overwrite.** If the file already has entries, append below the last one. Don't touch, reorder, or "clean up" earlier entries — they're a record of what actually happened at the time, not a draft.

3. **Format the entry using this exact template** (from `memory/README.md`):

   ```markdown
   ## <HH:MM ET> — <routine or activity name>

   **Summary:** one or two lines on what this was.

   **Findings:** what was observed (prices, indicators, scan hits, earnings dates, etc.)

   **Proposals:** any trade or watchlist proposal made, with rationale, size, and stop if applicable.

   **User decisions:** what the user approved, rejected, or changed.

   **Follow-ups:** anything to revisit next time.
   ```

   - Use the current time in ET (24-hour or 12-hour is fine, match the surrounding entries in the file).
   - The routine/activity name should be specific enough to scan at a glance later — e.g. "Daily Portfolio + Watchlist Check", "Weekly Scan — new candidates", "NVDA position proposal".
   - **Omit a section entirely if it doesn't apply** — e.g. a read-only daily check has no Proposals section. Don't write "N/A" or "None" as filler unless there's something meaningful to say by saying so (e.g. "None — this was scaffolding, not a trading decision" is fine because it's informative; a bare "N/A" isn't).

4. **Separate entries with a blank line** before the next `##` heading, matching the existing file's spacing.

5. **Write the file** (create + append, or append to existing) and confirm back to whoever invoked this skill what got logged and where.

## When you're missing pieces

If the caller hasn't given you enough to fill in Summary/Findings, ask rather than guessing or inventing findings — a wrong number in the journal is worse than a short pause to confirm it. It's fine to omit sections you have nothing to report, but never fabricate content to fill a section out.
