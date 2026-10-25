# Security Notes

This is a learning environment. The configuration should not be copied into a production environment without a security review.

- Never commit Wazuh API passwords, installation-password archives, JWTs, VirusTotal API keys, Shuffle `AUTH` values, webhook URLs containing secrets, TheHive credentials, or private key material.
- Do not expose Wazuh API port `55000`, agent communication ports `1514/1515`, TheHive's `9000`, or management interfaces directly to the public internet for convenience. Prefer private network reachability and explicit access controls.
- The `-k` cURL option disables certificate verification. It can help diagnose a self-signed certificate in an isolated lab, but it should not become an unexamined production default.
- The source-IP field must be validated before response. In earlier testing, `192.168.56.1` appeared as the source; verify whether it is the host-only gateway or the actual test client before blocking it.
- Keep human approval between enrichment and containment. Verify that the deny branch cannot reach the block action and that no separate Wazuh rule triggers broad automatic blocking.
- Docker socket access grants containers extensive control over the host. Orborus on the Wazuh server is convenient for a small lab, but a dedicated runtime VM gives stronger isolation.
- Keep the Windows host and host-only gateway reachable during tests. Make a rollback plan for any firewall response.
- Use only systems you own or are authorized to test.
