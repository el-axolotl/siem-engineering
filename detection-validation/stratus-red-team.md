# Stratus Red Team

[Stratus Red Team](https://github.com/DataDog/stratus-red-team) is an open-source adversary emulation tool for the cloud, often described as "Atomic Red Team for the cloud." It is maintained by Datadog and ships with a library of attack techniques mapped to MITRE ATT&CK. Techniques cover AWS, Azure, GCP, Microsoft Entra ID, and Kubernetes.

Stratus Red Team is SIEM agnostic. It runs attack techniques in your cloud environment, so any SIEM that collects that environment's logs can be used to look for the activity.

## How It Works

Stratus Red Team connects to your cloud service provider. It then detonates emulated attacks against test infrastructure. These are real API calls that an attacker would make, such as stopping CloudTrail logging or exfiltrating an EBS snapshot. Your cloud provider records the activity in its audit logs, such as AWS CloudTrail, the Azure Activity Log, or GCP Cloud Audit Logs. Your downstream SIEM, which collects those cloud logs, should then pick up the signals, and you can use them to build and validate detections. See [Right-Sizing Cloud Data](../data-collection/cloud-logging.md) for which cloud logs to collect.

> [!CAUTION]
> Run Stratus Red Team only in a dedicated test account, subscription, or project. Detonations make real changes, and some of them are intentionally destructive.

Each technique goes through these stages:

| Stage | What happens |
| --- | --- |
| Warm up | Creates the prerequisite infrastructure for the technique, such as an S3 bucket or an IAM user, using Terraform. |
| Detonate | Runs the attack technique against the live environment. |
| Revert | Undoes the detonation's side effects so the technique can be detonated again (for techniques that support it). |
| Clean up | Destroys the prerequisite infrastructure so no resources are left behind. |

## Resources

- [Stratus Red Team on GitHub](https://github.com/DataDog/stratus-red-team)
- [Stratus Red Team documentation: Getting Started and Concepts](https://stratus-red-team.cloud/user-guide/getting-started/#concepts)
