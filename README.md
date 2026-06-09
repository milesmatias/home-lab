Installing AdGuard Home on Proxmox VE Using Helper Scripts
Beginner-Friendly Step-by-Step Documentation

This guide walks you through installing AdGuard Home inside an LXC container on Proxmox VE using the community helper scripts. It is written for beginners and includes exact commands, setup steps, and troubleshooting tips.
1. Log in to Proxmox VE
Open your browser and go to your Proxmox web interface.

Example:
https://192.168.1.50:8006

Log in using your Proxmox username and password.
2. Open the Proxmox Shell
In the left sidebar, click:
Datacenter > Your Proxmox Node

Then click:
Shell

This opens the Proxmox command line.
3. Run the AdGuard Helper Script
Copy and paste the following command into the Proxmox shell:
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/ct/adguard.sh)"
4. Choose Install Settings
When prompted, select:
Default Settings

The helper script will automatically create the LXC container and install AdGuard Home.
5. Wait for Installation to Finish
Allow the script to complete. Do not close the shell window.

Once finished, the installer will display the AdGuard Home web interface address.
6. Open AdGuard Home
In your browser, navigate to:
http://ADGUARD-IP:3000

Example:
http://192.168.1.100:3000
7. Complete Initial Setup
Use the default beginner-friendly setup values:

Admin Interface: All interfaces
DNS Server: All interfaces
DNS Port: 53

Create your admin username and password.
8. Configure Upstream DNS Servers
Go to:
Settings > DNS Settings

Recommended DNS servers:
Cloudflare:
1.1.1.1
1.0.0.1

Google:
8.8.8.8
8.8.4.4

UPSTREAM DNS SERVERS
https://cloudflare-dns.com/dns-query 
https://dns.google/dns-query 
9. Point Your Router to AdGuard
Log into your router and locate the DNS settings.

Set the primary DNS server to the AdGuard Home IP address.

Example:
Primary DNS: 192.168.1.100
10. Test AdGuard
Open the AdGuard Dashboard and browse websites on your devices.

If you see DNS queries appearing, AdGuard is working correctly.
11. Reserve the AdGuard IP Address
In your router settings, create a DHCP reservation or static lease for the AdGuard container.

This prevents the IP address from changing.
12. Troubleshooting
If the AdGuard page does not load, make sure you are using:
http:// instead of https://

If ads are not blocking, verify your router DNS settings and reconnect devices to Wi-Fi.
13. Updating AdGuard Home
Updates are managed directly inside the AdGuard Home web interface.

Open the dashboard and apply updates when available.
Final Notes
Your setup should follow this structure:

Internet → Router → AdGuard Home on Proxmox → Your Devices

AdGuard Home filters DNS requests and blocks ads, trackers, and malicious domains across your network.

Created for Miles Matias Homelab Documentation
 
Router DNS Configuration Guide
This section explains how to configure your home router so every device on your network uses AdGuard Home for DNS filtering.
1. Find Your AdGuard Home IP Address
In Proxmox, click your AdGuard LXC container.

Go to:
Summary

Look for the IP address.

Example:
192.168.1.100
2. Log Into Your Router
Open a web browser and type your router IP address.

Common router addresses:
192.168.1.1
192.168.0.1
10.0.0.1

Enter your router username and password.
3. Locate DNS Settings
Every router looks different.

Look for sections such as:
Internet Settings
LAN Settings
DHCP Server
DNS Settings
Advanced Network Settings
4. Change DNS Servers
Replace the existing DNS servers with your AdGuard Home IP address.

Example:
Primary DNS: 192.168.1.100
Secondary DNS: Leave blank if possible

Leaving the secondary DNS blank helps prevent devices from bypassing AdGuard.
5. Save and Apply Changes
Click:
Save
Apply
or Reboot Router

Your router may restart after applying the settings.
6. Reconnect Your Devices
Disconnect and reconnect devices to Wi-Fi.

This forces them to obtain the new DNS settings from the router.
7. Verify AdGuard Is Working
Open the AdGuard Dashboard.

Browse websites from your phone or computer.

If DNS queries appear in the dashboard, the setup is working correctly.
Optional: Block Hardcoded DNS
Some devices use Google DNS or Cloudflare DNS directly.

Advanced users can create firewall rules on the router to redirect all DNS traffic back to AdGuard Home.

This is optional for beginners.
Updated with Router DNS Configuration Steps
