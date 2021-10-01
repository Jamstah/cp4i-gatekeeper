# cp4i-gatekeeper

Example of leveraging gatekeeper for mutating pods managed by IBM Cloud Pak for Integration operators.

When Cloud Pak for Integration operators, they will manage the Deployment/StatefulSet that creates the pods, but do not generally manage the pods themselves. This means that we can mutate the pods using a mutating webhook, and the operator will not try to reset them. If we attempt to mutate the Deployment or StatefulSet, the operator would continually reset them, which would add unnecessary load to the cluster.

I started from the gatekeeper experimental deployment with mutation enabled:
- https://github.com/open-policy-agent/gatekeeper/blob/master/deploy/experimental/gatekeeper-mutation.yaml

I added a few things for openshift:
- Remove the PodSecurityPolicy
- Add the use verb for the `nonroot` SCC
- Added `--emit-audit-events` to the audit pod for debug
- Added `--emit-admission-events`, `--log-mutations=true` to the controller pod for debug
- Dropped the replicas to 1 so I can just watch one log

That gave me https://github.com/Jamstah/cp4i-gatekeeper/raw/main/gatekeeper-with-mutations-on-openshift.yaml, which can just apply to an OpenShift cluster to install gatekeeper with mutations.

From there, tested with a few configurations, addind an annotation and a sidecar using examples from [this blog post]](https://cloud.redhat.com/blog/gatekeeper-mutations) and adding an [init container](https://github.com/Jamstah/cp4i-gatekeeper/raw/main/assign-initcontainer.yaml). Apply that to the cluster and every pod will have a new init container.

For my tests, I added the annotation, sidecar and init container to an MQ Queue Manager pod and an App Connect Toolkit pod, and they were successfully mutated without side effects in the operators.

Some quick warnings:
- The examples I used here will affect all pods on the cluster that aren't in a namespace labelled with `admission.gatekeeper.sh/ignore`. It would be better to use more targetted mutation rules.
- Anyone who can create `Assign`, and potentially even `AssignMetadata` resources effectively has (or could get) cluster admin privileges by mutating the right resources.
- Mutating resources created by operators will change the behaviour of the workloads they manage, and potentially cause problems that will not be supported by IBM.

For gatekeeper mutation doc, see https://open-policy-agent.github.io/gatekeeper/website/docs/mutation.

Thanks to https://cloud.redhat.com/blog/gatekeeper-mutations for examples and hints.
