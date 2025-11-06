# Configuration Guide - FranklinWH Unraid Plugin

**Version 1.0.0**

This guide provides detailed information about all configuration options and best practices for the FranklinWH Power Management Plugin.

## Configuration File Location

The plugin stores its configuration in:
```
/boot/config/plugins/franklinwh/franklinwh.cfg
```

This file persists across reboots since it's stored on the USB boot device.

## Configuration Parameters

### SERVICE
**Type**: String  
**Options**: `enable` | `disable`  
**Default**: `disable`

Controls whether the FranklinWH monitoring service runs.

```bash
SERVICE="enable"
```

- **enable**: Service starts automatically on boot and monitors battery
- **disable**: Service is stopped and does not monitor

**When to disable**:
- During initial setup/testing
- When troubleshooting other issues
- If traveling and server is at remote location

### USERNAME
**Type**: String  
**Format**: Email address  
**Required**: Yes (when service enabled)

Your FranklinWH account email address.

```bash
USERNAME="your@email.com"
```

### PASSWORD
**Type**: String  
**Required**: Yes (when service enabled)

Your FranklinWH account password.

```bash
PASSWORD="your_password"
```

**Security Note**: This password is stored in plain text in the configuration file. Ensure your Unraid server is properly secured.

### GATEWAY_ID
**Type**: String  
**Format**: `10060005A02X########`  
**Required**: Yes (when service enabled)

Your FranklinWH Gateway identifier (Serial Number).

```bash
GATEWAY_ID="10060005A02X24420359"
```

**How to find**:
1. Open FranklinWH mobile app
2. Navigate to More → Site Address
3. Copy the SN (Serial Number) displayed

**Security note**: While not as sensitive as your password, avoid sharing publicly.

### CHECK_INTERVAL
**Type**: Integer  
**Range**: 30 - 600 seconds  
**Default**: 60 seconds

How frequently the plugin queries your FranklinWH system for status updates.

```bash
CHECK_INTERVAL="60"
```

**Recommendations**:

| Use Case | Recommended | Reason |
|----------|-------------|---------|
| Normal operation | 60 seconds | Good balance of responsiveness and efficiency |
| Critical server | 30 seconds | Faster outage detection |
| Low priority | 120-300 seconds | Reduce system load |
| Testing | 30 seconds | Quick feedback during setup |

**Considerations**:
- Lower values = faster response but more API calls
- Higher values = less responsive but lower overhead
- FranklinWH API may have rate limits

### LOW_POWER_THRESHOLD
**Type**: Integer (percentage)  
**Range**: 10 - 100  
**Default**: 50

Battery percentage at which to trigger low power mode during grid outage.

```bash
LOW_POWER_THRESHOLD="50"
```

**Recommendations by battery capacity**:

| Battery Size | Threshold | Reasoning |
|-------------|-----------|-----------|
| 13.6 kWh (single) | 50% | ~6.8 kWh remaining |
| 27.2 kWh (double) | 40% | ~10.9 kWh remaining |
| 40.8 kWh (triple) | 30% | ~12.2 kWh remaining |

**What happens at this threshold**:
- Notification sent
- Custom low power actions executed
- Server reduces consumption (if configured)
- Arrays may spin down (if configured)

### SHUTDOWN_THRESHOLD
**Type**: Integer (percentage)  
**Range**: 5 - 50  
**Default**: 20

Battery percentage at which to initiate automatic shutdown.

```bash
SHUTDOWN_THRESHOLD="20"
```

**Critical guidelines**:
- Must be **lower** than LOW_POWER_THRESHOLD
- Consider server shutdown time (typically 2-5 minutes)
- Account for power draw during shutdown
- Leave buffer for battery variability

**Recommended values**:

| Server Type | Threshold | Reasoning |
|------------|-----------|-----------|
| Simple NAS | 15-20% | Quick shutdown |
| Heavy workload | 25-30% | More shutdown time needed |
| Development | 20-25% | Balance safety/uptime |

**Example calculation**:
```
Battery: 13.6 kWh
Threshold: 20%
Remaining: 2.72 kWh

Server power: 150W
Shutdown time: 3 minutes
Shutdown power: 150W × (3/60)h = 7.5 Wh = 0.0075 kWh

Safe? Yes (2.72 kWh >> 0.0075 kWh)
```

### GRID_OUTAGE_ACTION
**Type**: String  
**Options**: `monitor` | `low_power` | `shutdown`  
**Default**: `low_power`

Determines what action the plugin takes during a grid outage.

```bash
GRID_OUTAGE_ACTION="low_power"
```

#### Option 1: monitor
```bash
GRID_OUTAGE_ACTION="monitor"
```
- **Behavior**: Only displays status and sends notifications
- **No automatic actions**: Server continues normal operation
- **Use when**: You want manual control or just awareness

