# ECK on Minikube (Docker Driver) Setup Guide

## Prerequisites

- Docker installed
- Minikube installed (For installing minikube, visit <a href="https://minikube.sigs.k8s.io/docs/start/" target="_blank">minikube docs</a>)

## Step 1: Start Minikube with Docker Driver

```bash
minikube start --driver=docker --memory=4g --cpus=2
```

## Step 2: Install ECK Operator

```bash
# Install custom resource definitions with create:
kubectl create -f https://download.elastic.co/downloads/eck/2.16.1/crds.yaml
# Install the operator with its RBAC rules with apply:
kubectl apply -f https://download.elastic.co/downloads/eck/2.16.1/operator.yaml
```

## Step 3: Deploy Elasticsearch Cluster

```bash
kubectl apply -f els-deployment.yaml
```

> [!NOTE]
> If your Kubernetes cluster does not have any Kubernetes nodes with at least 2GiB of free memory, the pod will be stuck in Pending state. Check Manage compute resources for more information about resource requirements and how to configure them.

## Step 4: Check Elasticsearch Status

<p>
Get an overview of the current Elasticsearch clusters in the Kubernetes cluster with get, including health, version and number of nodes:
</p>

```bash
kubectl get elasticsearch
```

> Once the pod has finished coming up, the `HEALTH` will change to `green`.

<p>
You can check Elasticsearch pod status:
</p>

```bash
kubectl get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=quickstart'
```

> While the Elasticsearch pod is in the process of being started it will report `Pending`

## Step 5: Access Elasticsearch

### Option 1: Port Forwarding

<p>
Get the credentials:
</p>

> By default, a user named `elastic` is created with the password stored inside a <a target="_blank" href="https://kubernetes.io/docs/concepts/configuration/secret/">Kubernetes secret</a>. This default user can be disabled if desired, refer to <a target="_blank" href="https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-users-and-roles.html">Users and roles</a> for more information.

```bash
PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
```

### Test the connection:

<p>
From inside the Kubernetes cluster:
</p>

```bash
curl -u "elastic:$PASSWORD" -k "https://quickstart-es-http:9200"
```

<p>
From your local workstation:
</p>

<ul>
    <li>
        <p>
            Use the following command in a separate terminal:
        </p>
    </li>
</ul>

```bash
kubectl port-forward service/quickstart-es-http 9200
```

<ul>
    <li>
        <p>
            Request <code>localhost</code>:
        </p>
    </li>
</ul>

```bash
curl -u "elastic:$PASSWORD" -k "https://localhost:9200"
```

### Option 2: Access via Minikube Service URL

```bash
minikube service quickstart-es-http --url
```

## Troubleshooting

<ul>
    <li>If you encounter resource constraints, adjust the memory and CPU allocated to minikube</li>
    <li>For networking issues, ensure Docker has proper network access</li>
    <li>Check minikube status with <code>minikube status</code></li>
    <li>View logs with <code>kubectl logs -f -n elastic-system statefulset.apps/elastic-operator</code></li>
</ul>
