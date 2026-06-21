# moltable agent skills

[![skills.sh](https://skills.sh/b/moltable/skills)](https://skills.sh/moltable/skills)

**The best go-to-market agent skills, powered by [moltable](https://moltable.io).**

Lead enrichment, account research, TAM building, and outbound prep — done by your AI agent, in your terminal, against your live data. These skills turn natural-language asks into production-grade enrichment workflows on moltable, the agent-native enrichment-table platform.

Compatible with 18+ AI agents including Claude Code, GitHub Copilot, Cursor, Cline, and many others via the [Agent Skills](https://agentskills.io/) open standard.

## What you can do

Once installed, your agent picks up these prompts automatically:

```
Enrich this CSV of 500 leads with company info, funding stage, tech stack, and verified emails.
```

```
Build me a list of every YC W26 company + the LinkedIn of each founder + their last raise.
```

```
Research these 50 enterprise accounts and tell me which ones have a head of revops to pitch.
```

```
Find companies in the EU using Snowflake + Salesforce that aren't customers of <competitor>.
```

```
Watch the running enrichment job and tell me when it's done. Show me which rows failed.
```

```
Rotate my moltable API key and update my CI secret.
```

Behind the scenes the agent uses the [moltable CLI](https://github.com/moltable/cli) + this skills bundle to scaffold the workbook, pick the right column sources (LinkedIn lookups, web search, scrape providers, BYOK LLM enrichment, integrations like Salesforce/HubSpot), run the job, watch progress, and hand back results — all without you clicking through a dashboard.

## Installation

### Install all skills

```bash
npx skills add moltable/skills
```

### Install a specific skill

```bash
npx skills add moltable/skills --skill auth-and-profiles
npx skills add moltable/skills --skill build-enrichment-table
npx skills add moltable/skills --skill long-tail-fallback
npx skills add moltable/skills --skill run-and-watch-jobs
```

By default `npx skills add` installs to your project's `.claude/skills/`. Pass `-g` to install globally to `~/.claude/skills/`.

### Claude Code Plugin

You can also install the skills as a Claude Code plugin, which exposes them under the `/moltable:<skill-name>` slash-command namespace:

```bash
# 1. Add the marketplace
claude plugin marketplace add moltable/skills

# 2. Install the plugin
claude plugin install moltable@moltable-skills
```

## The foundation skills

Four skills compose every GTM workflow above. Each is single-purpose; the agent picks the right combination based on what you ask.

<details>
<summary><strong>build-enrichment-table</strong> — the GTM workhorse</summary>

End-to-end recipe for turning "a list of entities + things to look up about each one" into a fully populated enrichment table. Scaffolds the workbook, picks the right input + Moltygent columns (LinkedIn lookups, web search, scrape providers, BYOK LLMs, native integrations), imports rows, runs with live progress, and exports.

**Use when:**

- Enriching a list of leads, companies, accounts, or any entities
- Building a TAM list (sourcing + filtering + qualifying)
- Researching accounts before outbound
- "Enrich this CSV", "research these companies", "build me a list of X"

</details>

<details>
<summary><strong>run-and-watch-jobs</strong> — execution + monitoring</summary>

Kick off, monitor, and stop moltable enrichment runs. Three modes: whole-table re-run, single-cell retry (for failed rows), or reattach to an in-flight job started elsewhere. Surfaces server-sent-events progress as a live stream so the agent can report status, surface failures, and decide when to move on.

**Use when:**

- Re-executing enrichment on an existing table
- Watching a long-running job in progress
- Stopping a stuck or expensive run
- Debugging a job that finished with errors

</details>

<details>
<summary><strong>auth-and-profiles</strong> — credentials</summary>

Set up and switch between moltable credentials. Browser-handoff login (no token pasting), multi-profile management for work + personal workspaces, and the resolver order the CLI walks when picking a key.

**Use when:**

- `moltable auth check` errors with "no auth configured"
- Adding a second workspace (work + personal)
- Commands fail with 401
- "Which moltable account am I using?" / "Log me in" / "Switch profiles"

</details>

<details>
<summary><strong>long-tail-fallback</strong> — advanced operations</summary>

Direct REST API recipes (via curl) for moltable operations the CLI doesn't cover — saved views, Salesforce / HubSpot / Apollo connections, webhook ingest, sample sets, checkpoints, API key rotation, folder reorganization. The agent reaches for this skill any time the standard CLI verbs come up short.

**Use when:**

- Setting up a Salesforce or HubSpot connection
- Configuring a webhook ingest endpoint
- Rotating an API key, creating a saved view, reorganizing folders
- Any moltable operation that doesn't map to a marquee CLI verb

</details>

## Usage

Skills auto-activate when the model decides the task matches the skill's description. You usually don't need to invoke them explicitly — just describe the GTM task you want done.

**Example agent session:**

> **You:** Here's a CSV of 250 prospects from a recent webinar. Can you enrich each one with their current company's tech stack, funding stage, and a verified email address? Then save it as a moltable so I can re-run it monthly.
>
> **Agent (via `build-enrichment-table`):** Created workbook "webinar-followup-2026Q2". Built table with 5 columns: name (input), company (input), tech stack (BuiltWith), funding stage (Crunchbase), verified email (Findymail waterfall → ZeroBounce). Importing 250 rows...
>
> **Agent (via `run-and-watch-jobs`):** Job started (job_2YR3K). Estimated 4 minutes. Watching...
>
> **Agent:** Done. 247 rows enriched. 3 failed on email verification (typos in source data). Table `tb_8XQ2P` ready. Want me to export to CSV or set up a saved view?

## Skill structure

Each skill follows the [Agent Skills Open Standard](https://agentskills.io/):

- `SKILL.md` — frontmatter (`name`, `description`, `when_to_use`) + markdown body the agent loads when the skill activates
- `references/` — (optional) deep-dive docs the agent fetches on demand

Plugin metadata in `.claude-plugin/marketplace.json` enumerates the skills bundled into the `moltable` plugin so the Claude Code plugin marketplace flow exposes them under the `/moltable:<skill-name>` namespace.

## Built on moltable

These skills assume the moltable CLI is installed:

```sh
curl -fsSL https://get.moltable.io | sh
moltable auth login
```

[moltable](https://moltable.io) is the agent-native enrichment-table platform: spreadsheets-meets-AI for GTM teams. Build a list, pick what to look up, hit run. Sources include LinkedIn lookups, web search, scrape providers (Apify, Bright Data, scrape.do), BYOK LLM enrichment (Claude / GPT / Gemini via OpenRouter), and integrations (Salesforce, HubSpot, Apollo, Crustdata, Bettercontact, +30 more).

## Contributing

Open a PR with a new skill in `skills/<your-skill>/SKILL.md` and add an entry to the plugin's `skills:` array in `.claude-plugin/marketplace.json`. Conventional commits + small, focused changes welcome.

Skill ideas we'd love to see:

- `tam-builder` — source companies by industry / size / signals → enrich → qualify
- `account-research-deep-dive` — one account, full personnel + news + intent
- `outbound-prep` — pick a lead, return a personalized cold-email context bundle
- `competitor-vendor-scan` — find companies using vendor X (via BuiltWith / tech graph)
- `intent-signal-watch` — alert when a list of accounts hits a buying signal

## License

MIT — see [LICENSE](./LICENSE).
