# QEMU-KVM / Windows 11 Guest Integration Cheat Sheet

**Purpose:** Set up full host-guest integration (clipboard, display, drivers, file transfer) for a Windows 11 VM on a Linux KVM host.  
**Assumptions:** Ubuntu 24.04 LTS host. Windows 11 25H2 guest. Connection via virt-manager / virt-viewer.  
**Last updated:** 2026-04-08

---

## Capability overview

| Capability | Requires (Host) | Requires (Guest) | Status |
|---|---|---|---|
| Text/image clipboard | `spice-client-gtk`, SPICE display | SPICE Guest Tools (`vdservice`) | Reliable |
| Display auto-resize | `spice-client-gtk` | SPICE Guest Tools (QXL driver) | Reliable |
| Mouse integration | `spice-client-gtk` | SPICE Guest Tools | Reliable |
| Optimized disk/net I/O | — | VirtIO Guest Tools | Reliable |
| Host management (graceful shutdown, snapshot quiesce) | `libvirt-clients` | QEMU Guest Agent (`QEMU-GA`) | Reliable |
| File transfer (drag-and-drop / shared folder) | `virt-viewer` | SPICE WebDAV daemon | Finicky on Windows |
| File transfer (network share) | Samba / OpenSSH | SMB client / OpenSSH | Reliable |

---

## 1. Host-side packages (Ubuntu 24.04)

```bash
sudo apt install qemu-kvm libvirt-daemon-system virt-manager \
  spice-client-gtk spice-vdagent libvirt-clients virt-viewer
```

### Key packages explained

| Package | Purpose |
|---|---|
| `qemu-kvm` | KVM hypervisor and QEMU emulator |
| `libvirt-daemon-system` | libvirt management daemon |
| `virt-manager` | GUI for VM management |
| `spice-client-gtk` | GTK widget for SPICE protocol — **required for clipboard** |
| `spice-vdagent` | Host-side SPICE agent (Linux hosts) |
| `libvirt-clients` | CLI tools (`virsh`, `virt-viewer`, etc.) |
| `virt-viewer` | Standalone SPICE/VNC viewer — recommended for daily use |

### Connect to VM via virt-viewer

```bash
virt-viewer --connect qemu:///system <vm-name>
```

---

## 2. VM configuration (libvirt XML)

Edit with `virsh edit <vm-name>`. All three channels below are required for full integration.

### 2.1 SPICE graphics (replaces VNC)

```xml
<graphics type='spice' autoport='yes'>
  <listen type='address'/>
</graphics>
```

### 2.2 SPICE agent channel (clipboard, resolution, mouse)

```xml
<channel type='spicevmc'>
  <target type='virtio' name='com.redhat.spice.0'/>
</channel>
```

### 2.3 QEMU guest agent channel (host management commands)

```xml
<channel type='unix'>
  <target type='virtio' name='org.qemu.guest_agent.0'/>
</channel>
```

### 2.4 SPICE WebDAV channel (file sharing — optional)

```xml
<channel type='spiceport'>
  <source channel='org.spice-space.webdav.0'/>
  <target type='virtio' name='org.spice-space.webdav.0'/>
</channel>
```

> **Tip:** In virt-manager GUI, add these via **Add Hardware -> Channel**, then select the appropriate type from the dropdown.

---

## 3. Guest-side installs (Windows 11)

Install both packages inside the VM. Order doesn't matter. Reboot once after both are done.

### 3.1 SPICE Guest Tools — clipboard, resolution, mouse

**Download:**
- Latest (0.141): `https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-0.141/spice-guest-tools-0.141.exe`
- Auto-latest: `https://www.spice-space.org/download/windows/spice-guest-tools/spice-guest-tools-latest.exe`

**What it installs:**
- SPICE Agent service (`vdservice`) — clipboard sync, mouse, resolution
- QXL video driver — accelerated display for SPICE

**Install:** Run as Administrator -> accept driver prompts -> reboot.

### 3.2 VirtIO Guest Tools + QEMU Guest Agent — drivers, host management

