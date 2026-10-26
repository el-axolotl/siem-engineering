# SIEM Engineering

Notes, labs, and lessons learned from working on SIEM engineering problems.

## Contents

### Data Collection

- [Right-Sizing Cloud Data](data-collection/cloud-logging.md): which AWS, Azure, and GCP logs to ingest, how to scope high-volume data events, and which fields to extract.
- [Splunk](data-collection/splunk/README.md): ways to get logs into Splunk (technical add-ons, HEC tokens, and the Universal Forwarder) and where to start troubleshooting.
    - [Splunk Universal Forwarder](data-collection/splunk/universal_forwarder/README.md): Windows file locations, `splunk.exe` commands, and an example `outputs.conf` for Splunk Cloud.

### Detection Validation

- [Stratus Red Team](detection-validation/stratus-red-team.md): an open-source, SIEM-agnostic tool that detonates cloud attack techniques so you can test whether your detections fire.

### Triage and Investigation

- [Datadog Bits Security Analyst](triage-and-investigation/datadog/bits-security-analyst.md): an AI agent in Datadog Cloud SIEM that investigates signals and recommends a verdict.
