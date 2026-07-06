---
title: "Claude + Obsidian: your notes app just called you out"
source: "https://x.com/gippp69/status/2074059543164658162"
author:
  - "[[@gippp69]]"
published: 2026-07-06
created: 2026-07-06
description: "You have written the same idea three times this year. You don't know that yet. Your notes app does.Why this needs a loop, not a chat tab?You..."
tags:
  - "clippings"
---
![Imagen](https://pbs.twimg.com/media/HMdtwS7WwAA8EbG?format=jpg&name=large)

You have written the same idea three times this year. You don't know that yet. Your notes app does.

**Why this needs a loop, not a chat tab?**

You can already paste your old notes into Claude and ask it to find patterns. People do this. It works, once.

The problem is you're the trigger. You open the tab, you paste the file, you ask the question. Close the tab and the checking stops. Nothing rereads your vault unless you remember to make it.

A loop doesn't wait for you to remember. You give it a rule once, it runs the cycle on a schedule, checks its own output against a hard bar, fixes what's weak, and writes the result back into the same vault you already use.

The shape of a loop, always the same four parts:

```plaintext
TRIGGER   →  a schedule, or a new note being saved
DO        →  Claude reads the vault and does the actual work
VERIFY    →  checked against a hard rule, not a vibe
STOP      →  passes the rule, or hits a retry limit
```

Skip VERIFY and Claude grades its own homework and gives itself a 9 every time. Skip STOP and it reruns forever on a note it can't make sense of, quietly running up your API bill overnight. Both of those are where people get this wrong.

**The stack: Claude, Obsidian, a schedule?**

Three pieces, nothing exotic.

The vault. Obsidian. Free, local, every note is a markdown file you own on disk. A script can read and write into that folder directly, no export, no API wall between you and your own notes.

The source. Your AI chat history, exported once from whatever provider you use. This is the raw material the loops read from.

The brain. Claude, split by job. Sonnet for anything that needs real judgment, comparing ideas, catching contradictions. Haiku for cheap sorting and tagging, roughly a fraction of the cost per call. You don't run the expensive model to check a YAML field.

![Imagen](https://pbs.twimg.com/media/HMdsf6NW0AA6o_D?format=png&name=large)

**Loop 1: raw export becomes one note per idea?**

```plaintext
TRIGGER:   new export file dropped into raw/
STEPS:
  1. Read every conversation in the file
  2. Extract only the actual decision, the reasoning, and what
     happened after, if mentioned later
  3. For each distinct idea, write one note into notes/ using
     this frontmatter:
       ---
       topic: 
       created: 
       source_conversations: []
       status: open
       linked_notes: []
       ---
  4. If a note on this exact topic already exists, update it
     instead of creating a duplicate
VERIFY:    every note has all three body sections filled AND
           source_conversations has at least one entry.
           If not, reprocess that conversation alone.
STOP:      verify passes, or 3 retries, then flag for manual review
```

This is the one that turns a dead export into an actual vault. Nothing clever yet, just structure.

![Imagen](https://pbs.twimg.com/media/HMdtjgaWgAAxIPS?format=png&name=large)

This is what one processed note actually looks like. Not a summary, a decision with a paper trail

**Loop 2: the vault rereads itself for patterns?**

This is the one that actually earns its place. Four separate passes, each with its own job, each reading the same memory file before it starts and writing to it before it stops.

```plaintext
TRIGGER:   every 6 hours
STEPS:
  Pass 1, dropped threads:
    flag any open note untouched in 60+ days
  Pass 2, what worked:
    match past ideas against published output that performed well
  Pass 3, the honest read:
    one direct line on the last 14 days, no softening
  Pass 4, hidden duplicates:
    compare all notes for the same idea worded differently,
    across any time gap, list together, ask before merging
VERIFY:    each pass writes at least one entry to memory/MEMORY.md,
           and Pass 4's matches are confirmed by keyword overlap
           in the Decision field, not just similar phrasing
STOP:      all four passes complete, or a pass fails and gets
           logged, never silently skipped
```

Pass 4 is the one that actually surprises you. Two notes written months apart, worded completely differently, turn out to be the same unfinished idea. That match doesn't come from you. It comes from something that never gets bored of rereading you.

You cannot see your own patterns from inside your own head. You need something outside it that never gets bored of rereading you.

Never let Pass 4 auto-merge. Two notes about "quitting a bad client" can look identical in reasoning even when they're about different clients. Confirm before merging, every time.

![Imagen](https://pbs.twimg.com/media/HMdqj7WXoAAGXjt?format=jpg&name=large)

These are three clusters my vault found on its own, ideas I never noticed were connected until the loop pointed them out

**Try the manual version first?**

Before you schedule anything, run this once by hand in a Claude chat, pointed at a folder of your own notes:

```plaintext
You will work in a loop until the task meets the bar.

TASK:
Read every note in [folder]. Find ideas that repeat across
different notes, worded differently, and any project mentioned
that was never followed up on.

SUCCESS CRITERIA (strict, no soft passes):
- every match is backed by an actual quote from both notes
- every dropped thread includes how long it's been idle
- no vague "you seem to value X" style output

LOOP PROTOCOL, repeat every turn:
1. PLAN   - state the single next step
2. DO     - produce or improve the output
3. VERIFY - score 1-10 on each criterion, be brutally honest
4. DECIDE - if every criterion is 8+, print "FINAL" and stop

RULES:
- Never call it done until every criterion is 8+
- Do not ask me questions, make a sensible assumption and continue

Begin. Run the loop until FINAL.
```

If what comes back actually surprises you, the idea earns a schedule. If it doesn't, don't automate it yet.

![Imagen](https://pbs.twimg.com/media/HMdrfjiWoAAy8-L?format=jpg&name=large)

**The order that actually works?**

Get one manual run reliable, in the Claude chat.

Turn that into the two prompts in Loop 1 and Loop 2.

Add the VERIFY and STOP rules before you schedule anything.

Then, and only then, put it on a cron job or a recurring automation.

Skipping ahead, scheduling something you haven't proven by hand, is exactly how loops misfire while you're asleep. Prove it once, harden it, then automate it.

**What it actually costs?**

Loop 1 runs once per export, so it's a handful of Sonnet calls total, not recurring.

Loop 2 runs four passes every six hours. Move Pass 1 and Pass 4 to Haiku, they're comparison and tagging, not deep reasoning. Keep Pass 2 and Pass 3 on Sonnet, since matching against real performance data and giving an honest read both need actual judgment. Split this way, four passes a day costs less than the coffee you drink while reading what came back.

**What this actually means for you?**

The vault was never the point. The point is having something that keeps rereading you after you've stopped paying attention, so the pattern gets caught instead of repeated a fourth time.

The shift here isn't a new app or a smarter model. It's about who does the noticing. You stop being the only one checking your own thinking, and something else starts doing it in the background, on files you own, on a schedule you set.

My take: build Loop 1 first, let the vault fill up for two or three weeks before you touch Loop 2. The duplicate-finding pass is worthless until there's enough in the vault to actually collide against. Start with the manual version in this article, run it a few times in the Claude chat. If you keep reaching for it, that's when it earns the schedule.

If you want more breakdowns like this, I post one every couple of days on Telegram and X. Both free.

X - [https://x.com/gippp69](https://x.com/gippp69) Telegram - [https://t.me/GipArcAI](https://t.me/GipArcAI)