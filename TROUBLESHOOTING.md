# Troubleshooting Guide - FranklinWH Unraid Plugin

This guide helps you diagnose and resolve common issues with the FranklinWH Power Management Plugin.

## Quick Diagnostic Commands

Run these commands to gather information about your plugin status:

```bash
# Check if service is running
/etc/rc.d/rc.franklinwh status

# View recent logs
tail -50 /var/log/franklinwh.log

# Check current status
cat /var/local/emhttp/franklinwh_status.json | python -m json.tool

# View configuration
cat /boot/config/plugins/franklinwh/franklinwh.cfg

# Check if Python dependencies are installed
pip3 list | grep franklinwh

# View process details
ps aux | grep franklinwh
```

## Common Issues

### Issue 1: Service Won't Start

**Symptoms**:
- Service shows as "not running"
- Status page shows "No status data available"
- Nothing in logs

**Diagnosis**:
```bash
# Try to start manually
/etc/rc.d/rc.franklinwh start

# Check what happened
tail -20 /var/log/franklinwh.log

# Check for Python errors
python3 /usr/local/emhttp/plugins/franklinwh/franklinwh_monitor.py
```

**Common Causes & Solutions**:

#### Cause 1: Service Disabled in Config
```bash
# Check config
grep SERVICE /boot/config/plugins/franklinwh/franklinwh.cfg

# If shows: SERVICE="disable"
# Solution: Enable via web UI or edit config:
sed -i 's/SERVICE="disable"/SERVICE="enable"/' /boot/config/plugins/franklinwh/franklinwh.cfg
```

#### Cause 2: Missing Credentials
```bash
# Check if credentials are set
grep -E "GATEWAY_ID|ACCESS_TOKEN" /boot/config/plugins/franklinwh/franklinwh.cfg

# Should show:
# GATEWAY_ID="FWH-XXXXXX"
# ACCESS_TOKEN="some_long_token"

# Solution: Add credentials via web UI
```

#### Cause 3: Python Dependencies Missing
```bash
# Check if franklinwh package is installed
pip3 show franklinwh

# If not installed:
pip3 install franklinwh

# Or reinstall plugin:
installplg /boot/config/plugins/franklinwh.plg
```

#### Cause 4: Script Not Executable
```bash
# Check permissions
ls -l /usr/local/emhttp/plugins/franklinwh/franklinwh_monitor.py

# Should show: -rwxr-xr-x (executable)
# If not:
chmod +x /usr/local/emhttp/plugins/franklinwh/franklinwh_monitor.py
```

### Issue 2: "Failed to Connect to FranklinWH System"

**Symptoms**:
- Log shows: "Error connecting to FranklinWH"
- Service starts but no status data

**Diagnosis**:
```bash
# Test network connectivity to FranklinWH
# (Replace with your FranklinWH IP if known)
ping -c 4 192.168.1.XXX

# Test API credentials manually
python3 << 'EOF'
from franklinwh import FranklinWH
import os

# Read config
config = {}
with open('/boot/config/plugins/franklinwh/franklinwh.cfg', 'r') as f:
    for line in f:
        if '=' in line and not line.startswith('#'):
            key, val = line.strip().split('=', 1)
            config[key] = val.strip('"')

try:
    client = FranklinWH(config['GATEWAY_ID'], config['ACCESS_TOKEN'])
    print("✓ Connection successful!")
    data = client.get_data()
    print(f"✓ Battery: {data.get('soc', 'N/A')}%")
except Exception as e:
    print(f"✗ Connection failed: {e}")
EOF
```

**Solutions**:

#### Solution 1: Invalid Credentials
```bash
# Re-verify your Gateway ID and Access Token
# Gateway ID format: FWH-XXXXXX
# Access Token: Long alphanumeric string

# Update via web UI: Settings → FranklinWH
```

#### Solution 2: Network Issues
```bash
# Check if FranklinWH is on network
# Find FranklinWH IP:
nmap -sn 192.168.1.0/24 | grep -i franklin

# Or check your router's DHCP client list

# Test connectivity
ping YOUR_FRANKLINWH_IP

# If ping fails, check:
# - FranklinWH gateway is powered on
# - Network cable connected
# - Same network as Unraid
# - Firewall not blocking
```

