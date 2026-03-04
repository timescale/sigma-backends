---
name: sigma-backends
description: "Convert, evaluate, and deploy Sigma detection rules across SIEM backends. Covers sigma-cli (pySigma) for converting rules to Splunk SPL, Elasticsearch Lucene/ES|QL, Microsoft Sentinel KQL, QRadar AQL, CrowdStrike, and 20+ other backends. Covers rsigma for direct real-time evaluation against JSON events, linting (65 rules with --fix), validation, and running a detection daemon with correlation, hot-reload, and Prometheus metrics. Includes pipeline-to-backend mapping for ECS, Splunk CIM, Sysmon, and other field schemas. Use this skill whenever the user mentions converting Sigma rules, Sigma backends, sigma-cli, rsigma, SIEM queries, SPL, KQL, Lucene, AQL, LEQL, field mapping, ECS mapping, CIM mapping, detection engineering tooling, or asks how to run, test, validate, lint, or deploy Sigma rules -- even if they don't explicitly name a tool."
---

# Sigma Backends

Two tools for working with Sigma rules after authoring:

- **sigma-cli** (Python/pySigma): converts rules into backend-specific queries (SPL, Lucene, KQL, etc.) for import into a SIEM
- **rsigma** (Rust): evaluates rules directly against JSON log events in real time -- no SIEM required

Both support processing pipelines for field name mapping between generic Sigma fields and backend-specific schemas.

---

## sigma-cli Quick Start

### Install

```bash
pip install sigma-cli
```

### Install a Backend Plugin

```bash
sigma plugin install splunk
```

### Convert Rules

```bash
sigma convert -t splunk -p sysmon rules/windows/process_creation/
sigma convert -t elasticsearch -p ecs_windows -f kibana_ndjson rules/
sigma convert -t kusto -p sentinel_asim rules/
```

### List Available Backends, Formats, and Pipelines

```bash
sigma plugin list -t backend     # all available backend plugins
sigma list targets               # locally installed backends
sigma list formats splunk        # output formats for a backend
sigma list pipelines             # available processing pipelines
```

### Check Rules

```bash
sigma check rules/               # validate rule syntax
```

For the full sigma-cli command reference, see [references/sigma-cli.md](references/sigma-cli.md).

---

## rsigma Quick Start

### Install

```bash
cargo install rsigma
```

### Evaluate Events

```bash
# Single event (inline JSON)
rsigma eval -r rules/ -e '{"CommandLine": "cmd /c whoami"}'

# Stream NDJSON from stdin
cat events.ndjson | rsigma eval -r rules/

# With a processing pipeline
rsigma eval -r rules/ -p ecs.yml -e '{"process.command_line": "whoami"}'

# Read events from a file
rsigma eval -r rules/ -e @events.ndjson
```

### Lint Rules

```bash
rsigma lint rules/                        # 65 built-in lint rules
rsigma lint rules/ --fix                  # auto-fix 13 safe rules
rsigma lint rules/ --schema default       # + JSON schema validation
rsigma lint rules/ --disable missing_description,missing_author
```

### Validate Rules

```bash
rsigma validate rules/ -v                 # verbose validation
rsigma validate rules/ -p ecs.yml         # validate with pipeline
```

### Run Detection Daemon

```bash
# Long-running daemon with hot-reload, health checks, and Prometheus metrics
hel run | rsigma daemon -r rules/ -p ecs.yml --api-addr 0.0.0.0:9090

# With correlation state persistence
hel run | rsigma daemon -r rules/ -p ecs.yml --state-db ./state.db

# With suppression and correlation event inclusion
rsigma daemon -r rules/ --suppress 5m --correlation-event-mode full
```

For the full rsigma CLI reference, see [references/rsigma.md](references/rsigma.md).

---

## Backend Selection Guide

| SIEM / Tool | Backend ID | Pipeline | Query Language | State |
|-------------|-----------|----------|---------------|-------|
| Splunk | `splunk` | `splunk_cim_dm` / `splunk_windows` | SPL | Stable |
| Elasticsearch | `elasticsearch` | `ecs_windows` | Lucene / ES\|QL / EQL | Stable |
| OpenSearch | `opensearch` | `ecs_windows` | Lucene | Stable |
| Microsoft Sentinel | `kusto` | `sentinel_asim` | KQL | Stable |
| CrowdStrike Falcon | `crowdstrike` | (built-in) | CrowdStrike query | Stable |
| IBM QRadar | `qradar` / `ibm-qradar-aql` | (built-in) | AQL | Stable |
| Rapid7 InsightIDR | `insightidr` | (built-in) | LEQL | Stable |
| Grafana Loki | `loki` | (built-in) | LogQL | Stable |
| Carbon Black | `carbonblack` | (built-in) | CB query | Stable |
| Cortex XDR | `cortexxdr` | (built-in) | XQL | Stable |
| SentinelOne | `sentinelone` | (built-in) | Deep Visibility | Stable |
| Logpoint | `logpoint` | (built-in) | Logpoint query | Stable |
| Google SecOps | `secops` | (built-in) | UDM / YARA-L 2.0 | Development |
| rsigma (direct eval) | N/A | any pipeline YAML | JSON match output | Stable |

