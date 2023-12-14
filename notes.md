
git cli

clarify which repo to clone initially - move Dockerfile into original repo

issues with curl (on linux, but other linux ok)
ip address is different on 1 machine (10.88.0.2)

privileged port 80 on linux

 ✓ Ensuring node image (kindest/node:v1.27.3) 🖼
 ✗ Preparing nodes 📦 📦 📦  
Deleted nodes: ["kind-worker" "kind-worker2" "kind-control-plane"]
ERROR: failed to create cluster: command "podman run --name kind-control-plane --hostname kind-control-plane --label io.x-k8s.kind.role=control-plane --privileged --tmpfs /tmp --tmpfs /run --volume 4579cdb1dedf2e8c53a5d5b3cb489210f81cbaacbd709b0424d3796b9b19f364:/var:suid,exec,dev --volume /lib/modules:/lib/modules:ro -e KIND_EXPERIMENTAL_CONTAINERD_SNAPSHOTTER --detach --tty --net kind --label io.x-k8s.kind.cluster=kind -e container=podman --cgroupns=private --volume /dev/mapper:/dev/mapper --device /dev/fuse --publish=0.0.0.0:80:80/tcp --publish=0.0.0.0:443:443/tcp --publish=127.0.0.1:34131:6443/tcp -e KUBECONFIG=/etc/kubernetes/admin.conf docker.io/kindest/node@sha256:3966ac761ae0136263ffdb6cfd4db23ef8a83cba8a463690e98317add2c9ba72" failed with error: exit status 126
Command Output: Error: rootlessport cannot expose privileged port 80, you can add 'net.ipv4.ip_unprivileged_port_start=80' to /etc/sysctl.conf (currently 1024), or choose a larger port number (>= 1024): listen tcp 0.0.0.0:80: bind: permission denied

'open' not a cmd on linux
xdg-open on linux

prep 1st slide ahead of time.