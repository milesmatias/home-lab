Pi-hole Install Guide
Using Proxmox Helper Scripts + Unbound
Simple home-lab setup for DNS ad blocking, upstream DNS privacy, and basic network-wide filtering.
Example Environment Used
Proxmox VE node: proxmox | LXC ID: 101 | OS: Debian 13 | Pi-hole IP: 192.168.0.44 | Router/LAN: 192.168.0.0/24 | Pi-hole hostname: pihole

1. Overview
This guide documents a simple Pi-hole install using the Proxmox Community Helper Script. The helper script creates the LXC container, installs Pi-hole, and optionally installs Unbound. In this setup, Unbound was installed during the helper script process and configured as a DNS-over-TLS forwarding resolver.
•	Pi-hole filters DNS requests and blocks ads, trackers, and unwanted domains.
•	Unbound handles upstream DNS forwarding using DNS-over-TLS in this setup.
•	The router will hand out the Pi-hole IP address as the DNS server for the home network.
•	Client DNS flow: Devices -> Pi-hole -> Unbound -> encrypted upstream DNS provider.
2. Requirements
Item	Example / Notes
Proxmox VE	Installed and running. Example used PVE 9.1.9.
Network	Home LAN in the 192.168.0.0/24 range.
Static IP for Pi-hole	Example: 192.168.0.44.
Router access	Needed to set DHCP DNS to Pi-hole.
Browser access	Used to open http://192.168.0.44/admin.
3. Install Pi-hole with the Proxmox Helper Script
From the Proxmox shell, run the Pi-hole Proxmox Helper Script from the community-scripts site. Use the default settings unless you need to customize CPU, RAM, storage, or container ID.
# Run the Pi-hole helper script from the Proxmox shell
# Use the command provided by the Proxmox Community Scripts website.
In this install, the helper script used these default settings:
•	Container ID: 101
•	Operating System: Debian 13
•	Container Type: Unprivileged
•	Disk Size: 2 GB
•	CPU Cores: 1
•	RAM: 512 MiB
•	Network IP: 192.168.0.44
When prompted by the script, answer as follows:
Do you want to continue? y
Would you like to add Unbound? y
Configure Unbound as forwarding DNS server using DNS-over-TLS? y
Important Note
Answering yes to the DNS-over-TLS forwarding option means Unbound is not acting as a full recursive resolver. It forwards DNS queries upstream using encrypted DNS-over-TLS. This is still a good and simple home-lab setup.

4. Access the Pi-hole Dashboard
After the script completes, open the Pi-hole admin page in a browser:
http://192.168.0.44/admin
At first, the dashboard may show 0 queries and 0 active clients. That is normal until your router or devices are configured to use Pi-hole as DNS.
5. Configure Pi-hole Upstream DNS
Go to Pi-hole > Settings > DNS. Since Unbound was installed, Pi-hole should send DNS queries only to Unbound.
Uncheck public DNS providers such as:
•	Google
•	OpenDNS
•	Quad9
•	Cloudflare
•	Level3
•	Comodo
In Custom DNS servers, keep only:
127.0.0.1#5335
Remove other custom DNS servers such as:
1.0.0.1
149.112.112.11
Click Save & Apply.
6. Point the Router to Pi-hole
Log in to the router, go to LAN or DHCP settings, and set the DNS server handed out to clients as the Pi-hole IP address.
Primary DNS:   192.168.0.44
Secondary DNS: leave blank
Do Not Add a Secondary Public DNS Server
Do not set 8.8.8.8, 1.1.1.1, or another public DNS server as secondary DNS. Some devices may bypass Pi-hole if a secondary DNS server is available.

7. Renew DNS on Windows Clients
After changing router DNS settings, renew DNS on Windows so the computer starts using Pi-hole.
ipconfig /flushdns
ipconfig /release
ipconfig /renew
Test DNS resolution through Pi-hole:
nslookup google.com 192.168.0.44
Test ad-blocking behavior:
nslookup doubleclick.net 192.168.0.44
Then test what DNS server Windows is using by default:
nslookup google.com
The response should show the DNS server as Pi-hole, usually 192.168.0.44.
8. Recommended Blocklists
The default StevenBlack list works, but for a clean home setup, use HaGeZi Pro plus HaGeZi TIF. This gives strong everyday blocking without going overly aggressive.
Recommended blocklists to add:
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/pro.txt
https://raw.githubusercontent.com/hagezi/dns-blocklists/main/adblock/tif.txt
Optional default list already included by Pi-hole:
https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts
Recommended simple setup:
•	HaGeZi Pro
•	HaGeZi TIF
•	Optional: keep StevenBlack if you want extra overlap
After adding lists, update Gravity:
pihole -g
Or use the web UI: Tools > Update Gravity.
9. Quick Health Checks
Useful commands from inside the Pi-hole LXC:
pihole status
pihole restartdns
systemctl status unbound
dig google.com @127.0.0.1 -p 5335
Useful Proxmox LXC checks from the Proxmox host:
pct list
pct enter 101
10. Troubleshooting Notes
Dashboard shows 0 queries: Devices are not using Pi-hole yet. Set router DHCP DNS to 192.168.0.44 and renew client leases.
Websites are not loading: Temporarily disable blocking in Pi-hole, check upstream DNS, then test nslookup.
A specific website breaks: Check Query Log, then allowlist the blocked domain if needed.
Queries bypass Pi-hole: Remove secondary public DNS from router/client settings.
IPv6 not connected: Not a problem if your home network is IPv4-only. Keep IPv6 disabled unless you intentionally use it.
11. Final DNS Flow
Final home network DNS flow:
Client devices
   -> Router DHCP gives DNS 192.168.0.44
   -> Pi-hole filters DNS queries
   -> Unbound forwards upstream using DNS-over-TLS
   -> Encrypted upstream DNS provider
Final Result
Pi-hole is running at 192.168.0.44, Unbound is installed, upstream DNS is set to 127.0.0.1#5335, and the router should hand out 192.168.0.44 as the only DNS server.

 
Appendix: Copy/Paste Checklist
[ ] Run Pi-hole Proxmox Helper Script from Proxmox shell.
[ ] Use default LXC settings unless customization is needed.
[ ] Install Pi-hole.
[ ] Choose yes to install Unbound.
[ ] Choose yes for DNS-over-TLS forwarding if you want encrypted upstream forwarding.
[ ] Open http://192.168.0.44/admin.
[ ] Set Pi-hole upstream DNS to only 127.0.0.1#5335.
[ ] Remove other upstream DNS providers from Pi-hole.
[ ] Set router DHCP DNS to 192.168.0.44.
[ ] Leave secondary DNS blank.
[ ] Renew DNS lease on Windows clients.
[ ] Add HaGeZi Pro and HaGeZi TIF blocklists.
[ ] Run pihole -g or Tools > Update Gravity.
[ ] Verify queries appear in the Pi-hole dashboard.
