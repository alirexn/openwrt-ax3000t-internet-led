# AX3000T Smart Internet LED for OpenWrt

![OpenWrt](https://img.shields.io/badge/OpenWrt-24.10+-blue)
![Xiaomi AX3000T](https://img.shields.io/badge/Xiaomi-AX3000T-green)

A simple and smart script to restore the original Xiaomi firmware LED behavior on the **Xiaomi AX3000T** running OpenWrt:

- **Blue LED**: When real internet connection is available
- **Yellow LED**: When internet is completely down (even if the WAN cable is connected)

The script checks actual internet connectivity every 5 seconds by pinging Google and Cloudflare DNS servers. It works independently of the number of WAN interfaces or failover configurations.

Perfect for users who miss the smart LED indicators after switching to OpenWrt!

## Features

- Real internet detection (not just WAN link status)
- Works with multiple WANs and failover setups
- No additional package dependencies
- Lightweight and low resource usage
- Installation in under 1 minute

## Installation

1. Log in to your router via SSH and run the following command block :

```bash
cat > /etc/init.d/internet-led << 'EOF'
#!/bin/sh /etc/rc.common

START=99
STOP=10

LED_BLUE="/sys/class/leds/blue:status/brightness"
LED_YELLOW="/sys/class/leds/yellow:status/brightness"

start() {
    echo "[INFO] Internet LED monitor started."

    if ping -c 1 -W 3 8.8.8.8 >/dev/null 2>&1 || ping -c 1 -W 3 1.1.1.1 >/dev/null 2>&1; then
        echo 255 > $LED_BLUE
        echo 0 > $LED_YELLOW
    else
        echo 0 > $LED_BLUE
        echo 255 > $LED_YELLOW
    fi

    while true; do
        if ping -c 1 -W 3 8.8.8.8 >/dev/null 2>&1 || ping -c 1 -W 3 1.1.1.1 >/dev/null 2>&1; then
            echo 255 > $LED_BLUE
            echo 0 > $LED_YELLOW
        else
            echo 0 > $LED_BLUE
            echo 255 > $LED_YELLOW
        fi
        sleep 5
    done &
}

stop() {
    echo "[INFO] Internet LED monitor stopped."
    echo 255 > $LED_BLUE
    echo 0 > $LED_YELLOW
    killall sleep 2>/dev/null || true
}
EOF
```
2. Make it executable and enable the service :

```bash
   chmod +x /etc/init.d/internet-led
   /etc/init.d/internet-led enable
   /etc/init.d/internet-led start
   ```
3. (Optional) Reboot the router :

```bash
reboot
```

⭐ If you find this useful, please give it a star so others can discover it
