# HVAC Automation Requirements

## Summary

To save time and increase productivity and enjoyment of our townhome, I need to automate our existing HVAC system using Home Assistant and Zigbee.

### Success Criteria

| Metric | Target |
|--------|--------|
| Daily condenser mode switches | < 15 |
| Daytime room temperature (all rooms) | Within ±2°F of setpoint for >95% of day |
| Nighttime bedroom temperature | Within ±2°F of setpoint |
| Nighttime other rooms | Within ±5°F of setpoint (relaxed threshold) |

### Objective

Create a Home Assistant automation that is:
- Consistent, smart, and efficient in maintaining default setpoints
- Easy to add, test, and get running in Home Assistant
- Easy to configure in Home Assistant preferably via GUI

---

## Townhome

Our Seattle townhome is three stories and totals 1900 square feet divided roughly equally between the three floors. The townhome footprint is rectangular and the length runs north and south.

### Floors

| Floor | Description | Typical Occupancy | Head Unit Location |
|-------|-------------|-------------------|-------------------|
| 1 | Ground floor. About 450 sq ft in the large main room, concrete floor. Bathroom is about 150 sq ft. | Morning workouts, dogs kept there when out, 2-3x/year becomes guest bedroom | One large unit on south side |
| 2 | Middle floor. About 500 sq ft in the large main room | Sporadic 1-3 individuals throughout day for ~15 min each, sometimes 2 hours in evenings | One large unit on south side |
| 3 | Top floor. Divided roughly equally between bedroom, office, and bathroom. Gets good sunlight late spring/summer. | 6 PM to 5 AM: me, wife, and two ~40 lb dogs | One medium unit in bedroom, one medium unit in office |

### Rooms

| Room | Floor | Sq Ft | Head Unit Location | Day Setpoint | Night Setpoint |
|------|-------|-------|-------------------|--------------|----------------|
| Bedroom | 3 / Top | 250 | West side | 65°F | 65°F |
| Office | 3 / Top | 250 | South side | 68°F | 68°F |
| Living room | 2 / Middle | 500 | South side | 68°F | 68°F |
| First floor | 1 / Ground | 400 | South side | 68°F | 68°F |

---

## System Constraints

### Condenser State Limitation

The condenser can be in only one state at a time: **heating** or **cooling**. Therefore, all four head units must be in the same collective state. For example, we cannot heat the bedroom while simultaneously cooling the first floor.

The system must **never** output a "none", "off", or invalid condenser mode. When no room needs active conditioning, the condenser holds its current mode while head units use neutral temperature targets.

### Head Unit Temperature Limitation

The head unit sensors are unreliable (often off by 5-8°F). The system uses dedicated **Sonoff temperature sensors** as the authoritative readings. If a Sonoff sensor fails, the system should fall back to the head unit sensor via Sensibo until restored.

---

## Priority Room Selection

The system recalculates room priority every 1 minute.

### Eligibility Criteria

A room becomes eligible for priority only if **all** of these conditions are met:

1. Its temperature is outside its setpoint by ≥ the **deviation threshold** (default: ±2°F)
2. That deviation has been sustained continuously for ≥ the **deviation delay** (default: 20 minutes)
3. The room meets the applicable threshold for the current mode (day vs night)

**Important**: The deviation delay timer resets if the room returns within the deviation threshold, even briefly.

### Priority Selection

Among all eligible rooms:
- The room with the **lowest numeric priority rank** becomes the priority room
- There can only be **one** priority room at a time
- If no room meets eligibility criteria, priority room = "none" and no room influences condenser mode

### Priority Rankings

| Rank | Day Time | Night Time | Day + Guest Override | Night + Guest Override |
|------|----------|------------|---------------------|----------------------|
| 1 | Bedroom | Bedroom | First floor | Bedroom |
| 2 | Office | Office | Bedroom | First floor |
| 3 | First floor | First floor | Office | Office |
| 4 | Living room | Living room | Living room | Living room |

### Day vs Night Mode

| Setting | Day Mode | Night Mode |
|---------|----------|------------|
| Default hours | 7 AM - 10 PM | 10 PM - 7 AM |
| Bedroom deviation threshold | ±2°F | ±2°F |
| Other rooms deviation threshold | ±2°F | ±5°F (relaxed) |
| Configurable | Yes (dashboard) | Yes (dashboard) |

**Night mode logic**: The bedroom always qualifies for priority at ±2°F. Other rooms can only qualify if their deviation is ≥ the night non-priority threshold (default: 5°F). This prevents non-bedroom rooms from triggering mode switches at night unless they drift significantly.

### Priority Preemption

If a higher-ranked room becomes eligible while a lower-ranked room is priority:
- The higher-ranked room immediately becomes priority
- The lower-ranked room remains a "targeted room" (actively conditioned) but is no longer priority
- Condenser mode is determined solely by the new priority room

---

## Condenser Mode Selection

### Mode Determination

The condenser mode (heat or cool) is based **solely** on the priority room's temperature relative to its setpoint:
- Priority room temp < setpoint → **heat**
- Priority room temp > setpoint → **cool**

No other room may directly change the condenser mode. Lower-ranked rooms requiring a different mode may temporarily drift away from their setpoints while the priority room is being conditioned. This is expected and acceptable.

### Minimum Runtime

To prevent rapid oscillation, the condenser must remain in its current mode for at least the **minimum runtime** before a reversal is allowed.

| Parameter | Default |
|-----------|---------|
| Minimum runtime | 45 minutes |

### Mode Selection Summary

