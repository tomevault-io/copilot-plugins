## llm-d-workload-variant-autoscaler

> - Follow the standard Go code style and conventions. Use `gofmt` for formatting and adhere to idiomatic Go practices.

# Claude Code Assistant Guidelines

## Go Code Style

- Follow the standard Go code style and conventions. Use `gofmt` for formatting and adhere to idiomatic Go practices.
- Follow best practices from the [Effective Go](https://go.dev/doc/effective_go) guide:

### Naming Conventions
- Use **MixedCaps** or **mixedCaps** rather than underscores for multi-word names
- Package names should be short, lowercase, single-word names
- Getters don't use "Get" prefix (use `obj.Name()` not `obj.GetName()`)
- Interface names use "-er" suffix for single-method interfaces (e.g., `Reader`, `Writer`)

### Formatting
- Use `gofmt` for consistent formatting (tabs for indentation, spaces for alignment)
- Line length: no strict limit, but keep lines reasonable
- Group related declarations together

### Error Handling
- Return errors as the last return value
- Check errors immediately after the call
- Provide context with `fmt.Errorf` and error wrapping

### Logging
- Use `ctrl.Log` for structured logging
- Keep log fields consistent and meaningful
- Avoid logging sensitive data

### Documentation
- Every exported name should have a doc comment
- Start comments with the name being described
- Use complete sentences

### Concurrency
- Share memory by communicating; don't communicate by sharing memory
- Use channels to orchestrate goroutines
- Always handle goroutine cleanup and cancellation properly

### Project Structure
- Keep packages focused and cohesive
- Avoid circular dependencies
- Place tests in `*_test.go` files

## Documentation

Prefer placing documentation in the `docs/` directory.

There are 3 main types of documentation targeting different audiences:

1. **Developer Documentation** - For contributors and maintainers of this project
   - Architecture decisions
   - Development setup and workflow
   - Contributing guidelines
   - usually in the `docs/developer-guide/` subdirectory

2. **Administrator Documentation** - For operators deploying and managing the autoscaler controller
   - Installation and configuration
   - Deployment guidelines
   - Monitoring and troubleshooting
   - usually located under the `docs/user-guide/` directory (for example, in an admin-focused subdirectory)

3. **End-User Documentation** - For application developers creating applications that use the autoscaler
   - Usage guides and examples
   - API reference
   - Best practices and common patterns
   - usually located under the `docs/user-guide/` directory (for example, in an end-user-focused subdirectory)

## E2E Testing

- use make targets for running e2e tests (e.g., `make test-e2e-smoke` or `make test-e2e-full`) and document the process in `docs/developer-guide/testing.md`
- use `make test` for unit tests
- **Never use images from docker.io in e2e tests.** All container images must use fully-qualified registry paths (e.g., `registry.k8s.io/`, `quay.io/`, or a private registry). Do not rely on Docker Hub as a default registry.

## CLI Tools

### llm-d Inference Scheduler EPP CLI Reference

This section documents the command-line flags and environment variables supported by the llm-d inference scheduler EPP (Endpoint Picker). The EPP inherits its CLI from `gateway-api-inference-extension`.

#### Main Branch (Latest)

Uses `gateway-api-inference-extension` at commit `fd30cb97714a` (post-v1.3.0).

##### Command-Line Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--grpc-port` | int | `9002` | gRPC port used for communicating with Envoy proxy |
| `--ha-enable-leader-election` | bool | `false` | Enables leader election for high availability. When enabled, readiness probes will only pass on the leader |
| `--pool-group` | string | `inference.networking.k8s.io` | Kubernetes resource group of the InferencePool this Endpoint Picker is associated with |
| `--pool-namespace` | string | `""` | Namespace of the InferencePool this Endpoint Picker is associated with |
| `--pool-name` | string | `""` | Name of the InferencePool this Endpoint Picker is associated with |
| `--endpoint-selector` | string | `""` | Selector to filter model server pods on, only 'key=value' pairs are supported. Format: comma-separated list of key=value pairs (e.g., 'app=vllm-llama3-8b-instruct,env=prod') |
| `--endpoint-target-ports` | []int | `[]` | Target ports of model server pods. Format: comma-separated list of numbers (e.g., '3000,3001,3002') |
| `--disable-endpoint-subset-filter` | bool | `false` | Disables respecting the x-gateway-destination-endpoint-subset metadata for dispatching requests in EPP |
| `--model-server-metrics-scheme` | string | `http` | Protocol scheme used in scraping metrics from endpoints |
| `--model-server-metrics-path` | string | `/metrics` | URL path used in scraping metrics from endpoints |
| `--model-server-metrics-port` | int | `0` | **DEPRECATED**: Port to scrape metrics from endpoints |
| `--model-server-metrics-https-insecure-skip-verify` | bool | `true` | Disable certificate verification when using 'https' scheme for model-server-metrics-scheme |
| `--refresh-metrics-interval` | duration | `50ms` | Interval to refresh metrics |
| `--refresh-prometheus-metrics-interval` | duration | `5s` | Interval to flush Prometheus metrics |
| `--metrics-staleness-threshold` | duration | `2s` | Duration after which metrics are considered stale |
| `--total-queued-requests-metric` | string | `vllm:num_requests_waiting` | **DEPRECATED**: Use engineConfigs in EndpointPickerConfig instead |
| `--total-running-requests-metric` | string | `vllm:num_requests_running` | **DEPRECATED**: Use engineConfigs in EndpointPickerConfig instead |
| `--kv-cache-usage-percentage-metric` | string | `vllm:kv_cache_usage_perc` | **DEPRECATED**: Use engineConfigs in EndpointPickerConfig instead |
| `--lora-info-metric` | string | `vllm:lora_requests_info` | **DEPRECATED**: Use engineConfigs in EndpointPickerConfig instead |
| `--cache-info-metric` | string | `vllm:cache_config_info` | **DEPRECATED**: Use engineConfigs in EndpointPickerConfig instead |
| `-v`, `--v` | int | `0` | Number for the log level verbosity |
| `--zap-log-level` | string | | Zap log level (debug, info, warn, error) |
| `--zap-devel` | bool | `true` | Development Mode defaults (encoder=consoleEncoder,logLevel=Debug,stackTraceLevel=Warn) |
| `--zap-encoder` | string | | Zap log encoding ('json' or 'console') |
| `--zap-stacktrace-level` | string | | Zap Level at and above which stacktraces are captured |
| `--tracing` | bool | `true` | Enables emitting traces |
| `--health-checking` | bool | `false` | Enables health checking |
| `--metrics-port` | int | `9090` | The metrics port exposed by EPP |
| `--grpc-health-port` | int | `9003` | The port used for gRPC liveness and readiness probes |
| `--enable-pprof` | bool | `true` | Enables pprof handlers |
| `--cert-path` | string | `""` | The path to the certificate for secure serving. Certificate and private key files are assumed to be named tls.crt and tls.key |
| `--enable-cert-reload` | bool | `false` | Enables certificate reloading of the certificates specified in --cert-path |
| `--secure-serving` | bool | `true` | Enables secure serving |
| `--metrics-endpoint-auth` | bool | `true` | Enables authentication and authorization of the metrics endpoint |
| `--config-file` | string | `""` | The path to the configuration file |
| `--config-text` | string | `""` | The configuration specified as text, in lieu of a file |

##### Environment Variables

| Variable | Description | Deprecation |
|----------|-------------|-------------|
| `NAMESPACE` | Used to determine pool namespace when `--pool-namespace` is not set | - |
| `POD_NAME` | Used to determine EPP name when using `--endpoint-selector` mode | - |
| `ENABLE_EXPERIMENTAL_DATALAYER_V2` | Enables experimental pluggable data layer | **DEPRECATED**: Use FeatureGates in config file instead |
| `ENABLE_EXPERIMENTAL_FLOW_CONTROL_LAYER` | Enables experimental pluggable flow control layer | **DEPRECATED**: Use FeatureGates in config file instead |
| `SD_QUEUE_DEPTH_THRESHOLD` | Saturation detector queue depth threshold | **DEPRECATED**: Use config file instead |
| `SD_KV_CACHE_UTIL_THRESHOLD` | Saturation detector KV cache utilization threshold | **DEPRECATED**: Use config file instead |
| `SD_METRICS_STALENESS_THRESHOLD` | Saturation detector metrics staleness threshold | **DEPRECATED**: Use config file instead |

---

##### v0.5.0

Uses `gateway-api-inference-extension v1.3.0`.

##### Command-Line Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--grpc-port` | int | `9002` | gRPC port used for communicating with Envoy proxy |
| `--ha-enable-leader-election` | bool | `false` | Enables leader election for high availability. When enabled, readiness probes will only pass on the leader |
| `--pool-group` | string | `inference.networking.k8s.io` | Kubernetes resource group of the InferencePool this Endpoint Picker is associated with |
| `--pool-namespace` | string | `""` | Namespace of the InferencePool this Endpoint Picker is associated with |
| `--pool-name` | string | `""` | Name of the InferencePool this Endpoint Picker is associated with |
| `--endpoint-selector` | string | `""` | Selector to filter model server pods on, only 'key=value' pairs are supported. Format: comma-separated list of key=value pairs (e.g., 'app=vllm-llama3-8b-instruct,env=prod') |
| `--endpoint-target-ports` | []int | `[]` | Target ports of model server pods. Format: comma-separated list of numbers (e.g., '3000,3001,3002') |
| `--disable-endpoint-subset-filter` | bool | `false` | Disables respecting the x-gateway-destination-endpoint-subset metadata for dispatching requests in EPP |
| `--model-server-metrics-scheme` | string | `http` | Protocol scheme used in scraping metrics from endpoints |
| `--model-server-metrics-path` | string | `/metrics` | URL path used in scraping metrics from endpoints |
| `--model-server-metrics-port` | int | `0` | **DEPRECATED**: Port to scrape metrics from endpoints. Set to InferencePool.Spec.TargetPorts[0].Number if not defined |
| `--model-server-metrics-https-insecure-skip-verify` | bool | `true` | Disable certificate verification when using 'https' scheme for model-server-metrics-scheme |
| `--refresh-metrics-interval` | duration | `50ms` | Interval to refresh metrics |
| `--refresh-prometheus-metrics-interval` | duration | `5s` | Interval to flush Prometheus metrics |
| `--metrics-staleness-threshold` | duration | `2s` | Duration after which metrics are considered stale |
| `--total-queued-requests-metric` | string | `vllm:num_requests_waiting` | Prometheus metric for the number of queued requests |
| `--total-running-requests-metric` | string | `vllm:num_requests_running` | Prometheus metric for the number of running requests |
| `--kv-cache-usage-percentage-metric` | string | `vllm:kv_cache_usage_perc` | Prometheus metric for the fraction of KV-cache blocks currently in use (from 0 to 1) |
| `--lora-info-metric` | string | `vllm:lora_requests_info` | Prometheus metric for the LoRA info metrics (must be in vLLM label format) |
| `--cache-info-metric` | string | `vllm:cache_config_info` | Prometheus metric for the cache info metrics |
| `-v`, `--v` | int | `0` | Number for the log level verbosity |
| `--zap-log-level` | string | | Zap log level (debug, info, warn, error) |
| `--zap-devel` | bool | `true` | Development Mode defaults (encoder=consoleEncoder,logLevel=Debug,stackTraceLevel=Warn) |
| `--zap-encoder` | string | | Zap log encoding ('json' or 'console') |
| `--zap-stacktrace-level` | string | | Zap Level at and above which stacktraces are captured |
| `--tracing` | bool | `true` | Enables emitting traces |
| `--health-checking` | bool | `false` | Enables health checking |
| `--metrics-port` | int | `9090` | The metrics port exposed by EPP |
| `--grpc-health-port` | int | `9003` | The port used for gRPC liveness and readiness probes |
| `--enable-pprof` | bool | `true` | Enables pprof handlers |
| `--cert-path` | string | `""` | The path to the certificate for secure serving. Certificate and private key files are assumed to be named tls.crt and tls.key |
| `--enable-cert-reload` | bool | `false` | Enables certificate reloading of the certificates specified in --cert-path |
| `--secure-serving` | bool | `true` | Enables secure serving |
| `--metrics-endpoint-auth` | bool | `true` | Enables authentication and authorization of the metrics endpoint |
| `--config-file` | string | `""` | The path to the configuration file |
| `--config-text` | string | `""` | The configuration specified as text, in lieu of a file |

##### Environment Variables

| Variable | Description | Deprecation |
|----------|-------------|-------------|
| `NAMESPACE` | Used to determine pool namespace when `--pool-namespace` is not set | - |
| `POD_NAME` | Used to determine EPP name when using `--endpoint-selector` mode | - |
| `ENABLE_EXPERIMENTAL_DATALAYER_V2` | Enables experimental pluggable data layer | **DEPRECATED**: Use FeatureGates in config file instead |
| `ENABLE_EXPERIMENTAL_FLOW_CONTROL_LAYER` | Enables experimental pluggable flow control layer | **DEPRECATED**: Use FeatureGates in config file instead |
| `SD_QUEUE_DEPTH_THRESHOLD` | Saturation detector queue depth threshold | **DEPRECATED**: Use config file instead |
| `SD_KV_CACHE_UTIL_THRESHOLD` | Saturation detector KV cache utilization threshold | **DEPRECATED**: Use config file instead |
| `SD_METRICS_STALENESS_THRESHOLD` | Saturation detector metrics staleness threshold | **DEPRECATED**: Use config file instead |

#### Key Differences Between Main and v0.5.0

1. **Metric Flags**: In main branch, `--total-queued-requests-metric`, `--total-running-requests-metric`, `--kv-cache-usage-percentage-metric`, `--lora-info-metric`, and `--cache-info-metric` are deprecated and will error if explicitly set. In v0.5.0, these flags are functional.

2. **Configuration**: Main branch encourages using `EndpointPickerConfig` with `engineConfigs` for metrics configuration instead of CLI flags.

---

### llm-d Inference Simulator CLI Reference

This section documents the command-line flags and environment variables supported by the llm-d inference simulator (`llm-d-inference-sim`). The simulator is a vLLM server simulator supporting OpenAI API endpoints.

#### Main Branch (Latest)

##### Command-Line Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--config` | string | `""` | Path to a YAML configuration file. Command line values overwrite config file values |
| `--port` | int | `8000` | Port on which the simulator runs |
| `--model` | string | `""` | Currently 'loaded' model name (required) |
| `--served-model-name` | []string | `[]` | Model names exposed by the API (space-separated strings). Falls back to `--model` if not set |
| `--max-num-seqs` | int | `5` | Maximum number of inference requests that could be processed at the same time |
| `--max-waiting-queue-length` | int | `1000` | Maximum length of inference requests waiting queue |
| `--max-loras` | int | `1` | Maximum number of LoRAs in a single batch |
| `--max-cpu-loras` | int | (same as `--max-loras`) | Maximum number of LoRAs to store in CPU memory |
| `--max-model-len` | int | `1024` | Model's context window, maximum number of tokens in a single request including input and output |
| `--lora-modules` | []string | `[]` | List of LoRA adapters (space-separated JSON strings) |
| `--mode` | string | `random` | Simulator mode: `echo` returns input text; `random` returns random pre-defined sentences |
| `--seed` | int64 | (current Unix nano) | Random seed for operations |
| `--time-to-first-token` | duration | `0` | Time to first token (e.g., "100ms"). Integer format (milliseconds) is deprecated |
| `--time-to-first-token-std-dev` | duration | `0` | Standard deviation for time to first token (max 30% of TTFT) |
| `--inter-token-latency` | duration | `0` | Time to generate one token (e.g., "100ms"). Integer format is deprecated |
| `--inter-token-latency-std-dev` | duration | `0` | Standard deviation for inter-token latency (max 30% of ITL) |
| `--prefill-overhead` | duration | `0` | Time to prefill. Ignored if `--time-to-first-token` is set |
| `--prefill-time-per-token` | duration | `0` | Time to prefill per token |
| `--prefill-time-std-dev` | duration | `0` | Standard deviation for prefill time |
| `--kv-cache-transfer-latency` | duration | `0` | Time for KV-cache transfer from a remote vLLM (P/D mode) |
| `--kv-cache-transfer-latency-std-dev` | duration | `0` | Standard deviation for KV-cache transfer latency |
| `--kv-cache-transfer-time-per-token` | duration | `0` | Time for KV-cache transfer per token from a remote vLLM |
| `--kv-cache-transfer-time-std-dev` | duration | `0` | Standard deviation for KV-cache transfer time per token |
| `--time-factor-under-load` | float64 | `1.0` | Multiplicative factor affecting request time when parallel requests are processed (must be >= 1.0) |
| `--enable-kvcache` | bool | `false` | Enables KV cache feature |
| `--kv-cache-size` | int | `1024` | Maximum number of token blocks in KV cache |
| `--global-cache-hit-threshold` | float64 | `0` | Default cache hit threshold [0, 1] for all requests |
| `--block-size` | int | `16` | Token block size for contiguous chunks (valid: 8, 16, 32, 64, 128) |
| `--tokenizers-cache-dir` | string | `hf_cache` | Directory for caching tokenizers |
| `--hash-seed` | string | `""` | Seed for hash generation (falls back to `PYTHONHASHSEED` env var) |
| `--zmq-endpoint` | string | `tcp://localhost:5557` | ZMQ address to publish events |
| `--zmq-max-connect-attempts` | int | `0` | Maximum number of times to try ZMQ connect (max 10) |
| `--event-batch-size` | int | `16` | Maximum number of KV-cache events to be sent together |
| `--data-parallel-size` | int | `1` | Number of ranks to run (1-8) |
| `--data-parallel-rank` | int | `-1` | The rank when running each rank in a process |
| `--failure-injection-rate` | int | `0` | Probability (0-100) of injecting failures |
| `--failure-types` | []string | `[]` | Specific failure types to inject: `rate_limit`, `invalid_api_key`, `context_length`, `server_error`, `invalid_request`, `model_not_found` |
| `--fake-metrics` | string | `""` | JSON metrics to report to Prometheus instead of real metrics |
| `--ssl-certfile` | string | `""` | Path to SSL certificate file for HTTPS |
| `--ssl-keyfile` | string | `""` | Path to SSL private key file for HTTPS |
| `--self-signed-certs` | bool | `false` | Enable automatic generation of self-signed certificates for HTTPS |
| `--dataset-path` | string | `""` | Local path to SQLite database file for response generation from a dataset |
| `--dataset-url` | string | `""` | URL to download the SQLite database file for response generation |
| `--dataset-in-memory` | bool | `false` | Load the entire dataset into memory for faster access |
| `--enable-sleep-mode` | bool | `false` | Enable sleep mode |
| `--enable-request-id-headers` | bool | `false` | Enable including X-Request-Id header in responses |
| `--latency-calculator` | string | `""` | Name of the latency calculator: `constant` or `per-token` |
| `--max-tool-call-integer-param` | int | `100` | Maximum possible value of integer parameters in a tool call |
| `--min-tool-call-integer-param` | int | `0` | Minimum possible value of integer parameters in a tool call |
| `--max-tool-call-number-param` | float64 | `100` | Maximum possible value of number (float) parameters in a tool call |
| `--min-tool-call-number-param` | float64 | `0` | Minimum possible value of number (float) parameters in a tool call |
| `--max-tool-call-array-param-length` | int | `5` | Maximum possible length of array parameters in a tool call |
| `--min-tool-call-array-param-length` | int | `1` | Minimum possible length of array parameters in a tool call |
| `--tool-call-not-required-param-probability` | int | `50` | Probability (0-100) to add a non-required parameter in a tool call |
| `--object-tool-call-not-required-field-probability` | int | `50` | Probability (0-100) to add a non-required field in an object in a tool call |

##### Environment Variables

| Variable | Description |
|----------|-------------|
| `POD_NAME` | Pod name of simulator |
| `POD_NAMESPACE` | Namespace where simulator is running |
| `POD_IP` | IP address on which simulator runs |
| `PYTHONHASHSEED` | Fallback seed for hash generation if `--hash-seed` is not set |
| `VLLM_SERVER_DEV_MODE` | Set to `1` to enable development mode |

---

#### v0.5.0

##### Command-Line Flags

| Flag | Type | Default | Description |
|------|------|---------|-------------|
| `--config` | string | `""` | Path to a YAML configuration file. Command line values overwrite config file values |
| `--port` | int | `8000` | Port on which the simulator runs |
| `--model` | string | `""` | Currently 'loaded' model name (required) |
| `--served-model-name` | []string | `[]` | Model names exposed by the API (space-separated strings). Falls back to `--model` if not set |
| `--max-num-seqs` | int | `5` | Maximum number of inference requests that could be processed at the same time (parameter to simulate requests waiting queue) |
| `--max-loras` | int | `1` | Maximum number of LoRAs in a single batch |
| `--max-cpu-loras` | int | (same as `--max-loras`) | Maximum number of LoRAs to store in CPU memory |
| `--max-model-len` | int | `1024` | Model's context window, maximum number of tokens in a single request including input and output |
| `--lora-modules` | []string | `[]` | List of LoRA adapters (space-separated JSON strings) |
| `--mode` | string | `random` | Simulator mode: `echo` returns input text; `random` returns random pre-defined sentences |
| `--seed` | int64 | (current Unix nano) | Random seed for operations |
| `--time-to-first-token` | int | `0` | Time to first token in milliseconds |
| `--time-to-first-token-std-dev` | int | `0` | Standard deviation for time to first token in milliseconds (max 30% of TTFT) |
| `--inter-token-latency` | int | `0` | Time to generate one token in milliseconds |
| `--inter-token-latency-std-dev` | int | `0` | Standard deviation for inter-token latency in milliseconds (max 30% of ITL) |
| `--prefill-overhead` | int | `0` | Time to prefill in milliseconds. Ignored if `--time-to-first-token` is not 0 |
| `--prefill-time-per-token` | int | `0` | Time to prefill per token in milliseconds |
| `--prefill-time-std-dev` | int | `0` | Standard deviation for prefill time in milliseconds |
| `--kv-cache-transfer-latency` | int | `0` | Time for KV-cache transfer from a remote vLLM in milliseconds (P/D mode) |
| `--kv-cache-transfer-latency-std-dev` | int | `0` | Standard deviation for KV-cache transfer latency in milliseconds |
| `--kv-cache-transfer-time-per-token` | int | `0` | Time for KV-cache transfer per token from a remote vLLM in milliseconds |
| `--kv-cache-transfer-time-std-dev` | int | `0` | Standard deviation for KV-cache transfer time per token in milliseconds |
| `--time-factor-under-load` | float64 | `1.0` | Multiplicative factor affecting request time when parallel requests are processed (must be >= 1.0) |
| `--enable-kvcache` | bool | `false` | Enables KV cache feature |
| `--kv-cache-size` | int | `1024` | Maximum number of token blocks in KV cache |
| `--block-size` | int | `16` | Token block size for contiguous chunks (valid: 8, 16, 32, 64, 128) |
| `--tokenizers-cache-dir` | string | `""` | Directory for caching tokenizers |
| `--hash-seed` | string | `""` | Seed for hash generation (falls back to `PYTHONHASHSEED` env var) |
| `--zmq-endpoint` | string | `tcp://localhost:5557` | ZMQ address to publish events |
| `--zmq-max-connect-attempts` | uint | `0` | Maximum number of times to try ZMQ connect (max 10) |
| `--event-batch-size` | int | `16` | Maximum number of KV-cache events to be sent together |
| `--data-parallel-size` | int | `1` | Number of ranks to run (1-8) |
| `--failure-injection-rate` | int | `0` | Probability (0-100) of injecting failures |
| `--failure-types` | []string | `[]` | Specific failure types to inject: `rate_limit`, `invalid_api_key`, `context_length`, `server_error`, `invalid_request`, `model_not_found` |
| `--fake-metrics` | string | `""` | JSON metrics to report to Prometheus instead of real metrics |
| `--max-tool-call-integer-param` | int | `100` | Maximum possible value of integer parameters in a tool call |
| `--min-tool-call-integer-param` | int | `0` | Minimum possible value of integer parameters in a tool call |
| `--max-tool-call-number-param` | float64 | `100` | Maximum possible value of number (float) parameters in a tool call |
| `--min-tool-call-number-param` | float64 | `0` | Minimum possible value of number (float) parameters in a tool call |
| `--max-tool-call-array-param-length` | int | `5` | Maximum possible length of array parameters in a tool call |
| `--min-tool-call-array-param-length` | int | `1` | Minimum possible length of array parameters in a tool call |
| `--tool-call-not-required-param-probability` | int | `50` | Probability (0-100) to add a non-required parameter in a tool call |
| `--object-tool-call-not-required-field-probability` | int | `50` | Probability (0-100) to add a non-required field in an object in a tool call |

##### Environment Variables

| Variable | Description |
|----------|-------------|
| `POD_NAME` | Pod name of simulator |
| `POD_NAMESPACE` | Namespace where simulator is running |
| `PYTHONHASHSEED` | Fallback seed for hash generation if `--hash-seed` is not set |

##### Key Differences Between Main and v0.5.0

1. **Duration Parameters**: In main branch, latency-related parameters (`--time-to-first-token`, `--inter-token-latency`, etc.) use Go duration strings (e.g., "100ms", "1.5s"). In v0.5.0, these are integers representing milliseconds.

2. **New Flags in Main**: `--max-waiting-queue-length`, `--global-cache-hit-threshold`, `--data-parallel-rank`, `--ssl-certfile`, `--ssl-keyfile`, `--self-signed-certs`, `--dataset-path`, `--dataset-url`, `--dataset-in-memory`, `--enable-sleep-mode`, `--enable-request-id-headers`, `--latency-calculator`.

3. **Environment Variables**: Main branch adds `POD_IP` and `VLLM_SERVER_DEV_MODE`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/llm-d)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/llm-d)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
