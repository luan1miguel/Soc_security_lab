# SSTI Detection and Exploitation Scenario

## Objective

The objective of this scenario is to analyze and exploit a Server-Side Template Injection (SSTI) vulnerability in a web application. 

The application uses a template engine to process user-supplied input and render dynamic content. Due to insecure handling of user input, the application becomes vulnerable to SSTI (OWASP Top 10 - A05:2025 Injection).

This simulation focuses on the offensive and defensive aspects of the attack lifecycle, including vulnerability identification, exploitation, firewall logging, WAF monitoring, SIEM correlation, and detection engineering.

## Environment

All machines below are running in a virtualized lab environment using VirtualBox:

* Wazuh (SIEM)
* OPNsense (Firewall)
* Kali Linux (Attacker Machine)
* Linux Target Machine (Victim)
* Flask Web Application (Vulnerable Service)

## Vulnerable Application 

The image below shows that web app receive a parameter 'name' who has pass directly to 'RENDER_TEMPLATE' without any sanitization . Because the user input is interpreted by the Jinja2 template engine instead of being rendered as plain text, an attacker can inject template expressions that are executed on the server. 

<img width="824" height="416" alt="image" src="https://github.com/user-attachments/assets/c2e81ab5-ec85-44e5-98dc-71f2ed323d5e" />

Vulnerable Parameter: name
HTTP Method: GET

## Attack Simulation

the application receive a data from user in this case by the parameter 'name' ( but this could be from cookie,endpoint and any other way ) and reflect on site , the attacker use a wappalyzer to identifies whats technologies the application use in this case only python, so they maybe things about a template(jinja2 or flask). 

<img width="1566" height="990" alt="image" src="https://github.com/user-attachments/assets/5d1d4368-5ddc-4672-87d7-8f3f50af224e" />

so using some basic payloads (https://portswigger.net/web-security/server-side-template-injection) they test and identify a SSTI. in this case a basic {{7*7}} make the template calculate and render 49 so is vulnerable!!
and we can confirm the exactly template using the payloads describe on postswigger site , so {{7*'7'}} confirms that the site use jinja2

<img width="1383" height="863" alt="image" src="https://github.com/user-attachments/assets/42c55ddd-b89b-43a3-8d44-24f21a70abc1" />
<img width="1401" height="987" alt="image" src="https://github.com/user-attachments/assets/4f4616c0-6f4c-48ff-a00e-8f7b309e56db" />


since we know the site use jinja2 we can try a Command Execution via __import__: If the application uses Jinja2 and the template has access to Python’s built-in functions, you can use __import__('os').system('ls') to execute commands.
the image bellow use the following payloads: {{ __import__('os').popen('ls').read() }}. and so we have a command injection 

<img width="1365" height="888" alt="image" src="https://github.com/user-attachments/assets/24432ce7-529e-432b-b800-2a667576f364" />
(For the purpose of this demonstration, the request was sent directly to the application. WAF evasion techniques will be discussed in a dedicated section later in this document.)










  
