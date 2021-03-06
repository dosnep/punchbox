﻿# Elasticsearch Deployment

##  Prerequisites
- a cluster up and running with kubernetes installed
-  kubectl configured to deploy resources in your cluster
- min 3 nodes (4CPU 4GB )

##  Deployment types
In this repository two modes of deployment are available :
 - **Dedicated roles** : each nodes have specific roles like master role, data role, ingest role ...
 - **All roles** :  each nodes have all roles.
 
## Configuration 
You can configure your **Elasticsearch** nodes with global environment variables ([Elasticsearch configuration](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html))

To configure you just need to add yours environment variables in this field : `StatefulSet.spec.template.spec.containers.env`

##  Deploy  All Roles Mode
Before deploy elasticsearch you can customize your deployment by modify (CPU, RAM, number of nodes ...) in the file ``elasticsearch/all_roles.yaml``
You recommend to deploy at least **3 elasticsearch nodes** to avoid [split-brain problem.](https://blog.trifork.com/2013/10/24/how-to-avoid-the-split-brain-problem-in-elasticsearch/)

```sh
kubectl create -f kubernetes_resources/elasticsearch/all_roles/all_roles.yaml
```
This file will create 3 kubernetes objects :
- a **Namespace** to group elasticsearch, kibana and logstash resources
- a **Service** in charge to allow communication inter node and to handle customer request
- a **StatefulSet** in charge managing deployment and scaling of a set of Pods


##  Deploy  Dedicated Roles Mode
In this mode you are able to custom each specific role, for example add more storage to data node.
You can deploy following dedicated nodes : 
- **Master Node** are responsible for lightweight cluster-wide actions such as creating or deleting an index, tracking which elasticsearch nodes are part of the cluster, and deciding which shards to allocate to which nodes
- **Data Nodes** hold the shards that contain the documents you have indexed. Data nodes handle data related operations like CRUD, search, and aggregations. These operations are I/O-, memory-, and CPU-intensive.
- **Ingest Nodes** : can execute pre-processing pipelines, composed of one or more ingest processors.
- **Coordinating Nodes** are in charge of route requests, handle the search reduce phase, and distribute bulk indexing

We recommended to deploy at least **3 elasticsearch master nodes** to avoid [split-brain problem.](https://blog.trifork.com/2013/10/24/how-to-avoid-the-split-brain-problem-in-elasticsearch/)
To have a functionnal cluster you need to deploy at least following roles :
- master nodes,
-  data nodes
-  coordinating nodes.

##### deploy  Elasticsearch master nodes 
```sh
kubectl create -f kubernetes_resources/elasticsearch/dedicated_roles/es_master.yaml
```
##### deploy  Elasticsearch data nodes 
```sh
kubectl create -f kubernetes_resources/elasticsearch/dedicated_roles/es_data.yaml
```
##### deploy  Elasticsearch coordinating nodes 
```sh
kubectl create -f kubernetes_resources/elasticsearch/dedicated_roles/es_coordinating.yaml
```
##### deploy  Elasticsearch ingest nodes 
```sh
kubectl create -f kubernetes_resources/elasticsearch/dedicated_roles/es_ingest.yaml
```

## Testing Elasticsearch Cluster

To verify that the deployment works correctly run below commands :
1.  Check that all Pods running
```sh
kubectl get pod -n elk -w
```
2.  Check that a master was elected and nodes join the  cluster
#### For dedicated roles mode
```sh
kubectl logs po/es-master-0 -n elk | grep ClusterApplierService | jq
```
#### For all roles mode
```sh
kubectl logs po/es-cluster-0 -n elk | grep ClusterApplierService | jq
```

When you have a elasticsearch cluster running, you are able to send elasticsearch request.
First run a Pod in your kubernetes cluster to curl elasticsearch
```sh
kubectl run test-elasticsearch --rm -i --tty --image ubuntu -- bash
```
When you pod is running install curl.

```sh
apt-get update
apt-get install curl
```
Send some resquest to elasticsearch cluster

#### For dedicated roles mode
```sh
curl http://es-routing.elk:9200/_cluster/health?pretty
curl -X PUT http://es-routing.elk:9200/twitter?pretty
```
#### For all roles mode
```sh
curl http://es-svc.elk:9200/_cluster/health?pretty
curl -X PUT http://es-svc.elk:9200/twitter?pretty
```


## Scaling the Cluster

#### All Roles Mode
You can scale the cluster by updating the number of `StatefulSet.spec.replicas` field. You can perform this with the `kubectl scale` command.

```sh
kubectl scale sts es-cluster -n elk --replicas=4
```

#### Dedicated Roles Mode
In dedicated role mode you are able to scale each specific node by using `kubectl scale`.

##### scale  Elasticsearch master nodes 
```sh
kubectl scale sts es-master -n elk --replicas=4
```
##### scale  Elasticsearch data nodes 
```sh
kubectl scale sts es-data -n elk --replicas=4
```
##### scale  Elasticsearch coordinating nodes 
```sh
kubectl scale sts es-coordinating -n elk --replicas=4
```
##### scale  Elasticsearch ingest nodes 
```sh
kubectl scale sts es-ingest -n elk --replicas=4
```


