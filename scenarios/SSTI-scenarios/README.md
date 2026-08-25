<img width="1420" height="928" alt="image" src="https://github.com/user-attachments/assets/9e9a0e77-debf-4fe7-a85b-47c5c5a79a05" /># SSTI Detection and Exploitation Scenario

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

## Detection and response

in this case since the SSTI are explorable manually so only a few http request are send to the server, with no volume in this kind of attack in SIEM no alert is generated. 

so whats the SOC team has to do in this case ? 

1- make sure the WAF has the correctly rules are correctly implamented
2- monitoring the server and any commands out of usual, and deal with hardning the server 

## WAF Analisys 

During the assessment, the default OWASP CRS successfully blocked payloads associated with command execution. However, simple fingerprinting payloads (like {{7*7}} or {{7*'7'}}) were not considered malicious and therefore were allowed. This behavior allowed the attacker to confirm the existence of the SSTI vulnerability before attempting exploitation.

<img width="1565" height="983" alt="image" src="https://github.com/user-attachments/assets/a5bbda2c-474b-460a-b1f0-1d6a35cffef7" />

but when attacker try to execute any command on machine ( payload: {{ __import__('os').popen('ls').read() }}  ) the WAF correctly blocked this. and generade some logs but. like saiyng before this is only a single request,in this case the blocked has been sucessfull so the so the SIEM will registry only a single rule with level 2. 

<img width="911" height="284" alt="image" src="https://github.com/user-attachments/assets/7844722d-0ce2-4a25-8d2c-dc4da03937be" />

<img width="1919" height="1036" alt="image" src="https://github.com/user-attachments/assets/90497f77-a9e1-4f99-b809-3e411e699bd4" />

<img width="1145" height="919" alt="image" src="https://github.com/user-attachments/assets/a0fbade5-8e94-449d-b925-db0f2f716dfe" />

This illustrates an important challenge for SOC teams: not every successful detection should become a high-priority alert. Manual exploitation often generates only a few events, making traditional frequency-based correlation ineffective. 

## WAF bypass

Like any security control, a WAF has limitations and should not be considered the only line of defense. so the attacker can identifies the vulnerabilities and whats template are using on this web app.

The WAF relies on signatures and request analysis to identify malicious patterns. Understanding how these rules behave allows an attacker to evaluate possible weaknesses in the deployed policy.

the funcionallity send some kind of special characters? so the WAF maybe accept this kind of characters. this is a simple example but undestand that will be a good thing to make a payload to bypass the WAF 

in image below i use test excluding some part of a command injection payload and sent only (__import__('os').popen) this can be usefull to undestand the WAF , in this case the attacker is able to identifie then that WAF don't block the __import__ call or 'os' but block the popen() function 

<img width="1420" height="928" alt="image" src="https://github.com/user-attachments/assets/671db867-9121-402d-b307-3979a98eff33" />

Once the blocked pattern is identified, an attacker may attempt to modify the payload representation to determine whether the detection relies on static signatures or deeper semantic analysis.
using the following payload : {{ getattr(__import__('os'), 'po'+'pen')('ls').read() }} 

The getattr() function allows attribute access by name at runtime. By splitting the string 'popen' into 'po'+'pen', the static signature is broken at the HTTP request level. The WAF inspects the raw request and never sees the string popen — the concatenation is only resolved by the Python interpreter after the request is approved.

<img width="1918" height="994" alt="image" src="https://github.com/user-attachments/assets/c98308ff-72ee-4886-8829-1b3fc04e354a" />

In this laboratory, the web application can also be reached directly through an internal service port. This configuration exists exclusively for testing purposes and allows comparison between requests inspected by the WAF and requests sent directly to the application.

in this lab we can do this accessing the following ip 10.2.0.13:8080 ( we can discovery about this using a nmap scan)

<img width="1348" height="717" alt="image" src="https://github.com/user-attachments/assets/2501e32d-5c14-44ab-8bf3-bb7a56a31f14" />

This scenario focuses only on a small subset of WAF evaluation techniques. More advanced evasion methods were intentionally left outside the scope of this laboratory to keep the analysis focused on SSTI exploitation and defensive visibility.

## MITRE ATT&CK

**T1059 - Command and Scripting Interpreter**
https://attack.mitre.org/techniques/T1059/

The SSTI exploitation resulted in server-side code execution.
This behavior is mapped to T1059. No specific sub-technique was identified for the behavior demonstrated in this scenario.

## Mitigation

 in cases envolves a manual attack the mitigations methods will belongs to devoleps methods, WAF rules and hardning server.
 
 OWASP Top 10 2025 was used in this lab to understand and identify
 mitigation strategies for this type of vulnerability:

 https://owasp.org/Top10/2025/A05_2025-Injection/

 PortSwigger documentation was also used as a reference for
 understanding Server-Side Template Injection and its mitigations:

 https://portswigger.net/kb/issues/00101080_serversidetemplateinjection

### Development

- Never render user-controlled input directly into templates.
- Treat all user input as data, not executable template code.
- keep the development dependencies update
- Apply secure coding practices based on the OWASP Secure Coding Guidelines.

to Waf :

- monitoring all endpoint on application
-  Prevent direct access to backend services so every request passes through the WAF.
- Tune rules according to the application's expected behavior to reduce false positives.
- Keep the OWASP Core Rule Set updated.

### SIEM

- Monitor suspicious process execution initiated by the web server.
- Detect unexpected outbound network connections.
- Correlate WAF events with endpoint telemetry.
- Investigate abnormal privilege escalation attempts.

### Server Hardening

- Run the web application using a non-privileged user.
- Restrict outbound network connections to reduce the impact of reverse shells.
- Disable unnecessary services.
- Apply the principle of least privilege.
- Use strong authentication credentials and protect privileged accounts.

## Lessons Learned

- Fingerprinting is often enough to identify vulnerable technologies before exploitation.
- Successful exploitation may generate very little network noise.
- WAFs significantly increase the attacker's workload but should not replace secure application design.
- Small changes in payload representation can influence signature-based detections.
- Understanding defensive controls is an important part of offensive security.



















  
