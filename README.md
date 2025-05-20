
# 🧩 MCPConfiguration Spec – Unified Observability & Policy Configuration

The `MCPConfiguration` is a declarative YAML spec used to configure:

- ✅ **Log ingestion** across providers like Splunk, Elasticsearch, and SumoLogic  
- 📊 **Metrics collection** with Prometheus, Datadog, etc.  
- 🔐 **Secrets management** using Akeyless  
- 🚦 **API rate limiting** per route  
- 🛡️ **Policy enforcement** with OPA (Rego) for compliance and validation  

---

## 📁 Structure

```yaml
apiVersion: mcp.open/v1
kind: MCPConfiguration
metadata:
  name: example-mcp-config
  description: Unified configuration for logging, metrics, secrets, and policy in MCP
  version: 0.1.0
spec:
  logProviders: [...]
  metricsProviders: [...]
  secrets: {...}
  rateLimiting: {...}
  policy: {...}
```

---

## 🔍 Field Breakdown

### 🧾 Metadata

| Field       | Type   | Description                                 |
|-------------|--------|---------------------------------------------|
| `name`      | string | Unique name for the configuration           |
| `description` | string | Description of the config's purpose       |
| `version`   | string | Semantic version of the config              |

---

### 🪵 `logProviders[]`

Configure one or more log destinations:

| Field       | Type   | Description                                   |
|-------------|--------|-----------------------------------------------|
| `name`      | string | Name of the log provider                      |
| `type`      | enum   | One of `splunk`, `elasticsearch`, `sumologic` |
| `endpoint`  | string | Destination endpoint URL                      |
| `token`     | string | Token for Splunk                              |
| `authToken` | string | Token for SumoLogic                           |
| `auth`      | object | Basic auth for Elasticsearch                  |
| `index`     | string | Index or target location                      |
| `sourcetype`| string | (Splunk) sourcetype label                     |

---

### 📊 `metricsProviders[]`

Set up metrics collection services:

| Field          | Type   | Description                                   |
|----------------|--------|-----------------------------------------------|
| `name`         | string | Name of the metrics provider                  |
| `type`         | enum   | One of `prometheus`, `datadog`                |
| `endpoint`     | string | URL of the scraping or push location          |
| `scrapeInterval`| string| (Prometheus) Interval for scraping metrics    |
| `apiKey`       | string | (Datadog) API key                             |
| `appKey`       | string | (Datadog) App key                             |
| `region`       | string | (Datadog) Region (e.g., `us`, `eu`)           |

---

### 🔐 `secrets`

Akeyless-based secret injection:

```yaml
secrets:
  provider: akeyless
  config:
    accessId: ${AKEYLESS_ACCESS_ID}
    accessKey: ${AKEYLESS_ACCESS_KEY}
    apiUrl: https://api.akeyless.io
    secretsMapping:
      SPLUNK_TOKEN: path/to/splunk/token
      ELASTIC_USER: path/to/elastic/user
```

| Field          | Type   | Description                                |
|----------------|--------|--------------------------------------------|
| `provider`     | string | Secret manager type (e.g., `akeyless`)     |
| `accessId`     | string | Akeyless access ID                         |
| `accessKey`    | string | Akeyless secret key                        |
| `apiUrl`       | string | Akeyless API endpoint                      |
| `secretsMapping` | map | Key-value map from env var → secret path   |

---

### 🚦 `rateLimiting`

```yaml
rateLimiting:
  globalLimit:
    requestsPerMinute: 10000
  perRouteLimits:
    - route: /api/v1/logs
      limit: 1000
```

| Field                 | Type    | Description                                  |
|-----------------------|---------|----------------------------------------------|
| `globalLimit.requestsPerMinute` | integer | Global request cap per minute         |
| `perRouteLimits[]`    | list    | Route-specific rate limits                   |
| `route`               | string  | API route path                               |
| `limit`               | integer | Requests per minute for that route           |

---

### 🛡️ `policy`

OPA-based (Rego) validation and enforcement:

```yaml
policy:
  format: opa
  engine: rego
  policies:
    - id: enforce-namespace-is-set
      description: Require all log entries to have a `namespace` label
      type: validation
      rule: |
        package mcp.policies
        default allow = false
        allow {
          input.namespace != ""
        }
```

| Field        | Type    | Description                                     |
|--------------|---------|-------------------------------------------------|
| `format`     | string  | Policy format engine (e.g., `opa`)              |
| `engine`     | string  | Execution language (e.g., `rego`)               |
| `policies[]` | array   | List of policy rule definitions                 |
| `id`         | string  | Unique identifier for the policy                |
| `description`| string  | Human-readable description                      |
| `type`       | enum    | One of `validation`, `enforcement`             |
| `rule`       | string  | Rego policy script (OPA DSL)                    |

---

## ✅ PCI-DSS Compliance Policies Included

| Policy ID                        | Description                                                                  |
|----------------------------------|------------------------------------------------------------------------------|
| `disallow-sensitive-data-in-logs` | Prevents log messages from containing PCI-sensitive terms like card numbers |
| `enforce-strict-access-logging`   | Requires presence of `user_id`, `ip_address`, and `timestamp` in logs       |

These rules support PCI-DSS sections 10.2.x and 10.3.x.

---

## 📌 Future Roadmap

- [ ] Add Vault, AWS Secrets Manager support  
- [ ] Extend to support CEL or CUE policy formats  
- [ ] Dynamic provider plugins for observability stack  

---

## 📂 Example Directory Structure

```
.
├── config/
│   └── mcp.yaml
├── policies/
│   ├── pci_access.rego
│   └── sensitive_logs.rego
├── main.go
└── README.md
```

---

## 📣 Contribute

We welcome enhancements and provider plugins! Open an issue or submit a PR.
