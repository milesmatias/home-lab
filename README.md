# Deploy Tailscale in a Proxmox LXC Using Helper Scripts

This guide shows how to deploy Tailscale inside a Proxmox LXC using the Proxmox VE Helper Scripts / Community Scripts project. The goal is to create a small, lightweight Tailscale container that can securely connect to your tailnet. You can also turn it into a subnet router to access your Proxmox host and other LAN services remotely without opening ports on your router.

## What You Are Building

```text
Remote Laptop/Phone
        |
        v
Tailscale Tailnet
        |
        v
Tailscale LXC on Proxmox
        |
        v
Optional Subnet Route: 192.168.1.0/24
        |
        v
Proxmox, Pi-hole, NAS, Router, and other LAN services
```

## Requirements

- Proxmox VE server with internet access
- Root shell access to the Proxmox host
- Tailscale account
- Basic understanding of your LAN subnet, for example `192.168.1.0/24`
- A client device with Tailscale installed for testing

## Step 1 - Create a Debian LXC

Run this from the **Proxmox VE host shell**:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/debian.sh)"
```

Suggested settings:

| Setting | Recommendation |
|---|---|
| Hostname | `tailscale-lxc` |
| CPU | 1 core |
| RAM | 512 MB |
| Disk | 2-8 GB |
| Type | Unprivileged LXC |
| IP | Static IP preferred if using subnet routing |

## Step 2 - Add Tailscale to the LXC

Run this from the **Proxmox VE host shell**:

```bash
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/addon/add-tailscale-lxc.sh)"
```

Select the LXC you created. After the script finishes, reboot the container.

```bash
pct reboot <CTID>
```

## Step 3 - Authenticate Tailscale

Enter the LXC:

```bash
pct enter <CTID>
```

Bring Tailscale online:

```bash
tailscale up -ssh
```

Open the login URL, sign in, and authorize the device.

Verify:

```bash
tailscale status
tailscale ip -4
ip addr show tailscale0
```

## Optional - Configure the LXC as a Subnet Router

Use this if you want remote Tailscale devices to access your home LAN, such as Proxmox at `https://192.168.1.5:8006`.

Enable IP forwarding inside the LXC:

```bash
echo 'net.ipv4.ip_forward = 1' | tee -a /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | tee -a /etc/sysctl.d/99-tailscale.conf
sysctl -p /etc/sysctl.d/99-tailscale.conf
```

Advertise your LAN route. Replace this subnet with your actual LAN subnet:

```bash
tailscale up --advertise-routes=192.168.1.0/24 --accept-dns=false
```

Then approve the route in the Tailscale admin console:

1. Go to **Machines**
2. Select the Tailscale LXC
3. Edit route settings
4. Enable the advertised subnet route
5. Save

On some clients, enable route usage. Linux example:

```bash
sudo tailscale set --accept-routes
```

## Test

From a remote Tailscale device:

```bash
ping <tailscale-lxc-ip>
ping 192.168.1.1
```

Then open:

```text
https://<proxmox-lan-ip>:8006
```

Example:

```text
https://192.168.1.5:8006
```

## Troubleshooting

```bash
ls -l /dev/net/tun
tailscale status
systemctl status tailscaled
journalctl -u tailscaled --no-pager -n 50
```

Common fixes:

- Reboot the LXC after running the helper script.
- Confirm the LXC has internet access.
- Confirm the advertised route matches your real LAN subnet.
- Approve the subnet route in the Tailscale admin console.
- Make sure the remote client accepts subnet routes.
- Do not open router ports for this setup.

## Sources

- Proxmox VE Helper Scripts / Community Scripts: https://github.com/community-scripts/ProxmoxVE
- Tailscale LXC docs: https://tailscale.com/docs/features/containers/lxc/lxc-unprivileged
- Tailscale subnet router docs: https://tailscale.com/docs/features/subnet-routers
