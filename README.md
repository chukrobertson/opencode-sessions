# opencode-sessions

Sync [OpenCode](https://opencode.ai) chat history across devices by storing sessions in your project folders.

```
opencode-sessions export ~/my-project   # laptop: save sessions to .opencode/sessions/
# ... sync project folder via rsync / Syncthing / Dropbox / NFS / USB drive ...
opencode-sessions import ~/my-project   # desktop: load sessions into local DB
```

## Install

```bash
curl -fsSL https://raw.githubusercontent.com/chukrobertson/opencode-sessions/master/opencode-sessions -o ~/.local/bin/opencode-sessions
chmod +x ~/.local/bin/opencode-sessions
```

Requires Python 3. Zero dependencies.

Then install the OpenCode commands:

```bash
cp commands/*.md ~/.config/opencode/commands/
```

## OpenCode integration

After installing the commands, use these inside OpenCode:

| Command | Action |
|---|---|
| `/sessions-export` | Export sessions for the current project |
| `/sessions-import` | Import synced sessions from the project |
| `/sessions-status` | Show sync status |

Or run directly from your shell:

```
opencode-sessions export  <dir>    Write sessions from DB into .opencode/sessions/
opencode-sessions import  <dir>    Load .opencode/sessions/ JSON files into DB
opencode-sessions status  <dir>    Show sync status (in-DB vs in-project)
opencode-sessions list    [dir]    Browse all sessions
opencode-sessions sync    <dir>    Export + import in one step
```

## Workflow

**Laptop → desktop:**

```bash
# 1. On laptop, before syncing:
opencode-sessions export ~/my-project

# 2. Sync the project folder to your desktop (rsync, Syncthing, Dropbox, etc.)

# 3. On desktop, after syncing:
opencode-sessions import ~/my-project
```

Already-imported sessions are skipped — safe to re-run. Do the reverse to go desktop → laptop.

```
opencode-sessions status ~/my-project
```

```
  In DB only:      3      ← run 'export'
  In project only:  5      ← run 'import'
  Synced:          12
```

## How it works

- Reads/writes [OpenCode's SQLite database](https://github.com/anomalyco/opencode) at `~/.local/share/opencode/opencode.db`
- Each session becomes a JSON file in `.opencode/sessions/` — full conversation, messages, and tool outputs
- Sessions are UUID-keyed so duplicates are safely ignored
- Project paths are matched by filesystem path; missing projects are created automatically on import

## .gitignore

The first time you export or import in a project, the tool adds `.opencode/` to that
project's `.gitignore`. This keeps personal chat history out of version control.
The JSON files still sync via any file synchronization method — they just won't
end up in git commits.

To undo:

```
.opencode/   ← remove this line from .gitignore
```

## File structure

```
my-project/
├── .opencode/
│   └── sessions/
│       ├── ses_02d671523ffe...json
│       └── ...
├── src/
└── .gitignore
```

## Why not just sync the database?

`opencode.db` can be 100+ MB and references machine-specific project paths.
Per-project JSON files are smaller, portable, and only travel with the code
they're about.

## License

MIT
