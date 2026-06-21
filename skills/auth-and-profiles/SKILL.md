---
name: auth-and-profiles
description: Set up and switch between moltable credentials — browser-handoff login, multi-profile management, and verifying which identity is active.
when_to_use: When `moltable auth check` errors with "no auth configured" (exit code 2), when the user wants to add a second workspace or organization (work + personal), when commands fail with 401, or when the user asks "which moltable account am I using" / "log me in" / "switch profiles".
---

# Auth and Profiles

Set up moltable credentials, manage multiple profiles (work + personal workspaces), and verify which identity the CLI is currently using. moltable uses **org-scoped API keys** (prefix `molt_`) — one key per `(user, org)` pair, each held by exactly one profile in the local TOML config. Login is a one-command browser-handoff dance; no token pasting required. This skill is the prerequisite for every other moltable GTM workflow.

## Quick triage

Run `moltable auth check --json` first:

| Result | Action |
| --- | --- |
| Exit 0, prints `{profile, user, org_id, ...}` | Already logged in. Use "Add another profile" if a different one is wanted. |
| Exit 2, "no auth configured" | Run "First-time login" below. |
| 401 unauthorized | Key was revoked. `moltable auth logout` then re-run login. |

## First-time login

`moltable auth login` opens a browser handoff. The user signs in via Clerk, approves the CLI, and the key flows back to `~/.config/moltable/config.toml`.

```
moltable auth login
```

The first profile defaults to `personal` and becomes the default for subsequent commands.

If the browser doesn't open (SSH, headless), print the URL and the poll loop keeps running until the user opens it on their workstation.

## Add another profile (work + personal)

```
moltable auth login --profile work        # add a new named profile
moltable --profile work table list        # use it per-call
moltable profile use work                  # OR switch default permanently
```

## Inspect / list / remove profiles

```
moltable profile list --json
moltable profile use <name>     # set default
moltable profile remove <name>  # drop a profile (errors if it's the current default)
```

## Logout

```
moltable auth logout                # removes the default profile
moltable auth logout --profile work # removes a named profile
```

**`logout` removes the local profile, NOT the server-side API key.** If the machine is compromised, revoke in the web UI at `https://app.moltable.io/settings/api-keys`.

## Resolver order

When multiple sources set a key (highest wins):

1. `--api-key molt_…` flag
2. `MOLTABLE_API_KEY` env var
3. `--profile <name>` flag
4. `MOLTABLE_PROFILE` env var
5. `default_profile` in `config.toml`

`auth check --json` exposes which source won via the `source` field.

## CI

Skip the login dance — set the env var:

```yaml
env:
  MOLTABLE_API_KEY: ${{ secrets.MOLTABLE_API_KEY }}
run: |
  moltable row import --table tb_X --csv leads.csv --json
```

## Local development (`--dev`)

For users running the moltable backend locally, the `--dev` flag retargets every call at `https://localhost:8080` and skips TLS verification — but **only when the host is loopback**, so combining `--dev` with `MOLTABLE_API_BASE=https://something.example` keeps full TLS. Same effect via `MOLTABLE_DEV=1`.

```
moltable --dev auth login --profile dev
```

## Tips for agents

- Run `moltable auth check --json` first in any fresh session. Use the returned `org_id` and `profile` to ground the user on which workspace they're in.
- "Log me into work too" means a *second* profile — pass `--profile work`, don't overwrite the existing one.
- Don't read the TOML directly to grab the key — use `moltable config get api-key` (see `long-tail-fallback`).
