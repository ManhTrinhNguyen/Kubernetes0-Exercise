# Create Helm Chart for Boutique Microservices

## What is Helm Chart 

- Helm Chart is a bundle of Yaml file

- When I deploy an App with a bunch of Configuration Yaml file (Deployment, Service, ConfigMap, Secret, etc...) I can bundle . So that I can reuse it and make my deployment process more efficent

## 3 Ways to create Helm Chart 

1. Create separate Helm Chart when Configuration is very differnent

2. Create 1 Helm Chart when Configuration is very the same

3. Combination of both

## Basic Structure of HelmChart 

- To create Helm chart : `helm create <helm-chart-name>`

- Helm Chart Folder: 

```
<helm-chart-name>/
├── charts/
├── Chart.yaml
├── templates/
├── values.yaml
├── .helmignore
└── README.md
```

- `chart.yaml` : Describe metadata

- `charts/` folder : Chart Folder hold any dependencies charts that these charts might have

- `.helmignore` :

  - File don't want to include in Helm Chart
 
  - Basically this file tell Helm to ignore all of these Folder and Files when building a Package Archive from this Chart . Helm Chart can build and packaged and distributed just like any other artifacts
 
- `template` folder:

  - This is a core Helm Chart
 
  - This is where actual K8s file created
 
  - `.yaml` file : The attribute same as in the K8s Service Configuration file . Instead of hardcode value I have placeholder defined as `{{ .Values.<Name-of-Value>}}`
 
  - For example :
 
  ```
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: {{ .Values.AppName }}
  spec:
    replicas: {{ .Values.replicaCount }}
  ```

- `value.yaml`

  - This is a place where the acutal value are set will be then substitued in the template file

## Step By Step to create Helm Chart 

#### Step 1 : I create Helm Chart for Frontend, Redis, and other Microservices 

- To create Helm Chart : `helm create <helm-chart-name>`

- Redis and Frontend have to be separte bcs has different configuration 

#### Step 2 : Configure Deployment and Service in those Helm Chart 

- Instead of hardcode the Values I use syntax for dynamically Values : `{{ .Values.<Name-of-Value>}}`

#### Step 3 : Set Dynamic ENV for Deployment 

- For 1 single ENV I can have container ENV var like this:

```
env: 
- name: {{ .Values.containerEnvVar.key }}
  value: {{ .Values.containerEnvVar.value }}
```

- For working with List of something . In this case ENV, Helm has built in fucntion called **Range**

- **Range** is loop through a list and give me element one by one

- For example :

```
env:
{{- range .Values.ContainerEnvVars }}
- name: {{ .name }}
  value: {{ .value | quote}}
{{- end }}
```

- Note: Value ENV variable alway interpreted as strings . So I use a built-in function called **quote** and will use piping syntax `|`
  
- Value output will look like this :

```
ContainerEnvVars: 
- name: PORT
  value: "8080"
- name: DISABLE_PROFILER
  value: "1"
```






