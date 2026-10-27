# Security Notes

This is an educational lab. Do not copy the configuration into production without a security review.

- Never commit Wazuh API passwords, installation-password archives, JWTs, VirusTotal API keys, Shuffle `AUTH` values, webhook URLs containing secrets, TheHive credentials, or private key material.
- Do not expose Wazuh API port `55000`, agent communication ports `1514/1515`, TheHive's `9000`, or management interfaces directly to the public internet for convenience. Prefer private network reachability and explicit access controls.
- The cURL `-k` option disables certificate verification. It can help diagnose a self-signed certificate in an isolated lab, but it should not become an unexamined production default.
- Validate the source IP before response. In testing, `192.168.56.1` appeared as a source; verify whether it is the host-only gateway/host adapter or the actual test client before blocking.
- Confirm the agent ID targeted by the Wazuh API. The included dispatch screenshot refers to agent ID `004`, while earlier lab notes reference `003` for `LINUX-ENDPOINT`.
- Keep human approval between enrichment and containment. Verify that the No/deny branch cannot reach the block action and that no separate broad automatic Wazuh rule triggers blocking before approval.
- A successful API response confirms request dispatch, not necessarily that the intended firewall state persisted. Verify endpoint logs and firewall state when retesting.
- Docker socket access grants containers extensive control over the host. Orborus on the Wazuh server is convenient for a small lab, but a dedicated runtime VM gives stronger isolation.
- TheHive is not integrated in the completed workflow because of hardware/resource limitations. Do not describe case creation as a working feature until it is successfully tested.
- Keep the Windows host and host-only gateway reachable during tests. Make a rollback plan for any firewall response.
- Use only systems you own or are authorized to test.
