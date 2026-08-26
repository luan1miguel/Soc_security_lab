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

### 1. Dirb Enumeration

a simple dirb command to start a enumeration this command will use a default wordlist and has a objetive locate any paths on site : dirb http://10.2.0.13  


<img width="577" height="600" alt="image" src="https://github.com/user-attachments/assets/49b6a2d2-fec0-40fe-b3f7-34dcf0620eb4" />

---

## Detection 

on the target machine i run a tcpdump command to capture the traffeg and to anylisses this in another point of view

sudo tcpdump -ni any host 10.4.0.5 -U -w web_enum.pcap

<img width="1918" height="989" alt="image" src="https://github.com/user-attachments/assets/528049eb-ebad-4ff7-8524-e497b8dab66e" />

in a simple analyses using tcpdump we identifies the malicious IP from kali "10.4.0.5" , we can confirme this because that IP make too many requisition to us in a short time, and that IP use requisitions to /paths ( like xml,xmls... this is usually) in a alfabhet method.
this comportament is not common and dispert our attention 

looking in firewall we confirm a lot of connection from 10.4.0.5 in a short space of time.

<img width="1919" height="987" alt="image" src="https://github.com/user-attachments/assets/68886c06-b975-4c05-a505-dd3fb9f35d94" />














