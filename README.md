# SOC Alert Enrichment & Human-Approved Response Lab

A hands-on Security Operations Center (SOC) home lab that brings together **Wazuh**, **Shuffle**, **VirusTotal**, and a Kali Linux test machine to explore alert detection, indicator-of-compromise (IOC) enrichment, analyst notification, and controlled response.

> **Project status: COMPLETE — implemented scope.** The lab demonstrates Wazuh SSH-authentication alerting, forwarding selected alerts into Shuffle, VirusTotal IP-reputation enrichment, email notification/analyst approval, and Wazuh Active Response API dispatch. The repository includes evidence of workflow execution and API dispatch. Endpoint-level firewall state should still be verified during an end-to-end retest, especially after configuration changes. **I was not able to integrate TheHive into the final workflow because of hardware/resource limitations.** TheHive remains a planned future extension; automated TheHive case creation is not claimed as complete.

## Project overview

The project explores how a security alert can move from endpoint telemetry to enrichment, an analyst decision, and a controlled response. The principal use case is repeated failed SSH authentication attempts against a monitored Linux endpoint.

The implemented workflow covers:

1. **Generate telemetry:** Create controlled SSH authentication events in the lab.
2. **Detect:** Wazuh collects endpoint logs and triggers the configured SSH-authentication detection/correlation rule.
3. **Orchestrate:** Wazuh forwards selected alert JSON to a Shuffle webhook.
4. **Enrich and notify:** Shuffle extracts the source IP, queries VirusTotal for available reputation context, and sends an email asking the analyst whether to block the IP.
5. **Approve and respond:** The approved path requests a Wazuh Active Response using the `firewall-drop` command.
6. **Capture evidence:** Screenshots document the alert context, VirusTotal enrichment, and a Wazuh API dispatch result.

**Safety principle:** an alert alone should not initiate containment. The response action should only execute on the explicit Yes/approve branch. Confirm the source IP and target agent before approving a block.

## Architecture

![SOC automation lab architecture](assets/soc-automation-architecture.png)

The architecture diagram also shows TheHive as the intended case-management component. Because of hardware/resource limitations, TheHive was not integrated into the final automated workflow. It is outside the completed implementation scope.

## Execution evidence

### 1. Shuffle — SSH brute-force alert context

![Shuffle runtime alert context](assets/screenshots/01-shuffle-runtime-alert.png)

This screenshot shows the Shuffle runtime data for an SSH-authentication alert, including custom Wazuh rule ID `100003` and event context.

### 2. Wazuh — Active Response API result

![Wazuh Active Response API result](assets/screenshots/02-wazuh-active-response-result.png)

The captured result shows HTTP status `200`, one affected item, zero failed items, and the message `AR command was sent to all agents`. This demonstrates that the API accepted/dispatched the request in that captured run. It does **not**, by itself, confirm that the firewall rule was applied and remained active on the endpoint. The screenshot's request referenced agent ID `004`; verify the intended agent ID in the current lab before repeating the action.

### 3. VirusTotal — IP reputation lookup

![VirusTotal IP reputation lookup](assets/screenshots/03-virus-total-IOC-enrichment.png)

Shuffle queries VirusTotal using the source IP extracted from the Wazuh alert. The returned reputation and threat-intelligence context supports analyst triage before the containment decision. A reputation result is supporting intelligence, not proof by itself that an address is malicious.

## Technology stack

| Component | Purpose |
|---|---|
| Wazuh Manager and agents | Endpoint telemetry, SSH-authentication alerts, custom rules, and Active Response |
| Shuffle Cloud | Workflow authoring, alert handling, notifications, analyst approval, and orchestration |
| Self-hosted Shuffle Orborus and Docker Swarm worker | Executes workflow actions in the lab network |
| VirusTotal | IP-reputation / IOC enrichment |
| Kali Linux | Controlled lab testing and event generation |
| Ubuntu Server | Hosts Wazuh and the attempted TheHive stack |
| VirtualBox | Isolated lab networking with NAT and host-only interfaces |
| TheHive | Planned case-management extension; not integrated into the completed workflow due to hardware/resource limitations |

## Lab environment

The lab uses VirtualBox with NAT for updates and a host-only adapter for private lab traffic. These are reference values from the build notes; confirm each VM's current IP before using it in a workflow or response action.

