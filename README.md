# Influ2 Signal Emails

A Claude skill for Outpost sales reps. It watches the `#influ2-signals` Slack channel for Influ2 buyer-intent notifications that tag the rep, drafts a short outreach email for each contact in the rep's voice, and writes the subject and body onto the contact's HubSpot record. The rep's HubSpot sequence pulls those two properties into its first step. The skill never sends anything.

This README is for whoever maintains the skill. Reps don't need it; setup happens in chat.

## What a rep does

1. Upload `influ2-signal-emails.zip` at claude.ai/customize/skills (click **+**, then **Create skill**, then **Upload a skill**). Don't unzip it first.
2. Open a new Cowork chat and type: **set up Influ2 signal emails**
3. Answer Claude's questions. Setup checks connectors, confirms who to scan for, asks for a few sent emails to learn their voice, walks them through cloning the HubSpot sequence, shows two test drafts, then creates the recurring scheduled task. About ten minutes.

Prerequisites: Slack, HubSpot, and Influ2 connectors turned on in Claude, and a HubSpot user with contact edit permission. Setup checks all of this and tells the rep what to fix if something's missing.

## How it works

Two modes, detected from the prompt:

- **Setup mode** runs when a rep asks to set up or install. It collects per-user config (Slack ID, exclusions, voice profile, sequence link), runs a dry run, then creates a scheduled task whose prompt carries the config block. Each 4-hour run reads its config from that prompt, so nothing has to be edited in the skill file per person.
- **Run mode** runs when the scheduled task fires. Pulls the last 24 hours of the channel, filters to messages tagging the rep, dedupes via a "✅ Drafted" thread reply, finds the HubSpot contact (Influ2 lookup first, name + company fallback), drafts, writes to HubSpot, marks the thread.

Company-wide constants (channel ID, Influ2 workspace, HubSpot portal, property names, master sequence URL, Monica's Slack ID for setup notifications) live at the top of `SKILL.md`.

## Two reps on the same channel

If two people run this and a signal tags both, both would draft for the same contact and the second write would overwrite the first. Setup asks about this and lets the rep put the other person's Slack ID in their exclusion list. Only one side needs the exclusion; setup explains which.

## Repo layout

```
influ2-signal-emails/
├── README.md
├── influ2-signal-emails/
│   └── SKILL.md
└── influ2-signal-emails.zip     # what reps upload; rebuild after editing SKILL.md
```

## Rebuilding the zip after edits

From the repo root:

```
rm -f influ2-signal-emails.zip
zip -r influ2-signal-emails.zip influ2-signal-emails
```

The folder inside the zip must be named `influ2-signal-emails` and contain `SKILL.md`, or the upload fails.

## Changing the config for an existing rep

Voice profile, exclusions, and the sequence link live in the rep's scheduled task prompt, not in the skill. Ask Claude to "update my Influ2 voice profile" and it will rebuild the config block and offer to recreate the task.

## Maintainer

Monica Thorne (monica.thorne@outpost.us)
