
# Craystack System with minikube 1.33 and istio 1.12.0 on Kubernetes v1.24.17

# Deployment : nginx - working

# TODO: Upgrade to 1.18.7 using Helm

# Steps

## Install istio 1.12.0 via Helm
```sh
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
kubectl create namespace istio-system
helm install istio-base istio/base -n istio-system --version 1.12.0
helm install istiod istio/istiod -n istio-system --wait --version 1.12.0
kubectl create namespace istio-ingress
helm upgrade --install istio-ingress istio/gateway -n istio-ingress --wait --version 1.12.0
kubectl label namespace default istio-injection=enabled
k apply -f n.yaml
```

### Nginx Pod description ( We can see sidecar Istio 1.12.0 is being used by nginx)
```sh
apxd-cluster-3:~ # k describe po nginx-544dc8b7c4-csmkz  
Name:         nginx-544dc8b7c4-csmkz
Namespace:    default
Priority:     0
Node:         apxd-cluster-3/10.103.16.198
Start Time:   Tue, 07 May 2024 09:49:30 +0000
Labels:       app=nginx
              pod-template-hash=544dc8b7c4
              security.istio.io/tlsMode=istio
              service.istio.io/canonical-name=nginx
              service.istio.io/canonical-revision=latest
Annotations:  kubectl.kubernetes.io/default-container: nginx
              kubectl.kubernetes.io/default-logs-container: nginx
              prometheus.io/path: /stats/prometheus
              prometheus.io/port: 15020
              prometheus.io/scrape: true
              sidecar.istio.io/status:
                {"initContainers":["istio-init"],"containers":["istio-proxy"],"volumes":["istio-envoy","istio-data","istio-podinfo","istio-token","istiod-...
Status:       Running
IP:           10.244.0.5
IPs:
  IP:           10.244.0.5
Controlled By:  ReplicaSet/nginx-544dc8b7c4
Init Containers:
  istio-init:
    Container ID:  containerd://8eb6792475e29f1067e1347acea415f32cb3b319d32ba2c99a690ccfe04a72f4
    Image:         docker.io/istio/proxyv2:1.12.0
    Image ID:      docker.io/istio/proxyv2@sha256:6734c59bab78320fcb2f38cc5da0d6c8a40e484a8eaac5fa6709fe1e4ddec25e
    Port:          <none>
    Host Port:     <none>
    Args:
      istio-iptables
      -p
      15001
      -z
      15006
      -u
      1337
      -m
      REDIRECT
      -i
      *
      -x
      
      -b
      *
      -d
      15090,15021,15020
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 07 May 2024 09:49:30 +0000
      Finished:     Tue, 07 May 2024 09:49:30 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:        100m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dqrgx (ro)
Containers:
  nginx:
    Container ID:   containerd://4992ffed2ebf075ddb0534901023d9c24b175adcd1c09422e1c0c48e1791012e
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:32e76d4f34f80e479964a0fbd4c5b4f6967b5322c8d004e9cf0cb81c93510766
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 07 May 2024 09:49:41 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dqrgx (ro)
  istio-proxy:
    Container ID:  containerd://77b8d61f7a60885c4666addcf93545afd8890980c6867224ec8fe72db2b8f06a
    Image:         docker.io/istio/proxyv2:1.12.0
    Image ID:      docker.io/istio/proxyv2@sha256:6734c59bab78320fcb2f38cc5da0d6c8a40e484a8eaac5fa6709fe1e4ddec25e
    Port:          15090/TCP
    Host Port:     0/TCP
    Args:
      proxy
      sidecar
      --domain
      $(POD_NAMESPACE).svc.cluster.local
      --proxyLogLevel=warning
      --proxyComponentLogLevel=misc:error
      --log_output_level=default:info
      --concurrency
      2
    State:          Running
      Started:      Tue, 07 May 2024 09:49:41 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:      100m
      memory:   128Mi
    Readiness:  http-get http://:15021/healthz/ready delay=1s timeout=3s period=2s #success=1 #failure=30
    Environment:
      JWT_POLICY:                    third-party-jwt
      PILOT_CERT_PROVIDER:           istiod
      CA_ADDR:                       istiod.istio-system.svc:15012
      POD_NAME:                      nginx-544dc8b7c4-csmkz (v1:metadata.name)
      POD_NAMESPACE:                 default (v1:metadata.namespace)
      INSTANCE_IP:                    (v1:status.podIP)
      SERVICE_ACCOUNT:                (v1:spec.serviceAccountName)
      HOST_IP:                        (v1:status.hostIP)
      PROXY_CONFIG:                  {}
                                     
      ISTIO_META_POD_PORTS:          [
                                         {"containerPort":80,"protocol":"TCP"}
                                     ]
      ISTIO_META_APP_CONTAINERS:     nginx
      ISTIO_META_CLUSTER_ID:         Kubernetes
      ISTIO_META_INTERCEPTION_MODE:  REDIRECT
      ISTIO_META_WORKLOAD_NAME:      nginx
      ISTIO_META_OWNER:              kubernetes://apis/apps/v1/namespaces/default/deployments/nginx
      ISTIO_META_MESH_ID:            cluster.local
      TRUST_DOMAIN:                  cluster.local
    Mounts:
      /etc/istio/pod from istio-podinfo (rw)
      /etc/istio/proxy from istio-envoy (rw)
      /var/lib/istio/data from istio-data (rw)
      /var/run/secrets/istio from istiod-ca-cert (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-dqrgx (ro)
      /var/run/secrets/tokens from istio-token (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  istio-envoy:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     Memory
    SizeLimit:  <unset>
  istio-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  istio-podinfo:
    Type:  DownwardAPI (a volume populated by information about the pod)
    Items:
      metadata.labels -> labels
      metadata.annotations -> annotations
  istio-token:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  43200
  istiod-ca-cert:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      istio-ca-root-cert
    Optional:  false
  kube-api-access-dqrgx:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Normal   Scheduled         81s                default-scheduler  Successfully assigned default/nginx-544dc8b7c4-csmkz to apxd-cluster-3
  Normal   Pulled            81s                kubelet            Container image "docker.io/istio/proxyv2:1.12.0" already present on machine
  Normal   Created           81s                kubelet            Created container istio-init
  Normal   Started           81s                kubelet            Started container istio-init
  Normal   Pulling           80s                kubelet            Pulling image "nginx:latest"
  Normal   Pulled            70s                kubelet            Successfully pulled image "nginx:latest" in 9.483596725s
  Normal   Created           70s                kubelet            Created container nginx
  Normal   Started           70s                kubelet            Started container nginx
  Normal   Pulled            70s                kubelet            Container image "docker.io/istio/proxyv2:1.12.0" already present on machine
  Normal   Created           70s                kubelet            Created container istio-proxy
  Normal   Started           70s                kubelet            Started container istio-proxy
  Warning  DNSConfigForming  68s (x6 over 81s)  kubelet            Search Line limits were exceeded, some search paths have been omitted, the applied search line is: default.svc.cluster.local svc.cluster.local cluster.local hpc.amslabs.hpecorp.net us.cray.com americas.cray.com
  ```

