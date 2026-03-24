# Deep Domain Intelligence

A Claude skill for domain registration analysis. Search a keyword, discover recently registered domains containing it, and trace them through shared nameserver infrastructure to build a full operator profile.

## What It Does

Given any keyword, this skill guides Claude through a natural analysis chain:

1. **Search** — find all domains containing the keyword registered in the last 30 days
2. **Notice** — group results by nameserver, spot clusters that share the same NS and registration dates
3. **Dig** — reverse-lookup the nameserver to see the full portfolio of domains on it
4. **Profile** — summarize the relationship between domains, nameservers, and registration patterns

The key insight: domains registered by the same operator often share the same nameserver. One keyword search can lead to an entire portfolio you didn't know existed.

## Setup

### 1. Get a BrandArgus Research API Key (free)

This skill is powered by the BrandArgus MCP, built on the BrandArgus Research API — a free beta product open to researchers and students. An API key is required.

Apply at: **https://brandargus.com/research-api-docs**

### 2. Connect the BrandArgus MCP

Connect the BrandArgus MCP in your Claude environment. It provides four tools:

| Tool | What it does |
|---|---|
| `brandargus:nrds_count` | Count newly registered domains matching a keyword |
| `brandargus:nrds_search` | Search newly registered domains with full details |
| `brandargus:ns_reverse_count` | Count total domains on a nameserver |
| `brandargus:ns_reverse_search` | List all domains on a nameserver |

### 3. Add the Skill

Place `SKILL.md` in your Claude skills directory. Claude will read it automatically when relevant queries come in.

## Usage

Just ask Claude to analyze a keyword. For example:

> "Look into recent domain registrations containing google"

Claude will follow the Search → Notice → Dig → Profile chain, calling the BrandArgus MCP tools along the way.

## Data Coverage

- **NRDS**: All gTLD domains registered in the last 30 days
- **NS Reverse**: All active gTLD domains indexed by nameserver
- Data updates daily

## Links

- API Documentation & Key Application: https://brandargus.com/research-api-docs
