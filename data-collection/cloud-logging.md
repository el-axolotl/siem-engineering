# Right-Sizing Cloud Data

One of the biggest problems in SIEM engineering is right-sizing data. Cloud service providers can generate a huge volume of logs, and most SIEMs charge by ingestion volume. Ingesting everything drives up cost and buries useful signals in noise. Ingesting too little leaves gaps in detections and investigations.

The goal is to collect the data that supports your security use cases (detections, investigations, and compliance) and to be deliberate about everything else.

## AWS

### CloudTrail Management Events

At the very least, ingest **AWS CloudTrail management events**. Management events record control-plane activity in your AWS accounts, such as sign-ins, IAM changes, changes to logging and security services, and network configuration changes. They are the foundation for most AWS detections and investigations.

Management events still produce a lot of logs, and not all of them add value to security investigations. Plan to tune which management events you ingest after the baseline is in place.

The CIS AWS Foundations Benchmark recommends a log metric filter and alarm for each of the activities below. These events are a good baseline for SIEM detections.

| Monitor for | Events | Reason |
| --- | --- | --- |
| Root user activity | Any event where `userIdentity.type` is `Root` | The root user has unrestricted access and should almost never be used. |
| Unauthorized API calls | Any event where `errorCode` is `*UnauthorizedOperation` or `AccessDenied*` | Monitor for reconnaissance, permission probing, or misused credentials. |
| Console sign-in without MFA | `ConsoleLogin` where `additionalEventData.MFAUsed` is not `Yes` | Monitor for console access that bypasses MFA. |
| Console authentication failures | `ConsoleLogin` where `errorMessage` is `Failed authentication` | Monitor for brute-force or password-guessing attempts. |
| IAM policy changes | `CreatePolicy`, `DeletePolicy`, `CreatePolicyVersion`, `DeletePolicyVersion`, `PutUserPolicy`, `PutRolePolicy`, `PutGroupPolicy`, `DeleteUserPolicy`, `DeleteRolePolicy`, `DeleteGroupPolicy`, `AttachUserPolicy`, `AttachRolePolicy`, `AttachGroupPolicy`, `DetachUserPolicy`, `DetachRolePolicy`, `DetachGroupPolicy` | Monitor for privilege escalation and permission changes. |
| CloudTrail configuration changes | `CreateTrail`, `UpdateTrail`, `DeleteTrail`, `StartLogging`, `StopLogging` | Monitor for attempts to disable or tamper with audit logging. |
| Disabling or deleting KMS keys | `DisableKey`, `ScheduleKeyDeletion` | Monitor for actions that can make encrypted data permanently unreadable. |
| S3 bucket policy changes | `PutBucketAcl`, `PutBucketPolicy`, `PutBucketCors`, `PutBucketLifecycle`, `PutBucketReplication`, `DeleteBucketPolicy`, `DeleteBucketCors`, `DeleteBucketLifecycle`, `DeleteBucketReplication` | Monitor for buckets being exposed publicly or replicated outside the organization. |
| AWS Config changes | `StopConfigurationRecorder`, `DeleteDeliveryChannel`, `PutDeliveryChannel`, `PutConfigurationRecorder` | Monitor for attempts to disable configuration tracking. |
| Security group changes | `AuthorizeSecurityGroupIngress`, `AuthorizeSecurityGroupEgress`, `RevokeSecurityGroupIngress`, `RevokeSecurityGroupEgress`, `CreateSecurityGroup`, `DeleteSecurityGroup` | Monitor for firewall rules that open access to resources. |
| Network ACL changes | `CreateNetworkAcl`, `CreateNetworkAclEntry`, `DeleteNetworkAcl`, `DeleteNetworkAclEntry`, `ReplaceNetworkAclEntry`, `ReplaceNetworkAclAssociation` | Monitor for subnet-level firewall changes. |
| Network gateway changes | `CreateCustomerGateway`, `DeleteCustomerGateway`, `CreateInternetGateway`, `DeleteInternetGateway`, `AttachInternetGateway`, `DetachInternetGateway` | Monitor for new paths into or out of a VPC. |
| Route table changes | `CreateRoute`, `CreateRouteTable`, `ReplaceRoute`, `ReplaceRouteTableAssociation`, `DeleteRoute`, `DeleteRouteTable`, `DisassociateRouteTable` | Monitor for traffic being rerouted. |
| VPC changes | `CreateVpc`, `DeleteVpc`, `ModifyVpcAttribute`, `CreateVpcPeeringConnection`, `AcceptVpcPeeringConnection`, `DeleteVpcPeeringConnection`, `RejectVpcPeeringConnection` | Monitor for changes to network boundaries and new connections between VPCs. |

