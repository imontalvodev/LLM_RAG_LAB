# 72-Hour Update Report: INC-2026-0417 (FICTIONAL DOCUMENT FOR LAB USE)

**Author:** NorthLogic incident response (IR) lead
**Date:** 2026-10-14
**Status:** incident **open**; containment achieved on `web-prod-02`; investigation ongoing

## Corrections to earlier information

| Earlier claim | Current status |
|---|---|
| "All three servers are compromised" (chat, 11 Oct) | **Retracted.** Confirmed: 1 (`web-prod-02`). Suspected, not confirmed: 1 (`web-prod-03`). `web-prod-01` had been patched since 9 Oct and shows no malicious requests. |
| "A customer notified us" (chat, 11 Oct) | **Retracted** by its author. Detection was through the EDR alert at 02:21 UTC. |
| "The attacker is the usual group" (chat, 11 Oct) | **Unsupported.** There is no attribution. See forensic report. |
| "There is no evidence of a data leak" (statement, 12 Oct) | **Qualified.** The forensic report finds fragments consistent with customer data in an archive created by the attacker. What exactly was sent has not been confirmed. |

## State of the facts (as of 2026-10-14)

**Observed:**
- Webshell on `web-prod-02` created at 02:16 UTC on 11 Oct.
- EDR alert at 02:21 UTC; isolation at 06:30 UTC.
- Approximately 2.1 GB outbound to `203.0.113.45` between 02:40 and 03:55 UTC.
- Recovered fragments (≈12%) of a `data.tgz` archive with fields `name`, `email`, `phone`.

**Not determined:**
- Actual content of the 2.1 GB sent.
- Number of affected customers.
- Whether `web-prod-03` was compromised.
- Identity or affiliation of the attacker.
- Whether there was lateral movement to the database beyond the 02:52 UTC query.

**Risk assessment (IR opinion, not a fact):**
- The IR team considers it "**likely**" that personal data was exfiltrated, but does not treat it as confirmed.
- Internal incident severity: **high**. The vendor's CVSS score was 9.8; the internal team, after considering that the WAF filters part of the traffic, estimates 8.1 for its environment. This is an internal estimate and does not replace the vendor's.

## Actions

- Notification to the data protection authority: **stated** by the DPO on 13 Oct; acknowledgment **still pending** at the time of this report.
- Pending: analysis of `web-prod-03`; rotation of the `app_rw` account; review of the customer communication plan.
- Recommendation: do not issue statements claiming the absence of a data leak while the forensic investigation remains open.
