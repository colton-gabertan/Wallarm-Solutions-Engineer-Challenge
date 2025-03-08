# Wallarm Solutions Engineer Challenge

## Overview
Welcome! My solution to the technical evaluation involved deploying a Wallarm filtering node to my self-hosted K3S cluster. It is configured as an NGINX ingress to both expose an OWASP Juice Shop pod as well as monitor and protect it. 

## Task Breakdown
- [X] Deploy a Wallarm filtering node using the NGINX Ingress Controller
- [X] Configure the vulnerable OWASP Juice Shop image as a backend origin to receive test traffic
- [X] Use the **GoTestWAF** attack simulation tool to generate traffic and test the filtering node

## Solution
### Wallarm Deployment - Kubernetes NGINX Ingress Controller

I chose the NGINX Ingress Controller as it was a great solution for my cluster to provide both an ingress to expose my web apps as well as take advantage of Wallarm's enhanced security features.
> **Note**: the [official documentation](https://docs.wallarm.com/admin-en/installation-kubernetes-en/) covers this deployment in detail

#### 1. Generating a filtering node token

Login to the Wallarm cloud console and go to `settings` -> `API tokens`. Hit `+ Create token`, name it, and ensure that the `source role` is of type `Deploy`. 

![image](https://github.com/user-attachments/assets/b4224c23-99e1-45b2-a01b-5f90f7c6f1dd)

#### 2. Configure Helm and Kubernetes

Add the official [Wallarm chart repo](https://charts.wallarm.com/) via:
```
helm repo add wallarm https://charts.wallarm.com/
helm repo update wallarm
```

I've also created a custom [values.yml](/helm/values.yml) to pass to helm for our deployment. Let's break it down here:
```
controller:
  wallarm:
    enabled: true
    apiHost: us1.api.wallarm.com
    apiPort: 443
    apiSSL: true
    token: ""
    nodeGroup: defaultIngressGroup
    existingSecret:
      enabled: true
      secretKey: token
      secretName: wallarm-api-token
```
By default, Wallarm will install with `enabled: false`, as well as the `existingSecret` set to `false`, so we want to make sure to enable it either here or update the deployment post-installation. This ensures that our node can check-in to us1.api.wallarm.com as well as pull our API token from the Kubernetes secrets. 

At this point, Helm now knows which Wallarm cloud domain to point to for its API calls as well as retreive the API token from the secrets; however, we need to actually import our API key into kubernetes. We do this by creating the `wallarm` namespace and generating a secret to hold the token. This secret will be accessible to our node within the same namespace. 

Via `kubectl` we can do as follows:
```
kubectl create namespace <KUBERNETES_NAMESPACE>
kubectl -n <KUBERNETES_NAMESPACE> create secret generic wallarm-api-token --from-literal=token=<WALLARM_NODE_TOKEN>
```
> [Official documentation](https://docs.wallarm.com/admin-en/configure-kubernetes-en/#controllerwallarmexistingsecret) for more details regarding helm values and default configurations

Now that the secret is imported, we're ready to deploy via helm!:
```
helm install --version 5.3.8 <RELEASE_NAME> wallarm/wallarm-ingress -n <KUBERNETES_NAMESPACE> -f <PATH_TO_VALUES>
```

#### 3. Checking the installation

Firstly, using `kubectl` we can see if the pods deployed successfully:
```
$ kubectl get pods -n <KUBERNETES_NAMESPACE>
NAME                                                   READY   UP-TO-DATE   AVAILABLE   AGE
wallarm-wallarm-ingress-controller                     1/1     1            1           48m
wallarm-wallarm-ingress-controller-wallarm-tarantool   1/1     1            1           48m
```

Then, we can also check on the service object within our namespace to grab the IP of the proxy that will be routing traffic to our web apps.
```
$ kubectl get service -n <KUBERNETES_NAMESPACE>
NAME                                                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
wallarm-wallarm-ingress-controller                     LoadBalancer   10.43.95.204    10.0.0.80     80:30787/TCP,443:31596/TCP   54m
wallarm-wallarm-ingress-controller-admission           ClusterIP      10.43.196.175   <none>        443/TCP                      54m
wallarm-wallarm-ingress-controller-wallarm-tarantool   ClusterIP      10.43.112.60    <none>        3313/TCP                     54m
```
> **Note**: The EXTERNAL-IP was assigned by my `kube-vip` load balancer and will also be used to map domain names to this address for any web app(s) that will be deployed behind this proxy. My personal DNS setup within the homelab involves just creating the records and having my router handle internal DNS requests. 

This shows that our pods are READY and the service is up. Next, we can also go back to our cloud console and see is the node successfully checked in. We can reach it by going from the `Main Menu` -> `Configuration` -> `Nodes`. With this particular deployment, we have a `Self-hosted` node.

![image](https://github.com/user-attachments/assets/3253efff-a3d1-4ecb-8194-3dd57219d5cd)

With the installation steps complete, our cluster is ready to create ingresses with wallarm's security enhancements in order to expose web apps.

### Configure the OWASP Juice Shop as a backend origin to test the filtering node

I chose the OWASP Juice Shop as my backend to receive http(s) traffic as it is a popular pentesting image designed with a huge variety of web-based vulnerabilities. I was also curious as to how Wallarm would perform against this particular scenario.

#### 1. Creating the deployment





