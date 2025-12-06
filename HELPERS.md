# Required Home Assistant Helpers

## Production Helpers

### Input Numbers - Room Setpoints

| Entity ID | Min | Max | Default | Unit |
|-----------|-----|-----|---------|------|
| `input_number.bedroom_setpoint` | 55 | 80 | 65 | °F |
| `input_number.office_setpoint` | 55 | 80 | 68 | °F |
| `input_number.living_room_setpoint` | 55 | 80 | 68 | °F |
| `input_number.first_floor_setpoint` | 55 | 80 | 68 | °F |

### Input Numbers - Room Priority Rankings

Lower number = higher priority.

| Entity ID | Min | Max | Default |
|-----------|-----|-----|---------|
| `input_number.bedroom_priority` | 1 | 4 | 1 |
| `input_number.office_priority` | 1 | 4 | 2 |
| `input_number.first_floor_priority` | 1 | 4 | 3 |
| `input_number.living_room_priority` | 1 | 4 | 4 |

### Input Numbers - System Configuration

| Entity ID | Min | Max | Default | Unit |
|-----------|-----|-----|---------|------|
| `input_number.deviation_threshold` | 1 | 5 | 2 | °F |
| `input_number.deviation_delay_minutes` | 5 | 60 | 20 | min |
| `input_number.minimum_runtime_minutes` | 15 | 120 | 45 | min |
| `input_number.night_non_priority_threshold` | 2 | 10 | 5 | °F |
| `input_number.first_floor_temp_offset` | 0 | 10 | 3 | °F |

Note: `first_floor_temp_offset` compensates for the Mitsubishi unit's built-in sensor reading warmer than the actual room temperature (sensor is near ceiling). When heating, the target sent to the unit is increased by this offset; when cooling, it's decreased.

### Input Datetimes

| Entity ID | Has Date | Has Time | Default |
|-----------|----------|----------|---------|
| `input_datetime.night_mode_start` | No | Yes | 22:00:00 |
| `input_datetime.night_mode_end` | No | Yes | 07:00:00 |

### Input Booleans

| Entity ID | Default |
|-----------|---------|
| `input_boolean.guest_override` | off |

### Input Text

| Entity ID | Default |
|-----------|---------|
| `input_text.priority_room` | none |
| `input_text.condenser_mode_request` | heat |

---

## Test Framework Helpers

These are required for the automated test suite.

### Input Booleans - Test Control

| Entity ID | Default |
|-----------|---------|
| `input_boolean.hvac_test_mode` | off |
| `input_boolean.hvac_test_running` | off |

### Input Numbers - Simulated Temperatures

| Entity ID | Min | Max | Default | Unit |
|-----------|-----|-----|---------|------|
| `input_number.test_bedroom_temperature` | 40 | 100 | 65 | °F |
| `input_number.test_office_temperature` | 40 | 100 | 68 | °F |
| `input_number.test_living_room_temperature` | 40 | 100 | 68 | °F |
| `input_number.test_first_floor_temperature` | 40 | 100 | 68 | °F |

### Input Numbers - Test Counters

| Entity ID | Min | Max | Default |
|-----------|-----|-----|---------|
| `input_number.test_current_index` | 0 | 100 | 0 |
| `input_number.test_passed_count` | 0 | 100 | 0 |
| `input_number.test_failed_count` | 0 | 100 | 0 |
| `input_number.test_total_count` | 0 | 100 | 0 |

### Input Datetimes - Test Time Simulation

| Entity ID | Has Date | Has Time | Default |
|-----------|----------|----------|---------|
| `input_datetime.test_simulated_time` | No | Yes | 12:00:00 |

### Input Text - Test Status

| Entity ID | Default |
|-----------|---------|
| `input_text.test_current_name` | (empty) |
| `input_text.test_current_status` | (empty) |
| `input_text.test_failed_list` | (empty) |
| `input_text.test_last_run_result` | (empty) |
