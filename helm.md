# Helm Lab: Create and Deploy a Helm Chart

This guide will walk you through the process of creating a Helm chart, packaging it, and deploying it on Kubernetes.

## Prerequisites

Ensure you have Helm installed on your machine by running the following script:

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

## Step 1: Create a Helm Chart

### 1.1 Create a New Directory for Your Helm Chart

First, create a directory to hold your Helm chart files.

```bash
mkdir my-web-app-chart
cd my-web-app-chart
```

### 1.2 Define Your Chart

Create a `Chart.yaml` file with the following contents to define the metadata of your chart:

```yaml
apiVersion: v2
name: my-web-app
description: A Helm chart for a simple web application
version: 1.0.0
```

### 1.3 Define Default Values

Create a `values.yaml` file to set default values for your chart:

```yaml
replicaCount: 3
image:
  repository: my-web-app
  tag: latest
  pullPolicy: IfNotPresent
service:
  name: my-web-app
  type: ClusterIP
  port: 80
ingress:
  enabled: false
```

This `values.yaml` defines important settings for the application, including replica count, image details, and service configuration.

### 1.4 Create Kubernetes Resource Templates

Create a `templates/` directory where you'll define Kubernetes resources.

```bash
mkdir templates
```

#### 1.4.1 Create a Deployment Template

Inside the `templates/` directory, create a `deployment.yaml` file with the following contents:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ .Chart.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ .Chart.Name }}
  template:
    metadata:
      labels:
        app: {{ .Chart.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
```

This defines a Kubernetes Deployment resource with dynamic values from the Helm chart, such as the number of replicas and the image repository.

#### 1.4.2 Create a Service Template

Also, inside the `templates/` directory, create a `service.yaml` file with the following contents:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ .Values.service.name }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
  selector:
    app: {{ .Chart.Name }}
```

This defines a Kubernetes Service resource that exposes your web application using the defined port and service type.

### 1.5 Package the Chart

Once you have created the required files, package your Helm chart into a `.tgz` file:

```bash
helm package .
```

This command will create a `.tgz` file (e.g., `my-web-app-1.0.0.tgz`) that can be used for deployment.

## Step 2: Install the Helm Chart

Now, you can install the Helm chart using the `.tgz` file you packaged:

```bash
helm install <release_name> <path-to-chart.tgz>
```

For example:

```bash
helm install my-web-app ./my-web-app-1.0.0.tgz
```

This will deploy the chart onto your Kubernetes cluster.

## Step 3: Install an External Chart

You can also install charts from external repositories. For example, install the Bitnami WordPress chart using Helm:

```bash
helm repo add bitnami https://charts.bitnami.com/bitnami
helm install my-wordpress bitnami/wordpress --version 23.1.12
```

### 3.1 Update Your Helm Repositories

To make sure you're using the latest chart versions, always update your Helm repositories:

```bash
helm repo update
```

### 3.2 Pull and Unpack Charts from the Repo

You can download a chart from a repository for inspection by running:

```bash
helm pull <repo_name>/<chart_name> --untar
```

For example, to pull the WordPress chart from the Bitnami repository:

```bash
helm pull bitnami/wordpress --untar
```

This will download and unpack the chart so you can explore its contents locally.

---
