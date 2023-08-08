# openstack-stf

Guide to install service telemetry framework to extract metrics and events from 

CLIENT(S) ----->  openstack/devstack/packstack private cloud platform <br>
SERVER ----->  openshift container platform (eus version release only)
<br>
<br>

SERVER SIDE CONFIGURATION  - Openshift Container Platform

**Creating a namespace**<br>
Create a namespace to hold the STF components. The service-telemetry namespace is used throughout the documentation:
```
oc new-project service-telemetry
```
<br>
<br>
**Creating an OperatorGroup**<br>
Create an OperatorGroup in the namespace so that you can schedule the Operator pods.

```
oc create -f - <<EOF
> apiVersion: operators.coreos.com/v1
> kind: OperatorGroup
> metadata:
> name: service-telemetry-operator-group
> namespace: service-telemetry
> spec:
> targetNamespaces:
> - service-telemetry
> EOF
```
operatorgroup.operators.coreos.com/service-telemetry-operator-group created
<br>
<br>
**Enabling the OperatorHub.io Community Catalog Source**<br>
Before you install data storage and visualization operators, you must have access to the resources on the
OperatorHub.io Community Catalog Source

```
oc create -f - <<EOF
> apiVersion: operators.coreos.com/v1alpha1
> kind: CatalogSource
> metadata:
> name: operatorhubio-operators
> namespace: openshift-marketplace
> spec:
> sourceType: grpc
> image: quay.io/operatorhubio/catalog:latest
> displayName: OperatorHub.io Operators
> publisher: OperatorHub.io
> EOF
```
catalogsource.operators.coreos.com/operatorhubio-operators created

**Subscribing to the AMQ Certificate Manager Operator**<br>
Subscribe to the AMQ Certificate Manager Operator via the redhat-operators CatalogSource

```
oc create -f - <<EOF
> apiVersion: operators.coreos.com/v1alpha1
> kind: Subscription
> metadata:
> name: amq7-cert-manager-operator
> namespace: openshift-operators
> spec:
> channel: 1.x
> installPlanApproval: Automatic
> name: amq7-cert-manager-operator
> source: redhat-operators
> sourceNamespace: openshift-marketplace
> EOF
```
subscription.operators.coreos.com/amq7-cert-manager-operator created
<br>
<br>
**Subscribing to the Elastic Cloud on Kubernetes Operator**
To enable the Elastic Cloud on Kubernetes Operator, create the following manifest in your Red Hat
OpenShift Container Platform environment:
```
oc create -f - <<EOF
> apiVersion: operators.coreos.com/v1alpha1
> kind: Subscription
> metadata:
> name: elasticsearch-eck-operator-certified
> namespace: service-telemetry
> spec:
> channel: stable
> installPlanApproval: Automatic
> name: elasticsearch-eck-operator-certified
> source: certified-operators
> sourceNamespace: openshift-marketplace
> EOF
```
<br>

**Subscribing to the Service Telemetry Operator**<br>
To create the Service Telemetry Operator subscription, enter the oc create -f command:

```
oc create -f - <<EOF
> apiVersion: operators.coreos.com/v1alpha1
> kind: Subscription
> metadata:
> name: service-telemetry-operator
> namespace: service-telemetry
> spec:
> channel: stable-1.3
> installPlanApproval: Automatic
> name: service-telemetry-operator
> source: redhat-operators
> sourceNamespace: openshift-marketplace
> EOF
```
subscription.operators.coreos.com/service-telemetry-operator created<br><br>
Check the list of installed operators:

```
oc get csv -n service-telemetry
```

**Check Service Telemetry Framework pods**
```
oc get po
```
> EXPECTED OUTPUT : All the operator pods should be in running state


**Creating a ServiceTelemetry object with a override defaults**
```
apiVersion: infra.watch/v1beta1
kind: ServiceTelemetry
metadata:
  name: default
spec:
  alerting:
    enabled: true
    alertmanager:
      storage:
        storageClassName: "SC_NAME"
        size: XXG
        receivers:
          snmpTraps:
            enabled : false
            target: IPv4
  backends:
    events:
      elasticsearch:
      enabled: true
      storage:
        storageClassName: "SC_NAME"
        size: 400G
  metrics:
    prometheus:
      enabled: true
      scrapeInterval: 10s
      storage:
        storageClassName: "SC_NAME"
        size: XG
        retention: 90d
  graphing:
    enabled: false
  grafana:
    adminPassword: root
    adminUser: root
    disableSignoutMenu: false
    ingressEnabled: false
  highAvailability:
  enabled: false
  transports:
    qdr:
     enabled: true
    web:
     enabled: false
  clouds:
    - name: <cloudname>
      metrics:
        collectors:
          - collectorType: collectd
            subscriptionAddress: collectd/telemetry
            debugEnabled: false
          - collectorType: sensubility
            subscriptionAddress: sensubility/telemetry
          - collectorType: ceilometer
            subscriptionAddress: ceilometer/metering.sample
            debugEnabled: false
       events:
         collectors:
           - collectorType: collectd
             subscriptionAddress: collectd/notify
             debugEnabled: false
           - collectorType: ceilometer
             subscriptionAddress: ceilometer/event.sample
             debugEnabled: false
```
<br>
<br>

**To view the STF deployment logs in the Service Telemetry Operator, use the oc logs command**<br>
```
 oc logs --selector name=service-telemetry-operator
```

**To get Application Pod Status and PVC attached to openstack telemetry pods**<br><br>

```
oc get pods, pvc
```
