# pySigma Backends Reference

All available pySigma backends for converting Sigma rules to SIEM-specific queries. Install with `sigma plugin install <identifier>`.

For rsigma (direct evaluation against JSON events), no backend plugin is needed -- see [rsigma.md](rsigma.md).

---

## Major SIEMs

### Splunk -- `splunk`

**State:** Stable
**Query Language:** SPL, tstats data model queries
**Output Formats:** `default` (plain SPL), `savedsearches` (savedsearches.conf), `data_model` (tstats)
**Pipelines:** `splunk_windows`, `splunk_cim_dm`, `sysmon`

```bash
sigma plugin install splunk
sigma convert -t splunk -p splunk_cim_dm rules/
```

### Elasticsearch -- `elasticsearch`

**State:** Stable
**Query Language:** Lucene, ES|QL (with correlations), EQL
**Output Formats:** `default` (Lucene), `kibana_ndjson`, `esql`, `eql`, `dsl_lucene`
**Pipelines:** `ecs_windows`, `ecs_windows_old`, `sysmon`

```bash
sigma plugin install elasticsearch
sigma convert -t elasticsearch -p ecs_windows rules/
sigma convert -t elasticsearch -p ecs_windows -f esql rules/
```

### OpenSearch -- `opensearch`

**State:** Stable
**Query Language:** Lucene
**Output Formats:** `default` (Lucene), alerting rules
**Pipelines:** `ecs_windows`

```bash
sigma plugin install opensearch
sigma convert -t opensearch -p ecs_windows rules/
```

### Microsoft Sentinel / Azure -- `kusto`

**State:** Stable
**Query Language:** KQL (Kusto Query Language)
**Supports:** Microsoft XDR Advanced Hunting, Sentinel ASIM, Azure Monitor
**Pipelines:** `sentinel_asim`, Microsoft 365 Defender tables

```bash
sigma plugin install kusto
sigma convert -t kusto -p sentinel_asim rules/
```

### IBM QRadar -- `qradar` / `ibm-qradar-aql`

**State:** Stable
**Query Language:** AQL
**Two implementations:**
- `qradar` -- community backend with AQL and extension packages
- `ibm-qradar-aql` -- IBM-maintained backend

```bash
sigma plugin install qradar
sigma convert -t qradar rules/

# Or IBM's version
sigma plugin install ibm-qradar-aql
sigma convert -t ibm-qradar-aql rules/
```

### Rapid7 InsightIDR -- `insightidr`

**State:** Stable
**Query Language:** LEQL

```bash
sigma plugin install insightidr
sigma convert -t insightidr rules/
```

### Grafana Loki -- `loki`

**State:** Stable
**Query Language:** LogQL
**Output Formats:** `default` (LogQL), `ruler` (Loki ruler YAML for alerting)
**Pipelines:** Built-in mappings for Grafana and promtail Sysmon data

```bash
sigma plugin install loki
sigma convert -t loki rules/
sigma convert -t loki -f ruler rules/    # alerting rules
```

---

## EDR Platforms

### Carbon Black -- `carbonblack`

**State:** Stable
**Supports:** Enterprise EDR (Threat Hunter) and EDR (Response)

```bash
sigma plugin install carbonblack
sigma convert -t carbonblack rules/
```

### Cortex XDR -- `cortexxdr`

**State:** Stable
**Query Language:** XQL

```bash
sigma plugin install cortexxdr
sigma convert -t cortexxdr rules/
```

### CrowdStrike Falcon -- `crowdstrike`

**State:** Stable
**Includes:** Pipelines for CrowdStrike Falcon platform and Falcon Data Replicator (FDR) logs

```bash
sigma plugin install crowdstrike
sigma convert -t crowdstrike rules/
```

### SentinelOne -- `sentinelone`

**State:** Stable
**Query Language:** Deep Visibility queries

```bash
sigma plugin install sentinelone
sigma convert -t sentinelone rules/
```

### SentinelOne PowerQuery -- `sentinelone-pq`

**State:** Stable
**Query Language:** PowerQuery

```bash
sigma plugin install sentinelone-pq
sigma convert -t sentinelone-pq rules/
```

---

## Cloud and Other

### Google SecOps (Chronicle) -- `secops`

**State:** Development
**Query Language:** UDM searches and YARA-L 2.0 detection rules

```bash
sigma plugin install secops
sigma convert -t secops rules/
```

### Logpoint -- `logpoint`

**State:** Stable

```bash
sigma plugin install logpoint
sigma convert -t logpoint rules/
```

### Panther -- `panther`

**State:** Stable

```bash
sigma plugin install panther
sigma convert -t panther rules/
```

### Datadog Cloud SIEM -- `datadog`

**State:** Testing
**Query Language:** Datadog Query Syntax

```bash
sigma plugin install datadog
sigma convert -t datadog rules/
```

### uberAgent -- `uberagent`

**State:** Stable

```bash
sigma plugin install uberagent
sigma convert -t uberagent rules/
```

---

## Specialized / Niche

| Backend | State | Query Language | Install |
|---------|-------|---------------|---------|
| `dictquery` | Stable | DictQuery strings | `sigma plugin install dictquery` |
| `sqlite` | Testing | SQL (SQLite/Zircolite) | `sigma plugin install sqlite` |
| `stix` | Development | STIX 2.0 / STIX Shifter | `sigma plugin install stix` |
| `golangexpr` | Testing | Golang Expr | `sigma plugin install golangexpr` |
| `surrealql` | Testing | SurrealQL | `sigma plugin install surrealql` |
| `powershell` | Testing | PowerShell queries | `sigma plugin install powershell` |
| `hawk` | Testing | HAWK.io BETree queries | `sigma plugin install hawk` |
| `netwitness` | Testing | NetWitness application rules | `sigma plugin install netwitness` |
| `trellix_helix` | Development | Trellix Helix queries | `sigma plugin install trellix_helix` |
| `quickwit` | Development | Quickwit queries | `sigma plugin install quickwit` |
| `ala-socprime` | Development | Azure Log Analytics (SOC Prime) | `sigma plugin install ala-socprime` |

---

## Plugin States

| State | Meaning |
|-------|---------|
| **Stable** | Production-ready, actively maintained |
| **Testing** | Functional but may have gaps, community-maintained |
| **Development** | Experimental, expect breaking changes |

Use `sigma plugin list -t backend` for the current definitive list with states.
