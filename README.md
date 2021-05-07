# Anthos Service Mesh Getting Started

You can understand Anthos Service Mesh installation through this tutorial.

This tutorial includes the following contents:
- GKE Cluster Creation
- Sample Microservices Deployment
- Anthos Service Mesh Installation
- Istio ingress-gateway


## Description
### GKE Cluster Creation
#### 1. Initial Settings
Configure zone
```
$ gcloud config set compute/zone us-central1-b
```

Enable GKE API
```
$ gcloud services enable container.googleapis.com
```

Enable Anthos Service Mesh API
```
$ gcloud services enable meshca.googleapis.com
$ gcloud services enable gkehub.googleapis.com
$ gcloud services enable gkeconnect.googleapis.com
$ gcloud services enable meshconfig.googleapis.com
$ gcloud services enable meshtelemetry.googleapis.com
```

KPT Installation
```
$ gcloud components install kpt
```
or
```
$ brew tap GoogleContainerTools/kpt https://github.com/GoogleContainerTools/kpt.git
$ brew install kpt
```

### 2. Create GKE Cluster
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

### Sample Application Deployment
Deploy the following application:
- [GoogleCloudPlatform/bank-of-anthos](https://github.com/GoogleCloudPlatform/bank-of-anthos)

#### 1. Deploy Sample Application (Bank of Anthos)
Clone the repository
```
$ git clone https://github.com/GoogleCloudPlatform/bank-of-anthos.git
```

Deploy the application
```
# The demo JWT public key
$ kubectl apply -f bank-of-anthos/extras/jwt/jwt-secret.yaml

# The sample app to the cluster
$ kubectl apply -f bank-of-anthos/kubernetes-manifests
```

#### 2. Verification the app
Confirm endpoint as `EXTERNAL-IP`
```
$ kubectl get service frontend
```
```
$ open http://${EXTERNAL-IP}
```

This the application architecgure:
![image](https://user-images.githubusercontent.com/3072734/117401852-46787800-af40-11eb-87e6-bb42030e0b5e.png)



### Anthos Service Mesh Installation
#### 1. Retrieve the installation script
```
$ curl https://storage.googleapis.com/csm-artifacts/asm/install_asm_1.9 > install_asm
$ curl https://storage.googleapis.com/csm-artifacts/asm/install_asm_1.9.sha256 > install_asm.sha256
$ shasum -a 256 -c --ignore-missing install_asm.sha256
install_asm: OK
$ chmod +x install_asm
```

#### 2. Execute the installation script
Validate environment
```
$ mkdir asm_output
$ ./install_asm \
    --project_id (gcloud config get-value project) \
    --cluster_name bank-of-anthos \
    --cluster_location (gcloud config get-value compute/zone) \
    --mode install \
    --output_dir asm_output \
    --only_validate
```

Install ASM
```
$ rm -fr asm_output
$ mkdir asm_output
$ ./install_asm \
    --project_id (gcloud config get-value project) \
    --cluster_name bank-of-anthos \
    --cluster_location (gcloud config get-value compute/zone) \
    --mode install \
    --output_dir asm_output \
    --enable-all
```

#### 4. Apply ASM to Sample Application
Get ASM Revision
```
$ kubectl -n istio-system get pods -l app=istiod -ojson | jq -r '.items[0].metadata.labels["istio.io/rev"]'

asm-193-2
```

Enable Sidecar Auto Injection
```
$ kubectl label namespace default istio.io/rev=${ASM_REVISION} istio-injection- --overwrite
$ kubectl get ns default --show-labels
```

Re-Deploy Sample Application
(It makes sidecar proxy installed automatically)
```
$ kubectl rollout restart deployment -n default
```

Verify ASM Installation
```
$ open https://console.cloud.google.com/anthos/services
```

The following is the architecture which is applied ASM:
![image](https://user-images.githubusercontent.com/3072734/117401928-6e67db80-af40-11eb-9ac8-76c8d5a3340f.png)


### Istio Ingress-gateway
Istio Ingress-gateway is already created when ASM is installed.


#### 1. Apply Istio Ingress-gateway
Verify Istio Ingress-gateway
```
$ kubectl get service,deploy -n istio-system istio-ingressgateway

NAME                           TYPE           CLUSTER-IP   EXTERNAL-IP     PORT(S)                                                                      AGE
service/istio-ingressgateway   LoadBalancer   10.40.5.31   34.71.237.224   15021:30874/TCP,80:31897/TCP,443:32254/TCP,15012:31189/TCP,15443:31482/TCP   53m

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/istio-ingressgateway   2/2     2            2           53m
```

Apply the following configuration for `Gateway` and `VirtualService`
```yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: frontend-gateway
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: frontend-ingress
spec:
  hosts:
  - "*"
  gateways:
  - frontend-gateway
  http:
  - route:
    - destination:
        host: frontend
        port:
          number: 80
```
```
$ kubectl apply -f bank-of-anthos/istio-manifests/frontend-ingress.yaml
```

#### 2. Verify Istio Ingress-gateway
Confirm endpoint as `EXTERNAL-IP`
```
$ kubectl get services istio-ingressgateway -n istio-system
```
```
$ open http://${EXTERNAL-IP}
```
The following is the architecture which uses Istio Ingress-gateway:
![image](https://user-images.githubusercontent.com/3072734/117402115-c272c000-af40-11eb-96b6-0fc40f4eae52.png)

### Clean up
Delete Sample Application
```
$ kubectl delete -f bank-of-anthos/istio-manifests/
$ kubectl delete -f bank-of-anthos/kubernetes-manifests/
$ kubectl delete -f bank-of-anthos/extras/jwt/jwt-secret.yaml
```

Delete ASM
```
$ asm_output/istio-*/bin/istioctl manifest generate --set profile=asm-gcp | kubectl delete --ignore-not-found=true -f -
$ kubectl delete ns asm-system istio-system
```

Disable Sidecar Auto Injection
```
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
