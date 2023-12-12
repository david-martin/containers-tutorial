# Containers Tutorial

* Building an image from a Dockerfile
* Running a container locally with docker/podman

```shell
git clone git@github.com:GoogleCloudPlatform/kubernetes-engine-samples.git
cd ./kubernetes-engine-samples/quickstarts/languages/python
cat Dockerfile
docker build -t quickstart .
docker images
docker run -e "PORT=8080" --rm --name quickstart quickstart &
curl 172.17.0.2:8080
docker ps
docker stop quickstart
```

* Running a container in kubernetes (Deployments, Pods)

```shell
cd ../../../..
kind create cluster --config ./cluster.yaml
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/kubernetes-engine-samples/ddf42a2821e3542b6d26b4878d0e6014cd638d35/quickstarts/guestbook/all-in-one/guestbook-all-in-one.yaml
kubectl scale deployment frontend --replicas=1
kubectl get deployment,pod
kubectl logs -f $(kubectl get po -l app=guestbook -o name) &
```

* Accessing the container (Services, Ingress)

```shell
kubectl port-forward svc/frontend 8080:80 &
open http://localhost:8080
```

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s
```

```shell
kubectl apply -f ./ingress.yaml
kubectl get ingress example-ingress -o yaml
```

```shell
open http://localhost
```

* Setting Resources & QoS class

[Pod Quality of Service Classes](https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/)

```shell
kubectl get deployment frontend -n default -o jsonpath="{.spec.template.spec.containers[*].resources}"
```

* Scaling up manually & automatically (ReplicaSet, HorizontalPodAutoscaler)

```shell
kubectl get po
kubectl scale deployment frontend --replicas=2
kubectl get po -w
```

```shell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
kubectl top node
kubectl top pod -n default -l 'app=guestbook,tier=frontend'
kubectl autoscale deployment frontend --cpu-percent=50 --min=1 --max=10 -n default
kubectl get hpa frontend -n default
```

* Configuring via env vars (Secrets, ConfigMaps)

```shell
kubectl exec -it $(kubectl get po -l role=leader,app=redis -o name) -- redis-cli
CONFIG GET maxmemory
CONFIG GET maxmemory-policy
```

```shell
kubectl apply -f ./redis_config.yaml
kubectl patch deployment redis-leader -n default --type=json --patch-file=redis_patch.json
```

[Define container environment variables using ConfigMap data](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data)
[Using Secrets as environment variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

* Connecting to a Database & Storage (PersistentVolume(Claim))

[Kustomize](https://kubectl.docs.kubernetes.io/guides/introduction/kustomize/#usage)

```shell
kubectl apply -k ./
kubectl get deployment wordpress -o yaml
kubectl get pvc -n default
kubectl get pv
kubectl port-forward svc/wordpress 8081:80 &
open http://localhost:8081
```

[Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)

```shell
kubectl delete -k ./
kustomize build . > kustomize_output.yaml
```

* Deploying automatically with ArgoCD

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode && echo
open https://localhost:8080
```

```shell
kubectl apply -f ./argocd_application.yaml
```

* Resiliency with cross zone deployment & Taints and Tolerations

[Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
[Affinity and anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)
[topologySpreadConstraints](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/#topologyspreadconstraints-field)

```shell
kubectl get nodes --show-labels
```

`topology.kubernetes.io/zone` label in OpenShift clusters

* Allowing for app movement & node upgrades (PodDisruptionBudget)

[Voluntary and involuntary disruptions](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/#voluntary-and-involuntary-disruptions)

Example PDB:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myservice-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myservice
```

* Kubernetes Dashboard

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl proxy &
open http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
```

```shell
kubectl apply -f ./user.yaml
kubectl -n kubernetes-dashboard create token admin-user
```

* Other tools

[Kubernetes CLI To Manage Your Clusters In Style](https://k9scli.io/)
[kubectx + kubens: Power tools for kubectl](https://github.com/ahmetb/kubectx)

## Out of Scope

* CI - e.g. https://docs.openshift.com/container-platform/4.14/cicd/jenkins/migrating-from-jenkins-to-openshift-pipelines.html
* Jobs (e.g. db migrates)
* NetworkPolicy

## Relevant Courses

From https://learning.redhat.com

* Deploying Containerized Applications Technical Overview (DO080)
* Red Hat OpenShift Development I: Introduction to Containers with Podman (DO188) vILT
* Fundamentals Of Openshift
* OpenShift Administration I - Containers And Kubernetes (DO180) VILT

NOTE: Some courses may have a cost associated with them.

From https://www.linkedin.com/learning

* Learning Kubernetes - https://www.linkedin.com/learning/learning-kubernetes-16086900?u=2056732

## References

* https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx
* https://kubernetes.io/docs/tutorials/stateless-application/guestbook/
* https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/
* https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
