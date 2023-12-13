# Containers Tutorial

---

## Building an image from a Dockerfile & running a container locally with docker/podman

First, clone a repository containing Dockerfile and application code:

```shell
git clone git@github.com:GoogleCloudPlatform/kubernetes-engine-samples.git
cd ./kubernetes-engine-samples/quickstarts/languages/python
```

Next, view the contents of the Dockerfile to understand how the Docker image will be built:

```shell
cat Dockerfile
docker build -t quickstart .
```

Run a container from the built image. The -e flag sets an environment variable inside the container. The --rm flag removes the container once it stops, and --name gives a name to the running container:

```shell
docker images
docker run -e "PORT=8080" --rm --name quickstart quickstart &
```

Make a curl request to the application running in the container (replace 172.17.0.2 with the actual container IP):

```shell
curl 172.17.0.2:8080
```

Check the running containers to see details about the quickstart container before stopping it:

```shell
docker ps
docker stop quickstart
```

---

## Running a container in kubernetes (Deployments, Pods)

Create a local kind cluster (or use an existing cluster if you have one):

```shell
cd ../../../..
kind create cluster --config ./cluster.yaml
```

Deploy an example guestbook application:

```shell
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/kubernetes-engine-samples/ddf42a2821e3542b6d26b4878d0e6014cd638d35/quickstarts/guestbook/all-in-one/guestbook-all-in-one.yaml
```

Let's scale the app down for now & inspect some resources & logs:

```shell
kubectl scale deployment frontend --replicas=1
kubectl get deployment,pod
kubectl wait --namespace default --for=condition=ready pod --selector=app=guestbook,tier=frontend --timeout=90s
kubectl logs -f $(kubectl get po -l app=guestbook -o name) &
```

---

## Accessing the container (Services, Ingress)

