# AGENTS.md - Solar Optimizer Blueprint

## GitHub Token

This repo uses a dedicated GitHub token stored locally in `.gh-token` (gitignored).

Before any `gh` command (release, PR, issue...), source the token:

```bash
source .gh-token
```

The `.gh-token` file contains:
```
export GH_TOKEN=ghp_xxxxx...
```

This file is local only, never versioned. The global token from `secrets.env` is not used for this repo.

## Repo

- **GitHub**: https://github.com/ozirissp/ha-blueprint-solar-optimizer
- **Main branch**: `main`

## Releases

To create a GitHub release manually:

```bash
source .gh-token
git tag v0.x.0
git push origin v0.x.0
gh release create v0.x.0 blueprint.yaml \
  --title "v0.x.0" \
  --notes "Description of changes"
```

> Never commit `ghtoken` or `.gh-token`.

## Development Workflow

### New features → dedicated branch required

```bash
git checkout -b feat/<feature-name>
# ... development ...
git merge feat/<feature-name> --no-ff   # merge to main once validated
```

Never develop a feature directly on `main`.

### Documentation → always update the README

On every functional change:
- Update `README.md`
- Include the doc update in the same commit or the next one, before the release

### Do not commit without explicit request

---

## Project Overview

Home Assistant Blueprint that automatically regulates a solar inverter's output threshold
to maximize self-consumption. Keeps grid import close to a configurable target, ramps up
injection cautiously and cuts it aggressively to prevent any accidental export.

- **Main file**: `blueprint.yaml`
- **HACS**: compatible (single file at root)
- **Do not modify** `blueprint.yaml` without validating YAML syntax first

## Technology Stack

- **Language**: YAML (Home Assistant Blueprint syntax)
- **Domain**: Home Automation / Energy Management
- **Target**: Home Assistant Core / YAML-based automations

---

## Build & Validation Commands

No traditional build — pure YAML project.

```bash
# Validate YAML syntax
python3 -c "import yaml; yaml.safe_load(open('blueprint.yaml'))"

# YAML linter (if yamllint is installed)
yamllint blueprint.yaml
```

### Testing in Home Assistant

1. Copy the blueprint into Home Assistant:
   ```
   ~/.homeassistant/blueprints/automation/ozirissp/blueprint.yaml
   ```
2. Reload blueprints: Developer Tools → YAML → Reload Blueprints
3. Create an automation from the blueprint and test manually.

---

## Project Structure

```
ha-blueprint-solar-optimizer/
├── blueprint.yaml           # Main blueprint file
├── README.md               # Project documentation
├── LICENSE                 # MIT License
├── .gh-token               # Local GitHub token (never versioned)
├── .gitignore              # Ignores .gh-token and ghtoken
└── AGENTS.md               # This file
```

---

## Code Style Guidelines

### YAML Structure

```yaml
blueprint:
  name: Solar Optimizer
  description: >
    Automatically regulates solar inverter output.
    Keep grid import close to configurable target.
  domain: automation
  input:
    inverter_entity:
      name: Inverter Entity
      selector:
        entity:
          domain: number
    target_import:
      name: Target Grid Import (W)
      default: 50
      selector:
        number:
          min: 0
          max: 500
          step: 10
```

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Blueprint name | Title Case | `Solar Injection Regulator` |
| Input variables | snake_case | `inverter_entity`, `target_import` |
| Comments | English only | `# Skip if production is zero` |
| Log messages | English only | `[Solar Regulator] Grid=...` |

### Home Assistant Best Practices

1. **Entity Selectors**: always use typed entity selectors, never free text.
2. **Input Validation**: use `min`/`max` on number selectors, `mode: restart` if needed.
3. **Idempotency**: triggers and conditions must handle rapid state changes without side effects.
4. **`!input` in templates**: never use `!input` inside a Jinja2 `value_template`. Capture the
   input into a YAML variable first, then reference the variable inside the template.
5. **Dynamic triggers**: `time_pattern` does not support dynamic values from inputs. Use a
   `/1` trigger and filter with a condition on `now().minute % interval == 0`.
6. **`action:` not `service:`**: use `action:` keyword (required since HA 2024.8).
7. **Fallback defaults in filters**: always provide a fallback — `| int(100)` not `| int`.
8. **Templates**: test Jinja2 templates in Developer Tools → Templates before use.

### Formatting Rules

- Indentation: **2 spaces**
- Quotes on strings with special characters
- Block scalars (`|` or `>`) for long descriptions
- All text (names, descriptions, comments, log messages) in **English**

---

## Security

1. **No secrets in the blueprint**: never hardcode API keys or credentials
2. **Entity validation**: always validate that entity domains match expected types
3. **Jinja2 sandboxing**: templates run in Home Assistant's sandboxed environment

---

## References

- [Home Assistant Blueprint Documentation](https://www.home-assistant.io/docs/automation/using_blueprints/)
- [Home Assistant Automation Documentation](https://www.home-assistant.io/docs/automation/)
- [YAML Syntax](https://www.home-assistant.io/docs/configuration/yaml/)