For the full list of 25+ backends with install commands, see [references/backends.md](references/backends.md).

### Choosing Between sigma-cli and rsigma

| Use Case | Tool |
|----------|------|
| Import rules into an existing SIEM | sigma-cli (converts to native query language) |
| Evaluate rules against JSON events in real time | rsigma eval |
| Run a detection daemon alongside a log collector | rsigma daemon |
| Lint and validate rule syntax | rsigma lint (65 rules, auto-fix) |
| CI/CD rule validation | rsigma lint + rsigma validate |
| Batch convert rules for multiple SIEMs | sigma-cli with different `-t` targets |

---

## End-to-End Workflows

### Convert a Rule to Splunk SPL

```bash
# Install the Splunk backend
sigma plugin install splunk

# Convert with Sysmon pipeline
sigma convert -t splunk -p sysmon rules/windows/process_creation/shadow_copy_deletion.yml

# Convert as saved search config
sigma convert -t splunk -p sysmon -f savedsearches -o saved.conf rules/

# With backend options
sigma convert -t splunk -p sysmon -O index=main rules/
```

### Convert a Rule to Elasticsearch

```bash
sigma plugin install elasticsearch

# Lucene query (default)
sigma convert -t elasticsearch -p ecs_windows rules/

# ES|QL format
sigma convert -t elasticsearch -p ecs_windows -f esql rules/

# Kibana NDJSON (importable)
sigma convert -t elasticsearch -p ecs_windows -f kibana_ndjson -o export.ndjson rules/
```

### Convert a Rule to Microsoft Sentinel KQL

```bash
sigma plugin install kusto

# ASIM pipeline
sigma convert -t kusto -p sentinel_asim rules/
```

### Evaluate a Rule Against Live Events (rsigma)

```bash
# Single event test
rsigma eval -r rules/ -p ecs.yml -e '{"process.command_line": "vssadmin delete shadows /all"}'

# Stream from file with full event in output
rsigma eval -r rules/ -p ecs.yml --include-event -e @events.ndjson

# With jq extraction from wrapped events
rsigma eval -r rules/ --jq '.event' -e '{"ts":"...","event":{"CommandLine":"whoami"}}'
```

### Lint and Fix a Rule Directory (rsigma)

```bash
# Lint all rules
rsigma lint rules/

# Auto-fix safe issues (lowercase keys, remove duplicates, etc.)
rsigma lint rules/ --fix

# Lint with JSON schema validation
rsigma lint rules/ --schema default

# Lint with custom config
rsigma lint rules/ --config .rsigma-lint.yml
```

### Run a Detection Daemon with Correlation (rsigma)

```bash
# Basic daemon -- reads NDJSON from stdin, outputs matches to stdout
hel run | rsigma daemon -r rules/ -p ecs.yml

# With correlation state persistence (survives restarts)
hel run | rsigma daemon \
  -r rules/ \
  -p ecs.yml \
  --state-db /var/lib/rsigma/state.db \
  --suppress 5m \
  --action reset \
  --api-addr 0.0.0.0:9090

# Health and metrics
curl http://localhost:9090/healthz           # {"status": "ok"}
curl http://localhost:9090/metrics           # Prometheus format
curl http://localhost:9090/api/v1/status     # full daemon status
curl -X POST http://localhost:9090/api/v1/reload  # hot-reload rules
```

---

## Pipeline Selection

Pipelines transform Sigma rule fields to match your backend's data model. Stack multiple pipelines with repeated `-p` flags.

### Common Patterns

| Data Model | Pipeline | Use With |
|-----------|----------|----------|
| Elastic Common Schema (ECS) | `ecs_windows` | `elasticsearch`, `opensearch`, rsigma |
| Splunk Common Information Model | `splunk_cim_dm` | `splunk` |
| Splunk Windows TA | `splunk_windows` | `splunk` |
| Sysmon field names | `sysmon` | any backend |
| Microsoft Sentinel ASIM | `sentinel_asim` | `kusto` |

### Stacking Pipelines

Pipelines run in priority order (lower priority number = runs first):

```bash
# Log source pipeline (priority 10) + backend pipeline (priority 50)
sigma convert -t splunk -p sysmon -p splunk_cim_dm rules/

# rsigma: same stacking with -p
rsigma eval -r rules/ -p sysmon.yml -p ecs.yml -e '...'
```

### Custom Pipelines

Write your own pipeline YAML for organization-specific field mappings:

```yaml
name: My Organization ECS
priority: 20
transformations:
  - type: field_name_mapping
    mapping:
      CommandLine: process.command_line
      Image: process.executable
      User: user.name
    rule_conditions:
      - type: logsource
        product: windows
```

For detailed pipeline-to-SIEM mapping and field mapping tables, see [references/pipeline-mapping.md](references/pipeline-mapping.md).

---

## Additional References

- [sigma-cli command reference](references/sigma-cli.md) -- all commands, flags, and output formats
- [rsigma CLI reference](references/rsigma.md) -- eval, lint, validate, daemon, parse
- [All backends](references/backends.md) -- 25+ pySigma backends with install commands
- [Pipeline-to-backend mapping](references/pipeline-mapping.md) -- field mapping tables for ECS, CIM, Sysmon
