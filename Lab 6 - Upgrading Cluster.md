# Lab 6 - Upgrading Kubernetes on DC/OS
As the velocity of the Kubernetes project continues to grow, non-disruptive upgrades are increasingly becoming more important so that Operations teams can deliver the latest and greatest features and improvements to their Development teams

Below is information from documentation, which stronly suggest you backup before upgrading in Production. You can find the full documentation on upgrading the cluster [here](https://docs.mesosphere.com/services/kubernetes/1.1.1-1.10.4/upgrade/)


### Step 1: View/List available package versions:
```
$ dcos package describe kubernetes --package-versions
[
  "1.2.0-1.10.5",
  "1.1.1-1.10.4",
  "1.1.0-1.10.3",
  "1.0.3-1.9.7",
  "1.0.2-1.9.6",
  "1.0.1-1.9.4",
  "1.0.0-1.9.3"
]
```

## Step 2: Update DC/OS Kubernetes CLI
Before starting the update process, it is mandatory to update the CLI to the new version, in our case 1.2.0-1.10.5:
```
$ dcos package install kubernetes --cli --package-version=1.2.0-1.10.5
```

## Step 3: Initiate Upgrade:
```
$ dcos kubernetes update --package-version=1.2.0-1.10.5
About to start an update from version <CURRENT_VERSION> to <NEW_VERSION>

Updating these components means the Kubernetes cluster may experience some
downtime or, in the worst-case scenario, cease to function properly.
Before updating proceed cautiously and always backup your data.

This operation is long-running and has to run to completion.
Are you sure you want to continue? [yes/no] yes

2018/03/01 15:40:14 starting update process...
2018/03/01 15:40:15 waiting for update to finish...
2018/03/01 15:41:56 update complete!
```

## Step 4: Validation:

**Method 1:** If you monitor the Kubernetes framework through the DC/OS UI you will be able to see the Kubernetes framework upgrading in a rolling non-disruptive fashion

**Method 2:** Once the scheduler returns you can continue to use the below command:
```
$ dcos kubernetes plan status deploy
deploy (serial strategy) (IN_PROGRESS)
├─ etcd (serial strategy) (IN_PROGRESS)
│  ├─ etcd-0:[peer] (COMPLETE)
│  ├─ etcd-1:[peer] (STARTING)
│  └─ etcd-2:[peer] (PENDING)
├─ apiserver (serial strategy) (PENDING)
│  ├─ kube-apiserver-0:[instance] (PENDING)
│  ├─ kube-apiserver-1:[instance] (PENDING)
│  └─ kube-apiserver-2:[instance] (PENDING)
├─ kubernetes-api-proxy (serial strategy) (PENDING)
│  └─ kubernetes-api-proxy-0:[install] (PENDING)
├─ mandatory-addons (serial strategy) (PENDING)
│  ├─ mandatory-addons-0:[additional-cluster-role-bindings] (PENDING)
│  ├─ mandatory-addons-0:[kubelet-tls-bootstrapping] (PENDING)
│  ├─ mandatory-addons-0:[kube-dns] (PENDING)
│  ├─ mandatory-addons-0:[metrics-server] (PENDING)
│  ├─ mandatory-addons-0:[dashboard] (PENDING)
│  └─ mandatory-addons-0:[ark] (PENDING)
├─ controller-manager (serial strategy) (PENDING)
│  ├─ kube-controller-manager-0:[instance] (PENDING)
│  ├─ kube-controller-manager-1:[instance] (PENDING)
│  └─ kube-controller-manager-2:[instance] (PENDING)
├─ scheduler (serial strategy) (PENDING)
│  ├─ kube-scheduler-0:[instance] (PENDING)
│  ├─ kube-scheduler-1:[instance] (PENDING)
│  └─ kube-scheduler-2:[instance] (PENDING)
├─ node (serial strategy) (PENDING)
│  ├─ mark-kube-node-0-unschedulable (PENDING)
│  ├─ drain-kube-node-0 (PENDING)
│  ├─ kube-node-0:[kube-proxy, coredns, kubelet] (PENDING)
│  └─ mark-kube-node-0-schedulable (PENDING)
└─ public-node (serial strategy) (COMPLETE)
```