```
IF priority_room == "none":
    Hold current condenser mode (no change)

ELSE IF minimum_runtime has NOT elapsed:
    Hold current mode

ELSE IF priority room needs opposite mode:
    Switch to that mode

ELSE:
    Hold current mode
```

---

## Head Unit Control

Each head unit operates based on the global condenser mode and the room's individual deviation.

### Temperature Targets

| Room State | Condenser in Heat | Condenser in Cool |
|------------|------------------|------------------|
| Room needs conditioning (deviation ≥ 2°F, same direction as condenser) | Set to room setpoint | Set to room setpoint |
| Room does not need conditioning | Set to 60°F (neutral) | Set to 80°F (neutral) |

Head units must **never** be placed into fan-only mode or turned off. All head units must always match the condenser's heat or cool mode.

### Fan Speed Control

Fan speed is determined by the room's absolute deviation from setpoint:

| Absolute Deviation | Fan Speed |
|-------------------|-----------|
| > 3°F | High |
| 1.5°F - 3°F | Medium |
| ≤ 1.5°F | Low |

**Exception:** First floor always runs on **high** when being conditioned (regardless of deviation or priority status). This is because the first floor is 600 sq ft and requires more aggressive conditioning.

Rooms that do not need conditioning still run fans on **low** for circulation and mold prevention.

### Simultaneous Conditioning

If multiple rooms need the same type of conditioning (e.g., all need cooling), they should all be conditioned simultaneously while the condenser is in that mode. Once a room reaches its setpoint, it switches to a neutral temperature target but remains in the same heat/cool mode.

---

## Guest Override

### Activation

Guest mode activates when someone interacts with the first-floor wall thermostat. This:
- Overrides the first-floor setpoint with the guest's chosen temperature
- Changes priority rankings (first floor becomes rank 1 during day)
- Persists until manually deactivated

### Priority Impact

| Time | First Floor Priority (Guest Active) |
|------|-------------------------------------|
| Day | Rank 1 (highest) |
| Night | Rank 2 (bedroom remains rank 1) |

### Deactivation

Guest mode must be manually turned off via the Home Assistant dashboard after guests leave.

---

## Configuration

All configuration should be accessible via Home Assistant GUI/dashboard.

### Configurable Parameters

| Parameter | Default | Entity Type |
|-----------|---------|-------------|
| Room setpoints (per room) | See rooms table | `input_number` |
| Deviation threshold | 2°F | `input_number` |
| Deviation delay | 20 minutes | `input_number` |
| Minimum runtime | 45 minutes | `input_number` |
| Night non-priority threshold | 5°F | `input_number` |
| Night mode start time | 10:00 PM | `input_datetime` |
| Night mode end time | 7:00 AM | `input_datetime` |
| Guest override active | Off | `input_boolean` |
| Room priority ranks (per room) | See rankings table | `input_number` |

---

## Hardware & Software

| Component | Description | How It's Used |
|-----------|-------------|---------------|
| Mitsubishi Condenser | Single outdoor unit | Controls overall heating or cooling |
| Mitsubishi Head Units | 4 indoor units | Heat or cool individual spaces |
| Sensibo | 4 IR blasters near head units | Send commands to head units |
| Home Assistant | Automation platform | Central control system |
| Zigbee + ZHA | Protocol/dongle | Connect smart devices |
| Sonoff Temperature Sensors | 4 sensors, 1 per room | Authoritative temperature readings |
| Guest Thermostat | Zigbee wall thermostat | Guest temperature control for first floor |

### Home Assistant Entity IDs

| Room | Device Type | Entity ID |
|------|-------------|-----------|
| First floor | Temperature sensor | `sensor.temperature_sensor_first_floor_temperature` |
| First floor | Head unit | `climate.shop_air` |
| Living room | Temperature sensor | `sensor.temperature_sensor_living_room_temperature` |
| Living room | Head unit | `climate.living_room_air` |
| Office | Temperature sensor | `sensor.temperature_sensor_office_temperature` |
| Office | Head unit | `climate.office_air` |
| Bedroom | Temperature sensor | `sensor.temperature_sensor_bedroom_temperature` |
| Bedroom | Head unit | `climate.bedroom_air` |

---

## Recovery & Reliability

### Sensibo Signal Verification

The automation should verify that commands sent via Sensibo are received by head units. If a command fails:
1. Retry up to 5 times
2. If all retries fail, create an alert in Home Assistant (possible obstruction)

### Power Recovery

The automation should:
- Automatically restart after a power outage
- Resume operation immediately when the HVAC breaker is turned back on after cleaning

---

## Monitoring & Alerting

### Dashboard Requirements

- Real-time: condenser mode, each head unit status (mode, temp, setpoint)
- Historical: temperature graphs per room over time
- Condenser mode change log
- Guest override status and history
- Automation action log

### Alerts

- Automation stops running
- Condenser mode changed more than 15 times in a day
- Sensibo command delivery failure

### Data Retention

Historical data should be retained indefinitely and exportable for troubleshooting.

---

## Removed/Simplified from Original PRD

The following items from the original PRD have been removed or simplified based on analysis:

1. **±0.4°F "per-room need" threshold**: Removed. This was too granular and caused false conflicts. A single ±2°F threshold is used consistently.

2. **Outdoor temperature**: Removed as a decision input. Indoor sensors already reflect outdoor influence; added complexity for marginal benefit.

3. **Time-of-day mode bias**: Removed. Seattle's temperate climate doesn't have dramatic enough swings to warrant this complexity.

4. **Separate day/night setpoints**: Simplified. Setpoints remain constant; only priority rankings and deviation thresholds change between day and night.
