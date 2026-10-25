# Validation and Test Checklist

Use an isolated test source/target and retain evidence for each check. Do not test against the Windows host, host-only gateway, or any address whose ownership is uncertain.

## 1. Endpoint telemetry

- [ ] Confirm the expected agent is connected to Wazuh.
- [ ] Generate a controlled SSH authentication event on the lab endpoint.
- [ ] Confirm the event appears in Wazuh with the expected agent name/ID.
- [ ] Inspect the decoded alert and confirm whether `data.srcip` exists.

## 2. Detection rule

- [ ] Trigger the intended authentication detection with controlled test events.
- [ ] Confirm the correct rule ID, severity, source IP, timestamp, and description.
- [ ] Confirm the correlation behavior matches the rule's actual semantics. A frequency/timeframe rule can count repeated matches in a time window; do not describe it as strictly consecutive unless that behavior has been separately implemented and tested.

## 3. Shuffle intake and enrichment

- [ ] Confirm the selected Wazuh alert reaches the expected Shuffle webhook.
- [ ] Inspect the raw execution data and validate the source-IP expression.
- [ ] Confirm the VirusTotal/OSINT step receives only the intended IP and returns a response.
- [ ] Confirm an email is delivered with enough alert context for the analyst to make a decision.

## 4. Approval logic

- [ ] Click **No** and confirm the Wazuh response action does not execute.
- [ ] Click **Yes** with a verified test source address and confirm the response node runs.
- [ ] Confirm the source IP passed into the response action matches the reviewed alert.
- [ ] Confirm the Wazuh API returns a meaningful result and the endpoint logs the action.
- [ ] Confirm the resulting firewall state on the target endpoint.
- [ ] Remove any test block using a targeted rollback command; do not flush all firewall rules.

## 5. TheHive case management

- [ ] Send a test alert to the TheHive integration.
- [ ] Confirm a case/alert is created with the expected title, severity, source IP, timestamp, and description.
- [ ] Confirm repeated alerts behave as intended and do not create accidental duplicates.

## 6. Evidence for the portfolio

Capture redacted screenshots of:

- [ ] The architecture diagram.
- [ ] Wazuh alert details showing the relevant rule and source IP (use a lab IP, redact hostnames if needed).
- [ ] Shuffle workflow graph and a successful test execution.
- [ ] VirusTotal result with the tested IP and response summary.
- [ ] Approval decision and the corresponding response result.
- [ ] TheHive test case, once case creation has been confirmed.

Before publishing, redact passwords, JWTs, API keys, webhook URLs/tokens, private email addresses, and internal host details you do not want public.
