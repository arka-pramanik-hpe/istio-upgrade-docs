# Subtasks for ISTIO Upgrade

## Check upgrade notes from Previous ISTIO Versions

https://istio.io/latest/news/releases/1.12.x/announcing-1.12/upgrade-notes/

- TCP probes now working as expected 
- Default revision must be switched when performing a revision-based upgrade

https://istio.io/latest/news/releases/1.13.x/announcing-1.13/upgrade-notes/

- Health Probes will no longer re-use connections
- Multicluster Secret Authentication Changes
- Port 22 iptables capture changes


## Install Kiali version 1.49 for Istio 1.13

Kiali/Istio compatibility: https://kiali.io/docs/installation/installation-guide/prerequisites/

Istio: 1.13 Kiali: 1.49.0 (Kiali & kiali-operator charts can be found in https://github.com/kiali/helm-charts)

## Clone Upstream repo and apply necessary commits from our Cray-HPE work-1.11.8

git clone git@github.com:Cray-HPE/istio.git
cd istio
git checkout cray-master
git remote add upstream https://github.com/istio/istio.git
git fetch upstream
git checkout 1.11.8
git checkout -b work-1.13.9
git push --set-upstream origin work-1.13.9

Cherry-pick necessary Commits

## Build Images
- Build image through Jenkins Pipeline

## Setup Cleanup Scripts from necessary repos
- Check to see if we want to remove the add-on cleanup script added for istio 1.11 upgrade (find the right repo for the added cleanup scripts)

## Cray-istio Chart
- Add back priority class changes for the istio-operator chart

## Test on a bare metal system

## Verify the upgrade works & kiali (https://v1-49.kiali.io/docs/installation/installation-guide/prerequisites/) upgrade

## Check istio related idle timeout https://github.com/istio/istio/issues/36818 (see shonecyx commented on May 9, 2022) for resiliency testing issue

## Post upgrade Tasks

Things to check:

kubectl get pods -n istio-operator
kubectl get pods -n istio-system
kubectl get pods -n services | grep keycloak
kubectl get pods -n operators | grep kiali
  -- everything's running

kubectl get deployment -n istio-operator istio-operator -oyaml | grep image
kubectl get deployment -n istio-system istiod -oyaml | grep image
kubectl get deployment -n istio-system istio-ingressgateway -oyaml | grep image
kubectl get pods -n services cray-keycloak-0 -oyaml | grep image
  -- look for operator/pilot/proxyv2:1.10.6-cray1-distroless

After upgrade:
  kubectl rollout restart -n services statefulset cray-keycloak ; watch "kubectl get pods -n services | grep keycloak"

kubectl get deployment -n operators cray-kiali-kiali-operator -oyaml | grep image
kubectl get deployment -n istio-system kiali -oyaml | grep image
  -- look for v1.41.0

kubectl create -f - <<EOF
 apiVersion: security.istio.io/v1beta1
 kind: PeerAuthentication
 metadata:
   name: default-strict-mode
   namespace: istio-system
 spec:
   mtls:
     mode: STRICT
 EOF