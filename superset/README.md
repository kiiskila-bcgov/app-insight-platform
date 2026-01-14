# Apache Superset – Business Intelligence Platform

Apache Superset is an open-source business intelligence (BI) and data exploration platform used to build dashboards, charts, and SQL-based analytics on top of connected data sources.

This deployment installs Superset on OpenShift using the official Helm chart, with a custom values file.

---

## Prerequisites

* OpenShift cluster access with `oc` CLI configured
* Helm 3.x installed
* Target namespace already created (e.g., `a2edba-dev`)

---

## Quick Start

### 1. Install / Upgrade Superset

Superset was deployed using the official Helm chart and a custom values file.

```bash
helm upgrade superset superset/superset \
  -n a2edba-dev \
  -f values-superset.yaml
```

This will install Superset if it does not exist, or upgrade it if it does.

The `values-superset.yaml` file contains all Superset-specific configuration (image, resources, database connection, bootstrap settings, etc.).

---

### 2. Wait for Deployment

Superset may take a few minutes to initialize on first startup.

```bash
# Watch pod status
oc get pods -n a2edba-dev -l app=superset -w

# Wait until ready
oc wait -n a2edba-dev --for=condition=ready pod -l app=superset --timeout=300s
```

Check logs if startup seems slow:

```bash
oc logs -n a2edba-dev -l app=superset --tail=100
```

---

## Accessing Superset

Once the pod is ready and the route is active, Superset should be reachable at the configured hostname.

On first login, Superset may prompt for:

* Initial admin user (if not preconfigured)
* Database connections
* Feature configuration

---

## Troubleshooting

### Pods not starting

```bash
oc get pods -n a2edba-dev
oc describe pod <pod-name>
oc logs <pod-name> -n a2edba-dev
```

Common issues:

* **Database connection errors** – verify the metadata database is reachable and credentials are correct in `values-superset.yaml`
* **CrashLoopBackOff** – often due to bad config or missing secrets
* **Out of memory** – increase resource limits in `values-superset.yaml`

---

### Gateway route not working

* Verify the service name and port match the Superset service.
* Test internal access first:

```bash
oc port-forward svc/superset 8088:8088 -n a2edba-dev
```

Then open:
[http://localhost:8088](http://localhost:8088)

---

## Uninstall

```bash
helm uninstall superset -n a2edba-dev
```

This removes Superset resources but does not delete the metadata database unless it is managed separately.

---

## Resources

* Superset Kubernetes installation guide:
  [https://superset.apache.org/docs/installation/kubernetes/](https://superset.apache.org/docs/installation/kubernetes/)

* Superset documentation:
  [https://superset.apache.org/docs/intro](https://superset.apache.org/docs/intro)

* Helm chart repository:
  [https://artifacthub.io/packages/helm/superset/superset](https://artifacthub.io/packages/helm/superset/superset)

