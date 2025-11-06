# FranklinWH Power Management Plugin for Unraid

A comprehensive Unraid plugin that integrates with your FranklinWH home battery system to intelligently manage server power during grid outages. Similar to the NUT (Network UPS Tools) plugin for traditional UPS systems, this plugin monitors battery levels and can automatically reduce power consumption or initiate graceful shutdowns.

![FranklinWH Logo](https://www.franklinwh.com/images/logo.png)

## Features

### 🔋 Real-Time Monitoring
- **Battery State of Charge (SOC)**: Track remaining battery capacity
- **Grid Status**: Detect when the grid is online or offline
- **Power Flow**: Monitor battery, solar, and home consumption in real-time
- **Time Remaining**: Estimated runtime on battery power

### ⚡ Smart Power Management
- **Grid Outage Detection**: Automatic detection of power outages
- **Low Power Mode**: Reduce server power consumption at configurable battery threshold
- **Automatic Shutdown**: Graceful server shutdown when battery reaches critical level
- **Configurable Thresholds**: Set custom battery percentages for different actions

### 📊 Web Interface
- **Modern Dashboard**: Beautiful real-time status display
- **Easy Configuration**: Simple web-based settings
- **Visual Indicators**: Color-coded battery status and alerts
- **Auto-Refresh**: Status updates every 60 seconds

### 🔔 Notifications
- **Unraid Notifications**: Integration with native notification system
- **Grid Events**: Alerts when grid goes offline or comes back online
- **Battery Warnings**: Notifications at low power and critical battery levels
- **Shutdown Warnings**: Advanced notice before automatic shutdown

## Requirements

- **Unraid 6.9.0 or newer**
- **FranklinWH Home Battery System** with network connectivity
- **FranklinWH Account** with valid credentials
- **Python 3** (usually pre-installed on Unraid)

## Installation

### Method 1: Community Applications (Recommended)
*Coming soon - once plugin is published to Community Applications*

1. Open Unraid web interface
2. Navigate to **Apps** tab
3. Search for "FranklinWH"
4. Click **Install**

### Method 2: Manual Installation

1. Download the plugin file:
   ```bash
   wget https://raw.githubusercontent.com/JoshuaSeidel/franklinwh-unraid-plugin/master/franklinwh.plg -P /boot/config/plugins/
   ```

2. Install the plugin:
   ```bash
   installplg /boot/config/plugins/franklinwh.plg
   ```

3. Navigate to **Settings > FranklinWH** in the Unraid web interface

## Configuration

### Getting Your FranklinWH Credentials

1. **Install the FranklinWH App** on your smartphone
   - iOS: [App Store](https://apps.apple.com/app/franklinwh/id1533243540)
   - Android: [Google Play](https://play.google.com/store/apps/details?id=com.franklinwh.app)

2. **Log in to your FranklinWH account**

3. **Find Your Gateway ID**
   - Open the app
   - Go to Settings → System Info
   - Your Gateway ID is displayed (format: `FWH-XXXXXX`)

4. **Obtain Your Access Token**
   - Method 1: Contact FranklinWH support
   - Method 2: Use the Python API to authenticate:
     ```python
     from franklinwh import FranklinWH
     # Authentication method varies by region
     ```

### Plugin Configuration

1. Navigate to **Settings > FranklinWH** in Unraid web interface

2. **Basic Settings**:
   - **Service**: Enable/Disable the monitoring service
   - **Gateway ID**: Your FranklinWH Gateway ID
   - **Access Token**: Your FranklinWH API access token
   - **Check Interval**: How often to poll battery status (30-600 seconds, default: 60)

3. **Power Management Settings**:
   - **Low Power Threshold**: Battery % to trigger low power mode (default: 50%)
   - **Shutdown Threshold**: Battery % to trigger shutdown (default: 20%)
   - **Grid Outage Action**:
     - **Monitor Only**: Display status, no automatic actions
     - **Low Power Mode**: Reduce consumption at low power threshold
     - **Auto Shutdown**: Initiate shutdown at critical battery level
   - **Shutdown Delay**: Grace period before shutdown executes (60-3600 seconds, default: 300)

4. Click **Apply** to save settings

5. The service will start automatically if enabled

## Usage

### Viewing Status

Navigate to **Settings > FranklinWH** to view:
- Current battery charge percentage
- Grid status (Online/Offline)
- Battery power flow (charging/discharging)
- Solar power generation
- Home power consumption
- Estimated time remaining on battery

### During a Grid Outage

When the grid goes offline:

1. **Immediate Response**:
   - Plugin detects grid outage within check interval
   - Notification sent to Unraid notification system
   - Monitoring frequency continues as configured

2. **Low Power Mode** (if battery drops to threshold):
   - Array may spin down (configurable)
   - Notification sent
   - Custom actions can be scripted

3. **Critical Battery** (if battery drops to shutdown threshold):
   - Notification sent with shutdown warning
   - Countdown begins (default: 5 minutes)
   - Graceful shutdown initiated
   - Array stopped cleanly

4. **Grid Restoration**:
   - Plugin detects grid is back online
   - Notification sent
   - Low power mode cancelled
   - Normal operation resumes

## Troubleshooting

### Service Won't Start

Check the log file:
```bash
tail -f /var/log/franklinwh.log
```

Common issues:
- Invalid Gateway ID or Access Token
- Network connectivity to FranklinWH system
- Python dependencies not installed

### No Status Data

1. Verify service is running:
   ```bash
   /etc/rc.d/rc.franklinwh status
   ```

2. Check configuration:
   ```bash
   cat /boot/config/plugins/franklinwh/franklinwh.cfg
   ```

3. Restart service:
   ```bash
   /etc/rc.d/rc.franklinwh restart
   ```

### Testing the Plugin

Test without actual power outage:
```bash
# View current status
cat /var/local/emhttp/franklinwh_status.json | python -m json.tool

# Check service status
/etc/rc.d/rc.franklinwh status

# View logs
tail -n 50 /var/log/franklinwh.log
```

## Advanced Configuration

### Custom Actions

You can create custom scripts to run at different power events by modifying the monitor script or creating hooks.

Example: Create a script at `/boot/config/plugins/franklinwh/custom_actions.sh`:

```bash
#!/bin/bash
# Custom actions for FranklinWH events

case "$1" in
    grid_outage)
        # Run when grid goes offline
        echo "Grid outage detected" >> /var/log/custom_franklinwh.log
        # Stop non-essential Docker containers
        # docker stop container_name
        ;;
    low_power)
        # Run when battery hits low power threshold
        echo "Low power mode" >> /var/log/custom_franklinwh.log
        ;;
    critical)
        # Run when battery is critical
        echo "Critical battery" >> /var/log/custom_franklinwh.log
        ;;
    grid_restored)
        # Run when grid comes back online
        echo "Grid restored" >> /var/log/custom_franklinwh.log
        ;;
esac
```

Make it executable:
```bash
chmod +x /boot/config/plugins/franklinwh/custom_actions.sh
```

## API Reference

The plugin uses the [franklinwh](https://pypi.org/project/franklinwh/) Python package to communicate with your FranklinWH system.

Status data structure:
```json
{
  "timestamp": "2024-11-06T12:00:00",
  "battery_soc": 85,
  "grid_status": "online",
  "battery_power": -500,
  "solar_power": 3000,
  "home_consumption": 2500,
  "time_remaining": 420
}
```

## Comparison with NUT Plugin

| Feature | FranklinWH Plugin | NUT Plugin |
|---------|------------------|------------|
| Power Source | Home Battery System | UPS Device |
| Grid Monitoring | ✅ Yes | ✅ Yes |
| Battery Monitoring | ✅ Yes | ✅ Yes |
| Auto Shutdown | ✅ Yes | ✅ Yes |
| Low Power Mode | ✅ Yes | ✅ Yes |
| Solar Monitoring | ✅ Yes | ❌ No |
| Web Interface | ✅ Modern Dashboard | ✅ Basic |
| Notifications | ✅ Yes | ✅ Yes |

## Development

### Project Structure
```
franklinwh-unraid-plugin/
├── franklinwh.plg              # Main plugin manifest
├── README.md                   # This file
├── LICENSE                     # License file
└── docs/                       # Additional documentation
    ├── INSTALLATION.md
    ├── CONFIGURATION.md
    └── TROUBLESHOOTING.md
```

### Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test thoroughly on Unraid
5. Submit a pull request

### Building from Source

The plugin is self-contained in the `.plg` file. To modify:

1. Edit `franklinwh.plg`
2. Test installation: `installplg /path/to/franklinwh.plg`
3. Verify functionality
4. Commit changes

## Support

### Getting Help

- **GitHub Issues**: [Report bugs or request features](https://github.com/JoshuaSeidel/franklinwh-unraid-plugin/issues)
- **Unraid Forums**: [Post in the plugin support thread](https://forums.unraid.net/)
- **Documentation**: Check the `/docs` directory for detailed guides

### Logs

When reporting issues, please include:
```bash
# Plugin log
cat /var/log/franklinwh.log

# Configuration (redact sensitive info)
cat /boot/config/plugins/franklinwh/franklinwh.cfg

# System info
uname -a
python3 --version
```

## Security

- **Access Tokens**: Stored locally on your Unraid server
- **Network Traffic**: Direct communication with your FranklinWH gateway (local network)
- **No Cloud Dependency**: Works during internet outages (if FranklinWH is on local network)

**Important**: Keep your Access Token secure. Do not share your configuration file publicly.

## Changelog

### Version 2024.11.06 (Initial Release)
- ✨ Initial release
- 🔋 FranklinWH battery monitoring integration
- ⚡ Grid outage detection and response
- ⚙️ Configurable power thresholds for shutdown
- 🎨 Web UI for configuration and monitoring
- 🔔 Notification support
- 📝 Similar functionality to NUT plugin for UPS monitoring

## Roadmap

- [ ] Historical data logging and graphs
- [ ] Multiple FranklinWH gateway support
- [ ] Advanced scheduling (e.g., shutdown during specific hours)
- [ ] Integration with other Unraid plugins
- [ ] REST API for external monitoring
- [ ] Mobile app notifications
- [ ] Community Applications listing

## License

This plugin is released under the MIT License. See [LICENSE](LICENSE) file for details.

## Credits

- **Author**: Joshua Seidel
- **Inspired by**: Unraid NUT Plugin
- **FranklinWH API**: [franklinwh Python package](https://pypi.org/project/franklinwh/)
- **Unraid**: [Lime Technology](https://unraid.net/)

## Disclaimer

This plugin is not officially affiliated with or endorsed by FranklinWH or Lime Technology. Use at your own risk. Always test thoroughly before relying on automatic shutdown functionality.

---

**Made with ⚡ for the Unraid community**

