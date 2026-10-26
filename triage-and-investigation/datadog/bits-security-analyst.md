# Bits Security Analyst

Bits Security Analyst is an AI agent in Datadog Cloud SIEM, part of Datadog's Bits AI. It investigates Cloud SIEM signals on its own. For each signal, it plans investigation steps using MITRE ATT&CK as a guide, queries related signals and logs, and adjusts its steps based on what it finds. It then recommends a verdict of **Benign** or **Suspicious**, along with the evidence behind that verdict and suggested next steps.

It covers signals from cloud, identity, SaaS, and endpoint sources, such as AWS CloudTrail, Azure, GCP, Okta, Microsoft Entra ID, Microsoft 365, GitHub, CrowdStrike, and SentinelOne.

## Why It's Useful

- **Less manual triage:** Analysts no longer have to investigate every alert from scratch. Bits Security Analyst does the first pass, which helps with alert fatigue and lets analysts spend their time on signals that need human judgment.
- **Faster response:** Investigations start as soon as a signal fires, so by the time an analyst looks at a signal, the initial work is already done.
- **Transparent results:** Each investigation shows the queries Bits ran, the results, and the reasoning at each step, so analysts can check the work instead of taking the verdict on trust.
- **Actionable next steps:** It suggests remediation and rule tuning, such as suppressions for noisy attributes. Analysts can then act from the signal, for example by declaring an incident or running a SOAR workflow.

## Resources

- [Datadog blog: Bits AI Security Analyst](https://www.datadoghq.com/blog/bits-ai-security-analyst/)
- [Datadog docs: Bits Security Analyst](https://docs.datadoghq.com/bits_ai/bits_security_analyst)