#### Option 2: low_power
```bash
GRID_OUTAGE_ACTION="low_power"
```
- **Behavior**: Reduces power consumption at LOW_POWER_THRESHOLD
- **Actions**:
  - Send notification
  - Can spin down arrays
  - Custom scripts executed
  - Still monitors for SHUTDOWN_THRESHOLD
- **Use when**: You want automated conservation but not immediate shutdown

#### Option 3: shutdown
```bash
GRID_OUTAGE_ACTION="shutdown"
```
- **Behavior**: Immediately initiates shutdown at SHUTDOWN_THRESHOLD
- **No low power phase**: Goes straight to shutdown
- **Use when**: Data protection is highest priority

**Comparison**:

| Action | Low Power Mode | Auto Shutdown | Best For |
|--------|---------------|---------------|----------|
| monitor | ❌ | ❌ | Manual management |
| low_power | ✅ | ✅ | Balanced approach |
| shutdown | ❌ | ✅ | Quick protection |

### SHUTDOWN_DELAY
**Type**: Integer (seconds)  
**Range**: 60 - 3600  
**Default**: 300 (5 minutes)

Grace period between shutdown initiation and actual shutdown execution.

```bash
SHUTDOWN_DELAY="300"
```

**Purpose**:
- Allows time to cancel if grid returns
- Enables manual intervention
- Permits clean application shutdown
- Provides notification warning period

**Recommendations**:

| Scenario | Delay | Reasoning |
|----------|-------|-----------|
| Stable grid | 300s (5 min) | Standard grace period |
| Frequent outages | 600s (10 min) | Grid may return quickly |
| Remote server | 180s (3 min) | Can't physically intervene |
| Critical services | 120s (2 min) | Prioritize protection |

**Note**: This is the delay passed to the `shutdown` command, not additional delay added by the plugin.

### NOTIFY_EMAIL
**Type**: String (email address)  
**Optional**: Yes  
**Default**: Empty

Email address for notifications (if email notifications configured in Unraid).

```bash
NOTIFY_EMAIL="admin@example.com"
```

**Note**: This feature integrates with Unraid's native notification system. Configure email in:
- Settings → Notification Settings → Email

### NOTIFY_TELEGRAM
**Type**: String (Telegram chat ID)  
**Optional**: Yes  
**Default**: Empty

Telegram chat ID for notifications (if Telegram configured in Unraid).

```bash
NOTIFY_TELEGRAM="123456789"
```

**Note**: Configure Telegram in:
- Settings → Notification Settings → Telegram

## Configuration Profiles

### Profile 1: Conservative (Maximum Uptime)
Best for servers that rarely experience long outages.

```ini
SERVICE="enable"
GATEWAY_ID="FWH-XXXXXX"
ACCESS_TOKEN="your_token"
CHECK_INTERVAL="60"
SHUTDOWN_THRESHOLD="15"
LOW_POWER_THRESHOLD="40"
GRID_OUTAGE_ACTION="low_power"
SHUTDOWN_DELAY="600"
```

**Characteristics**:
- Late intervention (40% low power, 15% shutdown)
- Long grace period (10 minutes)
- Maximizes uptime during short outages

### Profile 2: Balanced (Recommended)
Good for most use cases.

```ini
SERVICE="enable"
GATEWAY_ID="FWH-XXXXXX"
ACCESS_TOKEN="your_token"
CHECK_INTERVAL="60"
SHUTDOWN_THRESHOLD="20"
LOW_POWER_THRESHOLD="50"
GRID_OUTAGE_ACTION="low_power"
SHUTDOWN_DELAY="300"
```

**Characteristics**:
- Moderate thresholds
- Standard grace period (5 minutes)
- Balances uptime and protection

### Profile 3: Aggressive (Maximum Protection)
Best for critical data or frequent long outages.

```ini
SERVICE="enable"
GATEWAY_ID="FWH-XXXXXX"
ACCESS_TOKEN="your_token"
CHECK_INTERVAL="30"
SHUTDOWN_THRESHOLD="30"
LOW_POWER_THRESHOLD="60"
GRID_OUTAGE_ACTION="shutdown"
SHUTDOWN_DELAY="180"
```

**Characteristics**:
- Early intervention (60% low power, 30% shutdown)
- Short grace period (3 minutes)
- Prioritizes data protection over uptime

### Profile 4: Monitor Only
For observation without automatic actions.

```ini
SERVICE="enable"
GATEWAY_ID="FWH-XXXXXX"
ACCESS_TOKEN="your_token"
CHECK_INTERVAL="120"
SHUTDOWN_THRESHOLD="10"
LOW_POWER_THRESHOLD="30"
GRID_OUTAGE_ACTION="monitor"
SHUTDOWN_DELAY="300"
```

**Characteristics**:
- No automatic actions
- Less frequent checks
- Full manual control

## Advanced Configuration

### Custom Scripts

You can create custom scripts that execute at specific events.

**Create custom action script**:
```bash
nano /boot/config/plugins/franklinwh/custom_actions.sh
```

