# Splunk Universal Forwarder

The Splunk Universal Forwarder (UF) is for when you have access to a log source's underlying OS. It is a small application you install on a host. You configure which directories on the host should be collected by the Splunk UF and sent to Splunk Cloud.

## Files in This Folder

- [outputs.conf](outputs.conf): Example outputs.conf that forwards to Splunk Cloud on port 9997 (default port).

## Universal Forwarder For Windows

### Important File Locations

- outputs.conf: A default exists at `C:\Program Files\SplunkUniversalForwarder\etc\system\default` which shouldn't be altered. Instead edit `C:\Program Files\SplunkUniversalForwarder\etc\system\local\outputs.conf`.

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
