# NorthLogic S.A. Statements on INC-2026-0417 (FICTIONAL DOCUMENT FOR LAB USE)

## Internal statement to employees: 2026-10-12, 18:00 CEST

From: Operations Management

"We have suffered a security incident that affected **one server** of our web platform. The incident is **contained**. According to the technical team, **there is no evidence that customer data has been leaked**. Systems are operating normally."

## Press release: 2026-10-13, 10:00 CEST

"NorthLogic detected on 11 October an unauthorized access to one of its servers. The company acted quickly and isolated the machine in less than four hours. Customers do not need to take any action. NorthLogic has notified the incident to the data protection authority."

## Verification notes (added by the IR team, 2026-10-14)

- "Isolated the machine in less than four hours": isolation took place at 06:30 UTC; the first logged malicious request was at 02:14 UTC, i.e. **4 h 16 min** earlier. Counting from the EDR alert (02:21 UTC) it is 4 h 09 min, which also exceeds four hours. The IR team considers the phrase **incorrect**.
- "No evidence that data has been leaked": the internal statement was written **before** the forensic report of 13 October was received. The lack of evidence at that time is not the same as having ruled it out.
- "Has notified the incident to the data protection authority": stated by the DPO. The IR team has not seen the acknowledgment of receipt of the notification.
- "One server": the SOC keeps open the suspicion about a second server (`web-prod-03`).
