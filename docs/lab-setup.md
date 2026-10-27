# Lab Setup Notes

This document records the lab build and relevant configuration. It is not a one-command installation guide; verify package versions and settings against the versions actually installed.

## Virtual machine specifications

| VM | OS | vCPU | RAM | Storage | Network adapters |
|---|---|---:|---:|---:|---|
| `WAZUH-SERVER` | Ubuntu Server | 4 | 8 GB | 100 GB | NAT + Host-only |
| `THEHIVE-SERVER` | Ubuntu Server | 4 | 12 GB | 100 GB | NAT + Host-only |
| `KALI-LINUX` | Kali Linux | 2 | 4 GB | 80 GB | NAT + Host-only |

The host-only network uses the `192.168.56.0/24` range. NAT is used for operating-system updates; host-only networking is used for private lab traffic. Do not expose management services directly to the public internet.

## Wazuh server

- Reference host-only address: `192.168.56.10/24`.
- Wazuh was installed using the installation assistant.
- The manager and endpoint have generated SSH authentication alerts, including the custom rule ID `100003`.
- Wazuh Active Response command dispatch was exercised via the API. A captured Shuffle result returned HTTP `200`, one affected item, zero failed items, and `AR command was sent to all agents`.
- The screenshot for this captured action references agent ID `004`; older lab notes referenced ID `003` for `LINUX-ENDPOINT`. Confirm the correct agent ID from the current Wazuh manager before testing.

## TheHive server

- Reference host-only address: `192.168.56.30/24`.
- The attempted installation used Cassandra, Elasticsearch, and TheHive.
- The earlier build used `127.0.0.1:9200` for local Elasticsearch access and disabled Elasticsearch HTTP TLS because certificate trust could not be resolved in the isolated lab. Treat this strictly as a lab-specific configuration, not a production recommendation.
- The original TheHive base URL was configured as `http://192.168.56.30:9000` for host-only access.
- **TheHive case creation was not integrated into the final Shuffle workflow because of hardware/resource limitations.** It remains a potential future extension and is not represented as completed.

## Kali Linux and monitored endpoints

- Kali was intended to send controlled authentication activity to the lab and was enrolled as a Wazuh agent.
- Historical build notes listed Kali at `192.168.56.107`; other test configurations used different addresses. Verify the current interface configuration with `ip -br addr` before testing.
- For SSH use cases, verify the SSH server is installed, active, listening on the host-only interface, and permitted by the endpoint firewall. An installed SSH client does not mean the machine accepts incoming connections.
- Ensure each Wazuh agent's manager address matches the lab's current Wazuh server address.

## Shuffle Cloud and self-hosted Orborus

- Shuffle Cloud was used to author the workflow.
- A self-hosted Orborus runtime named `SOC Lab` was deployed on `WAZUH-SERVER` so workflow workers could access private lab services.
- At the last successful service check, `shuffle-workers` was `1/1`; HTTP, AI, Subflow, and Tools services were `2/2`.
- Reference Wazuh API endpoint: `https://192.168.56.10:55000`, provided that the API is listening on the address and is reachable from the worker.
- Wazuh API credentials are distinct from the Ubuntu login. Obtain a JWT from the Wazuh API and pass it according to the expected authentication scheme. Never commit the token, password archive, or credentials.
- The API Active Response action was successfully dispatched in the captured test; keep the target agent ID synchronized with the current Wazuh installation.

## Resource limitation

The completed workflow focuses on Wazuh detection, Shuffle orchestration/notification/approval, and Wazuh Active Response dispatch. I was not able to integrate TheHive into the final workflow because the available host hardware/resources were insufficient to support the intended combined lab reliably.

## Sources to consult during maintenance

- Wazuh documentation: https://documentation.wazuh.com/
- TheHive documentation: https://docs.strangebee.com/
- Docker Engine for Ubuntu: https://docs.docker.com/engine/install/ubuntu/
- Shuffle documentation: https://shuffler.io/docs

Keep credentials and tokens out of shell history, screenshots, logs copied to public issues, and Git commits.
