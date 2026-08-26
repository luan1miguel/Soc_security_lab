# Web Enumeration Detection Scenario

## Objective

The objective of this scenario is to detect reconnaissance activity performed through web enumeration using common tools and scripts.

This simulation focuses on identifying suspicious enumeration behavior against web application assets through web server and firewall logs, combined with SIEM correlation rules.

---

## Attack Simulation

The attacker machine is located in a permitted network segment and is allowed to communicate with the target environment through the firewall.

The main objective of the attacker is to enumerate web paths and discover new application functionality that may be useful for further exploitation, without being detected by the SOC team.

In this scenario, tools such as FFUF, Dirb and Feroxbuster will be used. Other tools or custom scripts may be tested in future iterations to evaluate and improve the detection logic.

The detection logic is designed to identify enumeration behavior rather than relying exclusively on tool-specific signatures.
---

