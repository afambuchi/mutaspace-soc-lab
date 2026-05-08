# VM Resource Plan

## Learning Goal
A VM does not just need disk space. It also needs CPU and RAM while running. A 64 GB host can support a full lab, but not every VM needs to be powered on all the time.

The plan for 64 GB RAM:

VM	Purpose	vCPU	RAM	Disk
fw-01	Firewall/router	2	2 GB	20 GB
wazuh-01	SIEM	4-6	12-16 GB	150-250 GB
sensor-01	Suricata	2-4	4 GB	60 GB
win-client-01	Windows endpoint	2-4	6-8 GB	80-120 GB
ubuntu-app-01	Linux target	2	2-4 GB	40-80 GB
kali-01	Attack simulation	2-4	4-8 GB	80 GB
untrusted-01	Trust-boundary VM	2	2-4 GB	40-60 GB
nlp-01	Phishing/NLP research	4	8-12 GB	100 GB
dc-01	Active Directory, later	2-4	4-8 GB	80 GB

