# **PINOT BENCHMARKING POC**

## **Running Pinot with Kubernetes (Minikube)**

## **Prerequisites**

### **Kubernetes**

If you haven't yet set up a Kubernetes cluster, refer to the links below for instructions:

- [Enable Kubernetes on Docker-Desktop](https://docs.docker.com/docker-for-mac/kubernetes/)
- [Install Minikube for local setup](https://kubernetes.io/docs/tasks/tools/install-minikube/)
- [Set up a Kubernetes Cluster using Amazon Elastic Kubernetes Service (Amazon EKS)](https://docs.pinot.apache.org/basics/getting-started/public-cloud-examples/aws-quickstart)

Make sure to run Minikube with enough resources:

```bash
minikube start --vm=true --cpus=2 --memory=4g --disk-size=30g --driver=docker
```
## **Set up a Pinot cluster in Kubernetes**

Create a Working Directory

Create a directory for the Pinot proof of concept (POC) where you’ll save schema, config, and other required files for running Pinot and the benchmarking POC.

Start Pinot with Helm

Add the Pinot Helm repository and create the necessary namespace:
```bash
helm repo add pinot https://raw.githubusercontent.com/apache/pinot/master/helm

kubectl create ns pinot-benchmark
```

Install Pinot using Helm:

```bash
helm install pinot pinot/pinot \
  -n pinot-benchmark \
  --set cluster.name=pinot \
  --set server.replicaCount=1 \
  --set controller.replicaCount=1 \
  --set broker.replicaCount=1 \
  --set minion.enabled=false
```

If this command throws an error regarding the pinot-benchmark namespace, create a file named pinot-startup-fix.yaml with the following content:

```bash
apiVersion: v1
kind: Namespace
metadata:
  name: pinot-benchmark
  labels:
    app.kubernetes.io/managed-by: "Helm"
  annotations:
    meta.helm.sh/release-name: "pinot"
    meta.helm.sh/release-namespace: "pinot-benchmark"
```
Apply the fix:
```bash
kubectl apply -f pinot-startup-fix.yaml
```

Then run the Helm installation command again.

Check the running Pinot pods:
```bash
kubectl get pods -n pinot-benchmark
```
Port-forward the Pinot controller to access the UI:
```bash
kubectl port-forward service/pinot-controller 9000:9000 -n pinot-benchmark
```

You can now access the Pinot UI at [http://localhost:9000/](http://localhost:9000/)

## **Adding a Table to Pinot**
Create Schema and Table Configuration

- Create a schema file table-schema-shipment.json. You can copy a sample schema from the [GitHub repository](https://github.com/rajat-sr1704/pinot-POC/)
- Create a table configuration file table-Config-shipment.json. You can also find a sample table configuration in the [GitHub repository](https://github.com/rajat-sr1704/pinot-POC/)

**Upload Files to Pinot Cluster**

Copy the schema and table configuration files to the running Pinot cluster’s pod:
```bash
kubectl cp ./table-schema-shipments.json pinot-controller-0:/opt/pinot/data/table-schema-shipments.json -n pinot-benchmark
kubectl cp ./tableConfig-shipments.json pinot-controller-0:/opt/pinot/data/tableConfig-shipments.json -n pinot-benchmark
```
**Add the Table**

Access the Pinot controller pod:
```bash
kubectl exec -it pinot-controller-0 -n pinot-benchmark -- /bin/bash
```

Run the following command to add the table to Pinot:
```bash
/opt/pinot/bin/pinot-admin.sh AddTable \
  -schemaFile /opt/pinot/data/table-schema-shipments.json \
  -tableConfigFile /opt/pinot/data/tableConfig-shipments.json \
  -exec
```
The table will be added to the Pinot cluster. You can view the table content in the Pinot query editor by selecting the table name.

## **Screenshots**
Below is an example screenshot of the Pinot query editor showing the added table:
![image1](https://github.com/user-attachments/assets/011cb36a-a9e0-47aa-b9ef-de9deaa45ddf)
