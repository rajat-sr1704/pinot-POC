---
title: "[]{#_gjdgxs .anchor}Running Pinot with Kubernetes(Minikube)"
---

## **Prerequisites**

### **Kubernetes**

If you haven\'t yet set up a Kubernetes cluster, see the links below for
instructions:

-   [[Enable Kubernetes on
    > Docker-Desktop]{.underline}](https://docs.docker.com/docker-for-mac/kubernetes/)

-   [[Install Minikube for local
    > setup]{.underline}](https://kubernetes.io/docs/tasks/tools/install-minikube/)

Make sure to run with enough resources:

  -----------------------------------------------------------------------
  minikube start \--vm=true \--cpus=2 \--memory=4g \--disk-size=30g
  \--driver=docker
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

-   [[Set up a Kubernetes Cluster using Amazon Elastic Kubernetes
    > Service (Amazon
    > EKS)]{.underline}](https://docs.pinot.apache.org/basics/getting-started/public-cloud-examples/aws-quickstart)

### **Pinot**

## **Set up a Pinot cluster in Kubernetes**

Create a directory for Pinot POC where you'll save schema, config and
other required files for running pinot and the benchmarking POC.

### **Start Pinot with Helm**

  -----------------------------------------------------------------------
  helm repo add pinot
  https://raw.githubusercontent.com/apache/pinot/master/helm\
  kubectl create ns pinot-benchmark
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

After this step run:

+-----------------------------------------------------------------------+
| helm install pinot pinot/pinot \\                                     |
|                                                                       |
| -n pinot-benchmark \\                                                 |
|                                                                       |
| \--set cluster.name=pinot \\                                          |
|                                                                       |
| \--set server.replicaCount=1 \\                                       |
|                                                                       |
| \--set controller.replicaCount=1 \\                                   |
|                                                                       |
| \--set broker.replicaCount=1 \\                                       |
|                                                                       |
| \--set minion.enabled=false                                           |
+=======================================================================+
+-----------------------------------------------------------------------+

If this command throws an error regarding any namespace pinot-benchmark.

Go to the file directory of your Pinot POC and add a file named
'pinot-startup-fix.yaml' and write this code in it:

  -----------------------------------------------------------------------
  apiVersion: v1\
  kind: Namespace\
  metadata:\
  name: pinot-benchmark\
  labels:\
  app.kubernetes.io/managed-by: \"Helm\"\
  annotations:\
  meta.helm.sh/release-name: \"pinot\"\
  meta.helm.sh/release-namespace: \"pinot-benchmark\"
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

Then run:

kubectl apply -f pinot-startup-fix.yaml

In the terminal.

And then run: (you can modify these configurations according to your
pinot development)

+-----------------------------------------------------------------------+
| helm install pinot pinot/pinot \\                                     |
|                                                                       |
| -n pinot-benchmark \\                                                 |
|                                                                       |
| \--set cluster.name=pinot \\                                          |
|                                                                       |
| \--set server.replicaCount=1 \\                                       |
|                                                                       |
| \--set controller.replicaCount=1 \\                                   |
|                                                                       |
| \--set broker.replicaCount=1 \\                                       |
|                                                                       |
| \--set minion.enabled=false                                           |
+=======================================================================+
+-----------------------------------------------------------------------+

This will start creating containers of pinot in minikube without any
error.

Check for running pods for pinot using:

  -----------------------------------------------------------------------
  kubectl get pods -n pinot-benchmark
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

Then run:

  -----------------------------------------------------------------------
  kubectl port-forward service/pinot-controller 9000:9000 -n
  pinot-benchmark
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

It will start your pinot cluster UI on your local host. You can access
it using:

[[http://localhost:9000/]{.underline}](http://localhost:9000/)

## **Adding Table into Pinot** 

Creating Schema and Table Configuration for Data Ingestion.

Create a file table-schema-shipments.json in the same directory where
you add your table schema in it:

table-schema-shipment.json

Create your schema file according to your need. For making shipments
table you can copy it from the github repo from the file
table-schema-shipment.json into your system.

[[https://github.com/rajat-sr1704/pinot-POC]{.underline}](https://github.com/rajat-sr1704/pinot-POC)

Then create a file name as tableConfig-shipments.json and add the
following lines of code in it:(you can change it according to your
ingestion and pinot requirements):

table-Config-shipment.json

Create your own tableConfig File according to your need if you want to
go with shipments table you can clone it from this github repo:

[[https://github.com/rajat-sr1704/pinot-POC]{.underline}](https://github.com/rajat-sr1704/pinot-POC)

After that go to terminal and copy both of these files from your local
system into the running pinot cluster's pod files:

  -----------------------------------------------------------------------
  kubectl cp ./table_Config_shipment.json
  pinot-controller-0:/opt/pinot/data/table-config-shipment.json -n
  pinot-benchmark
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

  -----------------------------------------------------------------------
  kubectl cp ./table_schema_shipment.json
  pinot-controller-0:/opt/pinot/data/table-schema-shipment.json -n
  pinot-benchmark
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

Then go inside the running pod for running Addtable Command, go inside
the pod using this command:\
kubectl exec -it pinot-controller-0 -n pinot-benchmark \-- /bin/bash

Inside the pod give this command and this will add your table to the
pinot:

  -----------------------------------------------------------------------
  /opt/pinot/bin/pinot-admin.sh AddTable -schemaFile
  /opt/pinot/data/table-schema-shipment.json -tableConfigFile
  /opt/pinot/data/table-config-shipment.json -exec
  -----------------------------------------------------------------------

  -----------------------------------------------------------------------

Your table will get added to the pinot cluster you can go to query
editor in the pinot cluster and click on the table name to see the table
content:\
![](./image1.png){width="8.119792213473316in"
height="4.16459208223972in"}
