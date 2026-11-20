# log

Levelled logging to standard error for Neutron.

## Install

```console
$ ntsc pkg install log
```

```toml
[dependencies]
log = "^1.0.0"
```

## Use

```ntsc
use log

fun main() {
    log.set_level("debug")
    log.debug("connecting")
    log.info("ready")
    log.warn_from("db", "slow query")
    log.error("giving up")
}
```

Output:

```text
2026-11-20T11:54:49Z DEBUG prog: connecting
2026-11-20T11:54:49Z INFO  prog: ready
2026-11-20T11:54:49Z WARN  db: slow query
2026-11-20T11:54:49Z ERROR prog: giving up
```

The threshold is the `NTSC_LOG_LEVEL` environment variable, so it can also be
set from outside the program:

```console
$ NTSC_LOG_LEVEL=warn ./prog
```

## API

### Configuration

| Function | Returns | Behavior |
| --- | --- | --- |
| `log.set_level(level)` | `void` | Set the process-wide threshold. Throws on an unknown level. |
| `log.level()` | `string` | Current threshold; `info` when unset or unrecognized. |
| `log.enabled(level)` | `bool` | Whether a message at `level` would be written now. |
| `log.target()` | `string` | The default target: the program's file name, or `app`. |

Levels, in order: `debug`, `info`, `warn`, `error`, and `off` to disable
everything.

### Writing

| Function | Returns | Behavior |
| --- | --- | --- |
| `log.debug(msg)` / `info` / `warn` / `error` | `void` | Write one line under the default target. |
| `log.debug_from(target, msg)` / `info_from` / `warn_from` / `error_from` | `void` | Write one line under an explicit target. |
| `log.write(level, target, msg)` | `void` | Write at an explicit level. Throws on an unknown level. |
| `log.format(level, target, msg)` | `string` | Format a line without writing it. |

## Notes

- Everything goes to standard error; standard output stays clean for piped
  data.
- Timestamps are UTC with second precision (`%Y-%m-%dT%H:%M:%SZ`).
- The level is process-wide and is inherited by child processes, because it is
  stored in the environment. Set `NTSC_LOG_LEVEL=off` to silence a child.
- `set_level` validates its argument, so a typo fails immediately instead of
  silently logging nothing.
- Output is plain text. Colour it with the `term` package if needed.

## Files

| File | Contents |
| --- | --- |
| `src/lib.nt` | Public surface; configuration and the convenience helpers. |
| `src/level.nt` | Level validation, ranking and labels. |
| `src/clock.nt` | UTC timestamp formatting. |
| `src/format.nt` | Line layout: timestamp, label, target, message. |
| `src/sink.nt` | Writes one line to standard error. |
