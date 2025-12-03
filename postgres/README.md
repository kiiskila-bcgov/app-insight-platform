# PostgreSQL v17

This is a standard PostgreSQL v17 deployment for managed OpenShift environments.

## Prerequisites

- OpenShift cluster access with `oc` CLI configured
- Helm 3.x installed
- No cluster-admin permissions required

## Installation

### Deploy PostgreSQL

```bash
# Install using the standard templates
helm install postgres . 
```

### Verify Deployment

```bash
# Check the StatefulSet
oc get statefulset postgres

# Check the pods
oc get pods

# Wait for pod to be ready
oc wait --for=condition=ready pod/postgres-0 --timeout=300s
```

### Get Database Password

```bash
# Retrieve the generated password
oc get secret postgres-credentials \
  -o jsonpath='{.data.password}' | base64 -d
echo
```

### Connect to PostgreSQL

**From within the cluster:**
```
Host: postgres.[namespace ex: abcdef-dev].svc.cluster.local
Port: 5432
Database: postgres
Username: postgres
Password: (from secret above)
```

**Port-forward for local access:**
```bash
oc port-forward svc/postgres 5432:5432
```

Then connect:
```bash
psql -h localhost -p 5432 -U postgres
```

**Execute SQL directly in pod:**
```bash
oc exec -it postgres-0 -- psql -U postgres
```

## Configuration

Edit `values.yaml` to customize:

- **Image**: `postgres.image` (default: `postgres:17-alpine`)
- **Resources**: `resources.requests` and `resources.limits`
- **Storage**: `storage.size` (default: 1Gi)
- **PostgreSQL settings**: `postgresConfig.*`

## Upgrade

```bash
helm upgrade postgres . \
  -f values.yaml
```

## Uninstall

```bash
helm uninstall postgres

# Optionally delete the PVC
oc delete pvc postgres-data-postgres-0
```

## Backup & Restore

### Manual Backup

```bash
# Create a backup
oc exec postgres-0 -- \
  pg_dump -U postgres postgres > backup.sql

# Or backup all databases
oc exec postgres-0 -- \
  pg_dumpall -U postgres > backup-all.sql
```

### Restore

```bash
# Restore from backup
cat backup.sql | oc exec -i postgres-0 -- \
  psql -U postgres postgres
```

## Useful Commands

```bash
# View logs
oc logs postgres-0

# Follow logs
oc logs -f postgres-0

# Get pod details
oc describe pod postgres-0

# Check service
oc get svc postgres

# Check storage
oc get pvc
```

## Notes

- This is a **single instance** deployment (not HA)
- Data persists in a PersistentVolumeClaim
- Uses the official PostgreSQL 17 Alpine image from Docker Hub
- No operator or CRDs required
- Suitable for development and small production workloads
