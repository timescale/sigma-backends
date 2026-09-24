# Sigma Backends Skill

This repository is archived. The skill it published taught rsigma commands that no longer exist.

Use these instead:

- [timescale/sigma-rules](https://github.com/timescale/sigma-rules) for authoring Sigma YAML (detection, correlation, filters, pipelines, modifiers).
- [timescale/rsigma](https://github.com/timescale/rsigma) for the rsigma CLI and MCP loop, including convert, eval, lint, and daemon. The skill is published from `skills/rsigma/`.

```bash
npx skills add timescale/sigma-rules -g -y
npx skills add timescale/rsigma -g -y
```

The git history of this repository stays. Install the two skills above.
