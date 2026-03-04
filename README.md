# Sigma Backends Skill

An [Agent Skill](https://agentskill.sh/) for converting, evaluating, and deploying [Sigma](https://github.com/SigmaHQ/sigma) detection rules across SIEM backends.

## What This Skill Does

This skill teaches AI agents how to work with Sigma rules across backends:

- **sigma-cli** (pySigma): convert rules to Splunk SPL, Elasticsearch Lucene/ES|QL, Microsoft Sentinel KQL, QRadar AQL, and 20+ other query languages
- **rsigma**: evaluate rules directly against JSON events in real time, lint rules (65 checks with auto-fix), validate, and run a detection daemon with correlation, hot-reload, and Prometheus metrics
- **Pipeline mapping**: which pipeline to use for which SIEM, field mapping tables for ECS, Splunk CIM, and Sysmon

Complements the [sigma-rules](https://github.com/timescale/sigma-rules) skill for rule authoring.

## Install

```bash
npx skills add timescale/sigma-backends -g -y
```

Or install for a specific agent:

```bash
npx skills add timescale/sigma-backends -g -a cursor -y
npx skills add timescale/sigma-backends -g -a claude-code -y
```

## Structure

```
sigma-backends/
├── SKILL.md                          # Main skill — quick starts, backend guide, workflows
└── references/
    ├── sigma-cli.md                  # Full sigma-cli command reference
    ├── rsigma.md                     # Full rsigma CLI reference
    ├── backends.md                   # All 25+ pySigma backends
    └── pipeline-mapping.md           # SIEM-to-pipeline mapping and field tables
```

## Coverage

- **sigma-cli**: convert, list, plugin management, output formats
- **rsigma**: eval, lint (65 rules, --fix), validate, daemon (hot-reload, Prometheus, state persistence)
- **25+ backends**: Splunk, Elasticsearch, OpenSearch, Microsoft Sentinel, CrowdStrike, QRadar, InsightIDR, Loki, Carbon Black, Cortex XDR, SentinelOne, Google SecOps, and more
- **Pipeline mapping**: ECS, Splunk CIM, Sysmon field tables, pipeline stacking, priority conventions

## References

- [Sigma Specification](https://github.com/SigmaHQ/sigma-specification)
- [pySigma](https://github.com/SigmaHQ/pySigma)
- [sigma-cli](https://pypi.org/project/sigma-cli/)
- [rsigma](https://github.com/timescale/rsigma)

## License

MIT
