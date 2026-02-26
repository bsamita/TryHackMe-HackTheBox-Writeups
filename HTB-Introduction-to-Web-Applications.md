# HackTheBox: Introduction to Web Applications

**Platform:** HackTheBox Academy | **Module:** [Introduction to Web Applications](https://academy.hackthebox.com/achievement/2127127/75)  
**Author:** Brown Samita | **Date:** February 26, 2026  
 
**Certificate:** [View Achievement](https://academy.hackthebox.com/achievement/2127127/75)

---

## Overview

Web applications are at the core of almost every modern service — from banking portals and email platforms to social media and e-commerce. Unlike desktop applications, web apps run on remote servers and are accessed through a browser, making them universally accessible but also widely targeted by attackers.

This module provides a foundational understanding of how web applications are structured, what technologies power them, and the key security risks they face — building enough knowledge of the web stack to confidently approach offensive web application assessments.

**Key concepts covered:**
- Front-end technologies and their security implications (HTML Injection, XSS, CSRF)
- Back-end components: web servers, databases, frameworks, and APIs
- GET parameter manipulation and SQL Injection fundamentals
- OWASP Top 10 for Web Applications

**Tools & techniques used:** Browser DevTools • XSS payloads • GET parameter manipulation • `alert(document.cookie)`

---

## 1. Front-End Technologies

### 1.1 HTML, CSS, and JavaScript

Every web page is built from three core technologies: HTML structures the content, CSS controls the visual presentation, and JavaScript adds interactivity and dynamic behaviour. Understanding how these work together — and how they can be abused — is essential for web security.

### 1.2 HTML Injection

HTML Injection occurs when user-supplied input is rendered directly in the page without sanitisation. An attacker can inject arbitrary HTML tags to alter page content, redirect users, or craft convincing phishing pages hosted on a legitimate domain.

**Lab finding:** Hardcoded credentials discovered in an HTML comment — a common developer mistake where test credentials (`admin:HiddenInPlainSight`) were committed to source code and never removed before deployment.

> **Real-world relevance:** Source code review and browser DevTools inspection are standard SOC and vulnerability assessment techniques. Credentials left in HTML comments have been the root cause of real enterprise breaches.

---

## 2. Cross-Site Scripting (XSS)

### 2.1 What is XSS?

Cross-Site Scripting (XSS) allows an attacker to inject malicious JavaScript into a web page that is then executed by other users' browsers. It exploits the browser's trust of content served from a legitimate site.

| Type | Description |
|------|-------------|
| Reflected XSS | Payload is reflected off the server in an immediate response |
| Stored XSS | Payload is stored in the database and served to all users |
| DOM-based XSS | Payload executes through client-side JavaScript manipulation |

### 2.2 HTML Injection via XSS Payload

**Payload used:**
```html
<a href="http://www.hackthebox.com">Click Me</a>
```

Submitted to a name input field. Because the application did not sanitise input before rendering, the browser interpreted it as a real HTML link and displayed **"Click Me"** as a clickable hyperlink rather than raw text.

### 2.3 Cookie Theft via XSS

**Payload used:**
```javascript
<script>alert(document.cookie)</script>
```

By injecting this script into the vulnerable page, the browser executed the JavaScript in the context of the page and displayed the stored cookie value.

**Cookie retrieved:** `XSSisFun`

This demonstrates how XSS can be leveraged to steal session tokens and hijack authenticated sessions — one of the most impactful real-world consequences of XSS vulnerabilities.

> **Real-world relevance:** Session hijacking via XSS-stolen cookies is a common attack vector against web applications. A SOC analyst monitoring for this would look for unusual JavaScript in user input fields and unexpected outbound requests to attacker-controlled domains.

---

## 3. Back-End Components & APIs

### 3.1 Web Servers, Databases, and Frameworks

The back-end of a web application consists of several layers:

| Layer | Examples | Role |
|-------|----------|------|
| Web Server | Apache, Nginx, IIS | Receives and routes HTTP requests |
| Server-side Language | PHP, Python, Node.js | Processes application logic |
| Database | MySQL, PostgreSQL, MongoDB | Stores persistent data |

The interaction between these components is where many critical vulnerabilities originate.

### 3.2 GET Request Parameter Manipulation

**Request tested:**
```
GET /index.php?id=0
GET /index.php?id=1
```

By incrementing the `id` parameter, the application returned user data directly from the database. Setting `id=1` revealed the username of the first account: **`superadmin`**.

This type of vulnerability — where user-controlled input is passed directly to a database query — is the foundation of **SQL Injection attacks**, which appear in the OWASP Top 10 as one of the most critical web application risks.

> **Real-world relevance:** Unsanitised URL parameters are a primary attack surface for SQL injection. In a SOC context, detecting sequential parameter enumeration in web server logs (`?id=0`, `?id=1`, `?id=2`...) is an indicator of active reconnaissance or exploitation.

---

## 4. OWASP Top 10

The OWASP Top 10 provides a practical framework for identifying vulnerabilities and prioritising remediation in real-world applications.

| # | Category | Description |
|---|----------|-------------|
| 1 | Broken Access Control | Users accessing data or functions beyond their permissions |
| 2 | Cryptographic Failures | Sensitive data exposed due to weak or absent encryption |
| 3 | Injection | SQL, OS, and LDAP injection through unsanitised input |
| 4 | Insecure Design | Architectural flaws leading to vulnerable systems |
| 5 | Security Misconfiguration | Default settings, open cloud storage, verbose error messages |
| 6 | Vulnerable & Outdated Components | Libraries or frameworks with known CVEs |
| 7 | Auth & Session Failures | Weak session management and credential handling |
| 8 | CSRF | Tricking authenticated users into performing unwanted actions |

---

## 5. Conclusion & Key Takeaways

This module provided a comprehensive introduction to web application architecture and security fundamentals — building the foundation required to move into offensive web application testing.

| Takeaway | Detail |
|----------|--------|
| Input sanitisation is critical | HTML Injection and XSS both exploit unsanitised user input |
| Source code reveals secrets | Developer comments and hardcoded credentials are real attack vectors |
| Parameters are an attack surface | Unsanitised GET/POST parameters can expose database records |
| XSS impact is high | Cookie theft via XSS enables full session hijacking |
| OWASP Top 10 is a practical guide | Use it to structure both assessments and remediation priorities |

---

## Module Completion

✅ Module successfully completed on HackTheBox Academy  
🏆 Certificate: [https://academy.hackthebox.com/achievement/2127127/75](https://academy.hackthebox.com/achievement/2127127/75)
