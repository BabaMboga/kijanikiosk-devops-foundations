# KijaniKiosk API Server - Triage Report

**Date:** 2024-01-15 (log evidence) — investigation performed 2026-09-18
**Investigated by:** Ayim
**Server:** babamboga@Katerina (WSL2 Ubuntu, local dev/lab environment)
**Incident start (approximate):** 04:07:55 (first ERROR entry in application log)

## Summary

The application log shows a clear two-phase incident: a database connection pool exhaustion event starting at 04:07:55, followed by a complete database connectivity failure at 06:22:18. However, live investigation of the current system state found **no nginx or application process listening on any expected port** — meaning the service that Osei's briefing describes as "running" is not actually bound to port 80, 3000, or any other expected port at the time of this triage. The memory consumer process is real but trivial relative to available system resources. The most likely root cause of the *reported* latency issue is the database pool exhaustion documented in the logs; the *current* inability to reach the server at all is a separate, more severe finding that should be treated as higher priority.

## Process and Resource State

- The memory-consumer process (`python3`, PID **17749**) is running as **root**, using **3.2% memory** (RSS ≈ 524,276 KB / ~512 MB) and negligible CPU (0.2%).
- System-wide memory is **not under pressure**: `free -h` shows 15Gi total, only 1.0Gi used, 14Gi free, and 0B of the 4Gi swap in use. `/proc/meminfo` confirms `MemAvailable` at ~15.25M kB out of ~16.29M kB total.
- `top -bn1` confirms **0 zombie processes** and 8 of 9 tasks sleeping — system load average is effectively 0.00.
- The zombie-process check (`ps aux | awk '$8=="Z"'`) returned **no output** — confirmed no zombie or stuck processes.
- The open-file-descriptor check shows nothing abnormal; the highest fd count (5) belongs to a `bash` shell, and PID 17749 (the memory consumer) holds 0 open file descriptors — it is idle, just holding allocated memory, not actively doing I/O.

**Conclusion for this area:** the memory-consumer process exists and matches the lab's simulated condition, but at ~512MB against a 15.9GB system it is not, by itself, a resource pressure event on this machine.

## Filesystem and Disk

- Root partition (`/dev/sdd`, mounted at `/`) is at **3% used** (28G of 1007G) — no disk pressure at the partition level.
- However, `/var/log` contains two very large files **unrelated to KijaniKiosk**: `air_quality_cron.log` (1.3G) and `air_quality_consumer_cron.log` (504M) — both flagged by the `+50M` size search. These belong to a different project (the Air Quality Data Management System) sharing this host, not the KijaniKiosk app.
- The KijaniKiosk-specific log directory (`/var/log/kijanikiosk`) totals **271M**, which includes the simulated oversized `access.log.1` file from the lab setup.
- No files were modified in `/tmp` in the last 60 minutes — no evidence of transient/temp-file buildup.
- `ls -lhtr /var/log/` shows the `kijanikiosk` log directory was last touched Sep 9, and `apt`/`dpkg.log` are the most recently modified — consistent with routine package activity, not an active incident.

**Conclusion for this area:** disk space is not a contributing factor to the reported latency. The two large air-quality logs are a housekeeping concern (log rotation isn't happening) but are not related to this incident and not close to filling the partition.

## Log Analysis

- Error timestamps cluster into **two distinct windows**:
  - **04:07:55 – 04:08:01** — three ERROR entries (pool exhaustion → two query timeouts)
  - **06:22:18 – 06:22:28** — three ERROR entries (repeated `ECONNREFUSED database:5432`, ending in "Retry limit reached")
- Reading the full log narrative in order: connection pool climbed from 85% → 94% capacity (WARN), then was reported exhausted at 04:07:55, immediately followed by two query timeouts on `orders` and `products` at 04:08:01, then a memory warning at 04:09:12. Roughly two hours later, the database became fully unreachable (`ECONNREFUSED`) and stayed down through the retry-limit failure at 06:22:28.
- No OOM-killer, disk-I/O error, or disk-quota events found in `/var/log/syslog`.
- No `Accepted`/`Failed`/`Invalid` authentication events found in `/var/log/auth.log` — no evidence of unauthorized login activity around the incident window.

**Conclusion for this area:** the log evidence supports a single root-cause chain — database connection pool exhaustion leading to query timeouts, followed later by the database becoming completely unreachable. This is the strongest, most specific signal in the whole investigation.

## Network and Service State

- `ss -tlnp` shows **only one listening socket on the entire machine** — port 53 (DNS, internal to WSL2). **Neither nginx (port 80) nor a Node app (port 3000 or similar) is currently listening.**
- Filtering explicitly for the expected ports (`ss -tlnp | grep -E "80|443|3000|8080|5432"`) returned **no results** — confirms no web, API, or database service is bound to any of its expected ports right now.
- `curl` to both `http://localhost/` and `http://localhost/api/health` returned **HTTP 000** (connection failed, not even a valid HTTP response) in under 6ms — this is a connection refusal, not a slow response. It is not consistent with "high latency"; it's consistent with "nothing is listening."
- `ss -tan` shows a total of only 2 tracked entries (the header row and one LISTEN line) — there is effectively no active traffic on this box at all right now.

**Conclusion for this area:** this is the most significant discrepancy in the investigation. The incident briefing states "NGINX is running. The Node.js app process is running." The live evidence directly contradicts that — nothing is listening on the ports those services would use. Either both services have since stopped/crashed, were never started in this environment, or this WSL2 lab instance does not have them running as system services.

## Assessment

There are two separate findings that should not be conflated into one story:

1. **Historical (log-based) root cause:** the log evidence clearly documents a database connection pool exhaustion incident starting at 04:07:55, which caused query timeouts and elevated latency — consistent with what Osei reported. This is a defensible, evidence-backed hypothesis for the *original* P95 latency degradation.
2. **Current (live) finding:** at the time of this triage, neither nginx nor the application is listening on any port, and the database itself appears unreachable based on the historical logs. This means the system is not currently in a "degraded" state — it is in a **fully down** state as far as network service availability goes. This is more severe than the latency issue originally reported and should be flagged as a separate, higher-priority problem rather than assumed to be "the same issue, just worse."

Memory and disk were both ruled out as contributing factors — neither shows meaningful pressure on this host.

## Recommended Next Steps

1. **Immediately verify why nginx and the application process are not listening** — check `systemctl status nginx` and the app's process manager/service status, and restart them if stopped. This is the most urgent action since the service is currently fully unreachable, not just slow.
2. **Investigate the database connectivity** referenced in the 06:22 `ECONNREFUSED` errors — confirm the database service (port 5432) is running and reachable from this host, since the log evidence shows it failed after the pool-exhaustion event and never recovered.
3. **Set up log rotation** for `air_quality_cron.log` and `air_quality_consumer_cron.log` (1.3G and 504M respectively) — while not the root cause of this incident, unrotated logs on a shared host is a latent risk that could contribute to future disk pressure if left unaddressed.