### S3 Data Events

S3 data events record object-level activity (reads, writes, and deletes on objects in a bucket). They are not logged by default, and they can be extremely high volume, so you should not enable them on every bucket.

The CIS AWS Foundations Benchmark recommends object-level logging of both **read** and **write** events for S3 buckets. Collecting those logs for compliance is not the same as sending all of them to the SIEM. You can keep the full set in S3 for audits and forward only the high-value buckets to the SIEM.

**Identify the crown jewels first.** Before enabling S3 data events, meet with engineering architects and leaders to learn which buckets hold the most valuable or sensitive data. You are looking for buckets that contain:

- Sensitive data
- Proprietary data, such as source code buckets
- PII
- Customer data

Once these high-value (MVP) buckets are identified, scope S3 data event collection to those buckets and ingest the following event types:

| Event | Reason |
| --- | --- |
| `GetObject` | Monitor access to objects. |
| `PutObject` | Monitor for new objects. |
| `CopyObject` | Monitor for objects being copied. |
| `DeleteObject` | Monitor for objects being deleted. |
| `CreateMultipartUpload` | Monitor for large object uploads via multiple parts. |
| `RestoreObject` | Monitor for the restoration of archived objects. |

### Field Extraction

Think carefully about which fields you extract from AWS logs. CloudTrail events contain many nested fields, and ingesting every raw field adds a lot of volume that may not help your security goals. Extract the fields that support your detections and investigations, and keep the original event ID so analysts can find the full raw event when they need it.

For both management and data events, extract at least these fields. They include the fields that the CIS monitoring recommendations above rely on:

| Field | Description |
| --- | --- |
| `eventTime` | Time that the event occurred. |
| `eventID` | Original event ID for reference in raw data. |
| `eventName` | The API action that was called, such as `ConsoleLogin` or `StopLogging`. |
| `eventSource` | The AWS service that received the request, such as `iam.amazonaws.com`. |
| `eventType` | The type of event, used to separate user activity from AWS service events (`AwsServiceEvent`). |
| `recipientAccountId` | AWS account ID where the activity took place. |
| `awsRegion` | AWS region where the activity took place. |
| `sourceIPAddress` | The source IP of the requester. |
| `userAgent` | The user agent of the requester. |
| `userIdentity.type` | The type of identity that made the request, such as `Root`, `IAMUser`, or `AssumedRole`. |
| `userIdentity.arn` | The AWS ARN of the requester. |
| `userIdentity.accessKeyId` | The AWS access key ID. |
| `userIdentity.invokedBy` | The AWS service that made the request on the user's behalf, if any. Used to exclude service activity from root user alerts. |
| `errorCode` | Error code, if available. |
| `errorMessage` | The error message, if available. Used to find failed console sign-ins. |
| `additionalEventData.MFAUsed` | Whether MFA was used for a console sign-in. |
| `responseElements.ConsoleLogin` | Whether a console sign-in succeeded or failed. |

## Azure

### Activity Log

At the very least, ingest the **Azure Activity Log**. The Activity Log records control-plane activity in each subscription, such as creating, updating, and deleting resources, policy changes, and Microsoft Defender for Cloud alerts. It is Azure's equivalent of CloudTrail management events.

Azure keeps the Activity Log for 90 days but doesn't export it by default. The CIS Microsoft Azure Foundations Benchmark recommends a diagnostic setting on every subscription that exports at least the **Administrative**, **Alert**, **Policy**, and **Security** categories. Other categories, such as Service Health, Resource Health, Autoscale, and Recommendation, are operational and add little value to security investigations.

User sign-ins are not in the Activity Log. They are in the Microsoft Entra ID sign-in and audit logs, which are a separate log source.

The CIS Microsoft Azure Foundations Benchmark recommends an activity log alert for each of the operations below:

| Monitor for | Operations | Reason |
| --- | --- | --- |
| Policy assignment changes | `Microsoft.Authorization/policyAssignments/write`, `Microsoft.Authorization/policyAssignments/delete` | Monitor for attempts to remove or weaken Azure Policy guardrails. |
| Network security group changes | `Microsoft.Network/networkSecurityGroups/write`, `Microsoft.Network/networkSecurityGroups/delete` | Monitor for firewall rules that open access to resources. |
| Security solution changes | `Microsoft.Security/securitySolutions/write`, `Microsoft.Security/securitySolutions/delete` | Monitor for attempts to disable or change security tooling. |
| SQL Server firewall rule changes | `Microsoft.Sql/servers/firewallRules/write`, `Microsoft.Sql/servers/firewallRules/delete` | Monitor for databases being exposed to new networks. |
| Public IP address changes | `Microsoft.Network/publicIPAddresses/write`, `Microsoft.Network/publicIPAddresses/delete` | Monitor for resources being exposed to the internet. |

