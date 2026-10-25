# Lab Setup Notes

This document consolidates the supplied SOC lab build notes. It is a record of the lab configuration, not a one-command installation guide; commands and package versions should be checked against the versions actually installed.

## Virtual machine specifications

| VM | OS | vCPU | RAM | Storage | Network adapters |
|---|---|---:|---:|---:|---|
| `WAZUH-SERVER` | Ubuntu Server | 4 | 8 GB | 100 GB | NAT + Host-only |
| `THEHIVE-SERVER` | Ubuntu Server | 4 | 12 GB | 100 GB | NAT + Host-only |
| `KALI-LINUX` | Kali Linux | 2 | 4 GB | 80 GB | NAT + Host-only |

The lab host-only network uses the `192.168.56.0/24` range. NAT is used for operating-system updates; host-only networking is used for private lab traffic. Do not expose these management services directly to the public internet.

## Wazuh server

- Assign a static host-only address of `192.168.56.10/24`.
- Install Wazuh using the official installation assistant.
- Save installation credentials securely and outside the Git repository.
- Confirm that the manager, indexer, dashboard, and API services are healthy before adding integrations.

## TheHive server

- Assign a static host-only address of `192.168.56.30/24`.
- Install updates and enable SSH for administration.
- Install the required Java runtime and dependencies, then configure Cassandra, Elasticsearch, and TheHive according to the relevant version-specific documentation.
- The original build notes mention using Python 3.9 to run `cqlsh` because the system Python version caused compatibility issues. Avoid changing the system Python symlink; invoke the compatible interpreter explicitly.
- The original Elasticsearch setup used `127.0.0.1:9200` for local service access and disabled Elasticsearch HTTP TLS because certificate trust could not be resolved in the isolated lab. Treat this strictly as a lab-specific choice, not as a production recommendation.
- The original TheHive base URL was configured as `http://192.168.56.30:9000` for access from the host-only network.

## Kali Linux and monitored endpoints

- Install and enroll the Wazuh agent on the Kali VM if it is intended to send events to Wazuh.
- Original setup notes list Kali at `192.168.56.107`; verify the current interface configuration with `ip -br addr` before running tests because addresses changed during troubleshooting.
- For SSH use cases, verify the SSH server is installed, active, listening on the host-only interface, and permitted by the endpoint firewall. An installed SSH client does not mean the machine accepts incoming SSH connections.
- Ensure each Wazuh agent's configured manager address matches the lab's current Wazuh server address.

## Wazuh API and Shuffle runtime

- Shuffle Cloud is used for workflow design; a self-hosted Orborus runtime named `SOC Lab` was deployed on `WAZUH-SERVER` to reach services on the private lab network.
- At the last recorded check, the worker service was running and the HTTP, AI, Subflow, and Tools services had their desired replicas.
- The Wazuh API endpoint is normally available at `https://192.168.56.10:55000` within the lab, provided the API is listening on that address and the container has a route to it.
- API credentials are distinct from the Ubuntu login. Use the Wazuh API username/password to obtain a JWT, then pass the token in the API's expected authentication field/header. Do not commit the token or the password archive.
- The `get-api` workflow node previously returned `401 Unauthorized: Invalid credentials`; authenticate with the correct Wazuh API account before considering the API-response path verified.

## Sources to consult during maintenance

- Wazuh official documentation: https://documentation.wazuh.com/
- TheHive documentation: https://docs.strangebee.com/
- Docker Engine for Ubuntu: https://docs.docker.com/engine/install/ubuntu/
- Shuffle documentation: https://shuffler.io/docs

Review commands and settings against the installed versions before applying changes. Keep credentials and tokens out of shell history, screenshots, logs copied to public issues, and Git commits.
