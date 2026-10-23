# Challenge 3 — Python Pickle Deserialization

**Range:** AETHON Defense Systems / Operation Shadow Forge  
**Machine:** M3 — `aethon-dev`  
**Service:** Flask API on TCP `8080`; SSH on TCP `22`  
**Outcome:** **Challenge 3 core objectives verified**

## Executive summary

The Range3 Challenge 3 API accepts JSON containing a Base64-encoded value in the `profile` field and deserializes it with Python `pickle`. In the local lab, a controlled proof object caused the endpoint to execute a non-destructive `id` command and write the output to `/tmp/c3-proof.txt`. The response was HTTP 200 and the file showed execution as the `devops` service account. A subsequent controlled request wrote the challenge flag to a temporary output file and recovered Flag 3.

The session also verified the M2-to-M3 SSH pivot after correcting a malformed key fixture in the original setup. The test connected to `devops@192.168.56.103` from M2 and returned the hostname `aethon-dev`.

## Environment and network

- Guest OS observed: Ubuntu 26.04.1 LTS, kernel `7.0.0-38-generic`.
- API response: Python 3.14.4 / Werkzeug 2.3.7; app version `1.3.2`.
- M2 Host-only address: `192.168.56.104/24`.
- M3 Host-only address: `192.168.56.103/24`.
- M3 NAT address observed: `10.0.2.15/24`.
- Redis pivot configuration: `aethon-cache:6379`; the lab config comments state no Redis password is set.

The repository setup script used `config.ini` as a systemd `EnvironmentFile`, despite the INI section syntax. That incompatible line was removed from the **staged local copy** before installing M3. The upstream repository was not changed. The environment also differs from the README's nominal Ubuntu 22.04 baseline.

## Step-by-step record

### 1. Stage and validate M3 files

The contents of `machines/m3-dev-api/` were copied to `/opt/aethon-setup/`. The staged files included `setup/setup.sh`, `service/api.py`, `service/requirements.txt`, and `service/config.ini`. `bash -n` reported `M3 syntax check passed`.

### 2. Install and verify services

The staged service configuration was adjusted to remove the invalid `EnvironmentFile` reference. After installation:

- `aethon-api` reported `active (running)`.
- SSH reported active and listening on port 22.
- `curl http://127.0.0.1:8080/` returned HTTP 200 and API metadata.
- `/api/health` returned JSON with `status: ok`.

### 3. Configure private VM networking

The initial interface view showed NAT only. A Host-only adapter was then enabled on M2 and M3 while NAT remained available for package downloads. M3 used `192.168.56.103/24`; M2 used `192.168.56.104/24`. M2 successfully pinged M3 with 0% packet loss and `nc` confirmed port 22 was reachable.

### 4. Repair the SSH key fixture

The embedded private key failed to parse with `error in libcrypto`. A replacement Ed25519 key pair was generated on M2. Only the public key was transferred to M3 and installed as `/home/devops/.ssh/authorized_keys` with restrictive permissions and ownership. The private key itself is not reproduced here. The subsequent SSH command from M2 returned `aethon-dev`, confirming key-based access.

### 5. Inspect the API and configuration

The remote shell identified `hostname=aethon-dev` and user `devops` (UID/GID 1002 in the observed run). The `/opt/aethon-api/` directory contained `api.py`, `requirements.txt`, `config.ini`, a virtual environment, and `flag3.txt`. The API root listed `POST /api/load-profile` as requiring the `X-API-Key` header. Screenshot evidence containing the local API-key values has been redacted.

### 6. Demonstrate unsafe deserialization

The report's controlled test used a pickle proof object whose `__reduce__` returned `os.system` with a harmless command that wrote identity information to `/tmp/c3-proof.txt`. The request was sent to the local `http://127.0.0.1:8080/api/load-profile` endpoint with JSON content type and the configured lab API-key header. The server returned HTTP 200 with status `loaded`; the proof file recorded `uid=1002(devops) gid=1002(devops) groups=1002(devops)`. The API key value is intentionally omitted from this public document.

This demonstrates command execution as the service account; it does not demonstrate root access on M3.

### 7. Recover Flag 3

The first `cat /tmp/c3-flag.txt` attempt happened before the flag-retrieval request had created the file and therefore returned “No such file or directory.” The subsequent local test request wrote `/opt/aethon-api/flag3.txt` to `/tmp/c3-flag.txt`; the response was HTTP 200 and the file displayed the flag.

**Flag 3 recovered:**

```text
AETHON{p1ckl3_d3s3r14l1z4t10n_rce}
```

## Findings

- `pickle.loads()` on attacker-controlled serialized content enables code execution during deserialization.
- The proof executed as `devops` (UID/GID 1002 in this environment); root access was not demonstrated on M3.
- The `X-API-Key` header gates the code path but does not make pickle deserialization safe.
- `config.ini` disclosed the next-stage Redis hostname and port.
- The key fixture in the repository required local repair before the SSH pivot worked.

## Recommended remediation

- Do not deserialize untrusted pickle data; use a data-only format such as JSON and validate a strict schema.
- Keep the API unprivileged and restrict file and network access.
- Rotate lab API keys after exercises and keep values out of publicly shared screenshots.
- Correct setup scripts to parse INI configuration appropriately rather than passing it to systemd as an `EnvironmentFile`.
- Pin and test supported OS/Python versions.
- Keep intentionally vulnerable services on the private Host-only network.

## References

