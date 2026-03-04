# rsigma CLI Reference

Full reference for [rsigma](https://github.com/timescale/rsigma), a Rust CLI for parsing, validating, linting, evaluating, and running Sigma detection rules.

## Installation

```bash
cargo install rsigma
```

---

## rsigma eval

Evaluate JSON events against Sigma detection and correlation rules.

```bash
rsigma eval -r <rules> [-p <pipeline>]... [-e <event>] [options]
```

| Flag | Default | Description |
|------|---------|-------------|
| `-r`, `--rules` | required | Path to rule file or directory |
| `-e`, `--event` | stdin | Inline JSON, `@path` for NDJSON file, or omit for stdin |
| `-p`, `--pipeline` | `[]` | Pipeline YAML file (repeatable, priority-ordered) |
| `--jq` | none | jq filter for event extraction (conflicts with `--jsonpath`) |
| `--jsonpath` | none | JSONPath (RFC 9535) query (conflicts with `--jq`) |
| `--include-event` | `false` | Include full event JSON in match output |
| `--pretty` | `false` | Pretty-print JSON output |
| `--suppress` | none | Suppression window for correlation alerts (e.g. `5m`) |
| `--action` | none | `alert` or `reset` after correlation fires |
| `--no-detections` | `false` | Suppress detection output (only correlation alerts) |
| `--correlation-event-mode` | `none` | `none`, `full`, or `refs` |
| `--max-correlation-events` | `10` | Max events stored per correlation window |
| `--timestamp-field` | `[]` | Event field(s) for timestamp extraction (repeatable) |

### Event Input Modes

| Mode | Format | Behavior |
|------|--------|----------|
| `-e '{"key":"val"}'` | Inline JSON | Single event |
| `-e @path.ndjson` | NDJSON file | Streams line-by-line |
| (no `-e`) | stdin | NDJSON from stdin, exits at EOF |

### Examples

```bash
# Single event
rsigma eval -r rules/ -e '{"CommandLine": "whoami"}'

# NDJSON file
rsigma eval -r rules/ -e @events.ndjson

# Stream from stdin with pipeline
cat events.ndjson | rsigma eval -r rules/ -p ecs.yml

# Extract nested event with jq
rsigma eval -r rules/ --jq '.event' -e '{"wrapper":true,"event":{"CommandLine":"whoami"}}'

# Array unwrap
rsigma eval -r rules/ --jq '.records[]' -e '{"records":[{"EventID":1},{"EventID":2}]}'

# Full event in output
rsigma eval -r rules/ --include-event -e @events.ndjson

# Correlation with suppression
rsigma eval -r rules/ --suppress 5m --action reset < events.ndjson

# Only correlation alerts (no per-event detections)
rsigma eval -r rules/ --no-detections --correlation-event-mode full < events.ndjson
```

### Detection Match Output

```json
{
  "rule_title": "Detect Whoami",
  "rule_id": "abc-123-...",
  "level": "medium",
  "tags": ["attack.execution"],
  "matched_selections": ["selection"],
  "matched_fields": [
    { "field": "CommandLine", "value": "cmd /c whoami" }
  ],
  "event": null
}
```

### Correlation Match Output

```json
{
  "rule_title": "Brute Force",
  "rule_id": null,
  "level": "high",
  "tags": [],
  "correlation_type": "event_count",
  "group_key": [["User", "admin"]],
  "aggregated_value": 3.0,
  "timespan_secs": 300,
  "events": null,
  "event_refs": null
}
```

---

## rsigma lint

Run 65 built-in lint rules with optional JSON schema validation.

```bash
rsigma lint <path> [options]
```

| Flag | Default | Description |
|------|---------|-------------|
| `<path>` | required | Rule file or directory |
| `--schema`, `-s` | none | `"default"` for official schema (cached 7 days) or path to local schema |
| `--verbose`, `-v` | `false` | Show all files including passing |
| `--color` | `auto` | `auto`, `always`, or `never` |
| `--disable` | `""` | Comma-separated rule IDs to suppress |
| `--config` | none | Explicit path to `.rsigma-lint.yml` |
| `--fix` | `false` | Auto-fix 13 safe rules |

### Lint Categories

| Category | Rules | Examples |
|----------|-------|---------|
| Infrastructure | 4 | yaml_parse_error, not_a_mapping |
| Shared metadata | 16 | missing_title, invalid_id, invalid_status, invalid_date |
| Detection rules | 17 | missing_logsource, missing_detection, condition_references_unknown |
| Correlation rules | 13 | missing_correlation_type, invalid_timespan_format |
| Filter rules | 8 | missing_filter, filter_has_level |
| Detection logic | 7 | incompatible_modifiers, wildcard_only_value |

### Auto-Fixable Rules (13)

Invalid status, invalid level, non-lowercase keys, duplicate tags, duplicate references, duplicate fields, single value `|all`, `|all` with `|re`, wildcard-only value, logsource value not lowercase, filter has level, filter has status, unknown key (typo correction).

### Suppression

Three-tier system:
- **CLI**: `--disable rule1,rule2`
- **Config file**: `.rsigma-lint.yml` with `disabled_rules` and `severity_overrides`
- **Inline comments**: `# rsigma-disable`, `# rsigma-disable-next-line`

```yaml
# .rsigma-lint.yml
disabled_rules:
  - missing_description
  - missing_author
severity_overrides:
  title_too_long: info
```

### Examples

```bash
rsigma lint rules/                                    # lint all
rsigma lint rules/ -v                                 # verbose
rsigma lint rules/ --fix                              # auto-fix safe issues
rsigma lint rules/ --schema default                   # + JSON schema
rsigma lint rule.yml --schema my-schema.json          # local schema
rsigma lint rules/ --disable missing_description      # suppress rules
rsigma lint rules/ --config my-lint.yml               # explicit config
```

---

## rsigma validate

Parse and compile all rules in a directory, reporting errors.

```bash
rsigma validate <path> [-v] [-p <pipeline>]
```

| Flag | Default | Description |
|------|---------|-------------|
| `<path>` | required | Directory of Sigma YAML files |
| `-v`, `--verbose` | `false` | Show per-file details |
| `-p`, `--pipeline` | `[]` | Pipeline YAML file(s) to apply before compilation |

```bash
rsigma validate rules/ -v
rsigma validate rules/ -p ecs.yml
```

---

## rsigma daemon

Run as a long-running detection service with hot-reload and HTTP APIs.

```bash
rsigma daemon -r <rules> [-p <pipeline>]... [options]
```

| Flag | Default | Description |
|------|---------|-------------|
| `-r`, `--rules` | required | Path to rule file or directory |
| `-p`, `--pipeline` | `[]` | Pipeline YAML file(s) |
| `--jq` | none | jq filter for event extraction |
| `--jsonpath` | none | JSONPath (RFC 9535) query |
| `--include-event` | `false` | Include full event in matches |
| `--pretty` | `false` | Pretty-print output |
| `--api-addr` | `0.0.0.0:9090` | HTTP API bind address |
| `--suppress` | none | Correlation alert suppression window |
| `--action` | none | `alert` or `reset` after correlation fires |
| `--no-detections` | `false` | Only show correlation alerts |
| `--correlation-event-mode` | `none` | `none`, `full`, or `refs` |
| `--max-correlation-events` | `10` | Max events per correlation window |
| `--timestamp-field` | `[]` | Event field(s) for timestamps |
| `--state-db` | none | SQLite path for correlation state persistence |
| `--state-save-interval` | `30` | Seconds between state snapshots |

### HTTP Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/healthz` | GET | `{"status": "ok"}` |
| `/readyz` | GET | 200 when rules loaded, 503 otherwise |
| `/metrics` | GET | Prometheus metrics |
| `/api/v1/status` | GET | Full daemon status |
| `/api/v1/rules` | GET | Rule counts and path |
| `/api/v1/reload` | POST | Trigger rule reload |

### Prometheus Metrics

| Metric | Type | Description |
|--------|------|-------------|
| `rsigma_events_processed_total` | counter | Total events processed |
| `rsigma_detection_matches_total` | counter | Detection matches |
| `rsigma_correlation_matches_total` | counter | Correlation matches |
| `rsigma_events_parse_errors_total` | counter | JSON parse errors |
| `rsigma_detection_rules_loaded` | gauge | Detection rules loaded |
| `rsigma_correlation_rules_loaded` | gauge | Correlation rules loaded |
| `rsigma_correlation_state_entries` | gauge | Active correlation entries |
| `rsigma_reloads_total` | counter | Reload attempts |
| `rsigma_reloads_failed_total` | counter | Failed reloads |
| `rsigma_event_processing_seconds` | histogram | Per-event latency |
| `rsigma_uptime_seconds` | gauge | Daemon uptime |

### Hot-Reload Triggers

- File system changes to `.yml`/`.yaml` files (debounced 500ms)
- `SIGHUP` signal (Unix)
- `POST /api/v1/reload`

### State Persistence

With `--state-db`, correlation state (window entries, suppression timestamps, event buffers) is persisted to SQLite. State survives restarts -- a correlation that saw 2 of 3 required events before restart resumes from 2. Uses WAL journal mode; entries are keyed by stable rule identifiers.

---

## rsigma parse

Parse a Sigma YAML file and output the AST as JSON.

```bash
rsigma parse rule.yml
```

## rsigma condition

Parse a condition expression and output the AST as JSON.

```bash
rsigma condition 'selection and not filter'
```

## rsigma stdin

Read a Sigma YAML document from stdin and output the AST as JSON.

```bash
cat rule.yml | rsigma stdin
```

---

## Environment Variables

| Variable | Scope | Effect |
|----------|-------|--------|
| `NO_COLOR` | lint | Disables color output |
| `RUST_LOG` | daemon | Log level filter (default: `info`) |

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Error (parse failure, lint errors, missing arguments) |
