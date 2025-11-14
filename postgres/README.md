# PostgreSQL v16 - Standard Deployment

This is a standard PostgreSQL v17 deployment for managed OpenShift environments.

## Prerequisites

- OpenShift cluster access with `oc` CLI configured
- Helm 3.x installed
- Namespace `a2edba-dev` already created
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
oc get statefulset postgres -n a2edba-dev

# Check the pods
oc get pods -n a2edba-dev

# Wait for pod to be ready
oc wait --for=condition=ready pod/postgres-0 -n a2edba-dev --timeout=300s
```

### Get Database Password

```bash
# Retrieve the generated password
oc get secret postgres-credentials -n a2edba-dev \
  -o jsonpath='{.data.password}' | base64 -d
echo
```

### Connect to PostgreSQL

**From within the cluster:**
```
Host: postgres.a2edba-dev.svc.cluster.local
Port: 5432
Database: postgres
Username: postgres
Password: (from secret above)
```

**Port-forward for local access:**
```bash
oc port-forward svc/postgres -n a2edba-dev 5432:5432
```

Then connect:
```bash
psql -h localhost -p 5432 -U postgres
```

**Execute SQL directly in pod:**
```bash
oc exec -it postgres-0 -n a2edba-dev -- psql -U postgres
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
  -f values-standard.yaml \
  -n a2edba-dev
```

## Uninstall

```bash
helm uninstall postgres -n a2edba-dev

# Optionally delete the PVC
oc delete pvc postgres-data-postgres-0 -n a2edba-dev
```

## Backup & Restore

### Manual Backup

```bash
# Create a backup
oc exec postgres-0 -n a2edba-dev -- \
  pg_dump -U postgres postgres > backup.sql

# Or backup all databases
oc exec postgres-0 -n a2edba-dev -- \
  pg_dumpall -U postgres > backup-all.sql
```

### Restore

```bash
# Restore from backup
cat backup.sql | oc exec -i postgres-0 -n a2edba-dev -- \
  psql -U postgres postgres
```

## Useful Commands

```bash
# View logs
oc logs postgres-0 -n a2edba-dev

# Follow logs
oc logs -f postgres-0 -n a2edba-dev

# Get pod details
oc describe pod postgres-0 -n a2edba-dev

# Check service
oc get svc postgres -n a2edba-dev

# Check storage
oc get pvc -n a2edba-dev
```

## Notes

- This is a **single instance** deployment (not HA)
- Data persists in a PersistentVolumeClaim
- Uses the official PostgreSQL 17 Alpine image from Docker Hub
- No operator or CRDs required
- Suitable for development and small production workloads