- [Range3 repository](https://github.com/hacktifytechnologies/Range3)
- [Challenge 3 brief](https://github.com/hacktifytechnologies/Range3/blob/main/challenges/c3-red.md)
- [M3 setup script](https://github.com/hacktifytechnologies/Range3/blob/main/machines/m3-dev-api/setup/setup.sh)
- [M3 API source](https://github.com/hacktifytechnologies/Range3/blob/main/machines/m3-dev-api/service/api.py)
- [M3 config](https://github.com/hacktifytechnologies/Range3/blob/main/machines/m3-dev-api/service/config.ini)

## Screenshot evidence

All 20 report screenshots are included. The affected evidence images keep the surrounding execution/validation proof but redact the lab API-key literals.

### Evidence 01 — M3 files staged and shell syntax validated

![Evidence 01: M3 files staged and shell syntax validated](screenshots/challenge-3-pickle/01-m3-files-staged-and-shell-syntax-validated.png)

### Evidence 02 — API endpoint and SSH service checks

![Evidence 02: API endpoint and SSH service checks](screenshots/challenge-3-pickle/02-api-endpoint-and-ssh-service-checks.png)

### Evidence 03 — Staged systemd configuration adjusted for the INI-format config file

![Evidence 03: Staged systemd configuration adjusted for the INI-format config file](screenshots/challenge-3-pickle/03-staged-systemd-configuration-adjusted-for-the-ini-format-config.png)

### Evidence 04 — M3 installer completed and API service is active

![Evidence 04: M3 installer completed and API service is active](screenshots/challenge-3-pickle/04-m3-installer-completed-and-api-service-is-active.png)

### Evidence 05 — Health endpoint responded; initial interface state captured

![Evidence 05: Health endpoint responded; initial interface state captured](screenshots/challenge-3-pickle/05-health-endpoint-responded-initial-interface-state-captured.png)

### Evidence 06 — M2 Host-only adapter configuration

![Evidence 06: M2 Host-only adapter configuration](screenshots/challenge-3-pickle/06-m2-host-only-adapter-configuration.png)

### Evidence 07 — M3 Host-only adapter configuration

![Evidence 07: M3 Host-only adapter configuration](screenshots/challenge-3-pickle/07-m3-host-only-adapter-configuration.png)

### Evidence 08 — M3 private IPv4 address confirmed

![Evidence 08: M3 private IPv4 address confirmed](screenshots/challenge-3-pickle/08-m3-private-ipv4-address-confirmed.png)

### Evidence 09 — M2-to-M3 ping and SSH-port reachability verified

![Evidence 09: M2-to-M3 ping and SSH-port reachability verified](screenshots/challenge-3-pickle/09-m2-to-m3-ping-and-ssh-port-reachability-verified.png)

### Evidence 10 — Original SSH private-key fixture failed to parse

![Evidence 10: Original SSH private-key fixture failed to parse](screenshots/challenge-3-pickle/10-original-ssh-private-key-fixture-failed-to-parse.png)

### Evidence 11 — Repeated malformed-key failure recorded during troubleshooting

![Evidence 11: Repeated malformed-key failure recorded during troubleshooting](screenshots/challenge-3-pickle/11-repeated-malformed-key-failure-recorded-during-troubleshooting.png)

### Evidence 12 — Replacement Ed25519 key pair generated on M2

![Evidence 12: Replacement Ed25519 key pair generated on M2](screenshots/challenge-3-pickle/12-replacement-ed25519-key-pair-generated-on-m2.png)

### Evidence 13 — SSH retry before the replacement public key was authorized

![Evidence 13: SSH retry before the replacement public key was authorized](screenshots/challenge-3-pickle/13-ssh-retry-before-the-replacement-public-key-was-authorized.png)

### Evidence 14 — M2 private key validates and the public-key file is present

![Evidence 14: M2 private key validates and the public-key file is present](screenshots/challenge-3-pickle/14-m2-private-key-validates-and-the-public-key-file-is-present.png)

### Evidence 15 — M3 downloaded and validated the public key over Host-only networking

![Evidence 15: M3 downloaded and validated the public key over Host-only networking](screenshots/challenge-3-pickle/15-m3-downloaded-and-validated-the-public-key-over-host-only-networ.png)

### Evidence 16 — M2-to-M3 key-based SSH login succeeded and returned aethon-dev

![Evidence 16: M2-to-M3 key-based SSH login succeeded and returned aethon-dev](screenshots/challenge-3-pickle/16-m2-to-m3-key-based-ssh-login-succeeded-and-returned-aethon-dev.png)

### Evidence 17 — M3 API inventory and configuration — API-key values redacted for publication

![Evidence 17: M3 API inventory and configuration — API-key values redacted for publication](screenshots/challenge-3-pickle/17-m3-api-inventory-and-configuration-api-key-values-redacted-for-p.png)

### Evidence 18 — Controlled pickle command-execution proof — API-key values redacted for publication

![Evidence 18: Controlled pickle command-execution proof — API-key values redacted for publication](screenshots/challenge-3-pickle/18-controlled-pickle-command-execution-proof-api-key-values-redacte.png)

### Evidence 19 — Intermediate flag-file check occurred before the retrieval request created it

![Evidence 19: Intermediate flag-file check occurred before the retrieval request created it](screenshots/challenge-3-pickle/19-intermediate-flag-file-check-occurred-before-the-retrieval-reque.png)

### Evidence 20 — HTTP 200 response and Flag 3 retrieval confirmed — API key redacted

![Evidence 20: HTTP 200 response and Flag 3 retrieval confirmed — API key redacted](screenshots/challenge-3-pickle/20-http-200-response-and-flag-3-retrieval-confirmed-api-key-redacte.png)
