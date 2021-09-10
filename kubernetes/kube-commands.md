####Website
https://kubernetes.io/docs/reference/kubectl/overview/
####kubectl
>kubectl -n kube-system describe pod kube-scheduler-minikube | grep "Container ID"
---
> kubectl config view
---
> kubectl cluster-info
> kubectl cluster-info dump
---
> kubectl proxy (starts the dashboard, typically at: http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/)
---
####curl
> curl http://localhost:8001/
---
With the above curl request, we requested all the API endpoints from the API server.

We can explore several path combinations with curl or in a browser as well, such as:

- http://localhost:8001/api/v1

- http://localhost:8001/apis/apps/v1

- http://localhost:8001/healthz

- http://localhost:8001/metrics
####Authentication
**Get the token**
> TOKEN=$(kubectl describe secret -n kube-system $(kubectl get secrets -n kube-system | grep default | cut -f1 -d ' ') | grep -E '^token' | cut -f2 -d':' | tr -d '\t' | tr -d " ")
---
**Get the API server endpoint**
> APISERVER=$(kubectl config view | grep https | cut -f 2- -d ":" | tr -d " ")
---
**Accessing the API server**
>curl $APISERVER --header "Authorization: Bearer $TOKEN" --insecure
---
> curl $APISERVER --cert encoded-cert --key encoded-key --cacert encoded-ca
