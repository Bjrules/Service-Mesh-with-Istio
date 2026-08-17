# Service-Mesh-with-Istio
This Explores the benefits and advantages of service mesh using istio. as a better service to service secure/encrypted communication, Better  Load Balancing traffic routing , Observability and failovers.

- [x] Useful links to this project by istio [Istio official Bookinfo Guide](https://istio.io/latest/docs/examples/bookinfo/) 
- [x] Direct github repo to the project by istio [Click here](https://github.com/istio/istio/tree/master/samples/bookinfo)

# 1) Set up a Kubenetes Cluster

Awscli, Terraform(IaC) . Kubectl , Ekctl,


# 2) Download istio and also add bin directory to the system path

```yaml
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.30.3
export PATH=$PWD/bin:$PATH

```

![alt text](IMG-Screenshots/Screenshot_20260816_164437.png)

![alt text](IMG-Screenshots/Screenshot_20260816_161608.png)

# 3) Install Istio using istioctl 
`istioctl install --set profile=demo -y`
this installs the istio control plane `istiod` `istio-ingressgateway` and `istio-egressgateway` etc
![alt text](IMG-Screenshots/Screenshot_20260816_161744.png)

# 4) create and Label Namespace for Auto Sidecar Injection
This means that every pod inside the namespace will be attached with istio's Envoy sidecar container 
```
kubectl create ns webapps
kubectl label namespace webapps istio-injection=enabled

```
![alt text](IMG-Screenshots/Screenshot_20260816_161830.png)

# 5) Deploy the sample App (Bookinfo)
`kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml -n webapps`
This deploys:
    • productpage, details, reviews, ratings microservices

 ![alt text](IMG-Screenshots/Screenshot_20260816_162318.png) 

# 6) Expose App via Istio Gateway
Istio gateway will sit in front of the application as the Loadbalancer just like Nginx ingress or Gateway API.       But in this case since I'm are using service mesh to interact with our microservices it will be forwarding requests to the Envoy        sidecar containers attached to the various pods via mtls which         talks to thier respective pods securely.

**Get EXTERNAL IP:**
`kubectl get svc istio-ingressgateway -n istio-system`

![alt text](IMG-Screenshots/Screenshot_20260816_162550.png)

![alt text](IMG-Screenshots/Screenshot_20260816_163206.png)
>Application is exposed
![alt text](IMG-Screenshots/Screenshot_20260816_164437.png)

![alt text](IMG-Screenshots/Screenshot_20260816_164453.png)

# 7) Enable mTLS for the App
- [x] mtls is that feature of of istio which performs the internal end to end secure encryption of communication between pods
![alt text](IMG-Screenshots/Screenshot_20260816_172331.png)

![alt text](IMG-Screenshots/Screenshot_20260816_172305.png)

`sudo vi strict-mtls.yaml` ||| `kubectl apply -f strict-mtls.yaml`

# 8) Enable Observability
Deploy addons Directory  for prometheus and Grafana   
`kubectl apply -f samples/addons`

![alt text](IMG-Screenshots/Screenshot_20260816_194114.png)

> sell all deployed

![alt text](IMG-Screenshots/Screenshot_20260816_195041.png)

>Modify prometheus service from ClusterIP to LoadBalacer `kubectl edit svc prometheus istio-system`. Remember prometheus runs on port 9090
![alt text](IMG-Screenshots/Screenshot_20260816_203821.png)

>Modify grafana service from ClusterIP to LoadBalacer `kubectl edit svc grafana istio-system`

![alt text](IMG-Screenshots/Screenshot_20260816_195417.png)
>Grafana Screenshots of configuration to receive data from prometheus as it's Datasource for monitoring. Remember Grafana runs on port 3000

![alt text](IMG-Screenshots/Screenshot_20260816_203857.png)
![alt text](IMG-Screenshots/Screenshot_20260816_203640.png)
![alt text](IMG-Screenshots/Screenshot_20260816_203623.png)

![alt text](IMG-Screenshots/Screenshot_20260816_204444.png)

![alt text](IMG-Screenshots/Screenshot_20260816_205410.png)

> `kubectl get all -n istio-system`
![alt text](IMG-Screenshots/Screenshot_20260816_205443.png)

**Note Two very important concept in Istio are `DestinationRule` and `VirtualService`**

DestinationRule: The resource where pods are given specific Addresses called `labels` which serves as a unique ID for every pods.        

VirtualService: The resource where the percentage of traffice pods are specified and determine though `weight` parameters.

## This shows that istio is also a tool for canary Deployment and DevOps just like ArgoRollouts.


![alt text](IMG-Screenshots/Screenshot_20260816_191430.png)

kubectl apply -f drvs.yaml

```yaml

#Destination Rule:

apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: reviews
spec:
  host: reviews
  subsets:
  - name: v1
    labels:
      version: v1
  - name: v2
    labels:
      version: v2
  - name: v3
    labels:
      version: v3


# Virtual Service:

---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
  namespace: webapps
spec:
  hosts:
    - reviews
  http:
    - route:
        - destination:
            host: reviews
            subset: v1
          weight: 40
        - destination:
            host: reviews
            subset: v3
          weight: 30
        - destination:
            host: reviews
            subset: v2
          weight: 30

```

