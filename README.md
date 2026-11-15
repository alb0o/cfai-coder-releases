# cfai-coder — binaries

Compiled binaries of `cfai-code`, the terminal chat built on a Rust rewrite of the Claude Code engine and SDK.
The source and the development documentation are in a separate, private repository; this one only hosts the
releases people download and run.

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
release is built directly from the tagged source by the CI pipeline in the private repository (see the workflow
run linked from each release) and published here by that same pipeline — nobody uploads these files by hand.

## First run

`cfai-code` needs a way to reach a model: on first launch (with nothing already configured) it shows a sign-in
screen for a CFAI account, or run `cfai-code config` to use your own Anthropic key or your own model endpoint.
See [**USER_GUIDE.md**](USER_GUIDE.md) for the full walkthrough: installation, the three ways to connect to a
model, every setting of `cfai-code config`, the chat commands, non-interactive use with `-p`, where files are
saved, and common problems.

## Building from source

These binaries are produced from the private source repository; it is not public yet. If you want to build it
yourself, ask the maintainer for access.
