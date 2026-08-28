# Bad Reception — One Leading Digit to an Admin-Bot Channel

> **Intigriti August 2026 — Web Challenge Write-up.** A leading-digit validation bypass yields stored XSS in a moderation bot, which is pivoted to read an admin-only TV channel and recover the flag.

![Platform](https://img.shields.io/badge/Platform-Intigriti-8a2be2)
![Category](https://img.shields.io/badge/Category-Web-blue)
![Bug](https://img.shields.io/badge/Bug-Stored%20XSS%20%E2%86%92%20Bot%20RCE-critical)
![Status](https://img.shields.io/badge/Status-Solved-brightgreen)

| | |
|---|---|
| **Challenge**  | `https://challenge-0826.challenges.intigriti.io` |
| **Title**      | *Bad Reception* |
| **Researcher** | [rajib_mahmud](https://app.intigriti.com/profile/rajib_mahmud) (Intigriti) · [rajib-mahmud](https://github.com/rajib-mahmud) (GitHub) |
| **Bug class**  | Improper input validation → **stored XSS** → privilege escalation (CWE-20 / CWE-79) |
| **Impact**     | Arbitrary JS in the admin-bot context; read of an admin-only resource |
| **Flag**       | `INTIGRITI{019ff176-bc01-7543-9e81-46e417c8b39b}` |

**📖 Full write-up (styled):** [rajib-mahmud.github.io/intigriti-0826](https://rajib-mahmud.github.io/intigriti-0826/)

---

## TL;DR

The retro-TV app has channels 1–10 of static and a **"report bad reception"** button. The report form's `channelId` is validated by a single rule — **it must start with a digit** — so `1<img src=x onerror=...>` passes.

That value is reflected **unencoded** into `/moderate/{id}`, a page the privileged **admin bot** opens automatically. The payload runs in the bot's browser, fetches the admin-only **channel 11**, and exfiltrates its stream filename to a webhook. Download the video, read the flag off the frame.

```
weak validation (starts-with-digit)  →  stored XSS in channelId
    →  admin bot renders /moderate/{id}  →  our JS runs as admin
    →  fetch /api/channels/11/load  →  privileged .mp4 filename
    →  GET /static/streams/<hash>.mp4  →  flag on screen
```

---

## The key ideas

1. **"Starts with a digit" is not "is a number."** The validator inspects only the first character, so `1` + arbitrary HTML passes.
2. **Self-XSS becomes real XSS in the moderation bot.** The value only matters because a *privileged automated client* renders it unencoded.
3. **A stolen credential isn't a stolen capability.** Channel 11's privilege is bound to the bot's request context — replaying its cookie from the public internet still returns `channel not available`, so the read must happen *inside* the bot.
4. **Confirm, don't guess.** The `/static/streams/{filename}` template is verified from channels 1–10 first; channel 11 just plugs its bot-supplied hash into the same directory.

See the [full write-up](https://rajib-mahmud.github.io/intigriti-0826/) for reconnaissance, the exploit payload, a minimal Python PoC, mermaid diagrams of the full chain, remediation, and appendices on the unused JSONP gadget and the pitfalls that make the PoC look broken.

---

*Write-up by [rajib_mahmud](https://app.intigriti.com/profile/rajib_mahmud) (Intigriti) / [rajib-mahmud](https://github.com/rajib-mahmud) (GitHub) · Intigriti August 2026 Challenge · Educational / responsible-disclosure purposes only.*
