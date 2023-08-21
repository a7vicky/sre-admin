---
title: 'openshift 4'
origin:
---

OpenShift 4
===
This tutorial is developed for openshift administrator and automation engineer. We all face challanges in day to day life while working with complex openshift cluster and sometime we missed few commad line utilities which are already available and handy to use. 

## Table of Contents

[TBD]

## installer

ref: [troubleshooting](https://github.com/openshift/installer/blob/master/docs/user/troubleshooting.md)

## RHCOS

ref: [Documentation](https://access.redhat.com/documentation/en-us/openshift_container_platform/4.4/html/architecture/architecture-rhcos)

## access cluster

``` typescript
/* To access the cluster export kubeconfig file available in 
   installation_dir/auth directory*/
# export KUBECONFIG=<installation_dir>/auth/kubeconfig

// Now test the current user 
# oc whoami

// show server and colsole
# oc whoami --show-console
# oc whoami --show-server


// interactive completion of oc commands
# oc completion bash > bash_completion.sh
# source bash_completion.sh

// create a kubeconfig file out of credential
# oc login -u kubeadmin -p <your_password> \
  https://api.<cluster_name>.<base_domain>:6443
# oc config view --flatten > config
```

## debug cluster

``` typescript
/* collects the information from your cluster using the default 
   plug-in image and command, writing into ./must-gather.local.<rand> */
# oc adm must-gather

// gather information with a specific local folder to copy to
# oc adm must-gather --dest-dir=/local/directory

// Set a specific node to use - by default a random master will be used
# oc adm must-gather --node-name={NODE_NAME}

// gather information using multiple plug-in images
# oc adm must-gather --image=quay.io/kubevirt/must-gather \ 
  --image=quay.io/openshift/origin-must-gather

// gather information using a specific image stream plug-in 
// (helpful in disconnected environment)
# oc adm must-gather --image-stream=openshift/must-gather:latest

/* dumps the logs of all of the pods in the project, these logs are 
   dumped into different directories pod name */
# oc cluster-info dump --output-directory=/local

/* dump all namespaces pod logs into different directories
   based on namespace and pod name */
# oc cluster-info dump -A --output-directory=/local

// dump a set of namespaces to /local
# oc cluster-info dump --namespaces default,kube-system \
  --output-directory=/local

```

### debug node
``` typescript
// access RHCOS host as an administrator
# oc debug node/master-0

// Override the debug image used by the targeted container
# oc debug node/master-0 --image={IMAGE}

```

### node logs
``` typescript
// collect node logs 
# oc adm node-logs master-0

// Show kubelet logs from all masters
# oc adm node-logs --role master -u kubelet

// list all the available log path
# oc adm node-logs master-0 --path=/

// Retrieve the specified path within the node's /var/logs/ folder
# oc adm node-logs master-0 --path='journal' --tail=100
```

### manage node


## logLevel

### cri-o

``` typescript
// Increase cri-o logLevel
>  Options are: error (default), fatal, panic, warn, info, and debug
// create the custom resource to apply loglevel

$ cat <<EOF > custom-loglevel.yaml
apiVersion: machineconfiguration.openshift.io/v1
kind: ContainerRuntimeConfig
metadata:
 name: custom-loglevel
spec:
 machineConfigPoolSelector:
   matchLabels:
     custom-crio: custom-loglevel
 containerRuntimeConfig:
   logLevel: debug
EOF

$ oc create -f custom-loglevel.yaml

// Verify the resource created

$ oc get ctrcfg


/*To roll out the loglevel changes to all the worker nodes , 
 add custom-crio: custom-loglevel under labels in 
 the machineConfigPool config */

$ oc edit machineconfigpool worker

/*
apiVersion: machineconfiguration.openshift.io/v1
kind: MachineConfigPool
metadata:
  creationTimestamp: 2019-04-10T16:39:39Z
  generation: 1
  labels:
    custom-crio: custom-loglevel       ---------------> this needs to add
*/

/*Check to ensure that a new 99-worker-XXX-containerruntime 
  is created and that a new rendered worker is created*/

$ oc get machineconfigs
$ oc get mcp

```
> ref: for more insights follow 
> [ContainerRuntimeConfigDesign](https://github.com/openshift/machine-config-operator/blob/master/docs/ContainerRuntimeConfigDesign.md)  document.

### kubelet
``` typescript

```
> ref: [kubeletconfig](https://github.com/openshift/machine-config-operator/blob/master/docs/KubeletConfigDesign.md)
## certificate
``` typescript
// list all the certificatesigningrequests
# oc get csr

// approve pending csr
# oc get csr -o go-template='{{range .items}} \
{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}} \
{{end}}' | xargs oc adm certificate approve

```
> Read more about [certificate api](https://github.com/kubernetes/enhancements/blob/master/keps/sig-auth/20190607-certificates-api.md)
> and [kubelet tls bootstrap](https://github.com/kubernetes/community/blob/1fd524165bcf54d4bef99adb8332df72f4f88d5c/contributors/design-proposals/cluster-lifecycle/kubelet-tls-bootstrap.md)

## cluster version operator

``` typescript
// get the cluster version of openshift
# oc get clusterversion

// extract the current update image from the ClusterVersion object
# 

/* get the current cvo payload release info user "--help" to get more 
   descruptive details abount relese info command output */
# oc adm realease info
# oc adm release extract
# oc adm release extract --from=

// list the available updates 
# oc get clusterversion -o \
jsonpath="{range .items[*].status.availableUpdates[*]}{.image}{'\t'} \
{.version}{'\t'}{.force}{'\n'}"

// list the available updates (alternative of above command)
// user --help to get more details about the upgrade command
# oc adm upgrade

// upgrade to specific version
# oc adm upgrade --to={VERSION}

/* Provide a release image to upgrade to.
   WARNING: This option does not check for upgrade 
   compatibility and may break your cluster */
# oc adm upgrade --to-image=IMAGE

// To get a list of objects managed by the CVO


// swirch cluster version channel
# oc patch \
    --patch='{"spec": {"channel": "prerelease-4.1"}}' \
    --type=merge \
    clusterversion/version
 
// get a list of current overrides:
#  

/* When you just want to turn off the cluster-version operator
   instead of fiddling with per-object overrides, you can:*/
# oc scale --replicas 0 -n openshift-cluster-version \
  deployments/cluster-version-operator
```
>> Get more details [here](https://github.com/openshift/cluster-version-operator/tree/master/docs)

## cluster operators
``` typescript
// collect the status oc cluster operators
# oc get clusteroperators

/* get the details of cluster operator ; the message and reason 
   will be helpful in debugging if the operator is not available or 
   in degradded state*/   
# oc describe co/authentication

// Collect debugging data for all clusteroperators
# oc adm inspect clusteroperator

/* Collect debugging data for the specific operator example: 
  "machine-config" clusteroperator */
# oc adm inspect co/machine-config
```

>> We will discuss about each cluster operator one by one in the later section

### machine-config
> For more information follow the [doc](https://github.com/openshift/machine-config-operator/tree/master/docs)

``` typescript

```
### machine-api

### kube-apiserver

### kube-controller-manager

### kube-scheduler

### openshift-apiserver

### openshift-controller-manager

### service-catalog-apiserver

### service-catalog-controller-manager

### monitoring

### authentication

### authentication

### ingress

### image-registry

### network

### dns

### operator-lifecycle-manager

### operator-lifecycle-manager-catalog

### operator-lifecycle-manager-packageserver

### marketplace

## api

```typescript
// list of api groups
# oc api-versions

// list api-resources
# oc api-resources

// list api resources of a particular api group
# oc api-resources --api-group operators.coreos.com

// expalin will provide api documentaion
# oc explain pod.spec.containers

// explain resources for a particular api group
# oc explain --api-version=machineconfiguration.openshift.io/v1 kubeletconfigs

// collect the raw metrics of apiserver
# oc get --raw /metrics

// get the healthz of api
# oc get --raw /healthz

// list the subgroup,resource,verb and scope of created resources
# oc get --raw /apis/apps/v1 | python -m json.tool
```

>>Find kubernetes 1.18 api documentation [here](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.18/)

> In 4.x the [openshift apiserver](https://github.com/openshift/api) pod is running to extended ( using aggregation layer) the kubernetes api.
> Read more about [aggregated apiserver](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/api-machinery/aggregated-api-servers.md).
> Use [api-server-builder](https://github.com/kubernetes-sigs/apiserver-builder-alpha/blob/master/README.md)

### list of api groups
#### admissionregistration.k8s.io
#### apiextensions.k8s.io
ref: [authentication flow](https://kubernetes.io/docs/tasks/access-kubernetes-api/configure-aggregation-layer/#authentication-flow)
#### apiregistration.k8s.io
#### apps
#### authentication.k8s.io
#### authorization.k8s.io
#### autoscaling.openshift.io
#### autoscaling
#### batch
#### build.openshift.io
#### certificates.k8s.io
#### cloudcredential.openshift.io
#### config.openshift.io
#### console.openshift.io
#### coordination.k8s.io
#### events.k8s.io
#### extensions
#### image.openshift.io
#### imageregistry.operator.openshift.io
#### ingress.operator.openshift.io
#### k8s.cni.cncf.io
#### machine.openshift.io
#### machineconfiguration.openshift.io
#### metal3.io
#### metrics.k8s.io
#### monitoring.coreos.com
#### network.openshift.io
#### network.operator.openshift.io
#### networking.k8s.io
#### node.k8s.io
#### oauth.openshift.io
#### operator.openshift.io
#### operators.coreos.com
#### packages.operators.coreos.com
#### policy
#### project.openshift.io
#### quota.openshift.io
#### route.openshift.io
#### samples.operator.openshift.io
#### scheduling.k8s.io
#### security.openshift.io
#### storage.k8s.io
#### template.openshift.io
#### tuned.openshift.io
#### user.openshift.io
#### v1
#### whereabouts.cni.cncf.io

## apiextensions

### custom resource definition
### aggregation layer

## etcd

```typescript
// for generic etcdctl command user etcdctl container
// collect etcd metrics in ocp 4.4
# ETCDS=($(oc get pods -n openshift-etcd -l app=etcd -o jsonpath='{.items[*].metadata.name}'))
# for pod in "${ETCDS[@]}"; do
oc exec -it $pod -n "openshift-etcd" -c "etcd-metrics" -- sh -c  \
' CACERT=/etc/kubernetes/static-pod-certs/configmaps/etcd-metrics-proxy-serving-ca/ca-bundle.crt; \
CERT=/etc/kubernetes/static-pod-certs/secrets/etcd-all-serving-metrics/etcd-serving-metrics-`hostname -f`.crt; \
KEY=/etc/kubernetes/static-pod-certs/secrets/etcd-all-serving-metrics/etcd-serving-metrics-`hostname -f`.key; \
curl --cacert $CACERT --key "$KEY" --cert "$CERT" https://localhost:9979/metrics' &> ${pod}_metrics.txt; \
done

```

## disaster recovery 
## Appendix and FAQ