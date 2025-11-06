# FranklinWH Power Management Plugin for Unraid

**Version 1.0.0**

A comprehensive Unraid plugin that integrates with your FranklinWH home battery system to intelligently manage server power during grid outages. Similar to the NUT (Network UPS Tools) plugin for traditional UPS systems, this plugin monitors battery levels and can automatically reduce power consumption or initiate graceful shutdowns.

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
- **FranklinWH Account** with valid credentials (username/password)
- **Gateway ID** from your FranklinWH mobile app
- **Python 3** (install via NerdTools plugin if not already installed)

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

1. **FranklinWH Account**: Use your existing FranklinWH account username (email) and password

2. **Find Your Gateway ID**:
   - Install the FranklinWH App on your smartphone
     - iOS: [App Store](https://apps.apple.com/app/franklinwh/id1533243540)
     - Android: [Google Play](https://play.google.com/store/apps/details?id=com.franklinwh.app)
   - Log in to your account
   - Go to **More → Site Address**
   - Your Gateway ID is shown as **SN** (format: `10060005A02X########`)

### Plugin Configuration

1. Navigate to **Settings > FranklinWH** in Unraid web interface

2. **Basic Settings**:
   - **Service**: Enable/Disable the monitoring service
   - **FranklinWH Username (Email)**: Your FranklinWH account email
   - **FranklinWH Password**: Your FranklinWH account password
   - **FranklinWH Gateway ID**: Your Gateway ID (SN from the app)
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
- Invalid credentials (username, password, or Gateway ID)
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

- **Credentials**: Stored locally on your Unraid server (username, password, gateway ID)
- **API Communication**: Secure HTTPS connection to FranklinWH Cloud API
- **Authentication**: Uses franklinwh library v0.6.0 with token-based API access

**Important**: Keep your credentials secure. Do not share your configuration file publicly.

## Changelog

### Version 1.0.0 (November 6, 2024)

**Initial Release** 🎉

#### Core Features
- ✅ **Real-time Battery Monitoring**: Track SOC, power flow, grid status, and home consumption via FranklinWH Cloud API
- ✅ **Grid Outage Detection**: Automatic detection when grid goes offline/online
- ✅ **Smart Power Management**: Configurable thresholds for low power mode and automatic shutdown
- ✅ **Web-Based Configuration**: Integrated Settings page in Unraid web UI
- ✅ **Live Status Display**: Beautiful real-time dashboard with battery gauge and power metrics
- ✅ **Unraid Notifications**: Native integration with Unraid notification system
- ✅ **Python Monitoring Daemon**: Robust background service with graceful shutdown handling
- ✅ **Simple Authentication**: Username/password + Gateway ID (no manual token management)

#### Technical Details
- Uses franklinwh Python library v0.6.0
- Compatible with Unraid 6.9.0+
- Requires Python 3
- 60-second polling interval (configurable 30-600s)
- Persistent configuration on USB boot device

#### Documentation
- Comprehensive README with features and quick start
- Detailed installation guide with step-by-step instructions
- Complete configuration reference
- Troubleshooting guide with common issues and solutions

## Roadmap

Future enhancements being considered:

### v1.1.0 - Enhanced Monitoring
- [ ] Historical data logging and graphing
- [ ] Energy statistics (daily/weekly/monthly summaries)
- [ ] CSV/JSON export of historical data
- [ ] Customizable dashboard refresh intervals

### v1.2.0 - Multi-System Support
- [ ] Multiple FranklinWH gateway support
- [ ] System comparison and aggregation
- [ ] Per-gateway configuration profiles

### v1.3.0 - Advanced Automation
- [ ] Time-based scheduling (shutdown during specific hours)
- [ ] Calendar integration for planned outages
- [ ] Custom action scripts (pre-shutdown, post-restore)
- [ ] Integration with other Unraid plugins (VM manager, Docker)

### v2.0.0 - Extended Features
- [ ] REST API for external monitoring
- [ ] Prometheus/Grafana integration
- [ ] Email/SMS/Push notifications
- [ ] Local API support (reduce cloud dependency)
- [ ] Advanced power management modes

### Community
- [x] GitHub repository with issue tracking
- [ ] Community Applications listing
- [ ] Unraid forum thread
- [ ] User contributed scripts and configurations

**Want to contribute?** Open an issue or pull request on [GitHub](https://github.com/JoshuaSeidel/unraid-franklinwh-plugin)

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

