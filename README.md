# Meatmeet S Pro ESPHome Integration

ESPHome package for the Meatmeet S Pro 6-channel Bluetooth thermometer.

## Features

- 🌡️ 5 temperature sensors (Zone 1-5) with 0.01°C resolution
- 🏠 1 ambient temperature sensor (integer °C)
- 🔋 Battery level sensor
- 📡 Automatic BLE connection
- 🔄 Continuous data updates with configurable interval

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

## Example Configuration

See `meatmeet-example.yaml` for a complete example configuration.

## Technical Details

### BLE Protocol

- **Service UUID**: `A6ED0401-D344-460A-8075-B9E8EC90D71B`
- **Notification UUID**: `0000F5A1-0000-1000-8000-00805F9B34FB`
- **Write UUID**: `A6ED0404-D344-460A-8075-B9E8EC90D71B`
- **Data Packet**: 54 Bytes
- **Header**: `0x4D 0x45` ("ME")

### Data Structure

| Byte(s) | Content | Format |
|---------|--------|--------|
| 0-1 | Header "ME" | `0x4D 0x45` |
| 8 | Battery Level | uint8 (%) |
| 11-12 | Zone 1 Temperature | uint16 LE (0.01°C) |
| 13 | Ambient Temperature | uint8 (°C) |
| 16-17 | Zone 2 Temperature | uint16 LE (0.01°C) |
| 18-19 | Zone 3 Temperature | uint16 LE (0.01°C) |
| 20-21 | Zone 4 Temperature | uint16 LE (0.01°C) |
| 22-23 | Zone 5 Temperature | uint16 LE (0.01°C) |

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
