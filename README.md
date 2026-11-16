# Influ2 Signal Emails

A Claude skill for Outpost sales reps. It watches the `#influ2-signals` Slack channel for Influ2 notifications that tag the rep. When someone clicks an Outpost Gate Automation (OGA) ad, it writes a three-email follow-up from Outpost's writing rules and writes the subjects and bodies onto the contact's HubSpot record. A HubSpot workflow then enrolls the contact in the rep's sequence. The skill never sends or enrolls anything itself.

This README is for whoever maintains the skill. Reps don't need it; setup happens in chat.

## What a rep does

1. Save `influ2-signal-emails.zip` somewhere findable. Don't unzip it.
2. In the Claude desktop app, click **Customize** in the left sidebar, then **Skills**. Click **Add**, then **Upload a skill**, and pick the zip. (If Customize isn't in the sidebar, use claude.ai/customize/skills in a browser; skills are account-level and show up in the desktop app either way.) Click save.
3. Make sure the toggle next to "influ2-signal-emails" is on.
4. Open a new chat and type: **set up Influ2 signal emails**
5. Answer Claude's questions. Setup checks connectors, confirms who to scan for, walks them through copying the HubSpot sequence, shows a test sequence for two contacts, then creates the recurring scheduled task. About ten minutes.

Prerequisites: Slack, HubSpot, and Influ2 connectors turned on in Claude, a HubSpot user with contact edit permission, and the rep set as Gate Owner on their companies in HubSpot (Influ2 tags the Gate Owner). Setup checks the connectors and tells the rep what to fix if something's missing.

## Adding a new seller

1. Set the seller as Gate Owner on their companies in HubSpot, so Influ2 tags them in `#influ2-signals` and the workflow sends as them.
2. The seller installs the skill and runs setup (above). Setup DMs you their sequence link.
3. Connect their sequence copy to the enrollment workflow.

Nothing in the skill files changes per seller. Each email introduces the company's Gate Owner by first name, and the sequence sends from that same person.

## How it works

Two modes, detected from the prompt:

- **Setup mode** runs when a rep asks to set up or install. It collects per-rep config (Slack ID, exclusions, sequence link), runs a dry run without writing to HubSpot, then creates a scheduled task whose prompt carries the config block.
- **Run mode** runs when the scheduled task fires, every 4 hours:
  1. Pulls the last 24 hours of the channel and keeps Influ2 messages that tag the rep.
  2. Skips signals already marked in their Slack thread.
  3. Finds the HubSpot contact (Influ2 lookup first, then name and company).
  4. Skips the contact if it already has Influ2 emails, is in a sequence (Do Not Send sequences don't count), isn't an OGA signal, has a title on the skip list, or its company has no active Gate Owner.
  5. Picks the persona and three angles from the ad they clicked, and writes three emails that pass the check in `drafting-rules.md`.
  6. Writes all six properties in one update. Filling in Influ2 Email 1 Subject triggers the enrollment workflow, so Emails 2 and 3 have to be on the record at the same moment.
  7. Posts the result in the signal's Slack thread.

## Files

| File | What it does | Change it when |
|---|---|---|
| `SKILL.md` | The steps Claude follows in setup and every run, plus company-wide constants (channel, Influ2 workspace and programs, HubSpot portal and properties, Do Not Send sequence IDs, landing pages, master sequence, Monica's Slack ID) | A property, ID, URL, or step changes |
| `references/ad-angles.md` | Maps each OGA ad to a persona and the angle for Emails 1, 2, and 3, with each angle's question, solution, proof, offer, and subject | Ads are added, renamed, or retired |
| `references/drafting-rules.md` | Email structure, writing rules, banned words, subject rules, the HTML format for rich text bodies, and the check every email must pass | Writing rules change |
| `references/roles.md` | How the Security question ends for each role, and the titles to skip | A new role shows up often, or a skipped title turns out to be a buyer |
| `references/proof-library.md` | The only customers and claims emails may use, and which proof fits which segment | A customer is cleared, goes live, or a claim changes |
| `references/examples.md` | A full three-email sequence that passes every rule | Rules change enough that the examples no longer pass |

The reference files hold the rules. Claude writes new copy for every contact from them, so fixing a problem in the copy means editing the rule, never adding a template.

## HubSpot setup

**Contact properties.** Six properties hold the emails:

| Property | Type |
|---|---|
| `influ2_email_1_subject`, `influ2_email_2_subject`, `influ2_email_3_subject` | Single-line text |
| `influ2_email_1_body`, `influ2_email_2_body`, `influ2_email_3_body` | Rich text |

The skill writes bodies as HTML: one `<p>` per paragraph and one `<a href>` link to the Security or Operations landing page.

**Master sequence** (https://app.hubspot.com/sequences/22778013/sequence/311074822/edit). Three automated email steps, each laid out the same way:

```
Subject: [Contact: Influ2 Email N Subject]

[Contact: First Name (Hello)],

[Contact: Influ2 Email N Body]
```

The rep's signature is added automatically. The greeting has no "Hi," and falls back to "Hello," when the first name is missing.

**Enrollment workflow.** Triggers when Influ2 Email 1 Subject is known and enrolls the contact, sending as the company's Gate Owner. A contact in a Do Not Send sequence still gets drafts, so the workflow needs to handle that enrollment before it can enroll them here. HubSpot allows one active sequence per contact.

**Do Not Send sequences.** The skill treats enrollment in the sequence IDs listed in `SKILL.md` as not being in a sequence. Add any new Do Not Send sequence ID there.

**Test before rollout.** Write all six properties on a test contact you own, let the workflow enroll it, and check the emails that arrive:

- Paragraphs are separate, with no raw HTML tags showing
- The link is clickable and points to the right page
- Spacing looks right after the greeting and before the signature
- Emails 2 and 3 pull their own subject and body

If the bodies show raw HTML tags, HubSpot isn't rendering the rich text token in sequences, and the body format in `drafting-rules.md` needs to change before rollout.

## Two reps on the same channel

If two people run this and a signal tags both, both would draft for the same contact. The skill never overwrites existing drafts, so the second run skips the contact, but it's cleaner to avoid the race. Setup asks about this and lets the rep put the other person's Slack ID in their exclusion list. Only one side needs the exclusion; setup explains which.

## Repo layout

```
influ2-email-follow-up/
├── README.md
├── influ2-signal-emails/
│   ├── SKILL.md
│   └── references/
│       ├── ad-angles.md
│       ├── drafting-rules.md
│       ├── examples.md
│       ├── proof-library.md
│       └── roles.md
└── influ2-signal-emails.zip     # what reps upload; rebuild after editing any file in the folder
```

## Rebuilding the zip after edits

From the repo root:

```
rm -f influ2-signal-emails.zip
zip -r influ2-signal-emails.zip influ2-signal-emails
```

The folder inside the zip must be named `influ2-signal-emails` and contain `SKILL.md` and the `references` folder, or the upload fails.

## Updating reps who already installed the skill

Reps upload the new zip over the old skill (Customize → Skills). Their scheduled tasks keep working, because the config block only carries their Slack ID, exclusions, and sequence link. Config blocks from the previous version include a `VOICE_PROFILE` section; the skill ignores it.

## Maintainer

Monica Thorne (monica.thorne@outpost.us)
