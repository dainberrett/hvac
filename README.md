# HVAC Automation (Home Assistant)

This repository contains the full automation logic for controlling the Mitsubishi mini-split system at my home using Home Assistant. The goal is to maintain each room within ±2°F of its setpoint while minimizing condenser mode switching and keeping head units in fan-only mode whenever possible.

## Repository Structure

automations/
  priority.yaml       # Selects the priority room based on ranking and deviation
  condenser.yaml      # Controls heat/cool mode on the condenser
  head_units.yaml     # Controls each head unit based on condenser mode
  guest_override.yaml # Handles first-floor guest thermostat overrides

templates/
  rooms.yaml          # Template sensors for deviations, state tracking, and logic

## Philosophy

- Use dedicated Zigbee temperature sensors (Sonoff) as authoritative readings.
- Minimize condenser mode switching (2-minute delay between changes).
- Only the priority room is allowed to change condenser mode.
- All non-targeted rooms should run in fan-only mode.
- Guest thermostat interaction overrides normal priority logic.

## How to Use

1. Copy automations/*.yaml into /config/automations/ in Home Assistant.
2. Copy templates/rooms.yaml into /config/templates/.
3. Reload templates and automations in Home Assistant.
4. Ensure all required helpers (input_booleans, input_numbers, input_text) exist.
5. Test each automation from Developer Tools > Automations.

## Future Plans

- Add dashboards for monitoring room temperatures.
- Add historical logs of condenser mode switch count.
- Add watchdog logic for sensor failure and fallback.