### Nginx Pod testing
```sh
k port-forward svc/nginx 8080:8080 --address 10.103.16.198 &
[1] 26476
Forwarding from 10.103.16.198:8080 -> 80

curl http://10.103.16.198:8080

Handling connection for 8080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```
## Apply label and restart pod

```sh
kubectl label namespace default istio-injection=enabled
kubectl rollout restart deployment -n default
```

## Upgrade Istio to 1.18.7 ( WITH HELM In Place Upgrade )
 This guide assumes you have already performed an installation with Helm for a previous minor or patch version of Istio.

1. Download istio 1.18.7
wget https://github.com/istio/istio/releases/download/1.18.7/istio-1.18.7-linux-amd64.tar.gz
tar -xvf istio-1.18.7-linux-amd64.tar.gz 
cd istio-1.18.7

2. Before upgrading Istio, it is recommended to run the istioctl x precheck command to make sure the upgrade is compatible with your environment.
```sh
./bin/istioctl x precheck
✔ No issues found when checking the cluster. Istio is safe to install or upgrade!
To get started, check out https://istio.io/latest/docs/setup/getting-started/
```

3. Upgrade the Kubernetes custom resource definitions (CRDs):
```sh
kubectl apply -f manifests/charts/base/crds
```
3. Upgrade the Istio base chart:
```sh
helm upgrade istio-base manifests/charts/base -n istio-system --skip-crds --version 1.18.7
```
4. Upgrade the Istio discovery chart:
```sh
helm upgrade istiod istio/istiod -n istio-system --version 1.18.7
```
5. Upgrade and gateway charts installed in your cluster:
```sh
helm upgrade istio-ingress istio/gateway -n istio-ingress --version 1.18.7
```

