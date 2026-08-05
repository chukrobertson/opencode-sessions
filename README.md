# opencode-sessions

Sync [OpenCode](https://opencode.ai) chat history across devices by storing sessions in your project folders.

## Quick start

### Step 1 — install the tool

```bash
curl -fsSL https://raw.githubusercontent.com/chukrobertson/opencode-sessions/master/opencode-sessions -o ~/.local/bin/opencode-sessions
chmod +x ~/.local/bin/opencode-sessions
```

Requires Python 3. Zero dependencies.

### Step 2 — install the OpenCode commands

```bash
git clone https://github.com/chukrobertson/opencode-sessions /tmp/opencode-sessions
mkdir -p ~/.config/opencode/commands
cp /tmp/opencode-sessions/commands/*.md ~/.config/opencode/commands/
```

Then use these inside OpenCode on any project:

| Command | Action |
|---|---|
| `/sessions-export` | Save sessions into `.opencode/sessions/` |
| `/sessions-import` | Load `.opencode/sessions/` into your DB |
| `/sessions-status` | Show what needs syncing |

### Step 3 — export, sync, import

On your first machine:

```
/sessions-export
```

Sync the project folder to your second machine (rsync, Syncthing, Dropbox, etc.).

On your second machine:

```
/sessions-import
```

Your OpenCode sessions now appear on both machines. Repeat in either direction.

Already-imported sessions are skipped — safe to re-run anytime.

## Shell usage

Prefer the terminal?

```
opencode-sessions export  <dir>    Write sessions from DB into .opencode/sessions/
opencode-sessions import  <dir>    Load .opencode/sessions/ JSON files into DB
opencode-sessions status  <dir>    Show sync status (in-DB vs in-project)
opencode-sessions list    [dir]    Browse all sessions
opencode-sessions sync    <dir>    Export + import in one step
```

## Status output

```
$ opencode-sessions status ~/my-project

  In DB only:      3      ← run 'export'
  In project only:  5      ← run 'import'
  Synced:          12
```

## How it works

- Reads/writes OpenCode's SQLite database at `~/.local/share/opencode/opencode.db`
- Each session becomes a JSON file in `.opencode/sessions/` — full conversation, messages, and tool outputs
- Sessions are UUID-keyed so duplicates are safely ignored
- Project paths are matched by filesystem path; missing projects are created automatically on import

## .gitignore

The first time you export or import in a project, the tool adds `.opencode/` to that
project's `.gitignore`. Sessions stay private — they sync via your file sync tool,
not via git.

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
