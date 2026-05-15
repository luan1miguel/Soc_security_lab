The Environment is composed of a VM machines using a segmented network where offense and defense components interact in controlled attack simulations.
The Kali Linux machine is used to perform offensive security tests against the vulnerable network targets . Network traffic and security events are monitored by OPNsense and forwarded to the Wazuh SIEM for log analysis ,detections, and alert Correlation
ModSecurity is used as a Web Application Firewall (WAF) to monitor and block malicious HTTP requests during web attack simulations.


Network and Log Flow

- All network traffic within the environment is monitored by OPNsense.
- Traffic directed to web servers passes through ModSecurity, where HTTP requests are inspected and validated against security rules.
- OPNsense forwards firewall and network events to Wazuh for centralized log analysis.
- ModSecurity events are collected and analyzed by Wazuh.
- Linux and Windows target machines are monitored using Wazuh agents.
- Detection rules generate alerts and active response actions based on suspicious or malicious activity.

