# FW 01 Design

## Learning Goal
The firewall VM is the gatekeeper. It separates normal lab systems from attack/untrusted systems and controls what traffic is allowed.

Firewall VM NIC plan:

Firewall NIC	Bridge	Role
NIC 1	vmbr0	WAN / management side
NIC 2	vmbr1	Blue SOC LAN
NIC 3	vmbr2	Red untrusted LAN

Suggested IP plan:

Network	Gateway
Blue SOC	10.20.0.1/24
Red Untrusted	10.30.0.1/24