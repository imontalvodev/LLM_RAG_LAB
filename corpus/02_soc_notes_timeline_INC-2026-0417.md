# SOC Notes: Timeline INC-2026-0417 (FICTIONAL DOCUMENT FOR LAB USE)

**Affected organization (fictional):** NorthLogic S.A.
**Author:** Tier 2 SOC analyst (night shift)
**Convention:** `[OBS]` = observed directly in logs/tools. `[DEC]` = stated by a person or document, not verified by the SOC. `[HYP]` = analyst hypothesis.
**Time zone:** all times in UTC unless otherwise noted. (Business tickets use CEST = UTC+2.)

## Context

NorthLogic runs three web servers with PHP 8.3.x: `web-prod-01`, `web-prod-02`, `web-prod-03`.

## Timeline

- **2026-10-08** `[DEC]` Advisory PHP-SA-2026-014 (CVE-SIM-2026-0001) is published.
- **2026-10-09 14:00** `[OBS]` `web-prod-01` upgraded to PHP 8.3.15 (patch manager).
- **2026-10-10** `[DEC]` An infrastructure colleague mentions that "a public PoC is already circulating". The SOC has not seen the PoC nor confirmed its existence.
- **2026-10-11 02:14** `[OBS]` The WAF logs a POST request with an 8 KB header to `/api/tracking/upload` on `web-prod-02`. HTTP 200 response.
- **2026-10-11 02:21** `[OBS]` EDR alert: `php-fpm` process spawns `sh -c` on `web-prod-02`.
- **2026-10-11 02:40 – 03:55** `[OBS]` Firewall netflow: outbound traffic from `web-prod-02` to `203.0.113.45` (port 443), approximately 2.1 GB in total.
- **2026-10-11 02:52** `[OBS]` Database slow query log: `SELECT * FROM customers` executed with the `app_rw` account from the IP of `web-prod-02`.
- **2026-10-11 03:10** `[HYP]` The analyst believes the 2.1 GB corresponds to the `customers` table. There is no packet content (TLS) to confirm this.
- **2026-10-11 06:30** `[OBS]` `web-prod-02` is isolated from the network (quarantine).
- **2026-10-11 07:05** `[OBS]` On `web-prod-03` there are 3 requests with an 8 KB header in the WAF (02:30–02:33 UTC), all with HTTP 200 responses. The EDR did not raise an alert on `web-prod-03`.
- **2026-10-11 07:40** `[HYP]` The analyst suspects that `web-prod-03` was also compromised. No artifact has been found on disk yet.
- **2026-10-12 09:30** `[OBS]` `web-prod-02` (after rebuild) and `web-prod-03` upgraded to PHP 8.3.15.

## Open notes

- The original entry vector has not been determined beyond the 02:14 request.
- Pending: disk forensics of `web-prod-02` and `web-prod-03`.
- The analyst could not check how many rows the 02:52 query returned (the slow query log records rows examined, not rows returned).
