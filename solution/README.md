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

1. Generating a filtering node token

Login to the Wallarm cloud console and go to `settings` -> `API tokens`. Hit `+ Create token`, name it, and ensure that the `source role` is of type `deploy`. 

