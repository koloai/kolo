---
name: kolo
description: Kolo is a text-based Python debugger that captures every executed function, return value, local variable, HTTP request, and SQL query into greppable trace files. Use this skill for tricky debugging challenges, to verify that code behaves as expected at runtime, and to see the real data and values passing through the code. Use it to see the real data returned by sql queries or the real http data sent and received. Use it when the user reports something is not working, but you're not sure why - even if the user doesn't explicitly mention "kolo" or "debugging".
---

# Kolo - Text Based Python Debugger for AI agents

## The core workflow

1. Execute code with Kolo activated (`KOLO=1`)
2. Grep the trace data kolo emits in `.kolo/traces` — search for a function name, URL fragment, SQL table, or error string, then read the matching files
   - If you can't find the desired trace, view all recent traces with `kolo trace list`, then emit the desired trace manually with `kolo trace emit <trace_id>`
   - If you are not sure what to grep for or want to see an overview of everything that executed during a trace look at `.kolo/kolo.txt` or run `kolo cat <trace_id>`
3. Make desired code changes, then re-run (back to step 1)

## Installing Kolo

- `pip install kolo` - install kolo from PyPI as per the project's conventions and only if it's not already installed

## Enabling Kolo

- Set the `KOLO=1` environment variable.
  - Any python process that starts up will be traced with Kolo (this is powered by the .pth mechanism in Python)
  - This is the recommended and simplest way to activate Kolo
- Other ways to activate
  - `kolo run` - activate kolo for a specific python program or script: e.g. `kolo run python my_script.py`
  - `kolo.enable` - activate kolo for a specific code block. use as context manager: `with kolo.enable()` or as decorator: `@kolo.enable()`

## Using Kolo

Use the `kolo` cli to view trace data.

- Run bare `kolo` to see useful commands, recent traces and the location of `kolo.txt`.
- Use `kolo trace list` to list traces (most recent first). Specify number of traces to list with `--count <count>` (optional)

### Key insight: grep the traces directly

By default kolo will emit the 5 most recently captured traces in `.kolo/traces/`. Each trace is a directory containing the full trace data as plain text files with human-readable section headers — read one directly to see everything captured for a given call, no special parser needed.

Simply grepping for function names or HTTP request fragments will find the relevant data. The files contain:

- Full call stacks (how functions were invoked)
- Complete HTTP request/response (headers, body, status, timing)
- Function arguments and return values
- Execution timing for each call

Manually emit a trace using `kolo trace emit <trace_id>`. If you'd like to emit it to an output directory other than `.kolo/traces` specify it with `--output-dir`

### kolo.txt

`kolo.txt` is the "homepage" of kolo output. Read it with `head` - it contains useful commands, a list of recent traces, and the full compact (`kolo cat`) representation of the latest trace. Kolo automatically updates this file as new traces are captured.

### Other useful commands

- `kolo cat` — **prefer `Grep` first, not `cat`.** `cat` dumps an entire trace to stdout and can easily exceed your context window on real applications. Only reach for `cat` after you've narrowed to a specific small trace.
  - bare `kolo cat` outputs the most recent trace in compact format
  - `kolo cat TRACE_ID` outputs a specific trace
  - `kolo cat TRACE_ID --returns` includes input/return values
- `kolo trace node <trace_id> <node_index>` — fetch full details of a specific node (args, locals, return value) without grepping. Faster than `cat` when you already know the node index from the trace summary.

## Config

The kolo config file lives at `.kolo/config.toml`

### Filters

By default Kolo traces all code the project directory and ignores standard library or third party code. Filters customize this behavior.

#### Ignore Frames

`ignore_frames` is a list of file path fragments:

```toml
# .kolo/config.toml
[filters]
ignore_frames = ["ignore_me.py", "/ignore_directory/"]
```

This will ignore any file called `ignore_me.py` or any file with `/ignore_directory/` in its path.

#### Include Frames

```toml
# .kolo/config.toml
[filters]
include_frames = ["json", "django/db"]
```

This will include any frames from the `json` module of the standard library and from Django's database code in `django.db`.

> **Note:** Including standard library and third party code can generate a lot of extra data, which may slow down execution. If you notice slow execution, adjust ignores and includes to capture fewer frames.

More config options: https://docs.kolo.app/reference/config

## Further documentation

https://docs.kolo.app

Skill version: v0.1
