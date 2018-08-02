## Scaling to More Nodes

Sometimes you need more Kubernetes infrastructure to run more applications. DC/OS can easily scale the cluster. In this lab we will scale the current Kubernetes deployment for our next labs. We will show you how to scale Kubernetes through the GUI as well as the CLI.

Parameters to be changed:
-  "node_count": 2
-  "public_node_count": 1


## Lab 5a: From the UI

From the UI, go to Services > Kubernetes and choose "Edit" in top right. 

Under "kubernetes" in left hand menu, change the number of "node_count" to 2 and "public_node_count" to 1

![](https://i.imgur.com/0YJxn1r.png)

## Lab 5b: From the CLI

Export the current package configuration into a JSON file called config.json
```
dcos kubernetes describe > config.json
```

Use an editor such as vi to change the config file and update the `"node_count": 2` and `"public_node_count": 1`
```
"kubernetes": {
    "cloud_provider": "(none)",
    "high_availability": true,
    "network_provider": "dcos",
    "node_count": 2,
    "public_node_count": 1,
    "reserved_resources": {
      "kube_cpus": 2,
      "kube_disk": 10240,
      "kube_mem": 2048,
      "system_cpus": 1,
      "system_mem": 1024
    }
```

Scale the cluster: 
```
dcos kubernetes update --options=config.json
```

Verify from the UI:
Navigate to the DC/OS UI --> Services --> Kubernetes

Here you can see the new kube-node-1 as well as kube-public-node-0 that you just scaled

Verify from the CLI:
```
dcos kubernetes plan status deploy
```

## Done with Lab 5
After this lab you should have built a Kubernetes Cluster with these components:
- Highly Available Control Plane
	- 3x etcd instances
	- 3x kube-apiserver instances
	- 3x kube-scheduler instances
	- 3x kube-controller-manager instances
- 2x Kubernetes Worker Nodes
- 1x Kubernetes Public Node
