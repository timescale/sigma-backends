# Pipeline-to-Backend Mapping

Reference for which processing pipelines to use with which SIEM backends, and the field mappings they provide.

## Pipeline Selection by SIEM

| SIEM | Backend | Recommended Pipeline(s) | Data Model |
|------|---------|------------------------|------------|
| Splunk (CIM) | `splunk` | `splunk_cim_dm` | Splunk Common Information Model |
| Splunk (Windows TA) | `splunk` | `splunk_windows` | Windows TA field names |
| Elasticsearch | `elasticsearch` | `ecs_windows` | Elastic Common Schema (ECS) |
| OpenSearch | `opensearch` | `ecs_windows` | Elastic Common Schema (ECS) |
| Microsoft Sentinel | `kusto` | `sentinel_asim` | Advanced Security Information Model |
| Grafana Loki | `loki` | (built-in) | Promtail / Grafana labels |
| CrowdStrike | `crowdstrike` | (built-in) | Falcon event model |
| QRadar | `qradar` | (built-in) | QRadar field model |
| rsigma (direct eval) | N/A | any custom YAML | User-defined |

### Log Source Pipelines

These map Sigma's generic logsource to specific data sources:

| Pipeline | Purpose | Priority |
|----------|---------|----------|
| `sysmon` | Map Sysmon event fields | 10 |
| `windows_audit` | Map Windows Security/Audit events | 10 |
| `windows_logsource` | Map generic Windows logsource categories | 10 |

---

## Pipeline Priority Conventions

Pipelines run in ascending priority order. Standard convention:

| Priority | Layer | Purpose | Example |
|----------|-------|---------|---------|
| 10 | Log source | Map event source field names | `sysmon` |
| 20 | Custom / Organization | Organization-specific mappings | `my-org-ecs.yml` |
| 50 | Backend (built-in) | Backend auto-applies these | (automatic) |
| 60 | Output format | Format-specific transforms | (automatic) |

Stack multiple layers:

```bash
# Log source (10) + custom (20) → backend auto-applies its own (50)
sigma convert -t splunk -p sysmon -p splunk_cim_dm rules/

rsigma eval -r rules/ -p sysmon.yml -p ecs.yml -e '...'
```

---

## ECS Field Mapping (Elastic Common Schema)

Common field mappings for Windows process creation events:

| Sigma Field | ECS Field |
|-------------|-----------|
| `CommandLine` | `process.command_line` |
| `Image` | `process.executable` |
| `OriginalFileName` | `process.pe.original_file_name` |
| `ParentImage` | `process.parent.executable` |
| `ParentCommandLine` | `process.parent.command_line` |
| `User` | `user.name` |
| `Hashes` | `process.hash.*` |
| `IntegrityLevel` | `winlog.event_data.IntegrityLevel` |
| `LogonId` | `winlog.logon.id` |
| `CurrentDirectory` | `process.working_directory` |
| `ProcessId` | `process.pid` |
| `ParentProcessId` | `process.parent.pid` |

Network connection events:

| Sigma Field | ECS Field |
|-------------|-----------|
| `SourceIP` | `source.ip` |
| `DestinationIP` | `destination.ip` |
| `SourcePort` | `source.port` |
| `DestinationPort` | `destination.port` |
| `Protocol` | `network.transport` |
| `DestinationHostname` | `destination.domain` |

Authentication events:

| Sigma Field | ECS Field |
|-------------|-----------|
| `TargetUserName` | `user.name` |
| `TargetDomainName` | `user.domain` |
| `SourceAddress` | `source.ip` |
| `LogonType` | `winlog.event_data.LogonType` |
| `WorkstationName` | `source.domain` |

---

## Splunk CIM Field Mapping

Common field mappings for Splunk Common Information Model:

| Sigma Field | Splunk CIM Field |
|-------------|-----------------|
| `CommandLine` | `process` or `Processes.process` |
| `Image` | `process_name` or `Processes.process_name` |
| `ParentImage` | `parent_process_name` or `Processes.parent_process_name` |
| `User` | `user` or `Processes.user` |
| `SourceIP` | `src_ip` or `src` |
| `DestinationIP` | `dest_ip` or `dest` |
| `DestinationPort` | `dest_port` |
| `Protocol` | `transport` |

---

## Sysmon Field Names

Sysmon events use specific field names that differ from generic Sigma:

| Event ID | Category | Key Fields |
|----------|----------|------------|
| 1 | Process Creation | `Image`, `CommandLine`, `ParentImage`, `User`, `Hashes` |
| 3 | Network Connection | `SourceIp`, `DestinationIp`, `DestinationPort`, `Protocol` |
| 7 | Image Loaded | `ImageLoaded`, `Hashes`, `Signed`, `SignatureStatus` |
| 8 | CreateRemoteThread | `SourceImage`, `TargetImage`, `StartAddress` |
| 10 | ProcessAccess | `SourceImage`, `TargetImage`, `GrantedAccess` |
| 11 | FileCreate | `TargetFilename`, `Image` |
| 12/13/14 | Registry | `TargetObject`, `Details`, `EventType` |
| 22 | DNS Query | `QueryName`, `QueryResults`, `Image` |

The `sysmon` pipeline maps these Sysmon-specific field names to generic Sigma field names.

---

## Writing Custom Pipelines

For organization-specific field mappings:

```yaml
name: My Org Windows ECS
priority: 20
transformations:
  - type: field_name_mapping
    mapping:
      CommandLine: process.command_line
      Image: process.executable
      ParentImage: process.parent.executable
      User: user.name
      SourceIP: source.ip
      DestinationIP: destination.ip
    rule_conditions:
      - type: logsource
        product: windows

  - type: change_logsource
    product: my_product
    rule_conditions:
      - type: logsource
        product: windows
```

### Conditional Transforms by Category

```yaml
name: Category-Specific Mapping
priority: 20
transformations:
  - type: field_name_mapping
    mapping:
      Image: process.executable
      CommandLine: process.command_line
    rule_conditions:
      - type: logsource
        category: process_creation

  - type: field_name_mapping
    mapping:
      TargetFilename: file.path
      Image: process.executable
    rule_conditions:
      - type: logsource
        category: file_event
```

For the full list of 26 transformation types and all condition types, see the [sigma-rules skill's pipeline reference](../../sigma-rules/references/pipelines.md).