**Example script**:
```bash
#!/bin/bash
# Custom actions for FranklinWH events

LOGFILE="/var/log/franklinwh_custom.log"

log_event() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" >> "$LOGFILE"
}

case "$1" in
    grid_outage)
        log_event "Grid outage detected"
        # Stop non-essential Docker containers
        docker stop plex emby
        # Spin down specific disks
        # /usr/local/sbin/emcmd cmdSpindown=1
        ;;
        
    low_power)
        log_event "Entering low power mode"
        # Stop resource-intensive services
        /etc/rc.d/rc.plex stop
        # Reduce VM resources
        # virsh suspend vm-name
        ;;
        
    critical)
        log_event "Critical battery level"
        # Send custom notification
        curl -X POST "https://api.example.com/alert" \
             -d '{"event":"critical_battery"}'
        ;;
        
    grid_restored)
        log_event "Grid restored"
        # Restart services
        docker start plex emby
        /etc/rc.d/rc.plex start
        ;;
        
    *)
        log_event "Unknown event: $1"
        ;;
esac
```

**Make executable**:
```bash
chmod +x /boot/config/plugins/franklinwh/custom_actions.sh
```

**Modify monitor script to call your script**:
Edit `/usr/local/emhttp/plugins/franklinwh/franklinwh_monitor.py` and add calls to your script at appropriate points.

### Integration with Other Systems

#### Home Assistant Integration
```yaml
# configuration.yaml
sensor:
  - platform: command_line
    name: "Unraid Battery"
    command: "ssh root@unraid-ip 'cat /var/local/emhttp/franklinwh_status.json'"
    value_template: "{{ value_json.battery_soc }}"
    unit_of_measurement: "%"
    scan_interval: 60
```

#### Grafana Dashboard
Query the status JSON file periodically and push to time-series database for visualization.

## Configuration Best Practices

### 1. Test Before Relying
- Set conservative thresholds initially
- Test with simulated scenarios
- Verify notifications work
- Confirm shutdown process is clean

### 2. Document Your Settings
Keep notes on why you chose specific values:
```bash
# My Configuration Notes
# Battery: 13.6 kWh
# Average server draw: 150W
# Expected runtime: ~90 hours
# Shutdown threshold: 20% = 2.72 kWh = ~18 hours remaining
```

### 3. Seasonal Adjustments
Consider adjusting for different scenarios:
- **Summer**: Lower thresholds (more solar charging)
- **Winter**: Higher thresholds (less solar, longer nights)
- **Storm season**: More conservative settings

### 4. Regular Review
- Check logs monthly
- Verify battery capacity hasn't degraded
- Adjust thresholds if server power consumption changes
- Update after adding/removing hardware

### 5. Backup Configuration
```bash
# Backup config
cp /boot/config/plugins/franklinwh/franklinwh.cfg \
   /boot/config/plugins/franklinwh/franklinwh.cfg.backup

# With date
cp /boot/config/plugins/franklinwh/franklinwh.cfg \
   /boot/config/plugins/franklinwh/franklinwh.cfg.$(date +%Y%m%d)
```

## Troubleshooting Configuration

### Configuration Not Persisting
**Problem**: Changes don't survive reboot  
**Solution**: Ensure config file is on `/boot` (USB)
```bash
ls -l /boot/config/plugins/franklinwh/franklinwh.cfg
```

### Service Won't Start After Config Change
**Problem**: Service fails to start  
**Solution**: Check configuration syntax
```bash
# View config
cat /boot/config/plugins/franklinwh/franklinwh.cfg

# Check for syntax errors
# - Missing quotes
# - Invalid values
# - Special characters in token

# Check logs
tail -20 /var/log/franklinwh.log
```

### Thresholds Not Triggering
**Problem**: Actions don't occur at configured thresholds  
**Solution**: Verify battery reporting
```bash
# Check current status
cat /var/local/emhttp/franklinwh_status.json | python -m json.tool

# Verify threshold values
grep THRESHOLD /boot/config/plugins/franklinwh/franklinwh.cfg

# Check if grid outage action is correct
grep GRID_OUTAGE_ACTION /boot/config/plugins/franklinwh/franklinwh.cfg
```

## Configuration via Web Interface

Most users should configure via the web UI:

1. **Navigate**: Settings → FranklinWH
2. **Modify values** using form fields
3. **Click Apply** to save
4. **Service restarts** automatically with new settings

The web interface provides:
- ✅ Input validation
- ✅ Automatic service restart
- ✅ Clear descriptions
- ✅ Real-time status display

## Configuration File Format

The configuration file uses simple `KEY="VALUE"` format:

```bash
# Comments start with hash
VARIABLE="value"
ANOTHER_VAR="123"

# Empty lines are ignored

# Quotes are required for values
# No spaces around equals sign
```

---

**Next Steps**:
- Review [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for common issues
- Check [README.md](README.md) for feature overview
- See [INSTALLATION.md](INSTALLATION.md) for setup guide

