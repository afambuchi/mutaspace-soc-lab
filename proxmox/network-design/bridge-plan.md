# Bridge Plan

##Learning Goal
A Proxmox bridge is a virtual switch. VMs connect to bridges. The firewall VM controls how traffic moves between networks.

Bridge	Role	Purpose
vmbr0	Management / WAN	Proxmox access and firewall WAN
vmbr1	Blue SOC network	Wazuh, Windows, Linux, learner systems
vmbr2	Red / untrusted network	Kali and untrusted VM