# kubectl rollout & patch — Reference Guide

## kubectl rollout

Manages the rollout lifecycle of Deployments, StatefulSets, and DaemonSets.

### Status

Waits and streams progress until complete or timeout:

```bash
kubectl rollout status deployment/<name> -n <namespace>
kubectl rollout status deployment/<name> -n <namespace> --timeout=120s
```

### History

Shows revision numbers and change causes:

```bash
kubectl rollout history deployment/<name> -n <namespace>
kubectl rollout history deployment/<name> -n <namespace> --revision=3
```

### Undo (Rollback)

Reverts to the previous revision or a specific one:

```bash
kubectl rollout undo deployment/<name> -n <namespace>
kubectl rollout undo deployment/<name> -n <namespace> --to-revision=2
```

### Restart

Triggers a rolling restart without changing the spec. Use this to force pods to pick up updated Secrets or ConfigMaps:

```bash
kubectl rollout restart deployment/<name> -n <namespace>
```

### Pause & Resume

Lets you batch multiple changes before triggering a single rollout:

```bash
kubectl rollout pause deployment/<name> -n <namespace>
# make multiple edits / patches here
kubectl rollout resume deployment/<name> -n <namespace>
```

---

## kubectl patch

Modifies a live resource without rewriting the full manifest. Three patch types are available.

### --type=json (JSON Patch — RFC 6902)

Precise surgical changes using `op` / `path` / `value`. Supports `add`, `remove`, `replace`, `copy`, `move`, and `test`.

```bash
# Add envFrom referencing a Secret
kubectl patch deployment n8n -n n8n --type=json -p='[
  {
    "op": "add",
    "path": "/spec/template/spec/containers/0/envFrom",
    "value": [{"secretRef": {"name": "n8n-workflow-secrets"}}]
  }
]'

# Multiple ops in one call
kubectl patch deployment n8n -n n8n --type=json -p='[
  {"op": "replace", "path": "/spec/replicas", "value": 2},
  {"op": "remove",  "path": "/spec/template/spec/containers/0/envFrom/0"}
]'
```

### --type=merge (Merge Patch)

Deep-merges a partial object into the live resource. Simpler syntax but cannot remove keys.

```bash
kubectl patch deployment n8n -n n8n --type=merge -p='
{
  "spec": {"replicas": 2}
}'
```

### --type=strategic (Strategic Merge Patch — default)

Like merge but Kubernetes-aware. Knows how to merge lists by key (e.g. containers matched by `name`) instead of replacing the entire array.

```bash
kubectl patch deployment n8n -n n8n -p='
{
  "spec": {
    "template": {
      "spec": {
        "containers": [{
          "name": "n8n",
          "resources": {
            "limits": {"memory": "512Mi"}
          }
        }]
      }
    }
  }
}'
```

---

## When to Use Which

| Goal | Command |
|------|---------|
| Pick up new Secret or ConfigMap values | `rollout restart` |
| Revert a bad deploy | `rollout undo` |
| Wait for a deploy to finish in a script/CI | `rollout status --timeout=Xs` |
| Add or remove a specific field precisely | `patch --type=json` |
| Change replicas, image, or resource limits | `patch --type=merge` or strategic |
| Apply many changes as a single rollout | `rollout pause` → edit → `rollout resume` |

---

## Notes

- Every `kubectl patch` on a Deployment that touches `spec.template` automatically triggers a rolling restart — Kubernetes detects the pod spec changed and replaces pods one by one, respecting `maxUnavailable` and `maxSurge` from the rolling update strategy.
- `rollout undo` only works if revision history is retained. The number of retained revisions is controlled by `spec.revisionHistoryLimit` (default: 10).
- `rollout status` exits with code `0` on success and `1` on failure or timeout — useful for scripting deploy pipelines.
- Use `--dry-run=client` with any patch to preview changes without applying them:

```bash
kubectl patch deployment n8n -n n8n --type=merge \
  -p='{"spec":{"replicas":3}}' --dry-run=client
```