6. Perform a Rolling Upgrade of Nginx Pod
```sh
kubectl rollout restart deployment -n default
```

### Check that Nginx Sidecar has changed

```sh
k describe po nginx-689f4b8c69-zwvkc            
Name:         nginx-689f4b8c69-zwvkc
Namespace:    default
Priority:     0
Node:         apxd-cluster-3/10.103.16.198
Start Time:   Tue, 07 May 2024 09:57:21 +0000
Labels:       app=nginx
              pod-template-hash=689f4b8c69
              security.istio.io/tlsMode=istio
              service.istio.io/canonical-name=nginx
              service.istio.io/canonical-revision=latest
Annotations:  istio.io/rev: default
              kubectl.kubernetes.io/default-container: nginx
              kubectl.kubernetes.io/default-logs-container: nginx
              kubectl.kubernetes.io/restartedAt: 2024-05-07T09:57:21Z
              prometheus.io/path: /stats/prometheus
              prometheus.io/port: 15020
              prometheus.io/scrape: true
              sidecar.istio.io/status:
                {"initContainers":["istio-init"],"containers":["istio-proxy"],"volumes":["workload-socket","credential-socket","workload-certs","istio-env...
Status:       Running
IP:           10.244.0.8
IPs:
  IP:           10.244.0.8
Controlled By:  ReplicaSet/nginx-689f4b8c69
Init Containers:
  istio-init:
    Container ID:  containerd://4dced14d705c74dd5cc857a8ae729705cf0ca088a9e4f2623e4a1de5b013278a
    Image:         docker.io/istio/proxyv2:1.18.7
    Image ID:      docker.io/istio/proxyv2@sha256:880fb1c9e8c023eabda5f65460017f4aacff9089be371149d873cf536c8e527c
    Port:          <none>
    Host Port:     <none>
    Args:
      istio-iptables
      -p
      15001
      -z
      15006
      -u
      1337
      -m
      REDIRECT
      -i
      *
      -x
      
      -b
      *
      -d
      15090,15021,15020
      --log_output_level=default:info
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Tue, 07 May 2024 09:57:22 +0000
      Finished:     Tue, 07 May 2024 09:57:22 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:        100m
      memory:     128Mi
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-sql92 (ro)
Containers:
  nginx:
    Container ID:   containerd://3a9673f658abcb1663f914dfd92d7e483958332c9301b142767277aa090542e0
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:32e76d4f34f80e479964a0fbd4c5b4f6967b5322c8d004e9cf0cb81c93510766
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 07 May 2024 09:57:23 +0000
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-sql92 (ro)
  istio-proxy:
    Container ID:  containerd://422b9323b19f9c9b6be14075b9da443d1517f8cdab49a5c0c033574f685b9fbe
    Image:         docker.io/istio/proxyv2:1.18.7
    Image ID:      docker.io/istio/proxyv2@sha256:880fb1c9e8c023eabda5f65460017f4aacff9089be371149d873cf536c8e527c
    Port:          15090/TCP
    Host Port:     0/TCP
    Args:
      proxy
      sidecar
      --domain
      $(POD_NAMESPACE).svc.cluster.local
      --proxyLogLevel=warning
      --proxyComponentLogLevel=misc:error
      --log_output_level=default:info
    State:          Running
      Started:      Tue, 07 May 2024 09:57:24 +0000
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     2
      memory:  1Gi
    Requests:
      cpu:      100m
      memory:   128Mi
    Readiness:  http-get http://:15021/healthz/ready delay=1s timeout=3s period=2s #success=1 #failure=30
    Environment:
      JWT_POLICY:                    third-party-jwt
      PILOT_CERT_PROVIDER:           istiod
      CA_ADDR:                       istiod.istio-system.svc:15012
      POD_NAME:                      nginx-689f4b8c69-zwvkc (v1:metadata.name)
      POD_NAMESPACE:                 default (v1:metadata.namespace)
      INSTANCE_IP:                    (v1:status.podIP)
      SERVICE_ACCOUNT:                (v1:spec.serviceAccountName)
      HOST_IP:                        (v1:status.hostIP)
      ISTIO_CPU_LIMIT:               2 (limits.cpu)
      PROXY_CONFIG:                  {}
                                     
      ISTIO_META_POD_PORTS:          [
                                         {"containerPort":80,"protocol":"TCP"}
                                     ]
      ISTIO_META_APP_CONTAINERS:     nginx
      ISTIO_META_CLUSTER_ID:         Kubernetes
      ISTIO_META_NODE_NAME:           (v1:spec.nodeName)
      ISTIO_META_INTERCEPTION_MODE:  REDIRECT
      ISTIO_META_WORKLOAD_NAME:      nginx
      ISTIO_META_OWNER:              kubernetes://apis/apps/v1/namespaces/default/deployments/nginx
      ISTIO_META_MESH_ID:            cluster.local
      TRUST_DOMAIN:                  cluster.local
    Mounts:
      /etc/istio/pod from istio-podinfo (rw)
      /etc/istio/proxy from istio-envoy (rw)
      /var/lib/istio/data from istio-data (rw)
      /var/run/secrets/credential-uds from credential-socket (rw)
      /var/run/secrets/istio from istiod-ca-cert (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-sql92 (ro)
      /var/run/secrets/tokens from istio-token (rw)
      /var/run/secrets/workload-spiffe-credentials from workload-certs (rw)
      /var/run/secrets/workload-spiffe-uds from workload-socket (rw)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  workload-socket:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  credential-socket:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  workload-certs:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  istio-envoy:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     Memory
    SizeLimit:  <unset>
  istio-data:
    Type:       EmptyDir (a temporary directory that shares a pod's lifetime)
    Medium:     
    SizeLimit:  <unset>
  istio-podinfo:
    Type:  DownwardAPI (a volume populated by information about the pod)
    Items:
      metadata.labels -> labels
      metadata.annotations -> annotations
  istio-token:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  43200
  istiod-ca-cert:
    Type:      ConfigMap (a volume populated by a ConfigMap)
    Name:      istio-ca-root-cert
    Optional:  false
  kube-api-access-sql92:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason            Age                From               Message
  ----     ------            ----               ----               -------
  Normal   Scheduled         38s                default-scheduler  Successfully assigned default/nginx-689f4b8c69-zwvkc to apxd-cluster-3
  Normal   Pulled            37s                kubelet            Container image "docker.io/istio/proxyv2:1.18.7" already present on machine
  Normal   Created           37s                kubelet            Created container istio-init
  Normal   Started           37s                kubelet            Started container istio-init
  Normal   Pulling           37s                kubelet            Pulling image "nginx:latest"
  Normal   Pulled            36s                kubelet            Successfully pulled image "nginx:latest" in 889.231163ms
  Normal   Created           36s                kubelet            Created container nginx
  Normal   Started           36s                kubelet            Started container nginx
  Normal   Pulled            36s                kubelet            Container image "docker.io/istio/proxyv2:1.18.7" already present on machine
  Normal   Created           35s                kubelet            Created container istio-proxy
  Normal   Started           35s                kubelet            Started container istio-proxy
  Warning  DNSConfigForming  32s (x6 over 38s)  kubelet            Search Line limits were exceeded, some search paths have been omitted, the applied search line is: default.svc.cluster.local svc.cluster.local cluster.local hpc.amslabs.hpecorp.net us.cray.com americas.cray.com
```

We can see istio proxy version has been upgraded to 1.18.7


## REFERENCES

https://istio.io/v1.18/docs/setup/upgrade/helm/
https://istio.io/v1.12/docs/setup/install/helm/
https://github.com/istio/istio/wiki/Troubleshooting-Istio#sidecar-injection
https://istio.io/latest/blog/2019/data-plane-setup/
https://medium.com/johnjjung/enabling-and-disabling-kubernetes-istio-sidecar-injections-426c5e7d4811
https://rndwiki-pro.its.hpecorp.net/display/CASMPET/Installation+of+1.11.8 (Authored by Me)
