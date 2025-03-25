# PROD

This is the starting place for deploying gitea in production with mysql. To test do:

1. `ansible-playbook up.yaml` and `kubectl get pod`
2. To see adminer, With the pod name run `kubectl port-forward mysql-6d544974d7-pldt8 8080:8080`
3. Adminer is a sidecar to mysql so the host name is either `mysql` or `127.0.0.1`