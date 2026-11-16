## Object Deserialization scenario

web application :

<img width="1919" height="548" alt="image" src="https://github.com/user-attachments/assets/405db46c-11c8-4c91-820d-c69a08604b3b" />


# Objective

The objective of this scenario is to exploit and analyze an insecure object deserialization vulnerability in a Python web application.

This simulation focuses on the offensive and defensive aspects of the attack lifecycle, including vulnerability identification, exploitation, WAF monitoring, and SIEM correlation.


# Vulnerable application

The application has a simple login page. When a user signs in, the application generates a cookie containing serialized user information.

The cookie is encoded using Base64, which allows its contents to be easily decoded and inspected. However, the application does not properly protect the integrity of this client-side data before processing it.

<img width="835" height="344" alt="image" src="https://github.com/user-attachments/assets/18444ad9-5e8b-47d3-9668-74f2dfe3342e" />

In a second implementation, some of the cookie data is reflected in the web application. This provides additional information about how the application processes the serialized object.

The cookie also contains a role field. An attacker may attempt to modify this value from user to admin, potentially gaining access to features restricted to administrative users.

<img width="1130" height="583" alt="image" src="https://github.com/user-attachments/assets/9d41c2ca-29a5-4b27-b4e8-1a0cd74c1b84" />

However, the most critical issue is the application's use of pickle.loads() to deserialize the cookie contents.

Python Pickle can invoke object-specific methods during deserialization. If an attacker can supply a malicious serialized object, this behavior may be abused to execute arbitrary code on the server.

This makes the deserialization process a potential RCE attack vector.

<img width="553" height="285" alt="image" src="https://github.com/user-attachments/assets/bb7952a1-8633-4065-9466-b361b9ae3db2" />


# Attack Simulation

After logging into the application, an attacker may start enumerating available features, input fields, cookies, and other application behavior to understand how the application works.

In this case, the application reflects the logged-in user's information on the page.

<img width="1917" height="927" alt="image" src="https://github.com/user-attachments/assets/94f001e1-3379-4c8f-8e1a-5e1521ec666d" />

Using browser DevTools or other tools, the attacker can inspect and modify the session cookie.

<img width="1919" height="270" alt="image" src="https://github.com/user-attachments/assets/ceebcfa6-437e-4202-9625-01b397ff2341" />

After decoding the cookie from Base64, the attacker can inspect some of its contents, including readable information and other encoded data.

Based on the previous enumeration, the attacker identifies that the application is built with Python. The cookie structure also suggests that Python's Pickle serialization format may be involved.

Using a simple Python script, the attacker tests the cookie contents and confirms that it can be deserialized using Pickle.

<img width="809" height="161" alt="image" src="https://github.com/user-attachments/assets/bf334823-8b56-47c9-97f9-5c164c025f3d" />

The attacker can then modify the serialized object using another Python script and replace the original cookie through browser DevTools.

<img width="573" height="93" alt="image" src="https://github.com/user-attachments/assets/cdccef5b-8c90-4b6c-9818-a066cc1c0ca6" />

<img width="1918" height="928" alt="image" src="https://github.com/user-attachments/assets/a187811f-0247-4b34-b9d2-25c173a0b1a8" />

The attacker now controls the role field and can access functionality that would normally be restricted to administrative users. Depending on the application, this could result in significant privilege escalation and unauthorized access.

However, the attacker may attempt to achieve a greater impact by exploiting the application's insecure deserialization process.

To demonstrate this, a malicious serialized object is created using Python:

<img width="647" height="257" alt="image" src="https://github.com/user-attachments/assets/aa71a160-306e-4b76-bf4d-5116bf4c2581" />

After sending the modified object through the session cookie, the application deserializes it and executes the associated code. As a result, the file /tmp/pwned.txt is created on the web server.

<img width="842" height="93" alt="image" src="https://github.com/user-attachments/assets/53c7c707-d36c-41c0-bb75-aa8b154df7a3" />

This confirms that the vulnerability allows arbitrary code execution in the context of the web application process. An attacker could potentially use this access to perform further actions, depending on the application's privileges and server configuration.

# Detection and response

In this scenario, the object deserialization vulnerability can be exploited through a small number of HTTP requests. Since the attack does not necessarily generate a high volume of traffic, the existing SIEM correlation rules may not trigger any alerts.

So, what should the SOC team do in this situation?

WAF Monitoring: Ensure that the WAF rules are properly configured and updated. However, do not rely exclusively on the WAF to detect insecure deserialization.

Endpoint Monitoring: Monitor the web server for unexpected process execution, suspicious child processes, and unusual file creation or modification.

Log Analysis: Investigate application logs and correlate suspicious HTTP requests with endpoint events, especially when unexpected commands or file operations occur.

Server Hardening: Apply the principle of least privilege, restrict unnecessary permissions, and limit outbound network connections to reduce the potential impact of successful exploitation.

A low volume of HTTP requests does not necessarily mean that an attack is harmless or easy to detect. Endpoint telemetry and application monitoring are important additional sources of visibility.

# WAF Analys


The ModSecurity audit logs confirm that the HTTP requests were processed by the WAF, with the engine enabled and OWASP CRS 3.3.5 loaded.

However, no CRS rule matches or blocking actions were identified during the tested exploitation attempts.

The malicious serialized object was transmitted through the session cookie and processed by the application without being detected by the WAF. This demonstrates a limitation of relying on signature-based WAF inspection to identify insecure deserialization attacks.

The WAF was operational, but it did not identify or prevent the tested attack.

