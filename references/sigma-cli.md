# sigma-cli Command Reference

Full reference for the official [sigma-cli](https://pypi.org/project/sigma-cli/) tool (v2.x), which wraps [pySigma](https://github.com/SigmaHQ/pySigma) for command-line Sigma rule conversion.

## Installation

```bash
pip install sigma-cli
```

---

## sigma convert

Convert Sigma rules into backend-specific queries.

```bash
sigma convert -t <backend> [-p <pipeline>]... [-f <format>] [-o <output>] [-O <key=value>]... <input>
```

| Flag | Description |
|------|-------------|
| `-t`, `--target` | Backend identifier (required). E.g. `splunk`, `elasticsearch`, `kusto` |
| `-p`, `--pipeline` | Processing pipeline name or YAML file (repeatable, applied in order) |
| `-f`, `--format` | Output format (default: `default`). Use `sigma list formats <backend>` to see options |
| `-o`, `--output` | Output file path (default: stdout) |
| `-O`, `--backend-option` | Backend-specific key=value option (repeatable) |
| `<input>` | Path to a Sigma rule file or directory |

### Examples

```bash
# Convert to Splunk SPL with Sysmon pipeline
sigma convert -t splunk -p sysmon rules/windows/

# Convert to Elasticsearch Lucene with ECS pipeline
sigma convert -t elasticsearch -p ecs_windows rules/

# Convert to Kibana NDJSON format for import
sigma convert -t elasticsearch -p ecs_windows -f kibana_ndjson -o export.ndjson rules/

# Convert to Splunk saved searches config
sigma convert -t splunk -p sysmon -f savedsearches -o saved.conf rules/

# Convert to KQL for Microsoft Sentinel with ASIM
sigma convert -t kusto -p sentinel_asim rules/

# With backend options
sigma convert -t splunk -p sysmon -O index=main -O source=WinEventLog rules/

# Multiple pipelines (applied in order)
sigma convert -t splunk -p sysmon -p splunk_cim_dm rules/
```

---

## sigma list

List available backends, output formats, pipelines, and validators.

### sigma list targets

Show locally installed backends:

```bash
sigma list targets
```

### sigma list formats

Show output formats for a specific backend:

```bash
sigma list formats <backend>
```

Example output:

```
+----------------+----------------------------------------+
| Format         | Description                            |
+----------------+----------------------------------------+
| default        | Plain SPL queries                      |
| savedsearches  | Splunk savedsearches.conf format       |
| data_model     | Splunk data model queries              |
+----------------+----------------------------------------+
```

### sigma list pipelines

Show available processing pipelines:

```bash
sigma list pipelines
```

### sigma list validators

Show available rule validators:

```bash
sigma list validators
```

---

## sigma plugin

Manage backend plugins.

### sigma plugin list

Show all available plugins (backends, pipelines, validators):

```bash
sigma plugin list                # all plugins
sigma plugin list -t backend     # only backends
sigma plugin list -t pipeline    # only pipelines
```

Output columns: Identifier, Type, State (stable/testing/development), Description.

### sigma plugin install

Install a plugin:

```bash
sigma plugin install <identifier>
```

### sigma plugin uninstall

Remove a plugin:

```bash
sigma plugin uninstall <identifier>
```

---

## sigma check

Validate Sigma rule syntax:

```bash
sigma check <input>
```

Checks rule structure against the Sigma specification without converting.

---

## Output Formats by Backend

Each backend provides its own set of output formats. Common patterns:

| Backend | Format | Description |
|---------|--------|-------------|
| `splunk` | `default` | Plain SPL queries |
| `splunk` | `savedsearches` | savedsearches.conf format |
| `splunk` | `data_model` | Data model / tstats queries |
| `elasticsearch` | `default` | Lucene queries |
| `elasticsearch` | `kibana_ndjson` | Kibana importable NDJSON |
| `elasticsearch` | `esql` | ES\|QL queries |
| `elasticsearch` | `eql` | EQL queries |
| `elasticsearch` | `dsl_lucene` | Full DSL with Lucene query |
| `kusto` | `default` | KQL queries |
| `loki` | `default` | LogQL queries |
| `loki` | `ruler` | Loki ruler YAML for alerting |

Use `sigma list formats <backend>` for the definitive list after installing a plugin.

---

## Pipeline Specification

Pipelines can be specified by name (for built-in pipelines bundled with a backend plugin) or by file path (for custom YAML pipelines):

```bash
# By name (built-in)
sigma convert -t splunk -p sysmon rules/

# By file path (custom)
sigma convert -t splunk -p ./my-pipeline.yml rules/

# Multiple (stacked in order)
sigma convert -t elasticsearch -p sysmon -p ecs_windows -p ./custom.yml rules/
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| `0` | Success |
| `1` | Error (parse failure, conversion error, missing plugin) |
