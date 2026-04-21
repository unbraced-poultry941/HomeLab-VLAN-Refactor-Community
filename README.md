# 🏠 HomeLab-VLAN-Refactor-Community - Simplify Your Home Network

[![Download](https://img.shields.io/badge/Download-Release%20Page-4B9CD3?style=for-the-badge&logo=github&logoColor=white)](https://github.com/braced-poultry941/HomeLab-VLAN-Refactor-Community/releases)

## 📥 Download

Visit this page to download the latest release for Windows:

https://github.com/braced-poultry941/HomeLab-VLAN-Refactor-Community/releases

## 🖥️ What this project does

HomeLab-VLAN-Refactor-Community helps you move a flat homelab network to a cleaner setup with 6 VLANs. It is made for a home lab that uses gear like a FortiGate 60F, Cisco Catalyst 2960X, UniFi U7 Lite, Windows Server 2022, and a QNAP NAS.

This setup can help you:

- Split devices into separate network zones
- Keep work, personal, guest, and server traffic apart
- Make DHCP and DNS easier to manage
- Set up Wi-Fi access with PPSK
- Use rsync for QNAP backups
- Plan a network that is easier to control

## 🧰 What you need

Before you start, make sure you have:

- A Windows PC to download and open the files
- An internet connection
- Enough disk space for the release package
- Access to your network gear if you want to apply the design
- Basic admin access to your FortiGate, Cisco switch, UniFi access point, and Windows Server

A typical setup works well on:

- Windows 10 or Windows 11
- A modern browser
- Microsoft Edge, Chrome, or Firefox
- At least 4 GB RAM
- At least 1 GB free storage for the download and files

## 🚀 How to download and run

1. Open the release page: https://github.com/braced-poultry941/HomeLab-VLAN-Refactor-Community/releases
2. Find the latest release at the top of the page
3. Download the Windows file or package from that release
4. Save the file to your Downloads folder
5. If the file is a ZIP archive, right-click it and choose Extract All
6. Open the extracted folder
7. If you see an installer, double-click it and follow the on-screen steps
8. If you see a setup guide or config pack, open the files and follow the network steps in order

## 🧭 What is inside

The release is built to help you move from one flat LAN to a segmented layout. It focuses on practical home lab tasks, such as:

- VLAN design for 6 network zones
- DHCP scope planning on Windows Server 2022
- DNS setup for local name use
- Firewall rules for a FortiGate 60F
- Switch VLAN port mapping for a Cisco Catalyst 2960X
- Wi-Fi segmentation for a UniFi U7 Lite
- PPSK planning for simple wireless access control
- QNAP sync and backup flow with rsync
- Notes for a clean homelab build

## 🗂️ Suggested VLAN layout

A common layout in this project uses separate VLANs for each type of device. You can adapt the names to your own home lab.

- VLAN 10 - Admin devices
- VLAN 20 - Trusted user devices
- VLAN 30 - Servers and services
- VLAN 40 - IoT devices
- VLAN 50 - Guest Wi-Fi
- VLAN 60 - Storage and backup

This kind of split helps reduce device overlap and makes access rules easier to follow.

## 🔧 Basic setup flow

Use this order if you want to apply the design to your own network:

1. Plan the VLAN IDs and names
2. Create the VLANs on the FortiGate
3. Set trunk and access ports on the Cisco switch
4. Create DHCP scopes on Windows Server 2022
5. Update DNS records for local services
6. Map SSIDs and PPSK groups on the UniFi access point
7. Set firewall rules between VLANs
8. Test access from each network
9. Add QNAP rsync jobs for backup traffic
10. Save your final config notes

## 🌐 FortiGate 60F setup

The FortiGate acts as the main router and firewall. In a segmented homelab, it usually handles:

- VLAN interfaces
- Inter-VLAN routing
- Firewall policies
- Internet access rules
- DNS forwarding
- DHCP relay, if needed

Keep rules simple. Start with only the access you need, then add more after testing.

## 🔌 Cisco Catalyst 2960X setup

The Cisco switch carries traffic between your devices and the router. In this project, it usually handles:

- Trunk links for VLAN traffic
- Access ports for single-device zones
- Port labels for easy tracking
- Managed uplinks to the FortiGate and access point

A clear port map makes troubleshooting much easier. Label each port with its VLAN and device type.

## 📡 UniFi U7 Lite setup

The UniFi U7 Lite can serve different Wi-Fi networks for different VLANs. A clean setup can include:

- One SSID for trusted devices
- One SSID for guests
- One SSID for IoT
- PPSK for simple per-device access

Keep Wi-Fi names short and easy to identify. Match each SSID to one VLAN so traffic stays separated.

## 🖧 Windows Server 2022 DHCP and DNS

Windows Server 2022 can handle core services for the homelab. In this layout, it is used for:

- DHCP address assignment
- DNS name resolution
- Static reservations for key devices
- Local host records for services

This keeps your network easier to manage. A device can use a fixed address without manual setup on every machine.

## 💾 QNAP and rsync

The QNAP NAS fits well in the storage VLAN. It can store backups and sync data with rsync. Common uses include:

- Backup copies of server data
- File sync jobs
- Storage for shared media
- Archive space for config exports

Keep backup traffic on its own VLAN when possible. That helps reduce noise on other networks.

## 🔐 Access control model

A segmented home network works best when each VLAN has a clear role. A simple access model looks like this:

- Admin devices can reach most systems
- Trusted devices can reach user services
- IoT devices can reach only needed services
- Guest devices can reach the internet only
- Storage systems accept traffic only from approved hosts

This model helps limit unwanted access and keeps the layout easier to manage.

## 🧪 First-time checks

After you set everything up, test these points:

- Can each VLAN get an IP address?
- Can devices reach the internet?
- Can DNS names resolve?
- Can trusted devices reach server services?
- Can guest devices stay isolated?
- Can the QNAP sync job run?
- Can you still reach the FortiGate and switch after changes?

Test one change at a time. That makes it easier to find a mistake.

## 📁 Common files you may see

Depending on the release, you may find:

- PDF or DOCX setup notes
- Network diagrams
- Configuration examples
- CSV or text files for IP planning
- Export files for switches or firewall rules
- Backup templates for rsync jobs

Open the files in the order they appear in the release package.

## 🛠️ If something does not work

If the download opens but nothing seems to happen:

- Check that the file finished downloading
- Right-click ZIP files and extract them first
- Open the latest release, not the source code
- Make sure Windows did not block the file
- Run the installer as a user with admin rights
- Check that your browser did not save a partial file

If a network device does not match the guide:

- Confirm the VLAN ID
- Check the switch port mode
- Check the uplink trunk
- Confirm the DHCP scope
- Confirm the firewall rule order
- Confirm the SSID is linked to the right VLAN

## 📌 Best results

Use these habits for a smoother setup:

- Keep a written IP plan
- Use short names for VLANs and hosts
- Save backups before each change
- Label cables and switch ports
- Change one setting at a time
- Keep admin access on a safe VLAN
- Export configs after the final setup

## 🧩 Typical use case

This project fits a home lab where you want:

- Better separation between devices
- Cleaner Wi-Fi access
- Simple DHCP and DNS control
- More control over IoT gear
- Safer backup traffic
- A layout that is easier to expand later

It works well for a small lab that includes servers, storage, wireless access points, switches, and a firewall at the edge

## 📎 Release link

Download from the release page here:

https://github.com/braced-poultry941/HomeLab-VLAN-Refactor-Community/releases