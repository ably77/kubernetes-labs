Credit to Andrew Grzeskowiak for creating this demo for DC/OS

## Kafka Streams Demo

For this demo we will show you how easy it is to run your microservices and dataservices in one single DC/OS cluster using the same resources.

### Use Case
Today we will be simulating a predictive streaming application, more specifically we will be analyzing incoming real-time Airline flight data to predict whether flights are on-time or delayed

### Data Set
The raw dataset we will be using in our load generator is from (here)https://github.com/h2oai/h2o-2/wiki/Hacking-Airline-DataSet-with-H2O]

### Lab 8 - Kafka Streams Application

We will simulate an incoming real-time data stream by using the `load-generator.yaml` below that is a docker container built to send data to Confluent Kafka running on DC/OS. From the Kafka stream, we have an Airline Prediction app that will make predictions on flight delays based on incoming data.

### Explore Workload Generator Deployment configuration
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: kafka-streams-workload-generator
spec:
  selector:
    matchLabels:
      app: kafka-streams-workload-generator
  replicas: 2 # tells deployment to run 2 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: kafka-streams-workload-generator
    spec:
      containers:
      - name: kafka-streams-workload-generator
        image: greshwalk/kafka-streams-workload-generator:latest
```

### Explore Airline Prediction Deployment configuration
The Airline Prediction microservice is a docker container built to read directly off of the DC/OS Kafka broker endpoints to make analytic predictions on flight status:
```
apiVersion: apps/v1beta1
kind: Deployment
metadata:
  name: kafka-streams
spec:
  selector:
    matchLabels:
      app: kafka-streams
  replicas: 2 # tells deployment to run 2 pods matching the template
  template: # create pods using pod definition in this template
    metadata:
      labels:
        app: kafka-streams
    spec:
      containers:
      - name: kafka-streams
        image: greshwalk/kafka-streams:latest
```

## Getting Started

### Step 1: Deploy Confluent Kafka
```
dcos package install confluent-kafka --yes
```

Wait for the installation to complete, you can monitor by using:
```
dcos confluent-kafka plan status deploy
```

### Step 2: Create Confluent Kafka Topics
Once Confluent Kafka is installed, create two topics called `AirlineOutputTopic` and `AirlineInputTopic`
```
dcos confluent-kafka --name confluent-kafka topic create AirlineOutputTopic --partitions 10 --replication 3
```

```
dcos confluent-kafka --name confluent-kafka topic create AirlineInputTopic --partitions 10 --replication 3
```

### Step 3: Deploy Load Generator Service
Deploy the load generator service defined in the Architecture section above:
```
kubectl create -f https://raw.githubusercontent.com/ably77/kubernetes-labs/master/resources/load-generator.yaml
```

### Step 4: Deploy Airline Prediction Service
Deploy the predictive streaming application defined in the Architecture section above:
```
kubectl create -f https://raw.githubusercontent.com/ably77/kubernetes-labs/master/resources/airline-prediction.yaml
```

### Step 5: Confirm App Deployed
```
kubectl get pods
```

### Step 5: Check the App Logs for Output
Navigate to the K8s UI to see the log output of the running app:

If you haven't already, run:
```
kubectl proxy
```

Point your browser to:
```
http://127.0.0.1:8001/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/
```

From the Dashboard navigate to pods --> kafka-streams-XXXXX pod --> logs. Output should resemble below:
```
#####################
Flight Input:1999,10,14,3,741,730,912,849,PS,1451,NA,91,79,NA,23,11,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA,YES,YES
Label (aka prediction) is flight departure delayed: NO
Class probabilities: 0.5955978728809052,0.40440212711909485
#####################
#####################
Flight Input:1987,10,14,3,741,730,912,849,PS,1451,NA,91,79,NA,23,11,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA,YES,YES
Label (aka prediction) is flight departure delayed: YES
Class probabilities: 0.4319916897116479,0.5680083102883521
#####################
#####################
Flight Input:1999,10,14,3,741,730,912,849,PS,1451,NA,91,79,NA,23,11,SAN,SFO,447,NA,NA,0,NA,0,NA,NA,NA,NA,NA,YES,YES
Label (aka prediction) is flight departure delayed: NO
Class probabilities: 0.5955978728809052,0.40440212711909485
#####################
```

Alternatively you can also use the `kubectl log` command:
List running pods:
```
kubectl get pods
```

Output logs in streaming format (-f):
```
$ kubectl logs -f kafka-streams-<PodID>
```

### Step 7: Cleanup
```
$ kubectl get deployments

$ kubectl delete deployment kafka-streams

$ kubectl delete deployment kafka-streams-workload-generator

$ kubectl get deployments

$ kubectl get pods

$ dcos package uninstall confluent-kafka
```

## Done with Lab 8
You just deployed a microservice application on DC/OS that easily connects to a Confluent Kafka dataservice running on the same cluster!


