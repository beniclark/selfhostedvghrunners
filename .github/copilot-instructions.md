# Project Guidelines

## Overview

This is a **documentation-only** repository containing a comprehensive comparison of GitHub-hosted vs self-hosted runners for GitHub Actions. There is no application code, build system, or test suite — just Markdown content in [README.md](../README.md).

## Content Structure

The README follows a deliberate structure — preserve this order when editing:

1. **Overview** — Brief intro to runner categories
2. **Key Differences** — Summary comparison table (all four runner types as columns)
3. **Detailed Comparison** — Deep-dive subsections (Infrastructure, Hardware, Networking, Scaling, Cost)
4. **When to Recommend Each** — Decision guidance per runner type + hybrid approach
5. **Expected Customer Questions** — Numbered Q&A grouped by theme (Cost, Security, Performance, Migration, Architecture)
6. **Preparation Tips** — Actionable advice for customer conversations
7. **Reference Documentation** — Links table to official GitHub docs

## Writing Conventions

- Use comparison tables for feature-level differences; keep all four runner types as columns: GitHub-Hosted Standard, GitHub-Hosted Larger, Self-Hosted Manual, Self-Hosted ARC/K8s
- Use ✅/❌/⚠️ emoji in tables for quick scannability
- Customer Q&A entries are **numbered sequentially** across all subsections (1–24 currently) — maintain continuous numbering when adding new questions
- Q&A format: bold question in customer voice (`"How do we...?"`) followed by `>` blockquote answer
- Keep the Table of Contents in sync with all section headings
- Link to official GitHub docs (docs.github.com) in the Reference Documentation table; do not use third-party sources

## Key Terminology

- **ARC** = Actions Runner Controller (Kubernetes operator)
- **Runner Scale Sets** = Current recommended ARC scaling mode (replaces older listener-based approach)
- **Larger runners** = GitHub-hosted runners with enhanced specs (Team/Enterprise Cloud plans only)
- **VNET integration** = Azure Private Networking for larger runners
- **Runner groups** = Access control mechanism scoping runners to orgs/repos

## When Updating Content

- Verify any GitHub Actions pricing, plan requirements, or feature availability claims against the links in the Reference Documentation section
- The audience is internal — someone preparing for customer conversations about runner strategy
- Be specific and actionable; avoid generic CI/CD advice not tied to GitHub runners
