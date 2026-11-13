# HVAC Automation (Home Assistant)

This repository contains the full automation logic for controlling the Mitsubishi mini-split system using Home Assistant. The goal is to maintain each room within ±2°F of its setpoint while minimizing condenser mode switching.

## Repository Structure

```
automations/
  priority.yaml         # Selects the priority room based on ranking and deviation
  condenser.yaml        # Controls heat/cool mode on the condenser
  head_units.yaml       # Controls each head unit based on condenser mode
  guest_override.yaml   # Handles first-floor guest thermostat overrides
  system_recovery.yaml  # Handles power recovery and system restarts
  test_runner.yaml      # Automated test suite runner

templates/
  rooms.yaml            # Template sensors for deviations, night mode, target direction

tests/
  test_templates.yaml   # Test-mode-aware sensors for automated testing
  README.md             # Test framework documentation
```

## Philosophy

- Use dedicated Zigbee temperature sensors (Sonoff) as authoritative readings.
- Dynamic sensor offset calibration: compensates for inaccurate Mitsubishi head unit sensors by comparing them to Sonoff readings and adjusting setpoint commands accordingly.
- Trigger at threshold, continue until setpoint: conditioning starts when deviation exceeds the threshold (±2°F), but continues until the room actually reaches setpoint. This prevents oscillation where a room never reaches its target.
- Minimize condenser mode switching (45-minute minimum runtime before mode switch).
- Only the priority room is allowed to change condenser mode.
- All non-targeted rooms use neutral temperature targets (60°F for heat, 80°F for cool).
- Guest thermostat interaction overrides normal priority logic.

## How to Use

### File Organization on Home Assistant

```
/config/
  automations/
    hvac/
      priority.yaml
      condenser.yaml
      head_units.yaml
      guest_override.yaml
      system_recovery.yaml
      test_runner.yaml
  templates/
    hvac/
      rooms.yaml
      test_templates.yaml
```

### configuration.yaml

```yaml
automation ui: !include automations.yaml
automation hvac: !include_dir_merge_list automations/hvac/

template:
  - !include templates/hvac/rooms.yaml
  - !include templates/hvac/test_templates.yaml
  # ... other templates
```

### Setup Steps

1. Create required helpers (see `HELPERS.md`)
2. Copy automation files to `/config/automations/hvac/`
3. Copy template files to `/config/templates/hvac/`
4. Update `configuration.yaml` with the includes above
5. Restart Home Assistant
6. Run the test suite from the dashboard to verify

## Testing

The system includes an automated test suite with 18 tests covering:
- Deviation calculations
- Night mode thresholds
- Target direction logic
- Fan speed calculations

Run tests from the HVAC dashboard by clicking "Run Tests". See `tests/README.md` for details.

## Future Plans

- Add historical logs of condenser mode switch count
- Add watchdog logic for sensor failure and fallback
- Add alerts for excessive mode switching

