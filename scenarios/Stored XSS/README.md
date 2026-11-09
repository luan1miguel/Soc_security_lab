# Stored XSS Scenario

<img width="1919" height="926" alt="image" src="https://github.com/user-attachments/assets/a3f3f163-5775-4775-93be-2dd646b78ca2" />

# Objective

The objective of this scenario is to analyze and exploit a Stored Cross-Site Scripting (XSS) vulnerability and demonstrate its potential impact on a web application.

The application allows users to submit comments through a POST request. The submitted data is stored by the application and later displayed on the website without proper output encoding or sanitization. This allows an attacker to inject malicious content that can be executed when other users access the affected page.

This simulation focuses on both the offensive and defensive aspects of the attack, including vulnerability identification, exploitation, WAF monitoring, SIEM correlation, detection engineering, and the potential impact of Stored XSS.

# Vulnerable Application

The following screenshots show how the web application receives two parameters: Author and Body.

The application stores the submitted comments and displays them to other users in the comments section. However, the user-controlled input is inserted into the HTML response without proper output encoding.

As a result, malicious JavaScript submitted through these parameters can be executed in the browser of users who access the affected page.

Vulnerable parameters: Author and Body
HTTP method: POST
Vulnerability type: Stored XSS

<img width="580" height="424" alt="image" src="https://github.com/user-attachments/assets/9e4dbf0c-3fc1-4647-9051-97bd2afca31b" />

<img width="1131" height="938" alt="image" src="https://github.com/user-attachments/assets/365b430e-937c-43c1-afc2-3264a80d8222" />

# Attack Simulation

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






