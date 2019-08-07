# Service Mesh for OCP4

## Get started

```
git clone git@github.com:RedHat-EMEA-SSA-Team/service-mesh.git
cd service-mesh
```

Under manifests files *istio-operator.yml* and *istio.yml* point to upstream community images and files *servicemesh-operator.yml* and *servicemesh.yml* to RHEL 8 based Red Hat Service Mesh.

## Install Kiali Operator

```
bash <(curl -L https://git.io/getLatestKialiOperator) --operator-image-version v1.0.0 --operator-watch-namespace '**' --accessible-namespaces '**' --operator-install-kiali false
```


## Installa Jeager Operator

```
oc new-project observability # create the project for the jaeger operator
oc create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/crds/jaegertracing_v1_jaeger_crd.yaml
oc create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/service_account.yaml
oc create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/role.yaml
oc create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/role_binding.yaml
oc create -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/operator.yaml
```

## Deploy Istio operator


```
oc new-project istio-operator
oc apply -n istio-operator -f https://raw.githubusercontent.com/Maistra/istio-operator/maistra-0.12/deploy/maistra-operator.yaml
```

## Deploy Istio control plane

Multitenancy is enabled by default in TP12.

```
oc new-project istio-system
oc apply -f manifests/servicemesh.yml -n istio-system
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
oc delete smcp -n istio-system full-install
oc delete project istio-system
bash <(curl -L https://git.io/getLatestKialiOperator) --uninstall-mode true
oc delete -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/crds/jaegertracing_v1_jaeger_crd.yaml
oc delete -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/service_account.yaml
oc delete -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/role.yaml
oc delete -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/role_binding.yaml
oc delete -n observability -f https://raw.githubusercontent.com/jaegertracing/jaeger-operator/v1.13.1/deploy/operator.yaml
oc delete project observability
oc delete -n istio-operator -f https://raw.githubusercontent.com/Maistra/istio-operator/maistra-0.12/deploy/maistra-operator.yaml
oc delete project istio-operator
```
