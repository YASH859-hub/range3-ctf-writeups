# Range3 CTF Write-ups — Operation Shadow Forge

Five evidence-backed write-ups for the Range3/AETHON Defense Systems cyber range, documented from the corresponding Word reports and their submitted VirtualBox screenshots.

> **Suggested GitHub repository name:** `range3-ctf-writeups`  
> Alternative if you prefer a more neutral name: `range3-lab-documentation`.

## Contents

| File | Challenge | Target / focus | Evidence-backed status |
|---|---|---|---|
| [`challenge-1-xxe.md`](challenge-1-xxe.md) | 1 — Initial Access | M1 `aethon-web`; XML External Entity injection | Local XXE flag/config reads evidenced; M1→M2 pivot not verified |
| [`challenge-2-ssti.md`](challenge-2-ssti.md) | 2 — Credential Access + Execution | M2 `aethon-docs`; Jinja2 SSTI | Template evaluation, command execution as `docuser`, Flag 2 and pivot-artifact disclosure evidenced |
| [`challenge-3-pickle-deserialization.md`](challenge-3-pickle-deserialization.md) | 3 — Lateral Movement + Execution | M3 `aethon-dev`; unsafe pickle deserialization | M2→M3 SSH, API proof of execution and Flag 3 verified |
| [`challenge-4-redis.md`](challenge-4-redis.md) | 4 — Lateral Movement via Redis | M4 `aethon-cache`; unauthenticated Redis | Redis reachability, Flag 4 and M5 pivot data verified; Redis-to-SSH file write/login not verified |
| [`challenge-5-cap-setuid.md`](challenge-5-cap-setuid.md) | 5 — Final privilege escalation | M5 `aethon-core`; `cap_setuid` on Python | Local-console root transition and Flag 5 verified; M4→M5 SSH pivot not verified |

## Screenshot evidence

All **75 embedded screenshots** from the five Word reports are included under [`screenshots/`](screenshots/), organized by challenge. Evidence files are linked directly in each write-up, so GitHub renders them inline.

A few screenshots contain lab secrets from internal configuration output. Before publication, the affected areas were redacted in the evidence copies:

- M1 internal configuration credential values.
- M2 default lab credentials and disclosed private-key contents.
- M3 lab API-key values.

The source Word reports are not included in this package. The screenshot directory contains the sanitized evidence intended for GitHub publication; do not add unredacted originals.

## Overall result summary

The local objective evidence spans the five challenges, but **the whole five-machine attack chain is not claimed as end-to-end complete**. In particular:

- M1's local XXE tests succeeded; the M1→M2 pivot was not demonstrated.
- M2's SSTI and file-disclosure objectives were demonstrated; the later M2→M3 SSH connection was successfully verified during Challenge 3 after the malformed key fixture was replaced.
- M3's unsafe pickle deserialization proof executed as `devops` and Flag 3 was recovered.
- M4's Redis was reachable from M3, Flag 4 and the next-stage values were retrieved, but the expected M4 `authorized_keys` file was still absent in the latest local check. SSH as `cacheops` and the M4→M5 pivot are unverified.
- M5's `cap_setuid`-based UID 0 transition and Flag 5 were verified from the M5 local console, not via SSH from M4.

The individual challenge files distinguish observed successes from unverified steps. Do not treat missing pivots as complete based on flags or configuration values alone.

## Environment notes

The Range3 README describes Ubuntu 22.04 LTS and OpenStack network/DNS assumptions. This session used Ubuntu 26.04 guests under Oracle VirtualBox and required local compatibility changes to some staged setup scripts. The individual write-ups record those deviations.

## Safety and responsible use

This is documentation for a deliberately vulnerable training range. Keep all machines on a private/Host-only network, do not expose vulnerable ports to the public internet, and do not reuse the lab credentials or keys. After training, restore snapshots and remove intentionally weak settings such as unauthenticated Redis and unnecessary Linux capabilities.

## Upstream project

- [hacktifytechnologies/Range3](https://github.com/hacktifytechnologies/Range3)
- The per-challenge source links appear in each report.
