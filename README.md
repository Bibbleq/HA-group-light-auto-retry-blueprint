# HA Group Light Auto Retry Blueprint

This Home Assistant blueprint monitors service calls to `light.all_*` groups and automatically retries any lights that fail to respond correctly during bulk on/off events. It's particularly useful for Z-Wave networks where multiple simultaneous commands can overwhelm the mesh, leaving some lights out of sync.

## Features

- **Smart pattern matching**: Only triggers on `light.all_*` groups (configurable pattern)
- **Intelligent state detection**: Checks both state AND brightness to catch stuck lights
- **Configurable retry passes**: Choose 1-5 retry attempts (default: 2)
- **Detailed logging**: Logs what was caught, retried, and fixed on each pass
- **Parallel execution**: Can handle multiple group commands simultaneously
- **Handles edge cases**: Treats `unknown`/`unavailable` lights as out-of-sync
- **Smart early exit**: Stops retrying once all lights are synced
- **Resilient error handling**: Uses `continue_on_error` to ensure all lights are attempted even if some fail due to network congestion

## How It Works

1. **Trigger**: Detects any `light.turn_on` or `light.turn_off` service call
2. **Filter**: Only proceeds if a `light.all_*` group is in the targets
3. **Wait**: Delays (default 4s) for initial command to propagate
4. **Retry Loop**: For each pass (up to configured max):
   - Check which lights are out of sync (checks brightness for OFF commands)
   - If all synced, exit early
   - Otherwise, retry out-of-sync lights and log
   - Wait (default 3s) before next check
5. **Complete**: Final state logged

## State Detection Logic

- **For OFF commands**: Light is considered synced only if state is `off` AND brightness is 0
- **For ON commands**: Light is considered synced if state is `on`
- **Unknown/Unavailable**: Always treated as out-of-sync and retried

## Usage

1. Place `group_light_auto_retry_blueprint.yaml` in your `blueprints/automation` directory
2. Create a new automation from this blueprint
3. Configure the settings (or use defaults)
4. The automation runs automatically in the background

## Configuration Options

| Option | Description | Default |
|--------|-------------|---------|
| **Group Pattern** | Regex pattern for matching group names | `^light\.all_` |
| **Max Retry Passes** | Maximum number of retry attempts (1-5) | 2 |
| **Initial Delay** | Seconds to wait before first check | 4 |
| **Retry Delay** | Seconds to wait between retry passes | 3 |
| **Log Level** | System log level (info/warning/error) | warning |

## Example Scenario

```
1. You call light.turn_off on light.all_living_room
2. Blueprint waits 4 seconds
3. Pass 1: Finds light.living_room_lamp_3 is still on with brightness 128
4. Retries turn_off, logs: "[CATCHER PASS 1] Retried..."
5. Waits 3 seconds
6. Pass 2: Checks again - all synced, exits early ✓
```

## Logs

Check Home Assistant logs for entries like:
```
[CATCHER START] service=turn_off group_eid=light.all_living_room ...
[CATCHER PASS 1] Retried off for ['light.living_room_lamp_3'] (from light.all_living_room)
[CATCHER COMPLETE] All lights synced after 1 pass
```

## Original Automation

This blueprint is based on the "Catch all_* light group calls" automation, converted to a reusable blueprint.

## License

MIT