| VM | Operating system | vCPU | RAM | Disk | Reference IP / notes |
|---|---|---:|---:|---:|---|
| `WAZUH-SERVER` | Ubuntu Server | 4 | 8 GB | 100 GB | `192.168.56.10/24` |
| `THEHIVE-SERVER` | Ubuntu Server | 4 | 12 GB | 100 GB | `192.168.56.30/24`; web UI intended at `http://192.168.56.30:9000` |
| `LINUX-ENDPOINT` | Ubuntu/Linux | — | — | — | Reference address `192.168.56.100`; confirm current agent ID and IP |
| `KALI-LINUX` | Kali Linux | 2 | 4 GB | 80 GB | Build notes include `192.168.56.107`; verify current address with `ip -br addr` |

Hardware values are allocations recorded in the build notes, not minimum requirements. Adjust allocations to available host resources.

## Implementation status

| Capability | Status | Evidence / notes |
|---|---|---|
| Wazuh endpoint telemetry | **Complete** | SSH-authentication events have appeared in Wazuh. |
| SSH brute-force detection | **Implemented** | Custom rule `100003` is used for repeated matching failed SSH attempts in a time window. The frequency/timeframe logic should not be described as strictly consecutive attempts without a separate stateful correlator. |
| Shuffle Cloud + self-hosted Orborus | **Complete** | The `shuffle-workers` service was observed at `1/1`; the HTTP, AI, Subflow, and Tools services were observed at `2/2`. |
| Alert forwarding and email notification | **Complete** | The workflow reached the email step and asked the analyst whether to block the source IP. |
| VirusTotal IP enrichment | **Implemented** | Screenshot included; reputation results provide context for the approval decision. |
| Analyst approval gate | **Implemented** | Keep the Wazuh response action on the Yes/approve branch and verify the No branch after workflow edits. |
| Wazuh API Active Response dispatch | **Dispatch evidence captured; endpoint validation recommended** | One captured API response reported HTTP `200`, one affected item, zero failed items, and `AR command was sent to all agents`. Verify endpoint Active Response logs and firewall state when retesting. |
| TheHive case creation | **Not integrated — hardware/resource limitation** | I was not able to integrate TheHive into the final workflow because of the available lab hardware/resources. It remains a future extension; no successful automated case creation is claimed. |

## Repository layout

```text
soc-automation-lab/
├── README.md
├── .gitignore
├── assets/
│   ├── soc-automation-architecture.png
│   └── screenshots/
│       ├── 01-shuffle-runtime-alert.png
│       ├── 02-wazuh-active-response-result.png
│       └── 03-virus-total-IOC-enrichment.png
├── docs/
│   ├── lab-setup.md
│   ├── workflow.md
│   ├── testing-checklist.md
│   └── security-notes.md
└── examples/
    ├── local_rules.xml.example
    └── ossec-integration.xml.example
```

## Documentation

- [Lab setup notes](docs/lab-setup.md)
- [Workflow design and response logic](docs/workflow.md)
- [Testing checklist](docs/testing-checklist.md)
- [Security notes](docs/security-notes.md)
- [Example Wazuh SSH correlation rule](examples/local_rules.xml.example)
- [Example Wazuh-to-Shuffle integration](examples/ossec-integration.xml.example)

## Key learning outcomes

- Interpreting SSH-authentication events and Wazuh rule metadata.
- Creating a custom correlation rule for repeated failed authentication attempts.
- Passing event fields into Shuffle and inspecting runtime variables.
- Using API authentication and troubleshooting JSON request-body schemas.
- Enriching alerts with IP reputation context before analyst review.
- Designing a human approval gate before a response request.
- Deploying a self-hosted Shuffle runtime with Docker and Docker Swarm.
- Troubleshooting API dispatch, agent selection, endpoint response behavior, and hardware/resource constraints.

## Known limitation: TheHive

I was not able to integrate TheHive into the final automated workflow because of hardware/resource limitations in the available lab environment. The architecture diagram records the intended direction, but automated TheHive alert/case creation is not part of the completed scope.

## Security and publication notes

This is an educational lab, not a production-ready SOC deployment. Confirm the source IP and target agent before executing a block action. An earlier alert included `192.168.56.1`, which may be a host-only gateway or host adapter rather than the attacker; do not assume it is malicious without verification.

Do not publish passwords, JWTs, API keys, Shuffle authentication values, webhook URLs containing secrets, TheHive credentials, password archives, or unredacted logs. Review screenshots and configuration examples before committing them. Keep management services private and test only against systems you own or are authorized to assess.