* [Service Types](https://kubernetes.io/docs/concepts/services-networking/service/#publishing-services-service-types)
* [Ingress Controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/)


Access the application via the service first, using port forwarding:

```shell
kubectl port-forward svc/frontend 8080:80 &
open http://localhost:8080
```

Deploy an ingress controller (nginx):

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s
```

Create an Ingress for the application:

```shell
kubectl apply -f ./ingress.yaml
kubectl get ingress example-ingress -o yaml
```

Access the application via the Ingress:

```shell
open http://localhost
```

---

## Setting Resources & QoS class

* [Pod Quality of Service Classes](https://kubernetes.io/docs/concepts/workloads/pods/pod-qos/)

Determine the QoS class for the application:

```shell
kubectl get deployment frontend -n default -o yaml|grep -A 5 " resources:"
kubectl get pods -n default -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,QOS_CLASS:.status.qosClass
```

Let's change the class to 'Guaranteed'

```shell
cat patch-qos.yaml
kubectl patch deployment frontend -n default --patch-file patch-qos.yaml --type=strategic
kubectl get pods -n default -o custom-columns=NAME:.metadata.name,NAMESPACE:.metadata.namespace,QOS_CLASS:.status.qosClass
```

---

## Scaling up manually & automatically (ReplicaSet, HorizontalPodAutoscaler)

Scale the frontend to 2 replicas:

```shell
kubectl scale deployment frontend --replicas=2
kubectl get po -w
```

Install the metrics server to capture cpu (& other) metrics:

```shell
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
kubectl patch deployment metrics-server -n kube-system --type='json' -p='[{"op": "add", "path": "/spec/template/spec/containers/0/args/-", "value": "--kubelet-insecure-tls"}]'
kubectl wait --namespace kube-system --for=condition=ready pod --selector=k8s-app=metrics-server --timeout=90s
```

View metrics for the nodes and pods

```shell
kubectl top node
kubectl top pod -n default -l 'app=guestbook,tier=frontend'
```

Add a HorizontalPodAutoscaler:

```shell
kubectl autoscale deployment frontend --cpu-percent=50 --min=1 --max=10 -n default
kubectl get hpa frontend -n default -w
```

---

## Configuring via env vars (Secrets, ConfigMaps)

Check the current config of the redis instance used by the frontend:

```shell
kubectl exec -it $(kubectl get po -l role=leader,app=redis -o name) -- redis-cli
CONFIG GET maxmemory
CONFIG GET maxmemory-policy
```

Add a redis config file via a ConfigMap:

```shell
kubectl apply -f ./redis_config.yaml
kubectl patch deployment redis-leader -n default --type=json --patch-file=redis_patch.json
```

Check the config has been applied:

```shell
kubectl exec -it $(kubectl get po -l role=leader,app=redis -o name) -- redis-cli
CONFIG GET maxmemory
CONFIG GET maxmemory-policy
```

* [Define container environment variables using ConfigMap data](https://kubernetes.io/docs/tasks/configure-pod-container/configure-pod-configmap/#define-container-environment-variables-using-configmap-data)
* [Using Secrets as environment variables](https://kubernetes.io/docs/concepts/configuration/secret/#using-secrets-as-environment-variables)

---

## Resiliency with cross zone deployment & Taints and Tolerations

* [Taints and Tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/)
* [Affinity and anti-affinity](https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#affinity-and-anti-affinity)
* [topologySpreadConstraints](https://kubernetes.io/docs/concepts/scheduling-eviction/topology-spread-constraints/#topologyspreadconstraints-field)

Label nodes as zones:

```shell
kubectl label nodes kind-worker topology.kubernetes.io/zone=zone2
kubectl label nodes kind-worker2 topology.kubernetes.io/zone=zone3
kubectl get nodes --show-labels
```

Scale up the frontend guestbook deployment with anti affinity rules:

```shell
kubectl delete hpa frontend -n default
kubectl patch deployment frontend -n default --type=json --patch-file=patch-anti-affinity.json
kubectl scale deployment frontend --replicas=2
```

Check where the pods are scheduled:

```shell
kubectl get pods -l app=guestbook,tier=frontend -o custom-columns=NAME:.metadata.name,NODE:.spec.nodeName -n default
```

* [DaemonSets - Taints and tolerations](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/#taints-and-tolerations)

```shell
kubectl get DaemonSet/kindnet -n kube-system
```

---

## Allowing for app movement & node upgrades (PodDisruptionBudget)

* [Voluntary and involuntary disruptions](https://kubernetes.io/docs/concepts/workloads/pods/disruptions/#voluntary-and-involuntary-disruptions)

Example PDB:

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: frontend-pdb
  namespace: default
spec:
  minAvailable: 2  # or use "maxUnavailable: 1"
  selector:
    matchLabels:
      app: guestbook
      tier: frontend
```

---

## Connecting to a Database & Storage (PersistentVolume(Claim))

* [Kustomize](https://kubectl.docs.kubernetes.io/guides/introduction/kustomize/#usage)

Deploy a wordpress application with mysql database:

```shell
kubectl apply -k ./
```

Inspect the volume mounts, volumes and claims

```shell
kubectl get deployment wordpress -o yaml | grep  -A 11 " volumeMounts:"
kubectl get pvc -n default
kubectl get pv
```

Check the volume location:

```shell
docker exec -it kind-worker bash
```

Cleanup:

```shell
kubectl delete -k ./
```

Inspect the kustomization resources:

```shell
cat kustomization.yaml
kustomize build | less
```

---

## Deploying with ArgoCD

Install ArgoCD:

```shell
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl wait --namespace argocd --for=condition=ready pod --selector=app.kubernetes.io/name=argocd-server --timeout=90s
```

Access the console:

```shell
kubectl port-forward svc/argocd-server -n argocd 8080:443 &
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 --decode && echo
open https://localhost:8080
```

Create the wordpress application:

```shell
kubectl apply -f ./argocd_application.yaml
```

---

## Kubernetes Dashboard

Deploy the Kubernetes Dashboard:

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
kubectl wait --namespace kubernetes-dashboard --for=condition=ready pod --selector=k8s-app=kubernetes-dashboard --timeout=90s
```

Access the dashboard:

```shell
kubectl proxy &
open http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/
kubectl apply -f ./user.yaml
kubectl -n kubernetes-dashboard create token admin-user
```

---

## Other tools

* [Kubernetes CLI To Manage Your Clusters In Style](https://k9scli.io/)
* [kubectx + kubens: Power tools for kubectl](https://github.com/ahmetb/kubectx)

---

## Other interesting things

* CI - e.g. https://docs.openshift.com/container-platform/4.14/cicd/jenkins/migrating-from-jenkins-to-openshift-pipelines.html
* Jobs (e.g. db migrates) - https://kubernetes.io/docs/concepts/workloads/controllers/job/
* NetworkPolicy - https://kubernetes.io/docs/concepts/services-networking/network-policies/

---

## Relevant Courses

From https://learning.redhat.com

* Deploying Containerized Applications Technical Overview (DO080)
* Red Hat OpenShift Development I: Introduction to Containers with Podman (DO188) vILT
* Fundamentals Of Openshift
* OpenShift Administration I - Containers And Kubernetes (DO180) VILT

NOTE: Some courses may have a cost associated with them.

From https://www.linkedin.com/learning

* Learning Kubernetes - https://www.linkedin.com/learning/learning-kubernetes-16086900?u=2056732

---

## References

* https://kind.sigs.k8s.io/docs/user/ingress/#ingress-nginx
* https://kubernetes.io/docs/tutorials/stateless-application/guestbook/
* https://kubernetes.io/docs/tutorials/configuration/configure-redis-using-configmap/
* https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/
