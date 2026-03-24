# BrandArgus — For Research

Free domain threat intelligence tools for security researchers and cybersecurity students.

## What's Inside

### [`/api`](./api)
REST API access to BrandArgus domain intelligence feeds — newly registered domain search, nameserver reverse lookup, and bulk export. Bearer token authentication with IP whitelist.

### [`/mcp`](./mcp)
[Model Context Protocol] server for AI-assisted threat investigation. Query domain intelligence data directly from Claude and other MCP-compatible assistants.

### [`/skills`](./skills)
Pre-built investigation workflows for Claude — domain cluster analysis, registration pattern detection, and infrastructure mapping. Drop-in skills that turn Claude into a domain threat analyst.

## Who Is This For

- **Security researchers** tracking phishing campaigns and brand abuse infrastructure
- **Cybersecurity students** learning threat intelligence with real-world data
- **Academic researchers** studying domain abuse patterns and DNS ecosystems

## Getting Access

All tools are free for non-commercial research use.

Send your application to **info@lyalpha-gmbh.com** with:
1. Your name and affiliation
2. Intended use case
3. A static IP address or CIDR range (for API access)

## Data Coverage

- All major gTLDs (`.com`, `.net`, `.org`, `.xyz`, and 1,000+ others)
- Rolling 30-day window for newly registered domains
- Full active gTLD namespace for nameserver lookups
- Updated daily

## About

BrandArgus is powered by [ABTdomain](https://abtdomain.com), a domain intelligence platform that monitors 1M+ daily domain lifecycle events.

Built by [Lyalpha GmbH](https://lyalpha.com), Düsseldorf, Germany.

- Website: [brandargus.com](https://brandargus.com)
- Open-source: [DKSplit](https://github.com/ABTdomain/dksplit) — multilingual domain name segmentation
- Contact: info@lyalpha-gmbh.com

---

© 2026 Lyalpha GmbH. All rights reserved.
