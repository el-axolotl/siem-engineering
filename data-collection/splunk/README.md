# Splunk

Notes and lessons learned from working with Splunk as a SIEM.

## How to ingest logs

- Splunk Technical Addon
    - Prefered method
- Splunk HEC Token
    - For when a log source has a way to push logs to Splunk, or you only have access to a log API endpoint. You will need to construct your own method to push or pull logs in most cases.
- [Splunk Universal Forwarder](universal_forwarder/README.md)
    - For when you have access to a log source's underlying OS. This is a small application you install on a host. You configure which directories on the host should be collected by the Splunk UF and sent to Splunk Cloud.

## Troubleshooting

In Splunk Cloud, many errors and information logs can be found in the `_internal` index. If you are having problems with a Splunk technical-addon, I would begin looking for related logs in this index.
