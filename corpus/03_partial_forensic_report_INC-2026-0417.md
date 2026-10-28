# Partial Forensic Report INC-2026-0417 (FICTIONAL DOCUMENT FOR LAB USE)

**Author:** forensic analyst (external DFIR team under contract)
**Report date:** 2026-10-13
**Scope:** disk images of `web-prod-02` and `web-prod-03`, delivered on 2026-10-12

## Confirmed findings (observed in the images)

1. On `web-prod-02`, the file `/var/www/html/uploads/.cache.php` exists, created on 2026-10-11 at 02:16 UTC. It contains a webshell. SHA-256 (prefix): `9c41e7b2…`.
2. On `web-prod-02`, the `www-data` history shows execution of `tar`, `curl` and `whoami` between 02:21 and 02:38 UTC.
3. A file `/tmp/.x/data.tgz` of 1.9 GiB was found, deleted but partially recovered. The recovered fragments contain rows with fields such as `name`, `email` and `phone`. Approximately 12% of the file was recovered.
4. On `web-prod-03`, **no** webshell was found, nor files modified after 2026-10-08, nor anomalous processes in the available logs.

## Inconclusive findings

- The recovered fragments of `data.tgz` are consistent with customer data. With 12% recovered, the report **cannot state** that the entire archive contained personal data, nor how many records it held.
- It cannot be confirmed that `data.tgz` was the content sent to `203.0.113.45`. The volume (1.9 GiB compressed versus ~2.1 GB of outbound traffic) is of the same order of magnitude, but the report states that "a size match is not proof of identical content".
- For `web-prod-03`, the absence of artifacts may be because the attacker was unsuccessful or because the available logs do not cover what happened. The report classifies it as "compromise not confirmed; not ruled out".

## Attribution

The intrusion is not attributed to any actor. A third-party threat intelligence provider has verbally reported that the IP `203.0.113.45` appears in one of its reports associated with a group; the forensic team **has not seen that report** and considers it a "low-confidence data point".

## Limitations

- There is no packet capture. Traffic is TLS.
- `web-prod-03` logs are only retained from 2026-10-10 00:00 UTC.
