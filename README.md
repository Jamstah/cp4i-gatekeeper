# cp4i-gatekeeper
Investigation into using gatekeeper for mutating cp4i pods.

I started from the gatekeeper experimental deployment with mutation enabled:
- https://github.com/open-policy-agent/gatekeeper/blob/master/deploy/experimental/gatekeeper-mutation.yaml

I added a few things for openshift:
- Remove the PodSecurityPolicy
- Add the use verb for the `nonroot` SCC
- Added `--emit-audit-events` to the audit pod for debug
- Added `--emit-admission-events`, `--log-mutations=true` to the controller pod for debug
- Dropped the replicas to 1 so I can just watch one log

That gave me https://github.com/Jamstah/cp4i-gatekeeper/raw/main/gatekeeper-with-mutations-on-openshift.yaml, which can just apply to an OpenShift cluster to install gatekeeper with mutations.

From there, tested with a few configurations, one being an [init container](https://github.com/Jamstah/cp4i-gatekeeper/raw/main/assign-initcontainer.yaml). Apply that to the cluster and every pod will have a new init container.

For gatekeeper mutation doc, see https://open-policy-agent.github.io/gatekeeper/website/docs/mutation.

Thanks to https://cloud.redhat.com/blog/gatekeeper-mutations for examples and hints.
