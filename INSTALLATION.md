# Installation Guide - FranklinWH Unraid Plugin

This guide walks you through the complete installation and setup process for the FranklinWH Power Management Plugin.

## Prerequisites

Before installing the plugin, ensure you have:

1. ✅ **Unraid 6.9.0 or newer** installed and running
2. ✅ **FranklinWH Home Battery System** installed and operational
3. ✅ **Network connectivity** between Unraid server and FranklinWH gateway
4. ✅ **FranklinWH account credentials** (Gateway ID and Access Token)
5. ✅ **Internet access** on Unraid server (for initial installation only)

## Step 1: Obtain FranklinWH Credentials

### Gateway ID

1. Install the FranklinWH mobile app:
   - **iOS**: [Download from App Store](https://apps.apple.com/app/franklinwh/id1533243540)
   - **Android**: [Download from Google Play](https://play.google.com/store/apps/details?id=com.franklinwh.app)

2. Log in to your FranklinWH account

3. Navigate to: **Settings → System Info**

4. Your **Gateway ID** will be displayed (format: `FWH-XXXXXX`)
   - Write this down or take a screenshot

### Access Token

The Access Token is required for API access. You can obtain it through:

#### Option 1: Contact FranklinWH Support
- Email: support@franklinwh.com
- Explain you need API access for home automation
- Provide your Gateway ID

#### Option 2: Use the Python API (Advanced)
```bash
# SSH into your Unraid server
pip3 install franklinwh

# Run Python interactive shell
python3

# In Python shell:
from franklinwh import FranklinWH
# Follow authentication prompts
# Token will be provided after successful authentication
```

**Important**: Keep your Access Token secure! Treat it like a password.

## Step 2: Install the Plugin

### Method A: Manual Installation (Current)

1. **SSH into your Unraid server**:
   ```bash
   ssh root@your-unraid-ip
   ```

2. **Download the plugin**:
   ```bash
   wget https://raw.githubusercontent.com/JoshuaSeidel/franklinwh-unraid-plugin/master/franklinwh.plg \
        -P /boot/config/plugins/
   ```

3. **Install the plugin**:
   ```bash
   installplg /boot/config/plugins/franklinwh.plg
   ```

4. **Verify installation**:
   ```bash
   ls -l /usr/local/emhttp/plugins/franklinwh/
   ```

   You should see:
   - `franklinwh_monitor.py`
   - `franklinwh.php`
   - `franklinwh.page`
   - `README.md`

### Method B: Community Applications (Coming Soon)

Once the plugin is published to Community Applications:

1. Open Unraid web interface
2. Click **Apps** tab
3. Search for "FranklinWH"
4. Click **Install**
5. Wait for installation to complete

## Step 3: Initial Configuration

1. **Open Unraid Web Interface**
   - Navigate to `http://your-unraid-ip`

2. **Go to Plugin Settings**
   - Click **Settings** in the top menu
   - Select **FranklinWH** from the left sidebar

3. **Configure Basic Settings**:

   | Setting | Value | Description |
   |---------|-------|-------------|
   | **Service** | Disabled | Leave disabled during initial setup |
   | **Gateway ID** | `FWH-XXXXXX` | Your FranklinWH Gateway ID |
   | **Access Token** | `your-token-here` | Your API access token |
   | **Check Interval** | `60` | Seconds between status checks (30-600) |

4. **Click Apply** (but don't enable service yet)

## Step 4: Test Connection

Before enabling the service, test the connection:

1. **SSH into Unraid**

2. **Run test command**:
   ```bash
   cd /usr/local/emhttp/plugins/franklinwh
   python3 franklinwh_monitor.py
   ```

3. **Check for successful connection**:
   - Look for: `"Connected to FranklinWH system"`
   - Look for: `"Battery: X%, Grid: online"`
   - Press `Ctrl+C` to stop

4. **Check log file**:
   ```bash
   cat /var/log/franklinwh.log
   ```

### Troubleshooting Connection Issues

**Error: "Invalid credentials"**
- Verify Gateway ID and Access Token are correct
- Check for extra spaces or quotes
- Ensure Access Token hasn't expired

**Error: "Connection timeout"**
- Verify FranklinWH gateway is powered on
- Check network connectivity: `ping franklinwh-gateway-ip`
- Ensure firewall isn't blocking connection

**Error: "Module not found: franklinwh"**
- Reinstall Python package:
  ```bash
  pip3 install --force-reinstall franklinwh
  ```

## Step 5: Configure Power Management

Once connection test succeeds:

1. **Return to Web Interface** → **Settings** → **FranklinWH**

2. **Configure Power Thresholds**:

   | Setting | Recommended | Description |
   |---------|------------|-------------|
   | **Low Power Threshold** | `50%` | When to enter low power mode |
   | **Shutdown Threshold** | `20%` | When to initiate shutdown |
   | **Grid Outage Action** | `Low Power Mode` | What to do during outage |
   | **Shutdown Delay** | `300` (5 min) | Grace period before shutdown |

3. **Recommended Settings by Use Case**:

   **Home Server (24/7 uptime priority)**:
   - Low Power: 30%
   - Shutdown: 10%
   - Action: Low Power Mode
   - Delay: 600 seconds (10 min)

   **Media Server (data protection priority)**:
   - Low Power: 50%
   - Shutdown: 20%
   - Action: Auto Shutdown
   - Delay: 300 seconds (5 min)

   **Test/Development Server**:
   - Low Power: 60%
   - Shutdown: 30%
   - Action: Auto Shutdown
   - Delay: 180 seconds (3 min)

## Step 6: Enable the Service

1. **Set Service to "Enabled"**

2. **Click Apply**

3. **Verify Service Started**:
   ```bash
   /etc/rc.d/rc.franklinwh status
   ```

   Should show: `"FranklinWH monitor is running (PID: XXXX)"`

4. **Check Live Status**:
   - Refresh the Settings → FranklinWH page
   - You should see real-time battery status
   - Status updates every 60 seconds

## Step 7: Verify Notifications

Test that notifications work:

1. **Check Notification Settings**:
   - Settings → Notification Settings
   - Ensure you have notification agents configured (e.g., email, Telegram)

2. **Test Notification**:
   ```bash
   /usr/local/emhttp/webGui/scripts/notify \
     -s "FranklinWH Test" \
     -d "This is a test notification" \
     -i alert
   ```

3. **Verify you received the notification**

## Step 8: Set Up Auto-Start (Optional)

The plugin automatically starts on boot if enabled. To verify:

1. **Check go file**:
   ```bash
   cat /boot/config/go
   ```

2. **If not present, add**:
   ```bash
   # Start FranklinWH monitor
   /etc/rc.d/rc.franklinwh start
   ```

## Post-Installation Checklist

After installation, verify:

- ✅ Service is running: `/etc/rc.d/rc.franklinwh status`
- ✅ Status displays in web UI
- ✅ Log file is being written: `tail /var/log/franklinwh.log`
- ✅ Status file is updated: `cat /var/local/emhttp/franklinwh_status.json`
- ✅ Notifications work
- ✅ Configuration persists after reboot

## Testing the Plugin

### Safe Testing Without Real Outage

**DO NOT** actually disconnect your power. Instead:

1. **Monitor logs in real-time**:
   ```bash
   tail -f /var/log/franklinwh.log
   ```

2. **Watch status updates**:
   - Keep web UI open
   - Observe battery percentage
   - Check grid status

3. **Simulate low battery (for testing only)**:
   - Temporarily modify thresholds to be above current battery level
   - Observe plugin behavior
   - Check for notifications
   - Restore normal thresholds

### Full System Test (Advanced)

If you want to test actual grid outage response:

1. **Set conservative thresholds**:
   - Low Power: 80%
   - Shutdown: 70%

2. **Monitor closely**:
   - Have physical access to server
   - Watch web UI
   - Monitor logs

3. **Briefly disconnect grid** (at your own risk):
   - FranklinWH will switch to battery
   - Plugin should detect outage within check interval
   - Watch for notification

4. **Reconnect grid immediately**
   - Before thresholds are reached
   - Verify plugin detects grid restoration

5. **Restore normal thresholds**

## Uninstallation

If you need to remove the plugin:

1. **Disable Service**:
   - Settings → FranklinWH
   - Set Service to "Disabled"
   - Click Apply

2. **Uninstall Plugin**:
   ```bash
   removepkg franklinwh
   ```

   Or via web interface:
   - Plugins → Installed Plugins
   - Click **Remove** next to FranklinWH

3. **Verify Removal**:
   ```bash
   ls /usr/local/emhttp/plugins/ | grep franklinwh
   # Should return nothing
   ```

## Next Steps

- Read [CONFIGURATION.md](CONFIGURATION.md) for advanced settings
- Review [TROUBLESHOOTING.md](TROUBLESHOOTING.md) if issues arise
- Check [README.md](README.md) for feature documentation

## Support

Need help?
- Check logs: `/var/log/franklinwh.log`
- Visit: GitHub Issues
- Forum: Unraid Forums - Plugin Support

---

**Installation complete! Your Unraid server is now protected by FranklinWH power management.** ⚡

