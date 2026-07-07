# Web Hacking 101 — Whitebox Web Exploit Development

**DEF CON 34 Workshop** · half-day, hands-on
Cale Smith · Luke Cycon · Young Kim · Ruchik Dave

> **⏳ More content is being added here ahead of the con.** This repo will host the workshop
> materials — the vulnerable lab apps, setup instructions, and student handouts. Star/watch to get
> updates; full materials go live around the session.

---

## What this is

A hands-on intro to finding and weaponizing real web bugs — SQL injection, command injection, JWT
and session flaws, IDOR, XSS — and chaining an auth bypass with RCE into a remote-root exploit.
The theme is **whitebox + AI-resistant**: you get the source and the running box, and every
challenge is built so a blind copy-paste of the textbook payload quietly fails. Methodology is the
moat.

## What's coming to this repo

- 🎯 **The target app** — a self-contained, Dockerized vulnerable application (one command to run).
- 🧩 **Standalone warm-up labs** — one isolated vulnerability each, for a clean beginner on-ramp.
- 📘 **Student handouts** — objectives, a cheat sheet, and a progressive hint ladder.
- 🧪 **Setup + verification** — `docker compose up`, and a checker script.

*(Slides and full solution walkthroughs are shared at/after the session.)*

## Prerequisites (bring to the workshop)

A laptop on WiFi with browser dev tools, an SSH client, `curl`, Python 3, and **Burp Suite
Community** (free edition — no Pro needed). Targets and a shell box are provided in the room; Docker
(Docker Desktop / OrbStack) is optional for running locally.

## Contact

Questions before the con? Join the Discord: `<DISCORD INVITE COMING SOON>`

---
*Intentionally vulnerable training software. Run it only in an isolated lab environment.*
