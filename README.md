# NOTE: This README is a design sketch. No implementation is done

Maybe this should be incorporated in Karpenter. Then the nodepool selection could be done when the webhook is called and tolerations could he added based on the nodepool and affinity on architechture. Annotations could be added to the pod with information so that if it becomes pending Karpenter doesn't need to the repeat the work done during webhook evaluation.

<h1 align="center">Kubernetes Architecture Mutating Admission Webhook</h1>
<p align="center">An image architecture aware Kubernetes Mutating Admission Webhook</p>

<p align="center">
<a  target="_blank"><img src="https://img.shields.io/github/v/release/msvticket/kube-arch-webhook" /></a>
<a  target="_blank"><img src="https://img.shields.io/github/downloads/msvticket/kube-arch-webhook/total"/></a>
<a  target="_blank"><img src="https://img.shields.io/github/issues/msvticket/kube-arch-webhook"/></a>
<a  target="_blank"><img src="https://img.shields.io/github/go-mod/go-version/msvticket/kube-arch-webhook"/></a>
</p>

**kube-arch-webhook** is a kubernetes Mutating Admission Webhook that will add tolerations and/or affinity to pods based on the container image architectures (platforms) present in a Pod.

Pods that already has tolerations or affinities matching any of the configured ones will not be treated by the webhook. Containers with imagePullPolicy will not be considered.

## Deploy - Helm

```bash
helm repo add jxgh https://elementtech.github.io/kube-arch-scheduler/
helm repo update
helm install -n kube-system jxgh/kube-arch-webhook
```

## Core Values

These are the values most likely that you want to change with the default values.

```yaml
# The time to cache the architectures available for an image where the imagePullPolicy in the pod is IfNotPresent. The duration is parsed by github.com/mashiike/longduration.
ifNotPresentCacheTime: 7d

# The time to cache the architectures available for an image where the imagePullPolicy in the pod is PullAlways. null means don't cache.
pullAlwaysCacheTime: null

# This is relevant if your Kubernetes pod does not have permissions to your private registries.
# dockerconfig.json is a base64 encoded docker config file, it will be used
# to pull the image manifests. The pod needs to have the secret mounted at
# /root/.docker/config.json. This is only needed if you are using a private
# registry. If you are using a public registry, you can leave this empty.
dockerConfigSecretName: ""

# The tolerations or affinity to add to pod whose containers can run an architecture. requireNodeLabels is added to affinity.nodeAffinity.requiredDuringSchedulingIgnoredDuringExecution.nodeSelectorTerms
architectures: {}
# Example:
#  arm64:
#    taintsToTolerate: 
#    - effect: NoExecute
#      key: ARM
#      value: "true"
#    requireNodeLabels:
#    - kubernetes.io/arch: arm64

# MutatingWebhookConfiguration, see https://kubernetes.io/docs/reference/access-authn-authz/extensible-admission-controllers/#matching-requests-objectselector
failurePolicy: Ignore
namespaceSelector:
  matchExpressions:
  - key: kubernetes.io/metadata.name
    operator: NotIn
    values: [kube-system]
objectSelector: {}
reinvocationPolicy: IfNeeded
```

## Development

Use this command to run the scheduler locally while connected to your Kubernetes cluster's context:

```shell
go run main.go --authentication-kubeconfig ~/.kube/config --authorization-kubeconfig ~/.kube/config --config=./example/scheduler-config.yaml --v=2
```

You can use the [example deployment](example/busybox.yaml) in order to test the scheduler on a live pod:

```
kubectl deploy -f example/busybox.yaml
```

## Open questions

There are pods that should be ignored. Those that belong to a Daemonset certainly. Others?

## Code to use or be inspired by

- github.com/jellydator/ttlcache/v3
- github.com/mashiike/longduration
- https://github.com/viney-shih/go-cache
- https://github.com/seaguest/cache
- https://github.com/kubernetes-sigs/karpenter/blob/c5467c08c07a93d2ff2b09c677a4152c14f19d1d/pkg/scheduling/taints.go#L41

  ## References

  https://bshayr29.medium.com/build-your-own-admission-controllers-in-kubernetes-using-go-bef8ba38d595
