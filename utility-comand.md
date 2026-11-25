# 1. Quick Way: Imperative Command (kubectl run)

```bash
# Run a single Pod directly from the command line:
    kubectl run nginx-pod --image=nginx:latest

# Verification: Run kubectl get pods and confirm nginx-pod appears with status Running and 1/1 under READY
    kubectl get pods

# Delete the Pod
    kubectl delete pod <pod-name>
```
