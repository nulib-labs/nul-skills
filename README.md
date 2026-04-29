# nul-skills

Northwestern University Libraries community AI skills catalog. This repository is a
[Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugin-marketplaces)
and is also consumed by [`nul-cli`](https://github.com/nulib-labs/nul-cli),
which extracts the SKILL.md files out of each plugin and installs them for
non-Claude AI assistants (Codex, ChatGPT, Cursor, …).

Two install paths for skills:

- **Claude Code users** — `/plugin marketplace add nulib-labs/nul-skills` then
  `/plugin install <plugin>@nul-skills`. Skills become `/<plugin>:<skill>`.
- **Other agents** — `nul skills add <plugin>` writes each skill to
  `.agents/skills/<plugin>-<skill>/SKILL.md`.

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
Code-specific and are ignored by `nul-cli`, which only installs the SKILL.md
files. See [Claude Code's plugins reference](https://docs.claude.com/en/docs/claude-code/plugins-reference)
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
     "description": "What the plugin does.",
     "author": { "name": "Northwestern University Libraries" }
   }
   ```
   Omit `version` to let updates flow on every commit (Claude Code uses the
   commit SHA at the plugin's path); set `version` if you want explicit
   release gating.
3. Write `skills/<skill>/SKILL.md` with a `description:` front-matter that
   explains when Claude (or another agent) should invoke it.
4. Add an entry to `.claude-plugin/marketplace.json` under `plugins`:
   ```json
   {
     "name": "<plugin>",
     "source": "./plugins/<plugin>",
     "description": "One-line summary."
   }
   ```
5. Open a PR. No catalog version bump required — `nul-cli` upgrades each
   installed skill when its plugin's path gets a new commit.

## Validation

Run Claude Code's plugin validator from the repository root before opening a
PR:

```bash
claude plugin validate .
```

Expected output:

```text
Validating marketplace manifest:
/example/path/nul-skills/.claude-plugin/marketplace.json

✔ Validation passed
```

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