#### Solution 3: API Token Expired
```bash
# Access tokens may expire
# Solution: Request new token from FranklinWH support
# Or re-authenticate via FranklinWH app
```

### Issue 3: Status Not Updating

**Symptoms**:
- Status page shows old timestamp
- Same values don't change
- Service is running but appears frozen

**Diagnosis**:
```bash
# Check if process is actually running
ps aux | grep franklinwh_monitor

# Check last log entry
tail -1 /var/log/franklinwh.log

# Check status file modification time
ls -l /var/local/emhttp/franklinwh_status.json
stat /var/local/emhttp/franklinwh_status.json
```

**Solutions**:

#### Solution 1: Process Hung
```bash
# Restart service
/etc/rc.d/rc.franklinwh restart

# If restart fails, force kill:
pkill -9 -f franklinwh_monitor
/etc/rc.d/rc.franklinwh start
```

#### Solution 2: API Rate Limiting
```bash
# Check if CHECK_INTERVAL is too low
grep CHECK_INTERVAL /boot/config/plugins/franklinwh/franklinwh.cfg

# If below 30 seconds, increase:
# Settings → FranklinWH → Check Interval: 60
```

#### Solution 3: FranklinWH System Not Responding
```bash
# Test direct API access
python3 << 'EOF'
from franklinwh import FranklinWH
import sys

config = {}
with open('/boot/config/plugins/franklinwh/franklinwh.cfg', 'r') as f:
    for line in f:
        if '=' in line and not line.startswith('#'):
            key, val = line.strip().split('=', 1)
            config[key] = val.strip('"')

try:
    client = FranklinWH(config['GATEWAY_ID'], config['ACCESS_TOKEN'])
    data = client.get_data()
    print("API is responding")
    print(f"Battery: {data.get('soc')}%")
except Exception as e:
    print(f"API error: {e}")
    sys.exit(1)
EOF

# If this fails, issue is with FranklinWH system, not plugin
```

### Issue 4: Notifications Not Received

**Symptoms**:
- No notifications during grid events
- No test notifications received
- Plugin appears to be working otherwise

**Diagnosis**:
```bash
# Test Unraid notification system
/usr/local/emhttp/webGui/scripts/notify \
  -s "Test Notification" \
  -d "Testing notification system" \
  -i alert

# Did you receive notification?
# If no, issue is with Unraid notifications, not this plugin
# If yes, issue is specific to plugin
```

**Solutions**:

#### Solution 1: Unraid Notifications Not Configured
```bash
# Configure notifications:
# Settings → Notification Settings

# Enable at least one agent:
# - Email
# - Telegram
# - Discord
# - Pushover
# - etc.

# Test agent after configuration
```

#### Solution 2: Plugin Not Sending Notifications
```bash
# Check logs for notification attempts
grep -i "notification" /var/log/franklinwh.log

# If no entries, plugin may not be detecting events
# Check if grid outage actually occurred
cat /var/local/emhttp/franklinwh_status.json | grep grid_status

# Manually trigger notification test by simulating event
# (requires code modification for testing)
```

### Issue 5: Shutdown Not Triggering

**Symptoms**:
- Battery drops below threshold
- No shutdown occurs
- No shutdown notification

**Diagnosis**:
```bash
# Check current battery level
cat /var/local/emhttp/franklinwh_status.json | grep battery_soc

# Check shutdown threshold
grep SHUTDOWN_THRESHOLD /boot/config/plugins/franklinwh/franklinwh.cfg

# Check grid status
cat /var/local/emhttp/franklinwh_status.json | grep grid_status

# Check if shutdown was attempted
grep -i "shutdown" /var/log/franklinwh.log
```

**Important**: Shutdown only triggers if:
1. ✓ Grid is offline (`grid_status: "offline"`)
2. ✓ Battery below `SHUTDOWN_THRESHOLD`
3. ✓ `GRID_OUTAGE_ACTION` is set to `low_power` or `shutdown`
4. ✓ Service is running

**Solutions**:

