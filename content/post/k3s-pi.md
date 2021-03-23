---
title: Building a k3s cluster with Raspberry Pi 4s
author: Avery Wagar
date: 2021-03-23
draft: true
toc: false
tags:
  - K8s
  - Raspberry Pi
  - Cloud
  - Containers
---

I've been interested in learning Kubernetes for a while now. However, my biggest issue has been getting a reliable and realistic cluster built. I've had too many issues with [k3d.io](https://k3d.io), [Minikube](https://minikube.sigs.k8s.io/docs/), and [Microk8s](https://microk8s.io/). Plus, running a virtualized k8s cluster doesn't give me the benefits of actually having a cluster up 24/7.

I decided to build my own cluster using [k3s](https://k3s.io) and a few [Raspberry Pi 4s](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/).
