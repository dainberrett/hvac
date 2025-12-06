# HVAC Logic Test Framework

Fast, automated testing for HVAC automation logic. Runs 22 tests in under 5 seconds.

## What It Tests

| Test | Description |
|------|-------------|
| 1 | No conditioning needed - all rooms at setpoint |
| 2 | Bedroom needs heating (below setpoint) |
| 3 | Bedroom needs cooling (above setpoint) |
| 4 | Office cooling during daytime |
| 5 | Night mode - Office +4°F should NOT qualify (threshold is 5°F) |
| 6 | Night mode - Office +6°F SHOULD qualify |
| 7 | Night mode - Bedroom still uses 2°F threshold |
| 8 | Below threshold - should NOT qualify |
| 9 | Exactly at threshold - SHOULD qualify |
| 10 | Multiple rooms need same direction |
| 11 | Conflicting room needs (one heat, one cool) |
| 12 | First floor heating |
| 13-18 | Fan speed calculations based on deviation |
| 19 | First floor fan HIGH when conditioned (600 sq ft override) |
| 20 | Continue cooling below threshold (oscillation prevention) |
| 21 | Stop at setpoint (deviation = 0) |
| 22 | Continue heating below threshold (heat mode equivalent) |

## Setup

### 1. Create the test helpers

In Home Assistant, go to **Settings > Devices & Services > Helpers** and create:

| Entity ID | Type | Min | Max | Default | Unit |
|-----------|------|-----|-----|---------|------|
| `input_boolean.hvac_test_mode` | Boolean | - | - | off | - |
| `input_boolean.hvac_test_running` | Boolean | - | - | off | - |
| `input_number.test_bedroom_temperature` | Number | 40 | 100 | 65 | °F |
| `input_number.test_office_temperature` | Number | 40 | 100 | 68 | °F |
| `input_number.test_living_room_temperature` | Number | 40 | 100 | 68 | °F |
| `input_number.test_first_floor_temperature` | Number | 40 | 100 | 68 | °F |
| `input_number.test_current_index` | Number | 0 | 100 | 0 | - |
| `input_number.test_passed_count` | Number | 0 | 100 | 0 | - |
| `input_number.test_failed_count` | Number | 0 | 100 | 0 | - |
| `input_number.test_total_count` | Number | 0 | 100 | 0 | - |
| `input_datetime.test_simulated_time` | DateTime (time only) | - | - | 12:00:00 | - |
| `input_text.test_current_name` | Text | - | - | (empty) | - |
| `input_text.test_current_status` | Text | - | - | (empty) | - |
| `input_text.test_failed_list` | Text | - | - | (empty) | - |
| `input_text.test_last_run_result` | Text | - | - | (empty) | - |

### 2. Add test templates

Add to your `configuration.yaml`:

```yaml
template:
  - !include templates/hvac/rooms.yaml
  - !include templates/hvac/test_templates.yaml
```

Copy `test_templates.yaml` to `/config/templates/hvac/`. The file uses dictionary format (not list format) for compatibility with `- !include`.

### 3. Add test automation

Copy `test_runner.yaml` to `/config/automations/hvac/`.

### 4. Add dashboard card

Create a Manual card on your HVAC dashboard with test controls:

```yaml
type: vertical-stack
cards:
  - type: button
    name: Run Tests
    tap_action:
      action: call-service
      service: automation.trigger
      target:
        entity_id: automation.hvac_test_runner
  - type: entity
    entity: sensor.hvac_test_summary
    name: Test Results
```

### 5. Reload

In Home Assistant:
1. **Developer Tools > YAML > Reload Template Entities**
2. **Developer Tools > YAML > Reload Automations**
3. Refresh your dashboard

## Running Tests

### From Dashboard
Click the **Run Tests** button on the HVAC dashboard card.

### From Developer Tools
1. Go to **Developer Tools > Services**
2. Select `input_boolean.turn_on`
3. Target: `input_boolean.hvac_test_mode`
4. Click **Call Service**

### From Automation
```yaml
- service: input_boolean.turn_on
  target:
    entity_id: input_boolean.hvac_test_mode
```

## Understanding Results

- **Passed**: Test result matched expected outcome
- **Failed**: Test result did not match expected outcome

Failed tests show details like:
```
FAILED - Got: cool, Expected: heat
```

Results appear in:
1. Dashboard card (real-time)
2. Persistent notification
3. `input_text.test_last_run_result` entity

## How It Works

1. **Test mode activates** - `input_boolean.hvac_test_mode` turns on
2. **Simulated values set** - Test helpers get specific temperature values
3. **Templates evaluate** - `sensor.test_*` sensors calculate based on simulated inputs
4. **Results compared** - Actual vs expected values
5. **Pass/fail recorded** - Counters and logs updated
6. **Next test runs** - Immediately, no waiting
7. **Test mode deactivates** - Returns to normal operation

Each test takes ~50ms. The full suite completes in under 5 seconds.

## Adding New Tests

To add a test, append to the `test_runner.yaml` action sequence:

```yaml
# -------------------------------------------------------------------------
# TEST N: Description
# -------------------------------------------------------------------------
- service: input_text.set_value
  target:
    entity_id: input_text.test_current_name
  data:
    value: "N: Description"

# Set test conditions
- service: input_number.set_value
  target:
    entity_id: input_number.test_bedroom_temperature
  data:
    value: XX

- delay:
    milliseconds: 50

# Check result
- if:
    - condition: template
      value_template: >
        {{ states('sensor.test_bedroom_target_direction') == 'expected_value' }}
  then:
    # ... pass logic
  else:
    # ... fail logic
```

Don't forget to update `test_total_count` at the end of the automation.

## Troubleshooting

**Tests don't run:**
- Check that all helper entities exist
- Verify automation is enabled
- Check Home Assistant logs for errors

**All tests fail:**
- Verify template sensors are loaded (`sensor.test_*`)
- Check that `binary_sensor.hvac_night_mode_testable` exists
- Ensure production helpers exist (setpoints, thresholds, etc.)

**Tests run but results don't show:**
- Refresh dashboard
- Check `input_text.test_last_run_result` directly in Developer Tools > States
