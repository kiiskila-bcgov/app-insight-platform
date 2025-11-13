# Metabase - Business Intelligence Platform

Metabase is an open-source business intelligence (BI) platform that allows users to visualize, analyze, and share data by connecting to various databases.

## Prerequisites

- OpenShift cluster access with `oc` CLI configured
- Helm 3.x installed
- Namespace already created

## Quick Start

### 1. Install Metabase

```bash
cd metabase
helm install metabase .
```

### 2. Wait for Deployment

Metabase takes about 2-3 minutes to start up:

```bash
# Watch the pod status
oc get pods -l app=metabase -w

# Wait for it to be ready
oc wait --for=condition=ready pod -l app=metabase --timeout=300s
```

## 3. Setup the APS Gateway

See the [Gateway Documentation](../gateway/README.md) for more information on setting up an IDIR authenticated route.


## Troubleshooting

### Pod Not Starting

Check logs for errors:
```bash
oc logs -l app=metabase --tail=100
```

Common issues:
- **Database connection failed**: Verify postgres is running
- **Out of memory**: Increase memory limits in values.yaml
- **Timeout**: Increase `initialDelaySeconds` in deployment

## Backup & Restore

### Backup Metabase Configuration

```bash
# Backup the metabase database
oc exec postgres-0 -- \
  pg_dump -U postgres metabase > metabase-backup.sql
```

### Restore

```bash
cat metabase-backup.sql | oc exec -i postgres-0 -- \
  psql -U postgres metabase
```

## Uninstall

```bash
helm uninstall metabase

# Optionally drop the metabase database
oc exec -it postgres-0 - psql -U postgres -c "DROP DATABASE metabase;"
```

## Resources

- [Metabase Documentation](https://www.metabase.com/docs/latest/)
- [Metabase Questions](https://www.metabase.com/docs/latest/questions/start)
- [Metabase Dashboards](https://www.metabase.com/docs/latest/dashboards/start)