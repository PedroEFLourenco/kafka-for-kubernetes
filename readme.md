
# Simple Kafka Environment for Kubernetes

This repository contains all the files necessary to deploy on Kubernetes a simple Kafka environment comprised of: Zookeeper, Kafka, Schema Registry and Kafka Connect. Continue reading this file for a local deployment guide.

## Requirements
The requirements to deploy this kafka environment are:

* A Kubernetes Cluster (I suggest [Minikube](https://minikube.sigs.k8s.io/docs/start/) for local deployments).

* To use Minikube you'll also need [Docker](https://docs.docker.com/get-docker/).

* [Kubectl](https://kubernetes.io/docs/tasks/tools/) to operate the kubernetes cluster.

StackEdit stores your files in your browser, which means all your files are automatically saved locally and are accessible **offline!**

## Installation Guide

This guide will focus on local deployment of the kafka environment. To deploy on your cloud environment just skip the Minikube setup steps and don't forget to add auth configs to all deployments.

  
1.  **Minikube**

	i. `minikube start` (To start the Kubernetes Cluster)

	ii. `minikube dashboard --url` (Optional - To launch a web application for you to have a visual environment to check the kubernetes cluster and deployment's health).

	iii. `eval $(minikube docker-env)` (Optional - to use minikube's docker environment instead of local environment, so that image builds are available on minikube).

  

2. Deploy Applications - Respecting order of dependencies between deployments

	i. **Deploy Zookeeper**

	`kubectl create -f zookeeper-deployment.yaml`
	`kubectl create -f zookeeper-ip-service.yaml`

	ii. **Deploy Kafka**

	`kubectl create -f kafka/kafka-deployment.yaml`
	`kubectl create -f kafka/kafka-ip-service.yaml`

	iii. **Deploy Schema Registry**

	`kubectl create -f schema-registry/schema-registry-deployment.yaml`
	`kubectl create -f schema-registry/schema-registry-ip-service.yaml`

	iv. **Deploy Kafka Connect**

	`docker build -t kafka-connect-custom:1.0.0 kafka-connect`
	`kubectl create -f kafka-connect/kafka-connect-deployment.yaml`
	`kubectl create -f kafka-connect/kafka-connect-ip-service.yaml`

  

3. Make applications accessible outside of the Kubernetes cluster
	i. Kafka - `kubectl port-forward services/kafka-cluster 9092:9092`
	ii. Schema Registry - `kubectl port-forward service/kafka-schema-registry 8081:8081`
	iii. Kafka Connect - `kubectl port-forward service/kafka-connect 8083:8083`
	iv. Zookeeper - `kubectl port-forward service/zookeeper 2181:2181`

  
To make the applications accessible I suggest the port-forward approach instead of creating ingress resources on the kubernetes cluster because there are some issues with Minikube/Docker networking that prevent ingresses from working properly in some environments (namely Apple Silicon).
  

## Access to applications

The Environment is now up and running and applications should be accessible at:

*  **Schema Registry** - http://localhost:8081
*  **Kafka Connect** - http://localhost:8083
*  **Kafka** - localhost:9092
*  **Zookeeper** - localhost:2181

Obviously, you can interact with the applications programatically, but for ad-hoc operations, I suggest to download the [Confluent CLI](https://docs.confluent.io/confluent-cli/current/install.html), since it has the resources to create topics, launch producers and consumers with schema support. It is specially helpful for quickly testing stuff.


## Helpers

* Confluent CLI commands examples:

*  **Launch Producer with schema support -** kafka-avro-console-producer --topic test.topic --bootstrap-server localhost:9092 --property schema.registry.url=http://localhost:8081 --property value.schema.id=1 --property key.schema.id=2 --property key.serializer=org.apache.kafka.common.serialization.StringSerializer --property parse.key=true --property key.separator=":"

*  **Launch Consumer with schema support -** kafka-avro-console-consumer --topic test.topic --bootstrap-server localhost:9092 --property schema.registry.url=http://localhost:8081 --property value.schema.id=1 --property key.schema.id=2 --property key.deserializer=org.apache.kafka.common.serialization.StringDeserializer --property print.key=true --property key.separator=":" --from-beginning

*  **Create Topic -** kafka-topics --create --topic test.topic --bootstrap-server localhost:9092

* Schema registry request bodies examples:
  

***Body to create a schema for a message value***

Request URL: `<SchemaRegistryUrl>/subjects/<topic-name>-value/versions`

```
{
"schema":"{\"name\":\"event\",\"type\":\"record\",\"fields\":[{\"name\":\"status\",\"type\":\"string\"},{\"name\":\"created_at\",\"type\":\"string\"}]}",
"schemaType":"AVRO"
}
```  

***Body to create a schema for a message key***

Request URL: `<SchemaRegistryUrl>/subjects/<topic-name>-key/versions`

```
{
"schema":"{\"type\":\"string\"}",
"schemaType":"AVRO"
}
```
  

## Troubleshooting

In this repository, we are using Confluent's docker images made compatible with Apple Silicon.

If you have any issues with the images, check Confluent's [DockerHub](https://hub.docker.com/u/confluentinc) profile for the default images from Confluent and replace them in the yaml/dockerfile files.
