# Use Figma MCP Console

## Step 1 — Scan and analyze all components

## Step 2 — For each component, produce this Markdown block:

---
### [Component Name]

> [1-sentence purpose inferred from the name, variants, and property names]

**Import key**: `set_key`

**Variants:**
| Name | Key |
|---|---|
| Default | `key` |
| Loading | `key` |

**Boolean Properties:**
| Property | Default |
|---|---|
| `Show Icon?#1234:0` | false |

**Swap Properties:**
| Slot | Notes |
|---|---|
| `↪ Icon#1234:1` | Any icon component |

**Usage:** When to use this component in a page layout. 
Note if it must be nested inside Section template or can be placed at page level.