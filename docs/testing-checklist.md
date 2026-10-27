# Validation and Test Checklist

## Completion summary

The core SOC automation project is marked **complete** for its current scope. Evidence screenshots show the SSH-authentication alert context reaching Shuffle and a successful Wazuh API Active Response dispatch. TheHive integration was not completed because of hardware/resource limitations and is excluded from the completed scope.

This checklist distinguishes what has been evidenced from additional validation worth performing after future configuration changes.

## 1. Endpoint telemetry and detection

- [x] SSH authentication events were visible in Wazuh.
- [x] A custom SSH brute-force alert using rule ID `100003` appeared in the Shuffle runtime alert context.
- [x] The runtime data included a timestamp, rule ID, alert title, and source information.
- [ ] Re-run the detection test after any rule or agent changes and confirm the source IP belongs to the intended test client.

**Detection semantics:** the configured Wazuh frequency/timeframe rule detects repeated matching events within a time window. Do not call it strictly consecutive attempts unless a separate stateful mechanism has been implemented and tested.

## 2. Shuffle intake and notification

- [x] A selected Wazuh alert reached the Shuffle workflow.
- [x] The workflow reached the analyst email/approval stage.
- [x] A notification email asked whether the source IP should be blocked.
- [ ] After modifying the workflow, test the Yes and No branches end to end to confirm only Yes can reach the response action.
- [ ] Recheck the enrichment node result if its configuration or API key changes.

## 3. Wazuh API and Active Response

- [x] The Wazuh API action returned HTTP `200`.
- [x] The captured result reported `total_affected_items: 1`, `total_failed_items: 0`, and `AR command was sent to all agents`.
- [ ] Confirm the target agent ID is correct for the current installation. The screenshot's request URL references agent ID `004`; older lab notes use `003` for `LINUX-ENDPOINT`.
- [ ] Confirm the target agent's `active-responses.log` records execution.
- [ ] Confirm the expected firewall rule was applied on the target endpoint and test a targeted rollback.

A successful API dispatch response is valuable evidence but does not, on its own, prove that the resulting firewall rule remained applied. Do not test using the Windows host, host-only gateway, or an address whose ownership is uncertain.

## 4. TheHive integration

**Not completed — excluded due to hardware/resource limitations.**

- [ ] Automated case creation from Shuffle was not integrated or validated.
- [ ] If hardware resources become available, test TheHive API authentication and create a clearly labelled test case before connecting it to production-like alerts.

TheHive is shown in the architecture as a planned case-management extension, not as a working integration.

## 5. Portfolio evidence

- [x] Architecture diagram included.
- [x] Shuffle runtime alert screenshot included: `../assets/screenshots/01-shuffle-runtime-alert.png`.
- [x] Wazuh Active Response API result screenshot included: `../assets/screenshots/02-wazuh-active-response-result.png`.
- [ ] Add a redacted endpoint log/firewall verification screenshot if you later capture it.

Before publishing, redact passwords, JWTs, API keys, webhook URLs/tokens, private email addresses, and internal details you do not want public.
