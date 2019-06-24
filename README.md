# acicrome

## Get started

```
git clone git@github.com:RedHat-EMEA-SSA-Team/service-mesh.git
cd acicrome
```

Under manifests files *istio-operator.yml* and *istio.yml* point to upstream community images and files *servicemesh-operator.yml* and *servicemesh.yml* to RHEL 8 based Red Hat Service Mesh.

## Deploy Istio operator



```
oc new-project istio-operator
oc apply -f manifests/istio-operator.yaml -n istio-operator
```

## Deploy Istio control plane
```
oc new-project istio-system
oc apply -f manifests/istio.yml -n istio-system
```

## Deploy book info application

```
export BOOKINFO_PROJECT=bookinfo
oc new-project $BOOKINFO_PROJECT
oc adm policy add-scc-to-user anyuid -z default -n $BOOKINFO_PROJECT
oc adm policy add-scc-to-user privileged -z default -n $BOOKINFO_PROJECT
oc apply -f https://raw.githubusercontent.com/Maistra/bookinfo/master/bookinfo.yaml -n $BOOKINFO_PROJECT
oc apply -f https://raw.githubusercontent.com/Maistra/bookinfo/master/bookinfo-gateway.yaml -n $BOOKINFO_PROJECT

```



## Basic Istio uses cases with bookinfo application

Example are divied to following areas
- Traffic management: https://istio.io/docs/tasks/traffic-management/
- Security: https://istio.io/docs/tasks/security/
- Policies: https://istio.io/docs/tasks/policy-enforcement/
- Telemetry: https://istio.io/docs/tasks/telemetry/



To make demos work, you have to create basic destination rules. Instructions are in here https://istio.io/docs/examples/bookinfo/#apply-default-destination-rules

```
oc apply -f https://raw.githubusercontent.com/istio/istio/release-1.2/samples/bookinfo/networking/destination-rule-all.yaml -n $BOOKINFO_PROJECT
```
Verify that destination rules are in place

```
oc get dr -n $BOOKINFO_PROJECT
```
## Fault injection

```
oc apply -n $BOOKINFO_PROJECT -f rules/vs-fault-injection-base.yml
oc apply -n $BOOKINFO_PROJECT -f rules/vs-fault-injection-delay.yml
```

## Cleaning


```
oc delete project istio-system istio-operator
```
