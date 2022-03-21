# PlatyPaaS CORE
> **PlatyPaaS** - A Kubernetes based platform as a service that is simple, efficient and secure.

## What is CORE?
PlatyPaaS CORE is the following software:
- **CNI** - [Cilium](https://cilium.io/) has been chosen as the CNI for PlatyPaaS.
  - Simple. Using geneve overlay network to allow pods to communicate.
  - Scalable. By using an overlay network rather than an underlying cloud
    provider CNI, we can achieve higher density of pods on our nodes.
  - Secure. IPSec tunneling is configured out of the box to encrypt traffic between
    pods on different nodes. It allows configuration of L3, L4 and L7 network policies
    and has DNS name based enforcement built-in.
- **Ingress** - [Traefik Proxy](https://traefik.io/traefik/) is a reverse proxy
  and load balancer that integrates with Kubernetes using it's own CRDs or standard
  Kubernetes resources such as `Ingress` and `IngressRoute`. Traefik has bbeen configured
  to add B3 tracing headers and handle shutdowns gracefully.
- **Metrics Server** - [Kubernetes Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
  provides metrics that can be used by the in-cluster
  [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

PlatyPaaS Core makes a few assumptions about the Kubernetes environment you're
using:
- `kube-proxy` is installed and running in iptables mode.
- `coredns` is installed.
- Another `cni` is not installed.

## Answers I can think of
To questions you might have.

### Where's [service-mesh name]?
We've made the concsious decision to avoid a service mesh like Istio or Linkerd.
Experience has led us to realise that you probably don't need a service mesh, we
generally see them used for:
- Encryption with mutual TLS, we have IPSec transparent encryption with Cilium.
- Limiting HTTP capabilites to endpoints, we have Layer 7 rules in `CiliumNetworkPolicy`.
- Observability and visibility, we have [Hubble UI](https://docs.cilium.io/en/stable/gettingstarted/hubble/#hubble-gsg)
  and the ability to setup OpenTelemetry
- Traffic management, retries, circuit breakers, mirroring... YAGNI. These all require
  extra configuration that don't lend themselves to a simple continuous delivery
  pipeline. Developers often fall back to the familiar libraries to handle this even
  after having access to a service mesh.

Whatever the blog posts say, service meshes in reality bring more operational overhead
than they're worth. Service meshes are very new and it's not unusual to suffer outages
and breaking changes between versions. Most meshes inject sidecar proxies, this consumes
extra resources across your cluster costing you more in hard cash.

Cilium is adding more features to create an [eBPF Service Mesh](https://cilium.io/blog/2021/12/01/cilium-service-mesh-beta)
which will probably be embraced by PlatyPaaS when it is stable and mature enough.

### Cilium can run without `kube-proxy`, why do you use still use it?
Pretty much every Kubernetes cluster comes with `kube-proxy` ready installed. We
could replace `kube-proxy` with Cilium, this would create some friction in the
overall experience. You'd have to manually remove `kube-proxy`, or consciously not
install it.

#### I read about some place that had problems with `iptables` and `kube-proxy`
We could reconfigure `kube-proxy` with `ipvs`, but there's really
no need to. We're not running clusters with hundreds of nodes just yet, and we want
PlatyPaaS to be up and running quickly on a typical Kubernetes cluster with no tweaks.
