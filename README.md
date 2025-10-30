### Piximetre

This image relies on [lpasquali/wine](https://hub.docker.com/r/lpasquali/wine/) which has latest
wine on Debian GNU/Linux official slim stable image [debian:stable-slim](https://hub.docker.com/_/debian/) under Debian GNU/Linux

## Platform Compatibility

### Linux
I run piximetre like this: 

`docker run -it --rm -v /tmp/.X11-unix:/tmp/.X11-unix -e DISPLAY=unix$DISPLAY -v $HOME/Pictures:/home/semilanceata/Pictures lpasquali/piximetre`

### macOS (Intel and Apple Silicon)

**Compatible with:**
- macOS High Sierra and later
- macOS Ventura 13.4+ (including M1/M2/M3 MacBooks)

**Requirements for Apple Silicon (M1/M2/M3) Macs:**
- Docker Desktop for Mac 4.6.0 or later
- Rosetta 2 emulation enabled (this image uses x86_64 architecture)
- XQuartz for X11 display forwarding

**Note:** On Apple Silicon Macs, this container runs via Rosetta 2 emulation. Performance may vary compared to Intel-based Macs, but functionality remains the same.

#### macOS Launch Script

Under macOS with Docker Desktop, use this script to run Piximetre:

```bash
#!/bin/bash
# Check if XQuartz is installed
if [ ! -d "/Applications/Utilities/XQuartz.app" ] && [ ! -d "/Applications/XQuartz.app" ]; then
    echo "Error: XQuartz is not installed. Please install it from https://www.xquartz.org/ and try again."
    exit 1
fi
# Check if XQuartz is running
if ! pgrep -x "XQuartz" > /dev/null; then
    echo "Error: XQuartz is not running. Please start XQuartz before running this script."
    exit 1
fi
# Allow connections from localhost
xhost + 127.0.0.1

# Start socat to forward X11 display
screen -S socat-piximetre -d -m socat TCP-LISTEN:6000,reuseaddr,fork UNIX-CLIENT:\"$DISPLAY\"

# Get local IP address
ip=$(ifconfig en0 | grep inet | awk '$1=="inet" {print $2}')
if [ -z "$ip" ]; then
    # Fallback to Docker Desktop for Mac's internal host gateway if en0 interface not found
    ip="host.docker.internal"
fi
echo "Using IP: $ip"

# Allow X11 connections from this IP
xhost + ${ip}

# Run the container
docker run --rm -it \
  --platform linux/amd64 \
  -e DISPLAY=${ip}:0 \
  -v /tmp/.X11-unix:/tmp/.X11-unix \
  --name piximetre \
  lpasquali/piximetre

# Cleanup
xhost - ${ip}
xhost - 127.0.0.1
screen -S socat-piximetre -X quit
killall -9 socat
```

**Alternative for modern macOS (Ventura 13.4+):**

```bash
#!/bin/bash
# Simpler approach using host.docker.internal
xhost + 127.0.0.1
docker run --rm -it \
  --platform linux/amd64 \
  -e DISPLAY=host.docker.internal:0 \
  -v $HOME/Pictures:/home/semilanceata/Pictures \
  --name piximetre \
  lpasquali/piximetre
xhost - 127.0.0.1
```

**Important:** Make sure XQuartz is running and "Allow connections from network clients" is enabled in XQuartz preferences before running the script.
