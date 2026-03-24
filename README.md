# Atlassian Skills for OpenClaw

The complete Atlassian stack for your OpenClaw agent — **Jira**, **Confluence**, and **Bitbucket Cloud** as bash CLI wrapper skills.

Each skill is a self-contained folder with a `SKILL.md` and a bash script. Drop them into your OpenClaw workspace and your agent can manage tickets, read documentation, and review PRs through natural conversation.

## Why bash wrappers?

Most OpenClaw Atlassian integrations either give the agent raw curl commands in a SKILL.md (unreliable — the agent hallucinates URLs and flags) or require TypeScript plugins (heavyweight — needs compilation and custom container builds).

These skills sit in the middle: **bash scripts that wrap the REST APIs** with proper auth, error handling, URL encoding, and clean JSON output. The agent calls the script, gets structured data back, and never has to guess at API details.

Dependencies: `curl` + `python3` — both already in the stock OpenClaw container. No `jq`, no npm packages, no compilation.

## Skills

| Skill | Commands | Access |
|---|---|---|
| **Jira** | search, get, create, comment, transitions, transition, assign, update, users, projects | Read + Write |
| **Confluence** | spaces, pages, get, children, search, create, update, labels, add-labels | Read + Write |
| **Bitbucket** | repos, prs, pr, diffstat, diff, comments, pr-commits, branches, commits, file, ls | Read-Only |

## Installation

### From ClawHub

```bash
clawhub install pejovicvuk/jira
clawhub install pejovicvuk/confluence
clawhub install pejovicvuk/bitbucket
```

### Manual

Copy the skill folders into your OpenClaw workspace:

```bash
cp -r jira/ ~/.openclaw/workspace/skills/jira/
cp -r confluence/ ~/.openclaw/workspace/skills/confluence/
cp -r bitbucket/ ~/.openclaw/workspace/skills/bitbucket/

# Make scripts executable
chmod +x ~/.openclaw/workspace/skills/jira/jira-cli.sh
chmod +x ~/.openclaw/workspace/skills/confluence/confluence-cli.sh
chmod +x ~/.openclaw/workspace/skills/bitbucket/bitbucket-cli.sh
```

## Configuration

### Credentials

You need two API tokens from [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens):

1. **Atlassian API token** — used by Jira and Confluence (full access)
2. **Bitbucket API token** — separate token scoped to Bitbucket with **Repositories: Read** and **Pull requests: Read** only

### Environment variables

Set these on your OpenClaw gateway (in `docker-compose.yml`, `.env`, or your shell):

```bash
# Jira + Confluence (shared credentials)
ATLASSIAN_URL=https://yourcompany.atlassian.net
ATLASSIAN_EMAIL=you@yourcompany.com
ATLASSIAN_API_TOKEN=ATATT3xFfG...

# Bitbucket (separate read-only token)
BITBUCKET_API_TOKEN=ATATT3xFfG...  # different token, read-only scoped
BITBUCKET_WORKSPACE=yourcompany     # from bitbucket.org/{workspace}/{repo}
```

### Verify credentials

```bash
# Test Jira
curl -s -u "$ATLASSIAN_EMAIL:$ATLASSIAN_API_TOKEN" "$ATLASSIAN_URL/rest/api/3/myself"

# Test Confluence
curl -s -u "$ATLASSIAN_EMAIL:$ATLASSIAN_API_TOKEN" "$ATLASSIAN_URL/wiki/api/v2/spaces"

# Test Bitbucket
curl -s -u "$ATLASSIAN_EMAIL:$BITBUCKET_API_TOKEN" "https://api.bitbucket.org/2.0/user"
```

## Usage examples

Once installed, just talk to your agent:

- *"Show me my open Jira tickets"*
- *"Create a bug for the login page — users can't sign in with unicode emails"*
- *"What PRs are open in the backend repo?"*
- *"Show me the diff for PR #42"*
- *"Search Confluence for the deployment runbook"*
- *"Write a test report page in Confluence under the QA space"*

The agent picks the right skill based on context.

## Security

- **Bitbucket is read-only by design.** The skill uses a separate scoped token. Even if the agent is instructed to write, the API will reject it.
- **Jira and Confluence have write access** for creating tickets, adding comments, and publishing pages. The agent follows the rules in SKILL.md (e.g. "confirm before creating Critical bugs").
- **No credentials in SKILL.md.** All auth is via environment variables, never hardcoded.
- **No external dependencies.** The scripts use only `curl` and `python3` — no third-party packages to audit.

## Note on Bitbucket authentication

As of September 2025, Atlassian deprecated Bitbucket app passwords. Use **scoped API tokens** instead. The username for Bitbucket API auth is your **Atlassian account email** (not your Bitbucket username). See [Atlassian's migration guide](https://www.atlassian.com/blog/bitbucket/bitbucket-cloud-transitions-to-api-tokens-enhancing-security-with-app-password-deprecation) for details.

## License

MIT — see [LICENSE](LICENSE).

## Author

Built by [Viksi.ai](https://viksi.ai) — AI agent deployment and management for teams.
