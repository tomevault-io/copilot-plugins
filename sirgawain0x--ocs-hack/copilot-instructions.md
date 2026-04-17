## ocs-hack

> service quotas for Chainlink Runtime Environment (CRE) workflows.

This page documents the service quotas for Chainlink Runtime Environment (CRE) workflows.
note
SUBJECT TO CHANGE
The quotas documented on this page are subject to change. Check back regularly for updates.

Per-owner quotas
These quotas apply to each workflow owner (user account) within an organization.
Quota Key	Description	Value
PerOwner.WorkflowExecutionConcurrencyLimit	Maximum number of workflows that can execute simultaneously	5
PerOwner.VaultSecretsLimit	Maximum number of secrets that can be stored per owner	100
note

QUOTA ENFORCEMENT
When workflow executions exceed the quotas defined above, they are queued and automatically retried for up to 10 minutes. If an execution cannot be completed within this 10-minute window, it will be dropped and will not run.
Per-workflow quotas
These quotas apply to each individual workflow.
Workflow deployment quotas
Quota Key	Description	Value
PerWorkflow.WASMBinarySizeLimit	Maximum size of the compiled WASM binary	100 MB
PerWorkflow.WASMCompressedBinarySizeLimit	Maximum size of the compressed WASM binary	20 MB
PerWorkflow.WASMConfigSizeLimit	Maximum size of the workflow configuration	1 MB
Trigger quotas
Quota Key	Description	Value
PerWorkflow.TriggerSubscriptionLimit	Maximum number of triggers that can be registered to a single workflow	10
Execution quotas
Quota Key	Description	Value
PerWorkflow.ExecutionConcurrencyLimit	Maximum number of concurrent executions for a specific workflow	5
PerWorkflow.ExecutionTimeout	Maximum total execution time for a single workflow run	5 minutes
PerWorkflow.WASMMemoryLimit	Maximum memory allocated to a workflow	100 MB
PerWorkflow.ExecutionResponseLimit	Maximum size of the data a workflow can return	100 KB
General capability quotas
Quota Key	Description	Value
PerWorkflow.CapabilityConcurrencyLimit	Maximum concurrent capability calls (HTTP, EVM read/write, secrets) that can execute within a workflow	3
PerWorkflow.CapabilityCallTimeout	Maximum time a single capability call can take to complete	3 minutes
Secrets quotas
Quota Key	Description	Value
PerWorkflow.WASMSecretsSizeLimit	Maximum total size of secrets accessible to a workflow	1 MB
PerWorkflow.SecretsConcurrencyLimit	Maximum number of secrets that can be fetched concurrently. Learn how to fetch multiple secrets.	5
Consensus quotas
Quota Key	Description	Value
PerWorkflow.Consensus.ObservationSizeLimit	Maximum size of data that can be passed to consensus aggregation	25 KB
PerWorkflow.Consensus.CallLimit	Maximum number of consensus calls per workflow execution	2,000
Logging quotas
Quota Key	Description	Value
PerWorkflow.LogLineLimit	Maximum size of a single log line	1 KB
PerWorkflow.LogEventLimit	Maximum number of log events per workflow execution	1,000
Trigger-specific quotas
Cron trigger
Quota Key	Description	Value
PerWorkflow.CRONTrigger.FastestScheduleInterval	Minimum interval between cron trigger fires	30 seconds
note
MINIMUM CRON SCHEDULE
While the rate quota allows firing once per 30 seconds, you should consider the Concurrent Workflow Instances quota when setting your cron schedule to avoid overlapping executions.
HTTP trigger
Quota Key	Description	Value
PerWorkflow.HTTPTrigger.RateLimit	Maximum rate at which an HTTP trigger can fire	Rate: 1 per 30 seconds
Burst: 3
EVM log trigger
Quota Key	Description	Value
PerWorkflow.LogTrigger.EventRateLimit	Maximum rate at which log events can be processed	Rate: 10 per 6 seconds
Burst: 10
PerWorkflow.LogTrigger.FilterAddressLimit	Maximum number of contract addresses that can be monitored	5
PerWorkflow.LogTrigger.FilterTopicsPerSlotLimit	Maximum number of topic values that can be specified within a single topic position (Topics[0], Topics[1], Topics[2], or Topics[3]). Learn about topic filtering.	10
PerWorkflow.LogTrigger.EventSizeLimit	Maximum size of a single log event	5 KB
Capability-specific quotas
EVM write capability
Quota Key	Description	Value
PerWorkflow.ChainWrite.TargetsLimit	Maximum number of destination chains for write operations	10
PerWorkflow.ChainWrite.ReportSizeLimit	Maximum size of a report payload	5 KB
PerWorkflow.ChainWrite.EVM.TransactionGasLimit	Gas quota per EVM transaction	5,000,000
EVM read capability
Quota Key	Description	Value
PerWorkflow.ChainRead.CallLimit	Maximum number of EVM read calls per workflow execution	10
PerWorkflow.ChainRead.LogQueryBlockLimit	Maximum number of blocks that can be queried for historical logs	100
PerWorkflow.ChainRead.PayloadSizeLimit	Maximum size of an EVM read request payload	5 KB

HTTP capability
Quota Key	Description	Value
PerWorkflow.HTTPAction.CallLimit	Maximum number of HTTP requests per workflow execution	5
PerWorkflow.HTTPAction.ResponseSizeLimit	Maximum size of an HTTP response	100 KB
PerWorkflow.HTTPAction.ConnectionTimeout	Maximum time to establish an HTTP connection	10 seconds
PerWorkflow.HTTPAction.RequestSizeLimit	Maximum size of an HTTP request payload	10 KB
PerWorkflow.HTTPAction.CacheAgeLimit	Maximum time HTTP responses can be cached	10 minutes
Quota increases

Contact us to discuss quota increases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sirgawain0x) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
