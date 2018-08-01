## Step 1: Set Up DC/OS Command Line on your Local Machine

Set up the DC/OS command line by clicking on the top left and choosing "install CLI"

![CLI](https://i.imgur.com/p4kqIj6.png)

Click in the dialogue box to copy the command based off of your OS:

![Copy Command](https://i.imgur.com/3rQ2Unj.png)

When prompted for a password:
```
Username: bootstrapuser
Password: deleteme
```

Output should look like this:
```
[centos@ip-10-0-0-243 ~]$ [ -d /usr/local/bin ] || sudo mkdir -p /usr/local/bin &&
> curl https://downloads.dcos.io/binaries/cli/linux/x86-64/dcos-1.11/dcos -o dcos &&
> sudo mv dcos /usr/local/bin &&
> sudo chmod +x /usr/local/bin/dcos &&
> dcos cluster setup https://34.201.120.43 &&
> dcos
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100 14.0M  100 14.0M    0     0  3834k      0  0:00:03  0:00:03 --:--:-- 3835k
SHA256 fingerprint of cluster certificate bundle:
91:31:31:A8:83:E8:05:94:E1:9B:23:34:EE:41:21:32:C8:A1:EB:7F:0D:B1:4C:A5:67:5A:BF:ED:70:B0:1D:C2 [yes/no] yes
34.201.120.43's username: bootstrapuser
bootstrapuser@34.201.120.43's password:
Command line utility for the Mesosphere Datacenter Operating
System (DC/OS). The Mesosphere DC/OS is a distributed operating
system built around Apache Mesos. This utility provides tools
for easy management of a DC/OS installation.

Available DC/OS commands:

	auth           	Authenticate to DC/OS cluster
	cluster        	Manage your DC/OS clusters
	config         	Manage the DC/OS configuration file
	help           	Display help information about DC/OS
	job            	Deploy and manage jobs in DC/OS
	marathon       	Deploy and manage applications to DC/OS
	node           	View DC/OS node information
	package        	Install and manage DC/OS software packages
	service        	Manage DC/OS services
	task           	Manage DC/OS tasks

Get detailed command description with 'dcos <command> --help'.
```

Confirm that dcos is installed and connected to your cluster by running following command

```
dcos node
```

The output should be a list of nodes in the cluster:
```

   HOSTNAME        IP                         ID                     TYPE                 REGION          ZONE       
  10.0.0.101   10.0.0.101  94141db5-28df-4194-a1f2-4378214838a7-S0   agent            aws/us-west-2  aws/us-west-2a  
  10.0.2.100   10.0.2.100  94141db5-28df-4194-a1f2-4378214838a7-S4   agent            aws/us-west-2  aws/us-west-2a
```

### Optional: Set Up DC/OS Command Line on the DC/OS Bootstrap Bastion Host

#### Use this method if you for some reason cannot install the DC/OS CLI onto your local machine such as firewall rules that are out of your control

The instructor will give you access to IP address and credentials that you will need to SSH into.

Note that if you use the Bootstrap node for these labs the Kubernetes dashboard will not work by default, unless you set something up like a VNC server

### SSH into your Bootstrap Node
Navigate to the Lab Assignments spreadsheet that was given to you by your Lab instructor. SSH into the bootstrap using the Public IP
```
$ ssh centos@<BOOTSTRAP_PUBLIC_IP>
Password: <Provided by Instructor>
```
Follow the steps above on setting up the DC/OS CLI, except use LINUX as your operating system.

### Tour DC/OS Catalog

Your instructor will give you a tour of DC/OS UI and catalog. 



## Step 2: Install Kubernetes 
To begin we will be deploying Kubernetes v1.10.3. The latest is 1.10.5, however we will first deploy 1.10.3 so that we can show the non-disruptive upgrade to 1.10.5 in a later lab.

To install Kubernetes enter this command into your terminal

```
dcos package install kubernetes --package-version=1.1.0-1.10.3 --yes
```
**NOTE:** If you attempt to install Kubernetes from the GUI, install the CLI on your local machine with the following command:
```
dcos package install kubernetes --cli --package-version=1.1.0-1.10.3 --yes
```

You can see the installation runbook automation and status of installation of each component with this command

```
dcos kubernetes plan status deploy
```

It should look like this when completed

```
$ dcos kubernetes plan status deploy
deploy (serial strategy) (COMPLETE)
├─ etcd (serial strategy) (COMPLETE)
│  └─ etcd-0:[peer] (COMPLETE)
├─ apiserver (parallel strategy) (COMPLETE)
│  └─ kube-apiserver-0:[instance] (COMPLETE)
├─ mandatory-addons (serial strategy) (COMPLETE)
│  ├─ mandatory-addons-0:[additional-cluster-role-bindings] (COMPLETE)
│  ├─ mandatory-addons-0:[kube-dns] (COMPLETE)
│  ├─ mandatory-addons-0:[metrics-server] (COMPLETE)
│  ├─ mandatory-addons-0:[dashboard] (COMPLETE)
│  └─ mandatory-addons-0:[ark] (COMPLETE)
├─ kubernetes-api-proxy (parallel strategy) (COMPLETE)
│  └─ kubernetes-api-proxy-0:[install] (COMPLETE)
├─ controller-manager (parallel strategy) (COMPLETE)
│  └─ kube-controller-manager-0:[instance] (COMPLETE)
├─ scheduler (parallel strategy) (COMPLETE)
│  └─ kube-scheduler-0:[instance] (COMPLETE)
├─ node (parallel strategy) (COMPLETE)
│  └─ kube-node-0:[kube-proxy, coredns, kubelet] (COMPLETE)
└─ public-node (parallel strategy) (COMPLETE)
```

When all steps are "COMPLETE", confirm that the "dcos kubernetes" CLI was installed.

```
dcos kubernetes
```

## Step 3: Install Kubernetes kubectl Command Line

Install the Kubernetes command line by following instructions [here](https://kubernetes.io/docs/tasks/tools/install-kubectl/)

**For Macs** with brew installed the command is

```
brew install kubectl
```

**For CentOS** the command is:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl
```

**For Ubuntu** the command is

```
sudo apt-get update && sudo apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
sudo touch /etc/apt/sources.list.d/kubernetes.list 
echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update
sudo apt-get install -y kubectl
```

**For Windows** installation:
1. Download the latest release v1.11.0 from this [link](https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/windows/amd64/kubectl.exe)

Or if you have curl installed, use this command:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.11.0/bin/windows/amd64/kubectl.exe
```

2. Add the binary in to your PATH.

Confirm that kubectl is installed and in path /usr/local/bin (it will say it is not connected to dcos cluster yet which is ok)

```
kubectl version
```

## Step 4: Connecting kubectl to DC/OS
Deploy Marathon-LB:
```
dcos package install marathon-lb --yes
```

Save this json as `kubectl-proxy.json` on your local machine:
```
{
  "id": "/kubectl-proxy",
  "instances": 1,
  "cpus": 0.001,
  "mem": 16,
  "cmd": "tail -F /dev/null",
  "container": {
    "type": "MESOS"
  },
  "portDefinitions": [
    {
      "protocol": "tcp",
      "port": 0
    }
  ],
  "labels": {
    "HAPROXY_0_MODE": "http",
    "HAPROXY_GROUP": "external",
    "HAPROXY_0_SSL_CERT": "/etc/ssl/cert.pem",
    "HAPROXY_0_PORT": "6443",
    "HAPROXY_0_BACKEND_SERVER_OPTIONS": "  server kube-apiserver apiserver.kubernetes.l4lb.thisdcos.directory:6443 ssl verify none\n"
  }
}
```

Deploy the kubectl-proxy service:
```
dcos marathon app add kubectl-proxy.json
```

Navigate to the Marathon-LB service and determine:

Connect kubectl to DC/OS:
```
dcos kubernetes kubeconfig --insecure-skip-tls-verify --apiserver-url=https://<MARATHON-LB_PUBLIC_IP>:6443
```

Confirm connection:

```
kubectl get nodes
```

## Step 5: Kubernetes Dashboard (Official UI of Kubernetes)

To access the dashboard run:

```
kubectl proxy
```

Point your browser to:

```
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/
```
