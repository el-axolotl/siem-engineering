# Splunk
Labs I used to learn Splunk

## How to ingest logs

- Splunk Technical Addon
    - Prefered method
- Splunk HEC Token
    - For when a log source has a way to push logs to Splunk, or you only have access to a log API endpoint. You will need to construct your own method to push or pull logs in most cases.
- Splunk Universal Forwarder
    - For when you have access to a log source's underlying OS. This is a small application you install on a host. You configure which directories on the host should be collected by the Splunk UF and sent to Splunk Cloud.

## Universal Forwarder For Windows

### Important File Locations

- ouputs.conf: A default exists at `C:\Program Files\SplunkUniversalForwarder\etc\system\default` which shouldn't be altered. Instead edit `C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf` instead.

- splunkd.log: A log about the splunk forwarder service. Located at `C:\Program Files\SplunkUniversalForwarder\var\log\splunk\splunkd.log`

### Splunk.exe Commands

A command to debug outputs

```cmd
splunk.exe btool outputs list --debug
```

A command to add a receiver. This will create your outputs.conf file if none exists at `C:\Program Files\SplunkUniversalForwarder\etc\system\local`. 

```cmd
splunk.exe add forward-server <splunk-instance-name>.splunkcloud.com:9997
```

### Troubleshooting

In Splunk Cloud, many errors and information logs can be found in the `_internal` index. If you are having problems with a Splunk technical-addon, I would begin looking for related logs in this index.
