# codex-context

A tiny live context-health monitor for OpenAI Codex Desktop.

Codex threads can carry a surprisingly large amount of hidden bootstrap context before you start doing meaningful work: system instructions, tools, project rules, skills, repository metadata, and other environment context.

`codex-context` watches Codex's local session telemetry, automatically treats the initial context as a baseline, and tracks how much the working thread grows beyond it.

## Live meter

The meter updates in place every second. The filled bar, percentage, and status change color automatically as the thread grows.

```text
🟢  my-project  ███████████░░░░░░░░░░░░░  46%  46.2k / 100k  HEALTHY
🟡  my-project  █████████████████░░░░░░░  74%  74.0k / 100k  CONTEXT GETTING FULL
🟠  my-project  ████████████████████░░░░  85%  85.0k / 100k  WRAP UP SOON
🔴  my-project  ██████████████████████░░  93%  93.0k / 100k  NEW THREAD RECOMMENDED
```

> In a real terminal, the filled bar and percentage use matching ANSI colors: green → yellow → orange → red. GitHub README code blocks cannot render those terminal ANSI colors, so the colored status dots above show the same progression visually.

### Health states

| Thread growth | Meter color | Status |
| --- | --- | --- |
| `< 70k` | 🟢 Green | **HEALTHY** |
| `70–82k` | 🟡 Yellow | **CONTEXT GETTING FULL** |
| `82–90k` | 🟠 Orange | **WRAP UP SOON** |
| `90k+` | 🔴 Red | **NEW THREAD RECOMMENDED** |

These thresholds are intentionally opinionated workflow recommendations, not OpenAI product limits.

## Why

A fresh Codex thread may already report tens of thousands of active-context tokens after a tiny first message. That does not mean the visible conversation itself is that large.

`codex-context` separates those two concepts:

- **Baseline** — the initial Codex/system/project bootstrap context.
- **Thread growth** — context accumulated after that baseline.

The default meter applies a deliberately conservative **100,000-token working cap to thread growth**, rather than to the model's full technical context window. The 100k value is a workflow guardrail, not a claim about Codex's model limit.

## Features

- Live terminal meter, refreshing every second
- Automatic baseline per Codex thread
- Tracks thread growth instead of bootstrap context
- Monotonic accounting across context compaction
- Colored context-health states
- 100k conservative working cap
- Verbose view with baseline, current context, and actual model window
- No API key
- No network requests
- Reads Codex telemetry locally
- Works with Codex Desktop session rollouts

## Requirements

- macOS or another environment where Codex stores compatible local rollout files
- Python 3.9+
- Codex session telemetry under `~/.codex/sessions/`

You can override the Codex home directory with `CODEX_HOME`.

## Install

```bash
git clone https://github.com/bvzk/codex-context.git
cd codex-context
mkdir -p ~/.local/bin
cp codex-context ~/.local/bin/codex-context
chmod +x ~/.local/bin/codex-context
```

If `~/.local/bin` is not already on your `PATH`:

```bash
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

## Usage

Run it from the same project directory as the Codex thread you want to monitor:

```bash
codex-context
```

Live mode is the default. Stop with `Ctrl+C`.

### One snapshot

```bash
codex-context --once
```

### Verbose mode

```bash
codex-context -v
```

Example:

```text
my-project

██████████░░░░░░░░░░░░░░  42%
42.3k / 100k thread growth
57.7k remaining to safe cap

Baseline        24.1k
Current context 66.4k
Model window    258.4k

🟢 HEALTHY
```

### Change refresh interval

```bash
codex-context --interval 0.5
```

### Select another project directory

```bash
codex-context --cwd /path/to/project
```

## How it works

Codex writes local rollout events under `~/.codex/sessions/`. The monitor looks for `token_count` telemetry and reads the latest active-context token count from:

```text
payload.info.last_token_usage.total_tokens
```

It also reads `payload.info.model_context_window` for the model's reported context window in verbose mode.

The first observed token count becomes the thread's baseline. Subsequent positive changes are accumulated as thread growth.

For example:

```text
24k → 30k → 42k → 18k → 25k
```

If the drop from `42k` to `18k` is a context compaction/reset, the negative jump is ignored. Growth continues from subsequent positive deltas instead of making the meter suddenly move backwards.

This is intentionally a practical heuristic for monitoring long-running coding threads. It is not an official OpenAI context-accounting API.

## Privacy

Everything happens locally. `codex-context` does not upload prompts, source code, session data, or token telemetry anywhere.

## Current limitations

- Depends on Codex's local rollout/event format, which may change.
- Thread selection is primarily based on the current working directory and recent sessions.
- The 100k cap and health thresholds are opinionated defaults.
- Baseline/thread-growth accounting is a heuristic intended for workflow health, not billing or API usage measurement.

## Roadmap

- Configurable safe cap and thresholds
- macOS menu-bar mode
- Better thread-name detection
- Homebrew installation
- Tests against recorded sanitized rollout fixtures
- Notifications when a thread crosses a health threshold

## Disclaimer

This is an unofficial community tool and is not affiliated with or endorsed by OpenAI.

## License

MIT
