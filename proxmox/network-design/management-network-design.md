# Management Network Design

## Purpose
The Proxmox management IP is how I access the server. If I choose the wrong network settings, I may not be able to reach the web dashboard.

| Item               | Recommendation                   |
| ------------------ | -------------------------------- |
| Hostname           | `pve-mutaspace-01`               |
| Proxmox role       | Official SOC lab host            |
| Management network | Your normal wired LAN            |
| Management IP      | Static IP on your router network |
| DNS                | Router or trusted DNS            |
| Time zone          | America/Chicago                  |
