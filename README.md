# 🏡 VillageConnect

**An Integrated Local Communication System for Rural Communities**

[![Status](https://img.shields.io/badge/Status-Complete-success)]()
[![License](https://img.shields.io/badge/License-MIT-blue)]()
[![Platform](https://img.shields.io/badge/Platform-GNS3%20%7C%20Ubuntu-orange)]()

> A complete self-contained network providing VoIP calls, a web portal, and file sharing to rural villages — without requiring internet access.

---

## 📖 About the Project

VillageConnect simulates a complete enterprise-grade network for a village of approximately 1,000 residents. The system delivers three essential services over a local network:

- 📞 **Voice Calls (VoIP)** — Free calls between villagers using Asterisk PBX
- 🌐 **Web Portal** — Community announcements and directory (Arabic RTL)
- 📁 **File Sharing** — Centralized educational resources via Samba

**No internet required.** Everything runs on a single Ubuntu server connected to a simulated Cisco network.

---

## 👥 Team

| Name | Role |
|------|------|
| **Owais AbuSalah** | Network Design & Configuration |
| **Abdulah Dwekat** | Server & Services Setup |
| **Anas Habash** | Testing & Documentation |

**Supervised by:** Dr. Jihad Hamamra
**Academic Year:** 2025–2026
**Course:** Computer Networks

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────┐
│           Core Router (c3745)               │
│     OSPF Area 0 | Router-on-a-Stick         │
└──────────────────┬──────────────────────────┘
                   │ Trunk
┌──────────────────┴──────────────────────────┐
│           Core Switch (IOSvL2)              │
│         VLANs 10, 20, 30, 40                │
└─────┬───────┬────────┬────────┬─────────────┘
      │       │        │        │
   ┌──┴──┐ ┌─┴───┐ ┌──┴───┐ ┌──┴──────────┐
   │Admin│ │Public│ │Voice │ │ Ubuntu      │
   │VLAN │ │VLAN  │ │VLAN  │ │ Server      │
   │ 10  │ │ 20   │ │ 30   │ │ (VLAN 40)   │
   └─────┘ └──────┘ └──────┘ └─────────────┘
```

### VLAN Design

| VLAN | Name | Subnet | Purpose |
|:----:|:----:|:------:|:--------|
| 10 | Admin | `192.168.10.0/24` | Village administration |
| 20 | Public | `192.168.20.0/24` | General residents |
| 30 | Voice | `192.168.30.0/24` | IP phones (QoS) |
| 40 | Servers | `192.168.40.0/24` | Backend services |

---

## 🛠️ Technology Stack

- **GNS3** — Network simulation platform
- **Cisco IOS** — c3745 router + IOSvL2 switches
- **VMware Workstation** — Virtualization for Ubuntu
- **Ubuntu Server 24.04 LTS** — Host OS for services
- **Asterisk PBX 20** — VoIP server
- **Apache HTTP Server** — Community web portal
- **Samba** — SMB/CIFS file sharing
- **Twinkle** — SIP softphone for testing

---

## 📞 VoIP Extensions

| Extension | Role | Credentials |
|:---------:|:----:|:-----------:|
| 1000 | Admin | `village1000` |
| 1001 | Mayor | `village1001` |
| 1002 | Clinic | `village1002` |
| 2000 | School | `village2000` |
| 911 | Emergency | `emergency911` |
| `*43` | Echo Test | — |
| `100` | Announcements | — |

---

## 🗂️ Repository Structure

```
VillageConnect/
├── README.md                           # This file
├── presentation/
│   ├── VillageConnect_Presentation_Final.pptx
│   └── VillageConnect_Presentation_Final.pdf
├── reports/
│   ├── VillageConnect_Report_English.docx
│   ├── VillageConnect_Report_English.pdf
│   
│   
├── configs/
│   ├── cisco/
│   │   ├── Core-Router.txt
│   │   ├── Core-Switch.txt
│   │   └── Access-Switches.txt
│   ├── asterisk/
│   │   ├── sip.conf
│   │   └── extensions.conf
│   ├── samba/
│   │   └── smb.conf
│   └── system/
│       └── sysctl.conf
├── website/
│   └── index.html
└── screenshots/                        # Add your screenshots here
```

---

## 🚀 Deployment Guide

### Prerequisites

- GNS3 2.2+ with GNS3 VM
- VMware Workstation 17+
- Cisco IOS c3745 image
- Cisco IOSvL2 image
- Ubuntu Server 24.04 LTS ISO

### 1. Build the GNS3 Topology

Import the topology:
- Add **Core-Router** (c3745)
- Add **Core-Switch** (IOSvL2)
- Add 3× **Access Switches** (IOSvL2)
- Add 3× **VPCS** end-hosts (optional for testing)
- Add 1× **Cloud** node (for Ubuntu bridge)

Connect according to the architecture diagram.

### 2. Configure Cisco Devices

Apply configurations from `configs/cisco/`:
```
Core-Router.txt       → Core-Router console
Core-Switch.txt       → Core-Switch console
Access-Switches.txt   → Apply per-switch sections
```

### 3. Setup Ubuntu Server

In VMware:
1. Create VMnet2 (Host-only, `192.168.40.0/24`, DHCP off)
2. Install Ubuntu Server 24.04 with Custom: VMnet2 adapter
3. Set static IP `192.168.40.10/24`, gateway `192.168.40.1`

Apply kernel fix:
```bash
sudo cp configs/system/sysctl.conf /etc/sysctl.conf
sudo sysctl -p
```

### 4. Link GNS3 Cloud to VMnet2

In GNS3, edit the Cloud node → Select **VMware Network Adapter VMnet2** → Connect to Core-Switch Gi1/1.

### 5. Install Services

```bash
# Apache
sudo apt install -y apache2
sudo cp website/* /var/www/html/

# Samba
sudo apt install -y samba samba-common-bin
sudo cp configs/samba/smb.conf /etc/samba/
sudo mkdir -p /srv/village/{shared,uploads,admin}
sudo systemctl restart smbd nmbd

# Asterisk
sudo apt install -y asterisk
sudo cp configs/asterisk/*.conf /etc/asterisk/
sudo systemctl restart asterisk
```

### 6. Test

```bash
# Verify SIP peers
sudo asterisk -rx "sip show peers"

# Test web
curl http://192.168.40.10

# List Samba shares
smbclient -L //192.168.40.10 -N
```

---

## ✅ Test Results

| Test Case | Expected | Status |
|-----------|----------|:------:|
| Ping gateway from all VLANs | Reply received | ✅ PASS |
| Inter-VLAN routing (Admin → Server) | OSPF routes correctly | ✅ PASS |
| Web portal from PC-Admin | HTTP 200, pages render | ✅ PASS |
| Samba share listing | Files visible | ✅ PASS |
| SIP registration (Twinkle) | Peer online | ✅ PASS |
| Echo test (`*43`) | Voice echoed | ✅ PASS |
| Call 1000 → 1001 | Ringing + audio | ✅ PASS |

---

## 🔍 Challenges & Solutions

### 1. VMware-GNS3 Integration
**Problem:** LAN Segments don't appear in GNS3 Cloud interfaces.
**Solution:** Use VMnet2 Host-only adapter in Virtual Network Editor.

### 2. Reverse Path Filtering
**Problem:** Ubuntu drops replies to pings from other VLANs.
**Solution:** Disable `rp_filter` in `/etc/sysctl.conf`.

### 3. SIP Port Conflict
**Problem:** Softphone fails to bind to 5060 (taken by Asterisk).
**Solution:** Run Twinkle on port 5062.

### 4. Linphone Authentication
**Problem:** Linphone returns 401 without sending credentials.
**Solution:** Use Twinkle — implements SIP digest auth correctly.

---

## 📚 Documentation

- **[English Report](reports/VillageConnect_Report_English.pdf)** — Full technical report (PDF)
- **[Arabic Report](reports/VillageConnect_Report_Arabic.pdf)** — النسخة العربية
- **[Presentation](presentation/VillageConnect_Presentation_Final.pdf)** — 14-slide deck

---

## 🌟 Future Enhancements

- 📡 Wireless access points for mobile devices
- 🎥 Jitsi Meet for video conferencing
- 📚 Offline Wikipedia/Khan Academy cache
- 🌐 Mesh networking via BATMAN-adv
- ☀️ Solar-powered network cabinets
- 🛰️ Optional satellite internet gateway

---

## 📜 License

MIT License — Free to use, modify, and distribute.

---

## 🙏 Acknowledgments

Special thanks to **Dr. Jihad Hamamra** for supervision and guidance throughout this project.

---

<p align="center">
  <i>Connecting communities where internet cannot reach.</i>
</p>
