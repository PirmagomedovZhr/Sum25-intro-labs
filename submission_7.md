# Lab 7 — GitOps Fundamentals

## Task 1 · Git State Reconciliation

- **Repo initialised:** `git init`
- **Desired state recorded:** `version: 1.0` in *desired‑state.txt* and committed.
- **Drift introduced:** manually changed *current‑state.txt* to `version: 2.0`.

```console
$ ./reconcile.sh
Пт 11 июл 2025 11:24:49 UTC - DRIFT DETECTED! Reconciling...
```

`reconcile.sh` compared desired vs current, detected mismatch and restored the file, thus eliminating drift.

A background loop can be started when needed:

```bash
watch -n 5 ./reconcile.sh   # continuous reconciliation every 5 s
```

## Task 2 · Health Monitoring

`healthcheck.sh` appends status lines to *health.log*:

```console
Пт 11 июл 2025 11:25:25 UTC - OK: States synchronized
Пт 11 июл 2025 11:25:37 UTC - CRITICAL: State mismatch!
```

First run reported **OK** (files identical). After appending “unapproved change” to *current‑state.txt* the next run flagged **CRITICAL**.

### Cron integration

A five‑minute cron job ensures automated health checks:

```cron
*/5 * * * * ~/gitops-lab7/healthcheck.sh
```

---

### Key Takeaways

- **Declarative source of truth**: desired state lives in Git.
- **Automatic reconciliation**: shell script restores drift.
- **Continuous health**: log‑based monitoring + cron alert on mismatch.

**Submission prepared by:** Zhalil

