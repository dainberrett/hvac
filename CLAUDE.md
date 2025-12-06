# Claude Code Instructions

This is a Home Assistant HVAC automation project for a Mitsubishi mini-split system.

## Key Files

- `REQUIREMENTS.md` - The PRD/requirements document. **Read this first** to understand the system goals and constraints.
- `HELPERS.md` - Required Home Assistant helper entities (production + test).
- `automations/` - Home Assistant automation YAML files.
- `templates/rooms.yaml` - Template sensors for room state tracking (dictionary format for `!include`).
- `tests/test_templates.yaml` - Test-mode-aware sensors (dictionary format for `!include`).
- `tests/README.md` - Test framework documentation.

## Goals

1. Keep daily condenser mode switches under 15
2. Daytime: All rooms within ±2°F of setpoint
3. Nighttime: Bedroom within ±2°F, other rooms within ±5°F

## Template File Format

Template files use **dictionary format** (not list format) for compatibility with `- !include`:

```yaml
# Correct format for templates/rooms.yaml
binary_sensor:
  - name: hvac_night_mode
    state: ...

sensor:
  - name: bedroom_temperature_deviation
    state: ...
```

## Home Assistant File Structure

```
/config/
  automations/hvac/     # Automation YAML files
  templates/hvac/       # Template YAML files (rooms.yaml, test_templates.yaml)
```

## Before Making Changes

1. Read `REQUIREMENTS.md` to ensure changes align with requirements
2. Run the test suite after changes to verify logic
3. Check `HELPERS.md` for required input entities
