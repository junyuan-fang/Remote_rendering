 # Remote Rendering with VNC

This guide covers setting up VNC remote desktop access from a local machine to a remote Ubuntu machine using TurboVNC.

## Overview

VNC (Virtual Network Computing) allows you to remotely view and control a desktop environment from another computer. This setup uses TurboVNC for optimal performance.

## Prerequisites

- Local machine with network access to remote machine
- Remote Ubuntu machine with administrative privileges
- SSH access to the remote machine

## Setup Instructions

### 1. Install TurboVNC Viewer on Local Machine

Download and install the TurboVNC Viewer application on your local computer from the [official TurboVNC website](https://www.turbovnc.org/).

### 2. Install TurboVNC Server on Remote Machine

#### Add TurboVNC Repository

```bash
# Add GPG key
wget -q -O- https://packagecloud.io/dcommander/turbovnc/gpgkey | \
  gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/TurboVNC.gpg > /dev/null

# Add apt source
echo "deb [signed-by=/etc/apt/trusted.gpg.d/TurboVNC.gpg] https://packagecloud.io/dcommander/turbovnc/any/ any main" | \
  sudo tee /etc/apt/sources.list.d/TurboVNC.list

# Update package list and install
sudo apt update
sudo apt install turbovnc -y
```

#### Locate Installation Path

```bash
dpkg -L turbovnc | grep bin
```

Expected output:
```
/opt/TurboVNC/bin/vncserver
/opt/TurboVNC/bin/vncviewer
```

#### Add to PATH

```bash
export PATH=$PATH:/opt/TurboVNC/bin
```

To make this permanent, add the export line to your `~/.bashrc` or `~/.profile`:
```bash
echo 'export PATH=$PATH:/opt/TurboVNC/bin' >> ~/.bashrc
source ~/.bashrc
```

### 3. Configure VNC Password (Optional but Recommended)

Before starting the VNC server, you can set up a password for security:

```bash
# Set VNC password (recommended)
vncpasswd
```

This will prompt you to:
1. Enter a VNC password (up to 8 characters)
2. Optionally set a view-only password

### 4. Start VNC Server

#### Secure Mode (Recommended)
```bash
# Start VNC server with password authentication
vncserver :1

# View running VNC sessions
vncserver -list
```

#### No Password Mode (⚠️ Security Risk)
```bash
# Start VNC server without password (NOT RECOMMENDED for production)
vncserver :1 -SecurityTypes None
```

**⚠️ Security Warning**: Using `-SecurityTypes None` disables all authentication and allows anyone to connect to your VNC session. Only use this in trusted, isolated network environments or for testing purposes.

**Note**: If displays `:1` and `:2` are occupied by X11 or other services, you can use display `:3` or higher:

```bash
# Use display :3 if :1 and :2 are occupied
vncserver :3

# Or without password (security risk)
vncserver :3 -SecurityTypes None
```

### 5. Establish SSH Connection with Port Forwarding

From your local machine, create an SSH tunnel:

#### For Display :1
```bash
ssh -L 5901:localhost:5901 username@remote_machine_ip
```

#### For Display :3
```bash
ssh -L 5903:localhost:5903 username@remote_machine_ip
```

Replace:
- `username` with your remote machine username
- `remote_machine_ip` with the IP address of your remote machine
- Port numbers follow the pattern: 5900 + display number
  - Display `:1` → Port `5901`
  - Display `:3` → Port `5903`

### 5. Connect Using TurboVNC Viewer

1. Open TurboVNC Viewer on your local machine
2. Enter connection details:
   - **VNC Server**: `localhost:5901` (or `localhost:1`). If you opened port `5903`, please use it.
     ![VNC Start](https://github.com/junyuan-fang/Remote_rendering/blob/master/vnc_start.png)

   - **Password**: Enter the VNC password you set when starting the server
     ![VNC Password](https://github.com/junyuan-fang/Remote_rendering/blob/master/vnc_password.png)

3. Click **Connect**
![VNC Connection](https://github.com/junyuan-fang/Remote_rendering/blob/master/vnc_desktop.png)


## Usage Tips

- To stop a VNC session: `vncserver -kill :1`
- To change VNC password: `vncpasswd`
- For better performance, consider adjusting TurboVNC's compression settings
- Always use SSH tunneling for secure connections over the internet

## Troubleshooting

- If connection fails, check firewall settings on both machines
- Ensure the VNC server is running: `vncserver -list`
- Verify SSH tunnel is active: `netstat -ln | grep 5901`
