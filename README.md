Alpine Router Image
===================

Release: Alpine Linux 3.24.4 x86_64
Build date: 2026-11-15
Virtual disk size: 8 GiB
Boot: UEFI / GPT
Root filesystem: ext4, persistent read-write
Install mode: Alpine sys

Artifacts
---------
alpine-router-3.24.4-x86_64.raw
  Primary RAW disk image. Use directly as a disk image with a compatible hypervisor.

alpine-router-3.24.4-x86_64.vmdk
  Monolithic sparse VMDK conversion of the same RAW image.

SHA256SUMS
  SHA256 hashes for RAW and VMDK.

SOURCE-AND-BACKUP-SHA256SUMS
  SHA256 hashes for the Alpine installer ISO, MT7920 firmware files and QCOW2 backup.
  Paths in this file are relative to /var/tmp/alpine-router-build.

Network layout
--------------
WAN:
  MediaTek MT7920 PCI passthrough
  Linux interface: wlan0
  Driver family: mt7921e
  WPA supplicant config: /etc/wpa_supplicant/wpa_supplicant.conf
  DHCP client starts after WPA association.
  The WAN default route and DNS servers are learned from DHCP.
  Expected upstream gateway confirmed by user: 192.168.1.1

LAN:
  E1000 virtual NIC
  Alpine interface: eth0
  Alpine address: 192.168.100.1/24
  DHCP pool: 192.168.100.100 - 192.168.100.200
  LAN DNS server: 192.168.100.1
  LAN default gateway offered to clients: 192.168.100.1

FreeBSD host side:
  Expected TAP interface: tap10
  Expected host address: 192.168.100.2/24
  The host-side TAP is not created inside this image.

Wi-Fi credentials
-----------------
No SSID or passphrase is embedded. This is intentional: credentials were not available
as confirmed values in the source notes and were not guessed.

On the router, add a network block to /etc/wpa_supplicant/wpa_supplicant.conf:

network={
    ssid="YOUR_SSID"
    psk="YOUR_PASSPHRASE"
    key_mgmt=WPA-PSK
}

Then run:
  chmod 600 /etc/wpa_supplicant/wpa_supplicant.conf
  rc-service wpa_supplicant restart

The action hook starts udhcpc on wlan0 after WPA association and updates
/etc/resolv.conf from the upstream DHCP lease. dnsmasq on eth0 uses that file
for upstream DNS forwarding.

Management and security
----------------------
- OpenSSH is enabled.
- Root password authentication is enabled for initial management.
- IPv4 INPUT policy is DROP; LAN eth0 is trusted/allowed; WAN inbound SSH is not allowed.
- IPv4 forwarding is enabled.
- FORWARD default policy is DROP; eth0 -> wlan0 is allowed; established return traffic is allowed.
- wlan0 egress is masqueraded.
- Initial root password is stored separately on the build server at:
  /var/tmp/alpine-router-build/credentials/root-password.txt
  It is not included in this README, the image manifest, or the Notion build log.
- Change the initial root password after first access.

Validation boundaries
---------------------
Verified in QEMU TCG:
- UEFI boot from the installed image.
- GPT, EFI System Partition and ext4 root partition.
- Root filesystem mounted ext4 rw.
- apk package installation and persistence after reboot.
- E1000 LAN interface, static address and SSH access from LAN.
- dnsmasq DHCP lease delivery and DNS forwarding when a simulated WAN resolver exists.
- Linux forwarding, conntrack and MASQUERADE in an isolated virtual-interface test.
- OpenRC persistence for networking, iptables, dnsmasq and sshd.

Not verified in QEMU:
- Physical MT7920 PCI passthrough.
- Actual MT7920 firmware loading on the laptop.
- WPA authentication against the user's Wi-Fi.
- Actual WAN DHCP lease and Internet access on the physical laptop.

QEMU does not contain the user's physical MT7920 device. These final hardware checks
must be performed after attaching the card to the user's FreeBSD/bhyve environment.
