# PortSwigger Web Security Academy — Complete Lab Walkthroughs in Arabic (273/273)

[![PortSwigger Academy](https://img.shields.io/badge/PortSwigger%20Academy-273%2F273%20Completed-ff6633?style=for-the-badge&logo=burpsuite&logoColor=white)](https://portswigger.net/web-security)
[![Language](https://img.shields.io/badge/Language-Arabic%20(English%20Tech%20Terms%20%26%20Payloads)-blue?style=for-the-badge)](#)

A comprehensive, structured collection of detailed technical walkthroughs and remediation notes written in **Arabic** (retaining technical terms, HTTP requests, and payloads in **English**), covering **all 273 labs** available on the **PortSwigger Web Security Academy** at the time of publication.

---

## Overview

This repository serves as both a personal knowledge base and a practical Arabic reference for the cybersecurity community across **Apprentice**, **Practitioner**, and **Expert** levels. Every lab note in this repository follows a consistent structure:

1. **Goal (`الهدف`):** Clear definition of the lab objective and attack scenario.
2. **Exploitation Steps (`خطوات الحل`):** Step-by-step methodology, HTTP requests/responses, payloads, and custom scripts used to solve the lab.
3. **Remediation (`ترقيع الثغرة`):** Practical mitigation strategies and security best practices to prevent the vulnerability in production environments.

---

## Repository Structure

All labs are organized into topic-based directories matching the official PortSwigger Academy curriculum:

```text
├── Access-control-vulnerabilities/
├── API-testing/
├── Authentication/
├── Business-logic-vulnerabilities/
├── Clickjacking/
├── Cross-origin-resource-sharing-CORS/
├── Cross-site-request-forgery-CSRF/
├── Cross-site-scripting-XSS/
├── DOM-based-vulnerabilities/
├── Essential-skills/
├── File-upload-vulnerabilities/
├── GraphQL-API-vulnerabilities/
├── HTTP-Host-header-attacks/
├── HTTP-request-smuggling/
├── Information-disclosure-vulnerabilities/
├── Insecure-deserialization/
├── JWT-attacks/
├── NoSQL-injection/
├── OAuth-authentication/
├── OS-command-injection/
├── Path-traversal/
├── Prototype-pollution/
├── Race-conditions/
├── Server-side-request-forgery-SSRF/
├── Server-side-template-injection-SSTI/
├── SQL-injection/
├── Web-cache-deception/
├── Web-cache-poisoning/
├── Web-LLM-attacks/
├── WebSockets/
└── XML-external-entity-XXE-injection/
```

_(Note: A small number of multi-category labs appear in their secondary topic folder with a direct cross-reference link to the primary full writeup to avoid content duplication.)_

## How to Use This Repository

- **Browse by Vulnerability:** Navigate into any topic folder above to view all labs under that category.

- **Direct Lab Access:** Each markdown file starts with a direct hyperlink to the corresponding lab on PortSwigger Academy so you can practice alongside the walkthrough.

- **Quick Search:** Use GitHub's repository search (`t` shortcut) to jump directly to any lab by name.

## Disclaimer & Notes

- **Lab Count Scope:** The total count of **273 labs** reflects the complete curriculum available on PortSwigger Web Security Academy at the time this repository was published and may increase as new labs are added in the future.

- **Educational Purpose Only:** All techniques, payloads, and exploits documented here are intended strictly for educational purposes and authorized security testing.

* **Remediation Disclaimer:** The remediation sections (`ترقيع الثغرة`) represent personal research and best-effort analysis rather than official PortSwigger solutions; therefore, technical inaccuracies or incomplete edge cases may exist. Additionally, while the steps are written in an imperative tone ("يجب..."), they are not necessarily meant to be implemented all at once—several points represent alternative approaches or defense-in-depth layers, and applying even a subset of them may be sufficient depending on the application's architecture.

- **Typographical Note:** These notes were written organically while solving the labs as a personal study reference, so minor spelling or grammatical mistakes in Arabic or English are expected.

## Contact

You can connect with me on [X (Twitter)](https://x.com/RivenX173).