**Download:**
- ISO: `https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/virtio-win.iso`

**What it installs:**
- VirtIO drivers (disk, network, balloon/memory, serial, SCSI)
- QEMU Guest Agent (`QEMU-GA` service)

**Install:** Mount ISO in guest -> run `virtio-win-guest-tools.exe` -> reboot.

### 3.3 SPICE WebDAV Daemon — file sharing (optional)

**Download:**
- `https://www.spice-space.org/download/windows/spice-webdavd/`

**What it enables:**
- Shared folder from host via `virt-viewer` (File -> Preferences -> Share Folder)
- Mapped as a network drive inside the guest

> Known to be unreliable on Windows guests. Consider Samba or SSH for production file transfer.

---

## 4. Post-install validation

### 4.1 Verify Windows services

Open `services.msc` inside the guest and confirm:

| Service Name | Display Name | Expected State |
|---|---|---|
| `vdservice` | SPICE Agent | Running (Automatic) |
| `QEMU-GA` | QEMU Guest Agent | Running (Automatic) |

### 4.2 Verify guest agent from host

```bash
# Ping the guest agent
virsh qemu-agent-command <vm-name> '{"execute":"guest-ping"}'
# Expected: {"return":{}}

# Query guest info
virsh qemu-agent-command <vm-name> '{"execute":"guest-info"}'
```

### 4.3 Test clipboard

1. Restart `virt-viewer` (clipboard channel initializes on connection, not mid-session).
2. Copy text on host -> paste in guest.
3. Copy text in guest -> paste on host.

---

## 5. File transfer options

| Method | Setup | Reliability | Speed | Notes |
|---|---|---|---|---|
| **SPICE WebDAV** | Low | Flaky | Moderate | Drag-and-drop via virt-viewer; needs WebDAV daemon in guest |
| **Samba/SMB share** | Low | High | Fast | Host runs Samba, guest maps network drive — most dependable |
| **SSH/SCP/SFTP** | Low | High | Fast | Enable OpenSSH server on Windows; `scp` from host |
| **VirtioFS** | Medium | Maturing | Very fast | Requires WinFSP + VirtIO-FS driver; still rough on Win11 25H2 |
| **Tailscale + SSH** | Low | High | Fast | Works across host/guest boundary |

### Recommended: quick Samba setup on host

```bash
# Install
sudo apt install samba

# Create a shared directory
mkdir -p ~/vm-share
chmod 755 ~/vm-share

# Add Samba config
sudo tee -a /etc/samba/smb.conf > /dev/null << 'EOF'

[vmshare]
   path = /home/<your-user>/vm-share
   browseable = yes
   read only = no
   guest ok = no
   valid users = <your-user>
EOF

# Set Samba password
sudo smbpasswd -a <your-user>

# Restart Samba
sudo systemctl restart smbd

# Find host IP for the VM network
ip addr show virbr0
```

**In Windows guest:** Map network drive -> `\\<host-ip>\vmshare`

---

## 6. Troubleshooting

| Symptom | Check |
|---|---|
| Clipboard not working | Is `vdservice` running? Did you reconnect virt-viewer after install? Is display type SPICE (not VNC)? |
| No `vdservice` in services.msc | SPICE Guest Tools not installed — download from spice-space.org |
| `virsh` commands hang | `QEMU-GA` service not running; check VirtIO Guest Tools install |
| Display stuck at low resolution | QXL driver not loaded — check Device Manager -> Display adapters |
| Guest agent ping fails | Verify `org.qemu.guest_agent.0` channel exists in VM XML |
| Shared folder not visible | WebDAV daemon not installed, or channel missing from XML |

---

## References

- SPICE Guest Tools (Windows): `https://www.spice-space.org/download.html`
- SPICE WebDAV Daemon (Windows): `https://www.spice-space.org/download/windows/spice-webdavd/`
- VirtIO-Win Drivers ISO: `https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/stable-virtio/`
- SPICE Project Home: `https://www.spice-space.org/`
- libvirt VM XML Reference: `https://libvirt.org/formatdomain.html`
