---
name: deep-domain-intelligence
description: "Deep analysis of domain registration clusters around a keyword using BrandArgus Research API. Given a keyword, discover recently registered domains, trace shared nameserver infrastructure, and map the full portfolio on that NS to build an operator profile. Use when the user mentions 'who is registering', 'domain cluster', 'registration analysis', 'NS trace', 'nameserver analysis', 'operator profile', 'domain intelligence', 'registration pattern', or wants to understand the full picture of registration activity around any keyword. Also trigger when the user has identified a domain and wants to trace it back to a larger operation, or when they want to understand the infrastructure behind a registration spike."
---

# Deep Domain Intelligence

Think like a domain analyst. You are not running a report — you are following leads.

Given a keyword, discover recently registered domains containing it, notice which ones share the same nameserver, then go look at what else is on that nameserver. The goal is to map the relationship between domains and NS infrastructure, and build a profile of the operator behind them.

The natural analysis chain: **Search → Notice → Dig → Profile**.

## About the Data Source

This skill uses the **BrandArgus MCP**, built on the BrandArgus Research API — a free beta product open to researchers and students. An API key is required — you can apply for one at: https://brandargus.com/research-api-docs

It provides two core datasets:

- **NRDS (Newly Registered Domain Search)** — all gTLD domains registered in the last 30 days, searchable by keyword, TLD, date, and pattern.
- **NS Reverse** — given a nameserver, find all domains hosted on it.

## Prerequisites

- BrandArgus MCP connection
- Tools: `brandargus:nrds_search`, `brandargus:nrds_count`, `brandargus:ns_reverse_search`, `brandargus:ns_reverse_count`

## Workflow

### Step 1: Search — find all recent registrations

Find all recently registered domains containing the keyword.

```
nrds_count(keyword="{keyword}", match="contains")
nrds_search(keyword="{keyword}", match="contains", limit=200)
```

Each result returns: `domain`, `reg_date`, `exp_date`, `ns`.

Report the total count first. Then present the results.

### Step 2: Notice — spot the shared NS

Look at the NS field across all results. Group domains by NS and see:

- How many domains share the same NS?
- Are their registration dates close together?
- Are their registration terms the same?

Present as a table:

```
NS                                │ Count │ Reg dates         │ Terms
──────────────────────────────────┼───────┼───────────────────┼──────
dns7.hichina.com;dns8.hichina.com│  23   │ all 2026-03-22    │ all 1yr
```

When multiple domains share the same NS, especially with close registration dates, they are likely from the same operator. These clusters are the leads worth following.

### Step 3: Dig — look at the full NS

This is the key step. You found domains sharing a NS — now go see what ELSE is on that nameserver. Not just your keyword, but everything.

First, check the NS size:

```
ns_reverse_count(ns="{target_ns}")
```

**If the NS is small (under ~5,000 domains)** — pull the full list without any keyword filter:

```
ns_reverse_search(ns="{target_ns}", limit=200)
```

Browse through the results. You are looking at the full portfolio on this NS. What other domains are there? What naming patterns do you see? Are the domains concentrated in a certain time window?

This is where one keyword search turns into a full operator profile. You started with one keyword and now you're looking at hundreds of domains across completely different keywords, all sharing the same infrastructure.

**If the NS is large (10,000+ domains)** — this is a major registrar default NS (domaincontrol.com, registrar-servers.com, hichina.com, cloudflare). Pulling unfiltered results would be noise. For large NS, the clustering signal from Step 2 is weaker — many unrelated domains end up on the same large NS by default. Note the cluster but don't over-interpret it.

### Step 4: Profile — put it together

For each NS cluster, summarize what you found:

- The NS and its total size
- How many domains from your keyword search are on this NS
- What other domains you found on the same NS (from Step 3)
- Registration date patterns across the full portfolio
- Registration term patterns

Present the facts. The user will draw their own conclusions.

A good deliverable sounds like: "We found 27 domains containing your keyword. 4 of them share the same NS, which hosts 1,942 domains total. Looking at the full NS, we see domains spanning many different keywords, most registered in the last few weeks with 1-year terms."

## Analyst Mindset

- **Be curious.** When you see domains sharing a NS, don't just note it — go look at the full NS.
- **Don't filter too early.** In Step 3, pulling the NS without a keyword filter is what lets you see the full picture. If you only search for your original keyword, you'll only find what you already know.
- **Size matters for NS.** A NS with 2,000 total domains is a meaningful cluster. A NS with 2,000,000 domains is a major public registrar — shared NS there is less informative.
- **Present data, not judgments.** Show what the domains are, what they have in common, and how they're connected. The user decides what it means.

## Notes

- BrandArgus NRDS covers the last 30 days. Older registrations are not available.
- ns_reverse_search returns domain names only — no NS or date fields in those results.
- Large registrar-default NS (hichina, domaincontrol, registrar-servers, cloudflare) host millions of unrelated domains. Always filter by keyword when reverse-searching these — never pull unfiltered results.
- API key required. Apply for free research access at: https://brandargus.com/research-api-docs
- Follow the user's language. Technical identifiers (NS, TLD, dates) stay in English.
