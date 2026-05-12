# Unprotected Admin Functionality

> **Platform:** PortSwigger Web Security Academy **Category:** Access Control / Vertical Privilege Escalation **Difficulty:** Apprentice **Tool(s):** Browser only **Date:** 12/05/2026

---

## Overview

This lab demonstrates a vertical privilege escalation vulnerability where an admin panel is exposed without any authentication check. By reading the application's `robots.txt` file, I discovered the hidden admin endpoint and used it to delete a user — with no credentials, no exploitation tools, and no payload. Just a URL.

---

## Vulnerability Background

### What Is It?

**Unprotected admin functionality** is a type of broken access control — specifically vertical privilege escalation. It happens when an application exposes sensitive functions (like an admin panel) without enforcing who is allowed to reach them.

The assumption that _"users won't find it if we don't link to it"_ is not security. It's called security through obscurity, and it fails every time.

### How Does It Happen?

The root cause is simple: the developer built the admin functionality but forgot — or chose not — to add any server-side check that verifies the user has the right to be there.

There is no role check. No session validation. No authentication gate. If you know the URL, you're in.

### What Can an Attacker Do?

Once inside an unprotected admin panel, an attacker can typically:

- Delete, modify, or create user accounts
- Access private user data
- Escalate their own account to admin
- Trigger any administrative function the panel exposes

### What Kind of Apps Are Affected?

Any application where admin routes are not properly protected — common in quickly built internal tools, legacy applications, and apps where authorization was bolted on as an afterthought rather than designed in from the start.

---

## How It Works — Simple Example

Imagine an application with an admin panel at:

```
https://example.com/administrator-panel
```

A regular user navigating the site would never see a link to this page. But the page itself has no access control — anyone who knows the URL can visit it freely.

The question becomes: **how does an attacker find that URL?**

There are three realistic ways:

**1. robots.txt disclosure** The `robots.txt` file tells search engine crawlers which pages _not_ to index. Developers sometimes add admin paths here to keep them out of Google. Ironically, this makes them publicly readable to anyone who checks.
