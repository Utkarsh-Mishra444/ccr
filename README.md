# ccr

**Bring back all your Claude Code chats after you close your terminals.**

You have five Claude Code chats going, each in its own Terminal window. Then you close them all, or the Mac restarts, and getting back is a chore: open a window, `cd` to the right folder, run `claude --resume`, find the chat in the picker, and repeat for every chat.

`ccr` does it in one command. It finds the chats you were typing in and opens each one in its own Terminal window, resumed in the folder it belongs to.

```
$ ccr
Chats you typed in since Sat Oct 31 5 AM: 4 (1 already open)  [8 files scanned in 0.30s]
      your last message     folder                  chat
  1  Sat Oct 31 23:19 PDT  ~/thesis                Draft the related-work section from my notes
  2  Sat Oct 31 22:32 PDT  ~/site                  Why does the build fail on the image step?
  3  Sat Oct 31 21:05 PDT  ~                       Help me sort out my Downloads folder
open  Sun Nov 01 00:57 PDT  ~/api                   Add retries to the upload client
Open which? Enter = all 3, numbers like 1 3, q = quit:
```

Press Enter and three windows open, each with its chat resumed.

## Install

```sh
curl -fsSL https://raw.githubusercontent.com/Utkarsh-Mishra444/ccr/main/ccr -o ~/.local/bin/ccr
chmod +x ~/.local/bin/ccr
```

Any folder on your `PATH` works. You need:

- macOS with Terminal.app
- Python 3 (standard library only; tested on 3.13)
- the [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI, as `claude` on your `PATH`

## Usage

| Command | What it does |
|---|---|
| `ccr` | Chats you typed in since 5 AM yesterday, newest first |
| `ccr 12h` / `ccr 3d` | Chats you typed in during the last 12 hours / 3 days |
| `ccr 2026-10-31` | Chats you typed in on that day (5 AM to 5 AM the next morning) |
| `ccr -l` | List only, open nothing |
| `ccr -y` | Open every listed chat without asking |
| `ccr -p 1,3` | Open only these numbers from the list, without asking |
| `ccr --force` | Skip the 30-second repeat guard |

At the prompt: Enter opens all of them, numbers like `1 3` open some, `q` quits.

## Already-open chats are skipped

Run `ccr` while some chats are still open and it won't open them twice. It reads the session records Claude Code keeps in `~/.claude/sessions/` (each one counts only if its process is still a running `claude`) and checks the command line of every running `claude --resume`. Those chats show as `open` at the bottom of the list and are never launched.

## Safety limits

So a bad day can't open a hundred windows:

- **At most 10 windows per run**, whatever you select.
- **A second launch within 30 seconds is refused**, so a double Enter or a repeated command does nothing.
- Chats already open are skipped, and so are chats whose folder no longer exists.

## How it works

- It reads the transcripts Claude Code keeps locally in `~/.claude/projects/*/*.jsonl`, opening only files modified inside the time window, so a normal run takes well under a second.
- A chat is listed only if **you typed** in it during the window. Tool results, notifications, agent messages and bookkeeping lines don't count, so a chat that only received a background notice stays off the list. The time shown is your last message.
- Each chat opens with `cd <its folder> && claude --resume <id>`. The folder matters: `claude --resume` only finds a chat from the folder it was started in.
- Windows open through small `.command` files passed to `open -a Terminal`, not through AppleScript. It reuses at most ten of them, in `~/.claude/ccr/`.
- No model is called and nothing leaves your machine.

## Limits

- macOS and Terminal.app only for now (no iTerm2, Ghostty or Linux yet).
- It depends on Claude Code's local transcript format, which isn't a documented API, so a Claude Code update could break it.
- Only chats started from the interactive CLI are listed: transcripts whose `entrypoint` field isn't `cli` are skipped.
- If you switched chats inside a running window with `/resume`, it hasn't been tested whether `ccr` sees the new chat as open.

## Uninstall

Delete `~/.local/bin/ccr` (or wherever you put it) and the `~/.claude/ccr/` folder.

## License

MIT
