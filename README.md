# nul-skills

Northwestern University Libraries community AI skills catalog. This repository is a
[Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces)
and is also consumed by [`nul-cli`](https://github.com/nulib-labs/nul-cli),
which extracts each plugin's skill folders and installs them for
non-Claude AI assistants (Codex, ChatGPT, Cursor, …).

This catalog contains no Python package or Python dependencies. Use `uv` to
install and run `nul-cli` for the non-Claude install path.

Two install paths for skills:

- **Claude Code users** — `/plugin marketplace add nulib-labs/nul-skills` then
  `/plugin install <plugin>@nul-skills`. Skills become `/<plugin>:<skill>`.
- **Other agents** — `nul skills add <plugin>` writes each skill to
  `.agents/skills/<plugin>-<skill>/SKILL.md`.

Install Meadow directly in current Claude Code (2.1.275 or newer):

```text
/plugin install meadow --marketplace nulib-labs/nul-skills
```

For other agents:

```bash
uv tool install nul-cli
nul skills add meadow
```

## Layout

```
nul-skills/
├── .claude-plugin/
│   └── marketplace.json                     # catalog (Claude convention)
├── plugins/
│   └── <plugin-name>/
│       ├── .claude-plugin/
│       │   └── plugin.json                  # plugin manifest
│       └── skills/
│           └── <skill-name>/
│               └── SKILL.md
└── README.md
```

Each plugin can ship one or more skills. A plugin may also include
`commands/`, `agents/`, `hooks/`, `.mcp.json`, etc. — those are Claude
Code-specific and are ignored by `nul-cli`, which installs the skill folders
and their supporting files. See [Claude Code's plugins reference](https://code.claude.com/docs/en/plugins-reference)
for the full plugin schema.

## Adding a Plugin

1. Create the plugin directory:
   ```
   plugins/<plugin>/
     .claude-plugin/plugin.json
     skills/<skill>/SKILL.md
   ```
2. Fill in `plugin.json`:
   ```json
   {
     "name": "<plugin>",
     "version": "1.0.0",
     "description": "What the plugin does.",
     "author": { "name": "Northwestern University Libraries" }
   }
   ```
   Include a semantic `version` and bump it whenever you release plugin changes.
   Claude Code keeps a versioned plugin on that release until its version changes.
   Omitting the version uses commit-based updates but produces a validator warning.
3. Write `skills/<skill>/SKILL.md` with YAML frontmatter on the first line,
   including `name:` matching the skill directory and `description:` explaining
   when an agent should invoke it. Keep names lowercase with hyphens. Add scripts
   or reference files inside the skill directory and link to them from `SKILL.md`.
   Use standard Agent Skills fields for portability; Claude-specific fields and
   `${CLAUDE_PLUGIN_ROOT}` substitutions need corresponding support in other clients.
   See the [skills reference](https://code.claude.com/docs/en/skills).
4. Add an entry to `.claude-plugin/marketplace.json` under `plugins`:
   ```json
   {
     "name": "<plugin>",
     "source": "./plugins/<plugin>",
     "description": "One-line summary."
   }
   ```
5. Bump the changed plugin's `version` in `plugin.json`, then open a PR.
   No catalog version bump is required. `nul-cli` independently upgrades each
   installed skill when its plugin's path gets a new commit.

## Validation

Run Claude Code's plugin validator from the repository root before opening a
PR:

```bash
claude plugin validate . --strict
claude plugin validate ./plugins/meadow --strict
```

Validate each plugin as well as the marketplace. A plugin submitted to Anthropic's
public directory also needs a listing icon; it is optional for this team marketplace.

### Naming guidance

Inside Claude Code, skills are namespaced as `/<plugin-name>:<skill-name>`. So
`plugins/meadow/skills/run-tests/SKILL.md` becomes `/meadow:run-tests`.
Pick the inner skill name to read well after the colon. For non-Claude clients,
`nul-cli` installs each skill at `.agents/skills/<plugin-name>-<skill-name>/`
to avoid collisions between plugins.

## Versioning

`nul-cli` `skills upgrade` calls
`GET /repos/nulib-labs/nul-skills/commits?path=plugins/<plugin>` and compares
the resolved SHA against the manifest stamped at install time. Any commit
that touches the plugin's path triggers an upgrade for every installed skill
from that plugin.

Claude Code uses the plugin's `version` in `plugin.json` to detect new releases.
Bump it whenever the plugin's skills or supporting files change. This is separate
from `nul-cli`'s commit-based upgrades above.

The `metadata.version` field in `marketplace.json` is informational only.

## Local Development with `nul-cli`

```bash
export NUL_SKILLS_PATH=/path/to/nul-skills
nul skills list
nul skills add meadow --dest=/tmp/agents
# installs /tmp/agents/meadow-run-tests/
```

Without `NUL_SKILLS_PATH`, `nul-cli` fetches from `nulib-labs/nul-skills`.
Override the source via `NUL_SKILLS_REPO` and `NUL_SKILLS_REF`.