### Blob Storage Data Events

Blob Storage resource logs record object-level activity, similar to S3 data events. They are off by default and can be extremely high volume. The CIS benchmark recommends enabling the `StorageRead`, `StorageWrite`, and `StorageDelete` log categories for Blob Storage. As with S3, keep the full set for compliance and forward only the high-value storage accounts to the SIEM.

**Identify the crown jewels first.** Meet with engineering architects and leaders to identify the storage accounts and containers that hold sensitive data, proprietary data, PII, or customer data. Then ingest the following operations from those storage accounts:

| Operation | Reason |
| --- | --- |
| `GetBlob` | Monitor access to blobs. |
| `PutBlob` | Monitor for new blobs. |
| `CopyBlob` | Monitor for blobs being copied. |
| `DeleteBlob` | Monitor for blobs being deleted. |
| `PutBlock`, `PutBlockList` | Monitor for large blob uploads via multiple blocks. |
| `SetBlobTier`, `UndeleteBlob` | Monitor for blobs being restored from the archive tier or from soft delete. |

The CIS benchmark also recommends enabling logging for Azure Key Vault. Key Vault holds secrets, keys, and certificates, so it is usually worth ingesting.

### Field Extraction

The Activity Log and Blob Storage logs share the same top-level Azure resource log fields. Extract at least these fields:

| Field | Description |
| --- | --- |
| `time` | Time that the event occurred. |
| `correlationId` | ID for grouping related events and for finding the original event in raw data. |
| `operationName` | The operation that was performed, such as `Microsoft.Network/networkSecurityGroups/write` or `GetBlob`. |
| `category` | The log category, such as `Administrative` or `StorageRead`. |
| `resultType` | The result of the operation, such as success or failure. |
| `resourceId` | The full ID of the resource. The subscription and resource group can be parsed from it. |
| `callerIpAddress` | The source IP of the requester. |
| `identity` | The identity of the requester. In the Activity Log, this holds the caller's claims, such as UPN, object ID, tenant ID, and app ID. |
| `identity.type` | Blob Storage only. How the request was authorized, such as OAuth, SAS, or account key. Access with an account key or SAS is not tied to a user, which matters during investigations. |
| `statusText` | Blob Storage only. The result of the request, such as `Success` or `AuthorizationError`. |
| `uri` | Blob Storage only. The URI of the blob that was requested. |
| `properties.userAgentHeader` | Blob Storage only. The user agent of the requester. |

## GCP

### Cloud Audit Logs

At the very least, ingest **Admin Activity audit logs** from Cloud Audit Logs. They record API calls that change the configuration or metadata of resources, and they are GCP's equivalent of CloudTrail management events. Admin Activity logs are always on and can't be disabled. To get them into a SIEM, create an organization-level aggregated log sink to Pub/Sub, Cloud Storage, or BigQuery.

The CIS Google Cloud Platform Foundation Benchmark recommends a log metric filter and alert for each of the activities below:

| Monitor for | Filter | Reason |
| --- | --- | --- |
| Project ownership changes | `protoPayload.serviceName="cloudresourcemanager.googleapis.com"` and `ProjectOwnership` or `projectOwnerInvitee`, or `protoPayload.serviceData.policyDelta.bindingDeltas` that `ADD` or `REMOVE` `roles/owner` | Monitor for changes to the highest privilege in a project. |
| Audit configuration changes | `protoPayload.methodName="SetIamPolicy"` with `protoPayload.serviceData.policyDelta.auditConfigDeltas` | Monitor for attempts to disable or reduce audit logging. |
| Custom role changes | `google.iam.admin.v1.CreateRole`, `google.iam.admin.v1.DeleteRole`, `google.iam.admin.v1.UpdateRole` | Monitor for privilege escalation through custom roles. |
| VPC firewall rule changes | `compute.firewalls.insert`, `compute.firewalls.patch`, `compute.firewalls.delete` | Monitor for firewall rules that open access to resources. |
| VPC route changes | `compute.routes.insert`, `compute.routes.delete` | Monitor for traffic being rerouted. |
| VPC network changes | `compute.networks.insert`, `compute.networks.patch`, `compute.networks.delete`, `compute.networks.addPeering`, `compute.networks.removePeering` | Monitor for changes to network boundaries and new connections between networks. |
| Cloud Storage IAM changes | `storage.setIamPermissions` | Monitor for buckets being exposed publicly or to new identities. |
| Cloud SQL configuration changes | `cloudsql.instances.update` | Monitor for databases being exposed or having security settings changed. |

