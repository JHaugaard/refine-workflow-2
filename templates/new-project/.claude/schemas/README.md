# Handoff Document Schemas

JSON Schema definitions for validating workflow handoff documents.

## Purpose

These schemas define the data contracts between workflow skills. They enable:
- **Validation**: Verify handoff documents match expected structure
- **Documentation**: Self-documenting data formats
- **IDE Support**: Autocomplete and validation in JSON editors

## Schema Files

| Schema | Produced By | Consumed By |
|--------|-------------|-------------|
| `brief.schema.json` | project-brief-writer | solution-architect, tech-stack-advisor |
| `architecture-context.schema.json` | solution-architect | tech-stack-advisor |
| `tech-stack-decision.schema.json` | tech-stack-advisor | deployment-advisor, project-spinup |
| `deployment-strategy.schema.json` | deployment-advisor | project-spinup, deploy-guide, ci-cd-implement |
| `project-foundation.schema.json` | project-spinup | (downstream development) |
| `deployment-log.schema.json` | deploy-guide | ci-cd-implement |

## Critical Fields for Decision Gates

The workflow manifest uses conditions that read specific fields:

```yaml
# Example from workflow-manifest.yaml
condition:
  source: ".docs/deployment-strategy.json"
  field: "hosting.type"
  op: "equals"
  value: "localhost"
```

**Key fields used in decision gates:**
- `deployment-strategy.json` → `hosting.type` (localhost, vps-docker, cloudflare-pages, fly-io, hostinger-shared)

## Validation Usage

### With ajv (JavaScript)
```javascript
const Ajv = require('ajv');
const ajv = new Ajv();
const schema = require('./.claude/schemas/brief.schema.json');
const validate = ajv.compile(schema);

const brief = JSON.parse(fs.readFileSync('.docs/brief.json'));
const valid = validate(brief);
if (!valid) console.log(validate.errors);
```

### With jsonschema (Python)
```python
from jsonschema import validate
import json

with open('.claude/schemas/brief.schema.json') as f:
    schema = json.load(f)
with open('.docs/brief.json') as f:
    brief = json.load(f)

validate(instance=brief, schema=schema)
```

## Schema Versioning

Schemas follow semantic versioning via the `$id` field. Breaking changes increment the major version.

Current versions: All schemas are v1.0 (initial release).
