# AGENTS.md — ProtoForge Documentation Rules

Rules for Mintlify's auto-doc agent and other AI systems contributing to this documentation site.

## Code Example Standards

1. Every code example MUST show both the **request** and the **response**
2. Use `<CodeGroup>` with tabs for TypeScript, cURL, and Python (in that order)
3. TypeScript examples use `fetch` (not Axios or other libraries)
4. Python examples use `requests` (not urllib or aiohttp)
5. cURL examples use `-H` for headers and `-d` for request bodies
6. Response examples must be valid JSON with realistic (not placeholder) data
7. Include the HTTP status code in response comments (e.g., "Response (201 Created)")

## API Documentation Requirements

1. Every MDX page MUST have YAML frontmatter with `title` and `description`
2. Descriptions must be specific — never start with "Learn about", "Overview of", or "Introduction to"
3. Descriptions should be 10-30 words, written for `llms.txt` consumption
4. Every API endpoint page must show: request parameters, response schema, authentication, rate limit, and error codes
5. Rate limits must be documented per category (Search: 100/min, License: 30/min, Contract: 20/min, Bulk: 5/min)

## Style Guidelines

### Terminology (Canonical Terms)

| Use | Do NOT Use |
|-----|-----------|
| license | licence |
| track | song |
| organisation | organization |
| sync licensing | synchronisation licensing (except in definitions) |
| rights holder | rightsholder, rights-holder |
| API key | api key, apikey, API Key |

### Writing Style

1. Name the entity — never use vague pronouns ("it", "this", "that") without a clear antecedent
2. Use active voice: "ProtoForge validates the ISRC" not "The ISRC is validated"
3. Be specific: "returns a 422 error" not "returns an error"
4. Use present tense: "the API returns" not "the API will return"
5. Write for developers — assume familiarity with REST APIs, JSON, and HTTP

### Headings

1. Use descriptive headings: "ProtoForge Authentication with API Keys" not "Authentication" or "Overview"
2. H1 (#) — page title only, matches frontmatter `title`
3. H2 (##) — major sections
4. H3 (###) — subsections
5. Never skip heading levels (no H2 to H4)

## Content Structure Rules

### Mintlify Components

Use these components consistently:

| Component | When to Use |
|-----------|-------------|
| `<Steps>` | Sequential instructions (setup flows, tutorials) |
| `<CodeGroup>` | Multi-language code examples (always TS, cURL, Python) |
| `<Callout>` | Important warnings, tips, or notes |
| `<Card>` / `<CardGroup>` | Navigation links to related pages |
| `<Accordion>` | Expandable content (common mistakes, FAQs) |
| `<Snippet>` | Reusable content imported from `snippets/` |

### Page Structure (Guides)

1. Frontmatter (title, description)
2. H1 heading (matches title)
3. Callout — what this guide covers
4. Prerequisites — links to prior guides
5. Steps — main tutorial content with CodeGroup examples
6. Reference tables — parameters, status codes, etc.
7. Accordion — common mistakes
8. Card — next steps linking to the next guide

### Internal Links

- Root-relative paths without file extensions: `/guides/authentication` not `guides/authentication.mdx`
- Link text should be descriptive: "Authentication Guide" not "click here"
- Every guide should link to the next guide in the progression

## Content Validation

Before merging any content change:

1. Run `npx mintlify dev` and verify the page renders correctly
2. Check frontmatter has both `title` and `description`
3. Verify all code examples show request AND response
4. Search for "licence", "song", and vague pronouns
5. Confirm the page appears in `docs.json` navigation
