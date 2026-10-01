# Loyalty Points API

Created with the **Governed REST API** template in Developer Hub by `user:default/deanpeterson`.
Owner: `user:default/deanpeterson`. Lifecycle: `experimental`.

| What | Where |
|---|---|
| Endpoint | `https://api-loyalty-points.apps.salamander.aimlworkbench.com/api` (API key as `Authorization: Bearer <key>`) |
| Contract | [`openapi.yaml`](openapi.yaml), rendered in Developer Hub on the API's **Definition** tab |
| Keys and plans | Developer Hub → API products → **Loyalty Points API** → request a key (gold 1000/day, silver 200/day, bronze 20/day) |
| Dashboard | OpenShift console → Observe → Dashboards (Perses) → project `openshift-cluster-observability-operator` → **Loyalty Points API** |
| Traces | OpenShift console → Observe → Traces, service `loyalty-points-api` |
| Access log | OpenShift console → Observe → Logs, `{kubernetes_namespace_name="api-demo", kubernetes_container_name="istio-proxy"} \| json \| line_format "{{.message}}" \| json \| authority="api-loyalty-points.apps.salamander.aimlworkbench.com"` |

## What is in this repository

Everything the API needs on the cluster. OpenShift GitOps syncs it into the
namespace `api-loyalty-points` within a minute of a merge to `main`.

| File | Purpose |
|---|---|
| `openapi.yaml` | The contract. Mounted into the backend as a ConfigMap. |
| `gitops/namespace.yaml` | The API's own namespace, with a resource quota and default container sizes. |
| `gitops/backend.yaml` | The backend: the container image quay.io/kuadrant/authorino-examples:talker-api. |
| `gitops/httproute.yaml` | The API's route on the shared gateway `api-demo/api-gateway`. |
| `gitops/route.yaml` | The public hostname (TLS at the cluster edge). |
| `gitops/authpolicy.yaml` | Who may call: keys issued through the catalog for this product. Writes the consumer identity and plan into the gateway's access log. |
| `gitops/planpolicy.yaml` | How much: the gold, silver and bronze daily limits, enforced at the gateway. |
| `gitops/apiproduct.yaml` | The product consumers request keys for. Requests and approvals are recorded as objects on the cluster. |
| `gitops/instrumentation.yaml` | Tracing agent injection (OpenTelemetry), no code changes. |
| `gitops/dashboard.yaml` | The owner's dashboard, as code (Perses). |

Change any of them in a pull request. The merge is the approval; the commit history is the audit trail.

## Removing the API

Delete the GitOps application `loyalty-points-api` (project `governed-apis`) and the namespace `api-loyalty-points`; unregister the component in Developer Hub; archive this repository.

## Dashboards

Five Perses dashboards in the console (Observe → Dashboards), all as code in `gitops/`: `dashboard.yaml` (traffic, decisions, latency),
`dashboard-consumers.yaml` (who is calling, on which plan, allowance used today; per-consumer labels come from the gateway's
TelemetryPolicy), `dashboard-decisions.yaml` (the funnel every request goes through), `dashboard-cost.yaml` (what the key check
adds next to the end-to-end latency) `dashboard-posture.yaml` (route attached, policies enforced, GitOps in sync, backend up) and `dashboard-logs.yaml` (saved log questions, answered live from OpenShift Logging through the platform's Loki datasource).
