# Wallarm Solutions Engineer Challenge

## Overview
Welcome! My solution to the technical evaluation involved deploying a Wallarm filtering node to my self-hosted K3S cluster. It is configured as an nginx-ingress to both expose an OWASP Juice Shop pod as well as monitor and protect it. 

## Task Breakdown
- [X] Deploy a Wallarm filtering node using the NGINX Ingress Controller
- [X] Configure the vulnerable OWASP Juice Shop image as a backend origin to receive test traffic
- [X] Use the **GoTestWAF** attack simulation tool to generate traffic and test the filtering node