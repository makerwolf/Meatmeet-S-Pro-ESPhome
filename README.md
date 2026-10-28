# Meatmeet S Pro ESPHome Integration

ESPHome package for the Meatmeet S Pro 6-channel Bluetooth thermometer.

## Features

- 🌡️ 5 temperature sensors (Zone 1-5) with 0.01°C resolution
- 🏠 1 ambient temperature sensor (integer °C)
- 🔋 Battery level sensor
- 📡 Automatic BLE connection
- 🔄 Continuous data updates with configurable interval
- ⏰ Target temperature alarm on the base station (start/stop from Home Assistant)

## Installation

### Option 1: Local File

1. Download `meatmeet-package.yaml`
2. Copy the file to your ESPHome configuration directory
3. Add the following to your ESPHome configuration:

```yaml
substitutions:
  meatmeet_mac: "XX:XX:XX:XX:XX:XX"  # Your device MAC address
  update_interval: "3s"

packages:
  meatmeet: 
    file: !include meatmeet-package.yaml
```

### Option 2: From GitHub (Remote Package)

```yaml
substitutions:
  meatmeet_mac: "XX:XX:XX:XX:XX:XX"  # Your device MAC address
  update_interval: "3s"

# Meatmeet Package
packages:
  - url: https://github.com/makerwolf/Meatmeet-S-Pro-ESPhome
    file: meatmeet-s-pro-package.yaml
    refresh: 1s
```

## Configurable Parameters

| Parameter | Default | Description |
|-----------|----------|--------------|
| `meatmeet_mac` | `20:02:5A:12:00:D6` | MAC address of the thermometer |
| `update_interval` | `3s` | Interval for status requests |

## Available Sensors

After integration, the following sensors will be available in Home Assistant:

- **Zone 1-5**: Temperature in °C (0.1°C accuracy)
- **Ambient**: Grip-Temperature in °C (integer)
- **Battery**: Battery level in %
- **Cook State**: `Idle`, `Running` or `Stopped`

The following controls are also available:

- **Alarm Target**: Target temperature in °C (0-300, 0.5° steps)
- **Start Alarm**: Sends the target temperature to the base station and starts a cook. The base station beeps by itself when the probe reaches the target.
- **Stop Alarm**: Stops the cook and silences the alarm

The alarm commands are addressed to the probe ID, which is read automatically from the status packets and logged (`Probe ID: ...`). Wait until data is being received before pressing **Start Alarm**.

To use the current time as the cook start time, add a [time source](https://esphome.io/components/time/) such as `time: - platform: homeassistant` to your configuration. Without one, a fixed timestamp is sent, which the base station also accepts.

## Example Configuration

See `meatmeet-example.yaml` for a complete example configuration.

## Technical Details

### BLE Protocol

- **Service UUID**: `A6ED0401-D344-460A-8075-B9E8EC90D71B`
- **Notification UUID**: `0000F5A1-0000-1000-8000-00805F9B34FB`
- **Write UUID**: `A6ED0404-D344-460A-8075-B9E8EC90D71B`
- **Alarm Command UUID**: `0000F5A1-0000-1000-8000-00805F9B34FB` (service `0000F5A0-...`)
- **Data Packet**: 54 Bytes
- **Header**: `0x4D 0x45` ("ME")

### Data Structure

| Byte(s) | Content | Format |
|---------|--------|--------|
| 0-1 | Header "ME" | `0x4D 0x45` |
| 4 | Number of connected probes (probably) | uint8 |
| 5-10 | Probe ID | 6 bytes |
| 8 | Battery Level | uint8 (%) |
| 11-12 | Zone 1 Temperature | uint16 LE (0.01°C) |
| 13 | Ambient Temperature | uint8 (°C) |
| 16-17 | Zone 2 Temperature | uint16 LE (0.01°C) |
| 18-19 | Zone 3 Temperature | uint16 LE (0.01°C) |
| 20-21 | Zone 4 Temperature | uint16 LE (0.01°C) |
| 22-23 | Zone 5 Temperature | uint16 LE (0.01°C) |
| 28 | Cook State | uint8 (0 = idle, 1 = running, 2 = stopped) |

### Alarm Commands

Written to `0000F5A1-0000-1000-8000-00805F9B34FB`:

| Command | Bytes |
|---------|-------|
| Set target | `4D 45 29 08` + probe ID (6 bytes) + target (uint16 LE, 0.01°C) |
| Start cook | `4D 45 30 0C` + probe ID (6 bytes) + `01` + Unix time as 10 BCD digits (5 bytes) |
| Stop cook | `4D 45 30 07` + probe ID (6 bytes) + `02` |

## Requirements

- ESP32 board supporting BLE
- ESPHome 2023.12.0 or newer
- Home Assistant (optional but recommended)

## Troubleshooting

### Device Not Found

- Ensure the MAC address is correct
- Check if the thermometer is powered on
- Reduce the distance between ESP32 and thermometer

### No Data Received

- Check the logs for `ESP_GATTC_NOTIFY_EVT` events
- Ensure the device is connected
- Verify the BLE tracker is enabled

### Connection Drops

- Increase the `update_interval` (e.g., to `5s`)
- Reduce other BLE activities on the ESP32
- Check the distance to the device

## License

MIT License - Free to use for personal and commercial projects.

## Credits

Developed through reverse-engineering of the Meatmeet S Pro iOS app Bluetooth communication.