#### Solution 1: Grid Status Not Detected as Offline
```bash
# FranklinWH must report grid as offline
# Check current status
cat /var/local/emhttp/franklinwh_status.json

# If grid_status shows "online" but grid is actually off,
# FranklinWH may not be detecting outage correctly
# Check FranklinWH app to verify
```

#### Solution 2: Wrong Action Configured
```bash
# Check action setting
grep GRID_OUTAGE_ACTION /boot/config/plugins/franklinwh/franklinwh.cfg

# If set to "monitor", no shutdown will occur
# Change to "low_power" or "shutdown":
# Settings → FranklinWH → Grid Outage Action
```

#### Solution 3: Threshold Configuration Error
```bash
# Ensure threshold is valid number
grep SHUTDOWN_THRESHOLD /boot/config/plugins/franklinwh/franklinwh.cfg

# Should be numeric value between 5-50
# If invalid, fix via web UI
```

### Issue 6: Web Interface Shows "Page Not Found"

**Symptoms**:
- Settings → FranklinWH menu entry missing or broken
- Click menu item, get error

**Solutions**:

```bash
# Check if page files exist
ls -l /usr/local/emhttp/plugins/franklinwh/

# Should show:
# - franklinwh.page
# - franklinwh.php
# - franklinwh_monitor.py

# If missing, reinstall plugin:
installplg /boot/config/plugins/franklinwh.plg

# Refresh web browser
# Clear browser cache if needed
```

### Issue 7: High CPU Usage

**Symptoms**:
- `franklinwh_monitor.py` using significant CPU
- System slowdown
- High load average

**Diagnosis**:
```bash
# Check CPU usage
top -b -n 1 | grep franklinwh

# Check how often script is running
grep CHECK_INTERVAL /boot/config/plugins/franklinwh/franklinwh.cfg

# Check log for errors/loops
tail -100 /var/log/franklinwh.log | grep -i error
```

**Solutions**:

```bash
# Increase check interval to reduce frequency
# Settings → FranklinWH → Check Interval: 120 or higher

# Check for error loops in log
# If constantly retrying connection, fix connection issue first

# Restart service
/etc/rc.d/rc.franklinwh restart
```

### Issue 8: Plugin Not Starting After Reboot

**Symptoms**:
- Plugin works when manually started
- Doesn't auto-start after reboot

**Solutions**:

```bash
# Check if service is enabled
grep SERVICE /boot/config/plugins/franklinwh/franklinwh.cfg
# Should be: SERVICE="enable"

# Check if rc script is executable
ls -l /etc/rc.d/rc.franklinwh
# Should be: -rwxr-xr-x

# Add to go file for auto-start
echo '/etc/rc.d/rc.franklinwh start' >> /boot/config/go

# Test after next reboot
```

### Issue 9: "ModuleNotFoundError: No module named 'franklinwh'"

**Symptoms**:
- Service fails to start
- Log shows Python module error

**Solutions**:

```bash
# Install Python package
pip3 install franklinwh

# If pip3 not found, install pip:
curl https://bootstrap.pypa.io/get-pip.py -o /tmp/get-pip.py
python3 /tmp/get-pip.py
rm /tmp/get-pip.py

# Then install franklinwh
pip3 install franklinwh

# Verify installation
pip3 show franklinwh

# Restart service
/etc/rc.d/rc.franklinwh restart
```

### Issue 10: Status JSON File Contains Errors

**Symptoms**:
- Web UI shows parsing errors
- Status file corrupted

**Solutions**:

```bash
# Check if file is valid JSON
python3 -m json.tool /var/local/emhttp/franklinwh_status.json

# If invalid, remove and let plugin recreate:
rm /var/local/emhttp/franklinwh_status.json
/etc/rc.d/rc.franklinwh restart

# Wait one check interval, then verify:
cat /var/local/emhttp/franklinwh_status.json | python -m json.tool
```

## Advanced Troubleshooting

### Enable Debug Logging

Modify the monitor script to increase log verbosity:

