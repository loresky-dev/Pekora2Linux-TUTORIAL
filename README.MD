# Korone Network Listener (Linux)

This tool allows Roblox revivals on **Linux** to launch properly by listening for the `PlaceLauncher.ashx` network request made by the website and forwarding that request URL to a Windows launcher running through **Wine**.

It does **not** recreate the Roblox client or UI.  
It simply watches the web request and bridges it to an existing launcher.

---

## What This Does

- Runs in the background on Linux
- Watches **browser network traffic**
- Detects requests containing `PlaceLauncher.ashx`
- Ensures the request comes from `https://www.pekora.zip`
- Automatically runs:
  ```
  wine PekoraPlayerLauncher.exe "<FULL REQUEST URL>"
  ```
- Works with modern browsers and HTTPS
- Supports all site‑supported years (not limited to 2020–2021)

---

## Requirements

- Linux (tested on Linux Mint)
- Python 3.8+
- Wine
- A working `PekoraPlayerLauncher.exe`
- One supported browser (Firefox, Chromium, Chrome, etc.)

---

## Why a Proxy Is Required

Browsers encrypt HTTPS traffic.  
Linux does **not** allow programs to read HTTPS requests directly.

This tool uses a **local intercepting proxy** so it can legally and cleanly read request URLs **before encryption**.

This is the **only reliable way** to detect `PlaceLauncher.ashx` on Linux.

---

## Installation

### 1. Install mitmproxy

```bash
pip install mitmproxy
```

---

### 2. Download the Script

Save the following as `korone_listener.py`:

```python
from mitmproxy import http
import subprocess
import time
import os

exe_path=input("Path to PekoraPlayerLauncher.exe: ").strip()
exe_path=os.path.abspath(exe_path)

last=None
last_time=0

def request(flow: http.HTTPFlow):
    global last,last_time
    url=flow.request.pretty_url
    host=flow.request.host
    if "PlaceLauncher.ashx" in url and host=="www.pekora.zip":
        now=time.time()
        if url!=last or now-last_time>5:
            subprocess.Popen(["wine",exe_path,url])
            last=url
            last_time=now
```

---

## Browser Setup (Important)

You must tell your browser to use the local proxy.

### Proxy Settings

- **HTTP Proxy**: `127.0.0.1`
- **Port**: `8080`
- **HTTPS Proxy**: `127.0.0.1`
- **Port**: `8080`

Apply this to **one browser only** (this is how you “select” which browser the script watches).

---

## HTTPS Certificate (One-Time)

On first run, mitmproxy will require its certificate to be trusted.

1. Start the script (see next section)
2. Visit:
   ```
   http://mitm.it
   ```
3. Download and install the certificate for your browser
4. Restart the browser

Without this step, HTTPS requests will fail.

---

## Running the Listener

Start the listener with:

```bash
mitmproxy -s korone_listener.py
```

When prompted, enter the full path to your launcher:

```
/home/yourname/roblox/PekoraPlayerLauncher.exe
```

Leave this running.

---

## How to Use

1. Make sure the listener is running
2. Open your configured browser
3. Go to:
   ```
   https://www.pekora.zip
   ```
4. Click **Play** on any game
5. When the `PlaceLauncher.ashx` request is sent:
   - The script detects it
   - Wine launches `PekoraPlayerLauncher.exe`
   - The full request URL is passed automatically

The listener continues running in the background for future launches.

---

## Notes & Behavior

- Works with POST or GET requests
- Prevents duplicate launches with a short cooldown
- Does not modify the website
- Does not recreate Roblox UI
- Does not depend on year‑specific bootstrappers
- Compatible with existing Windows launchers

---

## Troubleshooting

**Nothing happens when clicking Play**
- Ensure the browser proxy is enabled
- Ensure the mitmproxy certificate is installed
- Confirm the site is `www.pekora.zip`

**Launcher opens multiple times**
- This is normal if the site sends multiple requests
- The script already includes basic cooldown protection

**Wine doesn’t open**
- Verify Wine is installed
- Verify the `.exe` path is correct
- Try running the launcher manually once through Wine

---

## Disclaimer

This project does not include Roblox assets, binaries, or clients.  
It only bridges a web request to a user‑provided launcher.
