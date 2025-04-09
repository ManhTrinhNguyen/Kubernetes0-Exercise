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
 
## Visualize the whole Picture

<img width="600" alt="Screenshot 2025-02-17 at 14 36 17" src="https://github.com/user-attachments/assets/672c5bd8-d481-4bde-aa8b-65fe41943d8d" />

## Image of each Container

- emailservice: gcr.io/google-samples/microservices-demo/emailservice:v0.8.0
- paymentservice: gcr.io/google-samples/microservices-demo/paymentservice:v0.8.0
- shippingservice: gcr.io/google-samples/microservices-demo/shippingservice:v0.8.0
- adservice: gcr.io/google-samples/microservices-demo/adservice:v0.8.0
- checkoutservice: gcr.io/google-samples/microservices-demo/checkoutservice:v0.8.0
- currencyservice: gcr.io/google-samples/microservices-demo/currencyservice:v0.8.0
- frontend: gcr.io/google-samples/microservices-demo/frontend:v0.8.0
- productcatalogservice: gcr.io/google-samples/microservices-demo/productcatalogservice:v0.8.0
- cartservice: gcr.io/google-samples/microservices-demo/cartservice:v0.8.0
- recommendationservice: gcr.io/google-samples/microservices-demo/recommendationservice:v0.8.0

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

#### Step 4: Set value for those Deployment and Service . Also Redis and Front end 

- I create a value folder : `mkdir values`

- Then I set values of each service . For example : I want to set a `emailservice`

  - Create `emailservice.yaml` : `touch emailservice.yaml` .
 
  ```
  AppName: emailservice
  ReplicaCount: 2
  ImageName: gcr.io/google-samples/microservices-demo/emailservice
  ImageVersion: v0.8.0
  ContainerPort: 8080
  ServicePort: 8080
  ServiceType: ClusterIP
  ContainerEnvVars: 
  - name: PORT
    value: "8080"
  - name: DISABLE_PROFILER
    value: "1"
  RequestMemory: 64Mi
  RequestCPU: 250m
  LimitMemory: 128Mi
  LimitCPU: 500m
  ```

#### Step 5: Create Helmfile to deploy HelmChart 

- To validate Template file : `helm template -f <value-file> <name-of-helm-chart>` . This will give me a nice preview from all the values sources 

- Then I can deploy Helm Chart by using `helm install -f <value-file> <realease-name> <chart-name>` . But this way will take a lot of time especially I have a alot of Services .

- **Best practice** : is to create `helmfile` then use that helm file to deploy helm chart

  - Install Helm file : `brew install helmfile`
 
  - Then I put the Name, Namespace, Chart folder, and values file in the Helm file . For example :
 
  ```
  releases:
  - name: email-service
    namespace: default
    chart: microservices
    values:
      - values/emailservice.yaml
  - name: adservice
    namespace: default
    chart: microservices
    values:
      - values/adservice.yaml
  - name: cartservice
    namespace: default
    chart: microservices
    values:
      - values/cartservice.yaml
  ```

  - Once I have all chart in `helmfile` I want to deploy I can use : `helmfile sync`
 
  - If I want to delete all of those service I can use `helmfile delete`
 
## Make my Frontend Accessible from the Browser (or anywhere outside Netword).

#### Step 6 : Deploy Ingress Controller by using Helm 

- Ingress Controller evaluate all the Rules in the Ingress component and implement the routing rules at Runtime

- To add NGINX repo : `helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx`

- Installing Helm Chart : `helm install ingress-controller ingress-nginx/ingress-nginx --set controller.publishService.enabled=true`
  
    - `--set controller.publishService.enabled=true`: This attribute make sure it automatically enable Public IP Address
 
- Ingress Controller use Cloud Native in the backgroud. AWS's own NodeBalancer dynamically created and provisioned as I create Ingress Controller

- This NodeBalancer will be come a Entry Point of my Cluster . It give me External IP, Ports .

- NodeBalncer will forward the request coming in into Cluster to the Ingress Controller and to the Service base on Ingress rule I create

#### Step 7: Create and Deploy a Ingress 

- When you deploy an Ingress Controller in a cloud provider environment, the Ingress Controller’s Service of type LoadBalancer automatically provisions a cloud-native load balancer.

- The cloud provider assigns a public DNS name or IP address to this load balancer.

- This becomes your public endpoint — users send requests here.

- The Ingress Controller receives the traffic from the load balancer and routes it internally according to the Ingress rules.

- Visual flow
  
```
[User Request] 
     ↓
[Cloud Load Balancer (Provisioned by Ingress Controller's LoadBalancer service)]
     ↓
[Ingress Controller]
     ↓
[Ingress Rules → Route traffic]
     ↓
[Kubernetes Services]
     ↓
[Pods / Apps]
```

- Then I will get that DNS file and put in my Ingress host :

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend
spec:
  ingressClassName: nginx
  rules:
  - host: a6667475a1d5c429bbfdf23cc8100bb3-164349241.us-west-1.elb.amazonaws.com # This could be valid domain name
    http:
      paths:
        - path: / # This is the path that will be used to access the service
          pathType: Prefix
          backend: # This is a target service that will used to forward the request
            service:
              name: frontend
              port:
                number: 8080
```






