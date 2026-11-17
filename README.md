# cfai-coder — binaries

Compiled binaries of `cfai-code`, the terminal chat built on a Rust rewrite of the Claude Code engine and SDK.
The source and the development documentation are in a separate, private repository; this one only hosts the
releases people download and run.

## Latest version: 0.2.1

0.2.1 is a maintenance release of 0.2.0: the way the agent works is unchanged. It no longer adds a default
attribution line to the commit messages and pull request descriptions it writes, and the `--help` and `--version`
texts are simpler. The changes below came with 0.2.0.

- **A check that keeps the agent honest.** With a `verify_command` set (for example your test suite), the work is
  checked as it goes, a failing check is sent back to the model, and a stronger model takes over when the small one
  cannot make it pass or goes around in circles without changing anything. If the work still ends with a failing
  check, the answer says so instead of claiming success.
- **Existing test files are protected** while a check is configured, so the agent fixes the code rather than the test.
- Clearer messages when the model sends a malformed tool call, and a plain-text fallback for a search that is not a
  valid regular expression.
- New settings: `verify_timeout`, `verify_fast`, `escalate_idle` (see [USER_GUIDE.md](USER_GUIDE.md)).

### How it was tested

On 16 problems of SWE-bench Lite (real bug reports on real open-source repositories, graded with the benchmark's
official tests), version 0.2.0 solved **16 out of 16**, at a cost of about 2-3 US cents per problem (0.2.1 differs
only in the points above and was not measured again). Read it as a
first signal, not a guarantee: the problems are public, they come with a ready-made test command that your own
projects may not have, and 16 problems is a small sample. The testing is ongoing and more problems will be added.

Each problem was started from a fresh copy of the project, with no hint of the solution: the agent got the bug
report and had to change the code until the project's own tests passed. The stronger model only steps in when the small one is stuck
or its checks keep failing.

| # | Problem | Repository | Result | Models | Cost |
|---|---|---|---|---|---|
| 1 | `django-12284` | django | solved | small + stronger | $0.026 |
| 2 | `django-12286` | django | solved | small only | $0.002 |
| 3 | `django-13220` | django | solved | small + stronger | $0.014 |
| 4 | `django-13551` | django | solved | small + stronger | $0.008 |
| 5 | `django-14238` | django | solved | small + stronger | $0.040 |
| 6 | `django-15320` | django | solved | small + stronger | $0.054 |
| 7 | `requests-3362` | psf | solved | small only | $0.006 |
| 8 | `pytest-5221` | pytest-dev | solved | small + stronger | $0.133 |
| 9 | `pytest-7168` | pytest-dev | solved | small + stronger | $0.024 |
| 10 | `sphinx-7975` | sphinx-doc | solved | small + stronger | $0.024 |
| 11 | `sphinx-8713` | sphinx-doc | solved | small only | $0.002 |
| 12 | `sympy-11400` | sympy | solved | small + stronger | $0.069 |
| 13 | `sympy-14774` | sympy | solved | small only | $0.003 |
| 14 | `sympy-21847` | sympy | solved | small only | $0.002 |
| 15 | `sympy-22714` | sympy | solved | small only | $0.009 |
| 16 | `sympy-24152` | sympy | solved | small only | $0.002 |
| | **16 problems** | | **16 / 16 solved** | | **$0.42 total** |

A separate set of 29 small everyday coding tasks (rename a function, fix a failing test, add a feature, and so on)
run through the same binary solved 58 of 58 runs with the checking enabled (57 of 58 without it). More results will be added here as the testing grows.

## Download

See the [Releases](../../releases) page. Each release has one archive per platform:

| Platform | File |
| --- | --- |
| Windows (x86_64) | `cfai-code-windows-x86_64.zip` |
| Linux (x86_64) | `cfai-code-linux-x86_64.tar.gz` |
| macOS (Intel) | `cfai-code-macos-x86_64.tar.gz` |
| macOS (Apple Silicon) | `cfai-code-macos-arm64.tar.gz` |

## Install

**Windows:** unzip, then run `cfai-code.exe` from a terminal (PowerShell or cmd) in the folder you want to work
in, or add it to your `PATH` to run it from anywhere.

**Linux / macOS:**
```sh
tar -xzf cfai-code-<platform>.tar.gz
chmod +x cfai-code
./cfai-code
```

## These binaries are not signed

There is no code-signing certificate (Windows Authenticode) and no Apple notarization yet. Your system will
warn you the first time you run one:

- **Windows:** SmartScreen says "Windows protected your PC". Click "More info", then "Run anyway".
- **macOS:** Gatekeeper refuses to open it. Either right-click (or Control-click) the file → "Open" → confirm in
  the dialog that appears, or from a terminal: `xattr -d com.apple.quarantine cfai-code` before running it.
- **Linux:** no warning, but the file needs the executable bit (`chmod +x`, above).

This is expected for an unsigned binary from a small project, not a sign that the file was tampered with. Each
release is built directly from the tagged source by an automated pipeline in the private repository and
published here by that same pipeline.

## First run

`cfai-code` needs a way to reach a model: on first launch (with nothing already configured) it shows a sign-in
screen for a CFAI account, or run `cfai-code config` to use your own Anthropic key or your own model endpoint.
See [**USER_GUIDE.md**](USER_GUIDE.md) for the full walkthrough: installation, the three ways to connect to a
model, every setting of `cfai-code config`, the chat commands, non-interactive use with `-p`, where files are
saved, and common problems.

## Building from source

These binaries are produced from the private source repository; it is not public yet. If you want to build it
yourself, ask the maintainer for access.
