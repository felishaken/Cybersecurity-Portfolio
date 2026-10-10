
# Project 2: Wazuh SIEM Deployment and Troubleshooting

## Objective
Deploy and troubleshoot Wazuh components on Kali Linux, understand how the Indexer and Dashboard communicate, and verify that the services are running.

## Environment
- Operating System: Kali Linux
- Virtualization: VirtualBox
- SIEM Platform: Wazuh
- Available RAM: Approximately 3 GB allocated to the Kali VM

## Components
- Wazuh Indexer: Stores and indexes security event data.
- Wazuh Dashboard: Provides a web interface for monitoring and investigation.
- Wazuh Manager: Handles security event analysis and alerting.

## Problem Encountered
The Wazuh Dashboard initially reported connection failures to the Indexer at `127.0.0.1:9200`. The Indexer service was failing to remain running during startup.

## Troubleshooting
1. Checked the Indexer service status with `systemctl`.
2. Inspected Indexer logs for startup errors.
3. Confirmed the configured JVM heap size.
4. Checked whether port 9200 was listening.
5. Adjusted the systemd startup timeout to five minutes.
6. Reloaded systemd and restarted the Indexer.
7. Verified that the Indexer became active and listened on port 9200.
8. Restarted the Wazuh Dashboard.

## Verification
- Indexer service reported `active (running)`.
- Port 9200 was listening.
- An HTTPS request to the Indexer returned HTTP 401 Unauthorized, confirming that the endpoint responded and required authentication.
- Dashboard service reported `active (running)`.

## Lessons Learned
- A running Dashboard depends on a reachable Indexer.
- HTTP 401 is different from connection refused: it indicates an authentication requirement rather than a failure to establish the connection.
- Service status, logs, listening ports, and HTTP responses provide complementary troubleshooting evidence.

## Next Steps
- Confirm successful Dashboard login.
- Explore Wazuh security events and alerts.
- Investigate how endpoint agents send security data to the Manager.

