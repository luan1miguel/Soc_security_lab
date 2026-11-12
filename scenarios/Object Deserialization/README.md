## Object Deserialization scenario

web application :

<img width="1919" height="548" alt="image" src="https://github.com/user-attachments/assets/405db46c-11c8-4c91-820d-c69a08604b3b" />

#Objective

The objective of this scenarios is exploit and analyze a object deserialization on python web application 

the python application use base64 to encode the object this is not a proper secure defense and are easy to decode and undestand the object by a attacker, the picke_loads , due with a insecure handling of a user input the application are vulnerable to CWE-502: Deserialization of Untrusted Data and yet a RCE.

This simulation focuses on the offensive and defensive aspects of the attack lifecycle, including vulnerability identification, exploitation, WAF monitoring, SIEM correlation.

# Vulnerable application

the application has a simple login page , and when user make a sing in the application set a cookie to a user the cookie has encoded in a simple base64 ,
