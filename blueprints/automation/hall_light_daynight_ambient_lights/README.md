# Light DayNight Ambient Lights Automation Blueprint

An advanced Home Assistant automation blueprint that intelligently controls lights based on ambient light levels, with support for multiple modes and customizable thresholds.

## Overview

This blueprint automatically manages light brightness by:
- Switching between active and ambient lighting modes based on ambient light sensor readings
- Supporting manual modes (On, Off) and automatic modes (Auto, Ambient, Schedule)
- Applying hysteresis with configurable thresholds to prevent flickering
- Providing variable delays for mode and schedule changes

## Modes

| Mode | Description |
|------|-------------|
| **Uit** (Off) | Lights are turned off |
| **Aan** (On) | Lights are set to active brightness level |
| **Auto** | Lights automatically switch between active and ambient based on light level and schedule |
| **Ambient** | Lights are set to ambient brightness when light level is dark enough |
| **Schedule** | Ambient lighting is controlled by the is_ambient schedule entity |

## Required Inputs

### Core Entities

- **Light (as light)**: Target light(s) to control
- **light_mode**: Input select entity defining the current mode (Uit, Aan, Auto, Ambient, Schedule)
- **active_level_value**: Input number entity for active brightness percentage (0-100)
- **light_level_value**: Input number entity for current ambient light level (0-100)
- **is_ambient**: Schedule entity that defines ambient time windows

### Brightness Levels

- **ambient_level**: Brightness percentage when in ambient mode (default: 20%)

### Light Level Thresholds

- **light_level_ambient_threshold**: Base light level threshold above which ambient lighting is not needed (default: 50)
- **light_level_darker_threshold**: Offset below ambient threshold to trigger ambient mode (default: 20)
  - Example: If ambient_threshold = 50 and darker_threshold = 20, switches to ambient below 30
- **light_level_lighter_threshold**: Offset above ambient threshold to exit ambient mode (default: 20)
  - Example: If ambient_threshold = 50 and lighter_threshold = 20, switches to active above 70

### Wait Times (in minutes)

- **light_level_darker_wait_time**: How long light level must stay in darker range before switching to ambient (default: 5 min)
- **light_level_lighter_wait_time**: How long light level must stay in lighter range before switching back to active (default: 5 min)

### Change Delays (in seconds)

- **light_mode_change_delay**: Delay before reacting to light_mode changes (default: 0 sec)
- **is_ambient_change_delay**: Delay before reacting to is_ambient schedule changes (default: 0 sec)

## How It Works

### Triggers

The automation reacts to the following events:

1. **Light level drops below darker threshold** → After configured wait time, activates ambient mode (if conditions allow)
2. **Light level rises above lighter threshold** → After configured wait time, activates active mode (if conditions allow)
3. **Light mode changes** → After optional delay, applies new mode logic
4. **Active level changes** → Immediately updates active brightness
5. **Schedule state changes** → After optional delay, applies schedule-based logic

### Logic Flow

The automation evaluates conditions in this order:

1. **Aan (On) mode**: Light turns on at active_level_value brightness
2. **Ambient mode**: Light turns on at ambient_level brightness if:
   - Mode is Ambient AND light level is below threshold, OR
   - Mode is Schedule AND schedule is active, OR
   - Mode is Auto AND schedule is active AND light level is below threshold
3. **Default**: Light turns off

## Example Setup

### Scenario: Bedroom with automatic day/night lighting

**Configuration:**
- `light_mode`: input_select.bedroom_mode (options: Uit, Aan, Auto, Ambient, Schedule)
- `active_level_value`: input_number.bedroom_brightness
- `light_level_value`: input_number.bedroom_ambient_light (from sensor)
- `is_ambient`: schedule.bedroom_evening_hours
- `ambient_level`: 15 (15% for nightlight)
- `light_level_ambient_threshold`: 40
- `light_level_darker_threshold`: 15 (switches to ambient below 25)
- `light_level_lighter_threshold`: 15 (switches back to active above 55)
- `light_level_darker_wait_time`: 3 (minutes)
- `light_level_lighter_wait_time`: 2 (minutes)

**Behavior:**
- When light level drops below 25 and stays there for 3 minutes → turns on nightlight at 15%
- When light level rises above 55 and stays there for 2 minutes → turns on full brightness
- Evening schedule activates → provides ambient lighting even if light level is moderate
- Set mode to Auto → fully automatic behavior based on light level and schedule

## Tips

- **Avoid flickering**: Increase wait times if you're experiencing rapid on/off cycles
- **Threshold tuning**: Start with conservative thresholds and adjust based on sensor behavior
- **Delay usage**: Add mode change delays to prevent accidental mode switches
- **Hysteresis**: The offset thresholds (darker/lighter) create hysteresis to prevent bouncing between modes

## Installation

1. Copy this blueprint file to your `config/blueprints/automation/` directory
2. In Home Assistant, go to **Settings → Automations & Scenes → Blueprints**
3. Click **Create Automation** and select "Light DayNight Ambient Lights"
4. Configure the required inputs and save

## Troubleshooting

- **Lights constantly switching**: Increase `light_level_darker_wait_time` and `light_level_lighter_wait_time`
- **Not responding to mode changes**: Ensure `light_mode` entity exists and has correct options
- **Schedule not working**: Verify `is_ambient` is a valid schedule entity with on/off states
- **Wrong brightness**: Check that `active_level_value` and `ambient_level` are set correctly
