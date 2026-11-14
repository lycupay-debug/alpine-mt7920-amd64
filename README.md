# Alpine MT7920 Router Image

Alpine Linux 3.24.4 x86_64 writable router disk image for UEFI virtual machines, configured for a MediaTek MT7920 Wi-Fi WAN and E1000 LAN.

## Release assets

- `alpine-router-3.24.4-x86_64.raw.gz` — gzip-compressed 8 GiB RAW disk image. Decompress before attaching it to a VM.
- `alpine-router-3.24.4-x86_64.vmdk` — monolithic sparse VMDK disk image.
- `SHA256SUMS` — hashes for distributed assets and the decompressed RAW image.
- `MANIFEST.txt` — build configuration and installed components.
- `BUILD-REPORT.txt` — validation evidence and remaining hardware checks.

## Image configuration

- Alpine Linux 3.24.4 x86_64; kernel `6.18.63-0-lts`
- UEFI / GPT; 8 GiB virtual disk; ext4 root filesystem mounted read-write
- WAN: MediaTek MT7920 PCI passthrough, `mt7921e`, interface `wlan0`
- LAN: E1000, `eth0`, `192.168.100.1/24`
- DHCP range: `192.168.100.100`–`192.168.100.200`
- IPv4 forwarding, iptables firewall, conntrack and NAT
- dnsmasq, wpa_supplicant, OpenSSH and OpenRC startup
- Upstream Wi-Fi gateway is obtained dynamically by DHCP; expected upstream gateway in the target network is `192.168.1.1`.

No Wi-Fi SSID or passphrase is embedded. Configure `/etc/wpa_supplicant/wpa_supplicant.conf` on first boot. See the network configuration instructions below.

## First boot

1. Decompress the RAW image if using it: `gzip -dk alpine-router-3.24.4-x86_64.raw.gz`.
2. Attach the image to a UEFI VM. Pass through the physical MT7920 device and provide an E1000 virtual NIC for LAN.
3. Configure the Wi-Fi credentials in `/etc/wpa_supplicant/wpa_supplicant.conf`:

   ```conf
   network={
       ssid="YOUR_SSID"
       psk="YOUR_PASSPHRASE"
       key_mgmt=WPA-PSK
   }
   ```

4. Protect the configuration and restart supplicant: `chmod 600 /etc/wpa_supplicant/wpa_supplicant.conf && rc-service wpa_supplicant restart`.
5. Change the initial root password after first access. The image enables SSH password authentication for initial LAN-side management; the firewall does not permit WAN-side inbound SSH.

The host-side TAP interface is not created inside this guest. For the documented FreeBSD host layout, configure the host TAP separately as `tap10` with `192.168.100.2/24`.

## Verification status

QEMU TCG tests passed for UEFI boot, GPT/ESP, ext4 read-write persistence, `apk` package persistence, E1000 LAN, DHCP, DNS forwarding with a simulated upstream resolver, and isolated forwarding/conntrack/MASQUERADE. VMDK and QCOW2 image integrity checks passed.

Physical MT7920 passthrough, firmware initialization on the actual card, WPA authentication, WAN DHCP and real Internet access have **not** yet been verified. QEMU test hardware did not include the physical MT7920. See `BUILD-REPORT.txt` for details.

## Source firmware

The MT7920 firmware pair was taken from the official `linux-firmware` repository at commit `87414f9ba8f875cb733281652474bccf93313e2f`. See `MANIFEST.txt` for file names and kernel module details.
