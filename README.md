# Anthos Service Mesh Getting Started

You can understand Anthos Service Mesh installation through this tutorial.

This tutorial includes the following contents:
- GKE Cluster Creation
- Sample Microservices Deployment
- Installation to the Application


## Description
### 1. Initial Settings
Configure zone
```
$ gcloud config set compute/zone us-central1-b
```

Enable GKE API
```
$ gcloud services enable container.googleapis.com
```

### 2. GKE Cluster Creation
Create GKE cluster based on [the Requirements for ASM](https://cloud.google.com/service-mesh/docs/scripted-install/asm-onboarding#requirements):
- Machine type: At least 4 vCPUs, such as `e2-standard-4`
- Release channels: `Regular release channel`

```
$ gcloud container clusters create bank-of-anthos \
    --project (gcloud config get-value project) \
    --zone (gcloud config get-value compute/zone) \
    --machine-type=e2-standard-4 \
    --num-nodes=3 \
    --subnetwork=default \
    --enable-ip-alias \
    --release-channel regular
```

### 3. Sample Application Deployment
Deploy the following application:
- [GoogleCloudPlatform/bank-of-anthos](https://github.com/GoogleCloudPlatform/bank-of-anthos)

Clone the repository
```
$ git clone https://github.com/GoogleCloudPlatform/bank-of-anthos.git
```

Deploy the application
```
$ cd bank-of-anthos
$ kubectl apply -f extras/jwt/jwt-secret.yaml
$ kubectl apply -f kubernetes-manifests
```

## Demo

## Features

- feature:1
- feature:2

## Requirement

## Usage

## Installation

## References

## Licence

Released under the [MIT license](https://gist.githubusercontent.com/shinyay/56e54ee4c0e22db8211e05e70a63247e/raw/34c6fdd50d54aa8e23560c296424aeb61599aa71/LICENSE)

## Author

[shinyay](https://github.com/shinyay)
