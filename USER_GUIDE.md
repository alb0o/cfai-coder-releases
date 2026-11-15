# User guide — cfai-code

This guide explains how to install and use `cfai-code`, the terminal chat built on this project's Rust engine.
It is not a development document: for the state of the port, conformance evidence and technical decisions, see
the private source repository's `README.md` and `research/` (not public). Entries without a date are stable
across several updates; when the binary changes, this guide should be updated alongside it.

## 1. Installation

Download the archive for your platform from this repository's [Releases](../../releases) page.

**Windows:** unzip, then run `cfai-code.exe` from a terminal (PowerShell or cmd) in the folder you want to work
in, or add it to your `PATH` to run it from anywhere.

**Linux / macOS:**
```sh
tar -xzf cfai-code-<platform>.tar.gz
chmod +x cfai-code
./cfai-code
```

`cfai-code` always reads and writes relative to the folder it is started from (`--cwd` to point it elsewhere).

These binaries are not signed yet (no Windows Authenticode, no macOS notarization) — see the main README for
what to expect from SmartScreen / Gatekeeper on first run.

## 2. Sign-in

`cfai-code` needs to know which model to talk to. There are three ways, simplest first:

1. **A CFAI account (recommended).** On first launch, if nothing else is configured, a welcome screen asks for
   your CFAI account's email and password. The token you get back is saved in your user configuration folder and
   reused on the next launches: nothing more to type. To sign in or out at any time, without opening the chat:
   ```sh
   cfai-code login
   cfai-code logout
   ```
2. **Your own Anthropic key.** `cfai-code config`, choose `provider: anthropic`, then enter your key (`api_key`)
   and, if you want, a model. In the distributed build the request still goes through the CFAI service (your key
   travels in a dedicated header and never becomes the account that pays).
3. **Your own model, even a local one.** `provider: custom`, with `base_url` pointing at an endpoint that speaks
   Anthropic's Messages API (for example a local server at `http://localhost:8080`) and, if the server requires
   one, `api_key`.

The environment always wins over saved settings: `ANTHROPIC_API_KEY`, `ANTHROPIC_MODEL` and `ANTHROPIC_BASE_URL`,
read from the terminal's environment or from a `.env` file in the working directory (`--env-file FILE` points at
a different one).

## 3. Settings (`cfai-code config`)

```sh
cfai-code config            # opens an interactive menu
cfai-code config list       # shows current values (the key is masked)
cfai-code config get <name>
cfai-code config set <name> <value>
cfai-code config reset      # clears all saved settings
```

The same menu is reachable from inside the chat with `/config` (or `/settings`). Available settings:

| Name | Values | Meaning |
| --- | --- | --- |
| `provider` | `cfai` (default) · `anthropic` · `custom` | the CFAI service, your own Anthropic key, or your own model |
| `model` | free text | model name; empty keeps the provider's default |
| `base_url` | free text | `custom` only: the endpoint's address |
| `api_key` | secret | `anthropic` and `custom` only (a local model that needs none can leave it empty) |
| `temperature` | number from 0 to 2 | empty keeps the provider's default |
| `reasoning` | `default` · `low` · `medium` · `high` | how much the CFAI service's model thinks before answering (`cfai` only) |
| `verify_command` | free text | the command that checks the work, e.g. `python -m unittest -q` or `cargo test`; empty: no check |
| `verify_every` | number from 0 to 1000 | runs the check on its own after this many tool calls since the last one (0: only when the model says it is done) |
| `verify_tries` | number from 0 to 20 | how many times a failing check is sent back to the model before the stronger one takes over |
| `escalate_model` | free text | the model that takes over when checks keep failing (`cfai-escalate` on the CFAI service); empty: none |
| `profile` | `local` · `gateway` | `cfai` only: whether the prompt for the small model is shortened here (`local`) or by the service (`gateway`, needs the service to support it) |

Saved settings act as defaults: an environment variable you already set always wins over them.

## 4. Starting the chat

```sh
cfai-code                     # opens the chat in the current folder
cfai-code --cwd /some/path    # opens the chat in another folder
cfai-code -p "do this"        # answers once and exits, without opening the chat
```

Some useful startup options (the full list is in `cfai-code --help`):

- `--model <name>` — use a different model than the default for this session.
- `-c`, `--continue` — resume this folder's last session.
- `-r`, `--resume [id]` — list saved sessions, or resume the one with the given id.
- `--permission-mode <mode>` — `default` (asks for confirmation), `acceptEdits`, `plan`, `dontAsk`, `bypassPermissions`.
- `--add-dir <folders...>` — also grant access to folders outside the working directory.
- `--no-project-instructions` — do not read `CLAUDE.md` files from the working directory and its parents.

Inside the chat: Enter sends the line; a trailing backslash (`\`) continues on the next line without sending.
Ctrl-C interrupts the current response without closing the program.

## 5. Chat commands

```
/help                 this list
/exit, /quit          leave
/clear, /new          start a new session with the next message
/compact [notes]      summarise the session to free context (notes steer the summary)
/compact list          number the messages you wrote, to cut at
/compact from N [notes]   summarise from your message N on and keep the earlier ones
/compact upto N [notes]   summarise what comes before your message N and keep the rest
/model [name]          show or change the model
/permissions [mode]    show or change the permission mode
/resume [number|id]    list the saved sessions, or continue one
/cost                  tokens and cost of this chat
/status                model, directory, mode and session in use
/config [list|set <name> <value>]   the settings described above
!command               run a shell command here, without the model
/name args             run a project command (.claude/commands), a skill or an MCP prompt
@file                  attach a file to the message
```

## 6. Non-interactive use (`-p`)

To use `cfai-code` from a script or a pipeline, without opening the chat:

```sh
cfai-code -p "add a test for function X" --output-format json
echo "prompt text" | cfai-code -p
```

`--output-format text|json|stream-json` chooses the response format (`stream-json` needs `--verbose`);
`--allowedTools`/`--disallowedTools` restrict or exclude individual tools; `--max-turns` limits conversation
turns. The process's exit code follows the outcome of the response.

## 7. Where data is saved

- **Settings** (`cfai-code config`): `cfai-settings.json` in the user's configuration folder.
- **Saved sign-in**: `cfai-session.json`, same folder; holds only the service token, not the password.
- **Chat sessions**: `.cfai/sessions` inside the working directory (`--journal-dir` to change it). This is the
  project's own format, not the original `claude` CLI's.

None of these files are sent by `cfai-code` to any server other than the one chosen in `provider`.

## 8. Common problems

- **"set --model or ANTHROPIC_MODEL"**: no model was chosen, neither with `--model` nor `ANTHROPIC_MODEL` nor
  `cfai-code config`. Set one of the three.
- **The program asks for email and password on every launch**: the sign-in was not saved, probably because the
  user's configuration folder is not writable. Try `cfai-code login` and check the printed error.
- **The model cannot see a file outside the working directory**: add the folder with `--add-dir`, or restart
  with `--cwd` on the right folder.
- **An API key does not seem to be the expected one**: the environment (`ANTHROPIC_API_KEY`/`ANTHROPIC_AUTH_TOKEN`/
  `ANTHROPIC_BASE_URL` already set in the terminal) always wins over settings saved with `cfai-code config`.
  Check what is saved with `cfai-code config list`, and what is in the environment with `echo $ANTHROPIC_API_KEY`
  (or `echo $env:ANTHROPIC_API_KEY` on PowerShell).