```bash
# Edit monitor script
nano /usr/local/emhttp/plugins/franklinwh/franklinwh_monitor.py

# Change logging level from INFO to DEBUG
# Find this line:
#   level=logging.INFO,
# Change to:
#   level=logging.DEBUG,

# Save and restart
/etc/rc.d/rc.franklinwh restart

# View debug logs
tail -f /var/log/franklinwh.log
```

### Manual API Testing

Test API connection independently:

```bash
python3 << 'EOF'
from franklinwh import FranklinWH
import json

GATEWAY_ID = "FWH-XXXXXX"  # Replace with yours
ACCESS_TOKEN = "your_token"  # Replace with yours

try:
    print("Connecting to FranklinWH...")
    client = FranklinWH(GATEWAY_ID, ACCESS_TOKEN)
    
    print("Fetching data...")
    data = client.get_data()
    
    print("\nSuccess! Data received:")
    print(json.dumps(data, indent=2))
    
except Exception as e:
    print(f"\nError: {e}")
    import traceback
    traceback.print_exc()
EOF
```

### Check File Permissions

Ensure all files have correct permissions:

```bash
# Plugin directory
chmod 755 /usr/local/emhttp/plugins/franklinwh
chmod 644 /usr/local/emhttp/plugins/franklinwh/*.php
chmod 644 /usr/local/emhttp/plugins/franklinwh/*.page
chmod 755 /usr/local/emhttp/plugins/franklinwh/*.py

# RC script
chmod 755 /etc/rc.d/rc.franklinwh

# Config directory
chmod 755 /boot/config/plugins/franklinwh
chmod 644 /boot/config/plugins/franklinwh/*.cfg

# Log file
touch /var/log/franklinwh.log
chmod 644 /var/log/franklinwh.log
```

### Reinstall Plugin

If all else fails:

```bash
# Stop service
/etc/rc.d/rc.franklinwh stop

# Backup configuration
cp /boot/config/plugins/franklinwh/franklinwh.cfg \
   /boot/config/plugins/franklinwh/franklinwh.cfg.backup

# Remove plugin
removepkg franklinwh 2>/dev/null
rm -rf /usr/local/emhttp/plugins/franklinwh
rm -f /etc/rc.d/rc.franklinwh

# Reinstall
installplg /boot/config/plugins/franklinwh.plg

# Restore configuration
mv /boot/config/plugins/franklinwh/franklinwh.cfg.backup \
   /boot/config/plugins/franklinwh/franklinwh.cfg

# Start service
/etc/rc.d/rc.franklinwh start
```

## Getting Help

If you've tried everything and still have issues:

### Gather Information

Create a diagnostic report:

```bash
cat << 'EOF' > /tmp/franklinwh_diagnostic.txt
=== FranklinWH Plugin Diagnostic Report ===
Generated: $(date)

--- System Info ---
$(uname -a)
$(python3 --version)

--- Service Status ---
$(/etc/rc.d/rc.franklinwh status)

--- Process Info ---
$(ps aux | grep franklinwh)

--- Configuration (redacted) ---
$(grep -v "ACCESS_TOKEN" /boot/config/plugins/franklinwh/franklinwh.cfg)

--- Recent Logs ---
$(tail -50 /var/log/franklinwh.log)

--- Status File ---
$(cat /var/local/emhttp/franklinwh_status.json 2>/dev/null || echo "No status file")

--- Python Packages ---
$(pip3 list | grep -E "franklinwh|requests")

--- File Permissions ---
$(ls -la /usr/local/emhttp/plugins/franklinwh/)
$(ls -la /etc/rc.d/rc.franklinwh)

=== End Report ===
EOF

cat /tmp/franklinwh_diagnostic.txt
```

### Where to Get Help

1. **GitHub Issues**: [Report bug with diagnostic report]
2. **Unraid Forums**: [Plugin Support Thread]
3. **FranklinWH Support**: For API/system issues

When posting for help, include:
- ✓ Diagnostic report (redact sensitive info!)
- ✓ Unraid version
- ✓ FranklinWH model
- ✓ What you've already tried
- ✓ Exact error messages

---

**Still having issues?** Check the [Configuration Guide](CONFIGURATION.md) or review the [Installation Guide](INSTALLATION.md).

