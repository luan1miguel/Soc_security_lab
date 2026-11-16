## Object Deserialization scenario

web application :

<img width="1919" height="548" alt="image" src="https://github.com/user-attachments/assets/405db46c-11c8-4c91-820d-c69a08604b3b" />

#Objective

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

<img width="753" height="463" alt="image" src="https://github.com/user-attachments/assets/b0a76724-e2e4-41ae-806d-38f02861337c" />