Compute Engine method names can include an API version prefix, such as `v1.compute.firewalls.insert` or `beta.compute.networks.insert`. Match on the method name with "contains" logic rather than an exact match.

### Cloud Storage Data Events

Data Access audit logs record reads and writes of user data, similar to S3 data events. They are off by default and can be extremely high volume. The CIS benchmark recommends enabling `DATA_READ` and `DATA_WRITE` logs for all services and all users. As with S3, keep the full set for compliance and forward only the high-value buckets to the SIEM.

Data Access logs are turned on per service at the project, folder, or organization level, not per bucket. Scope SIEM ingestion by filtering the log sink on the bucket name (`resource.labels.bucket_name`).

**Identify the crown jewels first.** Meet with engineering architects and leaders to identify the buckets that hold sensitive data, proprietary data, PII, or customer data. Then ingest the following operations from those buckets:

| Operation | Log type | Reason |
| --- | --- | --- |
| Get object data | `DATA_READ` | Monitor access to objects. |
| Create objects | `DATA_WRITE` | Monitor for new objects. |
| Copy objects | `DATA_READ` | Monitor for objects being copied. |
| Delete objects | `DATA_WRITE` | Monitor for objects being deleted. |
| XML API multipart uploads | `DATA_WRITE` | Monitor for large object uploads via multiple parts. |
| Restore soft-deleted objects | `DATA_WRITE` | Monitor for the restoration of deleted objects. |

### Field Extraction

Admin Activity and Data Access audit logs share the same format. Extract at least these fields:

| Field | Description |
| --- | --- |
| `timestamp` | Time that the event occurred. |
| `insertId` | Unique ID of the log entry for reference in raw data. |
| `logName` | The audit log the entry belongs to, such as `activity` or `data_access`. |
| `resource.type` | The type of resource, such as `gcs_bucket` or `gce_firewall_rule`. |
| `resource.labels.project_id` | The project where the activity took place. |
| `protoPayload.serviceName` | The service that received the request, such as `storage.googleapis.com`. |
| `protoPayload.methodName` | The API action that was called. |
| `protoPayload.resourceName` | The resource that was acted on. |
| `protoPayload.authenticationInfo.principalEmail` | The identity of the requester. |
| `protoPayload.requestMetadata.callerIp` | The source IP of the requester. |
| `protoPayload.requestMetadata.callerSuppliedUserAgent` | The user agent of the requester. |
| `protoPayload.status.code` | Error code, if the request failed. |
| `protoPayload.serviceData.policyDelta` | IAM policy changes. Used by the CIS project ownership and audit configuration filters. |

### Google Workspace

If the organization uses Google Workspace, also ingest the **Google Cloud Identity audit logs**. These are Google Workspace logs (such as admin console activity, sign-ins, and group membership changes), separate from Cloud Audit Logs.

## Resources

- [CIS Amazon Web Services Foundations Benchmark](https://www.cisecurity.org/benchmark/amazon_web_services)
- [CIS AWS Foundations Benchmark in AWS Security Hub](https://docs.aws.amazon.com/securityhub/latest/userguide/cis-aws-foundations-benchmark.html)
- [AWS Security Hub CloudWatch controls (CIS metric filter patterns)](https://docs.aws.amazon.com/securityhub/latest/userguide/cloudwatch-controls.html)
- [CIS Microsoft Azure Foundations Benchmark](https://www.cisecurity.org/benchmark/azure)
- [Azure Activity Log event schema](https://learn.microsoft.com/en-us/azure/azure-monitor/platform/activity-log-schema)
- [Azure Storage logged operations](https://learn.microsoft.com/en-us/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages)
- [CIS Google Cloud Platform Foundation Benchmark](https://www.cisecurity.org/benchmark/google_cloud_computing_platform)
- [Cloud Audit Logs overview](https://docs.cloud.google.com/logging/docs/audit)
- [Cloud Audit Logs with Cloud Storage](https://docs.cloud.google.com/storage/docs/audit-logging)
