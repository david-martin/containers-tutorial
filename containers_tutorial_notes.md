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
kubectl get deployment,pod
```

* Accessing the container (Services, Ingress)

```shell
kubectl port-forward svc/frontend 8080:80
curl http://localhost:8080
```

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
kubectl wait --namespace ingress-nginx --for=condition=ready pod --selector=app.kubernetes.io/component=controller --timeout=90s
```

```shell
kubectl apply -f ./ingress.yaml
```

```shell
curl http://localhost
```

* Setting Resources & QoS class

```shell
TODO
```

* Scaling up manually & automatically (ReplicaSet, HorizontalPodAutoscaler)

```shell
kubectl scale deployment frontend --replicas=2
kubectl get po
```

```shell
TODO: HorizontalPodAutoscaler
```

* Configuring via env vars (Secrets, ConfigMaps)
* Connecting to a Database & Storage (PersistentVolume(Claim))
* Resiliency with cross zone deployment & Taints and Tolerations
* Allowing for app movement & node upgrades (PodDisruptionBudget)
* Deploying automatically with ArgoCD

???

* dashboard proxy
* NetworkPolicy

## Out of Scope

* CI - e.g. https://docs.openshift.com/container-platform/4.14/cicd/jenkins/migrating-from-jenkins-to-openshift-pipelines.html
* Jobs (e.g. db migrates)

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
* 




