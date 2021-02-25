<!--
#
# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
-->

---

id: deployment-on-kubeSphere
title: Install Ingress APISIX on KubeSphere

---

This document explains how to install Ingress APISIX on [KubeSphere](https://kubesphere.io/).

KubeSphere is a distributed operating system managing cloud native applications with Kubernetes as its kernel, and provides plug-and-play architecture for the seamless integration of third-party applications to boost its ecosystem.

## Prerequisites

- Install [KubeSphere](https://kubesphere.io/docs/quick-start/), you can choose [All-in-one Installation on Linux](https://kubesphere.io/docs/quick-start/all-in-one-on-linux/) or [Minimal KubeSphere on Kubernetes](https://kubesphere.io/docs/quick-start/minimal-kubesphere-on-k8s/).
- Install [Helm](https://helm.sh/).
- Clone [Apache APISIX Charts](https://github.com/apache/apisix-helm-chart).
- Make sure your target namespace exists, kubectl operations of this document will be executed in namespace `ingress-apisix`.

## Install APISIX

[Apache APISIX](http://apisix.apache.org/) as the proxy plane of apisix-ingress-controller, should be deployed in advance.

```shell
cd /path/to/apisix-helm-chart
helm repo add bitnami https://charts.bitnami.com/bitnami
helm dependency update ./charts/apisix
helm install apisix ./charts/apisix \
  --set gateway.type=NodePort \
  --set allow.ipList="{0.0.0.0/0}" \
  --namespace ingress-apisix \
kubectl get service --namespace ingress-apisix
```

Two Service resources were created, one is `apisix-gateway`, which processes the real traffic; another is `apisix-admin`, which acts as the control plane to process all the configuration changes.

The gateway service type is set to `NodePort`, so that clients can access Apache APISIX through the Node IPs and the assigned port.
If you want to expose a `LoadBalancer` service, try to use [Porter](https://github.com/kubesphere/porter).

Another thing that should be concerned that the `allow.ipList` field should be customized according to the Pod CIDR settings, so that the apisix-ingress-controller instances can access the APISIX instances (resources pushing).

## Install apisix-ingress-controller

You can also install apisix-ingress-controller by Helm Charts. It's recommended to install it in the same namespace with Apache APISIX.

```shell
cd /path/to/apisix-helm-chart
# install apisix-ingress-controller
helm install apisix-ingress-controller ./charts/apisix-ingress-controller \
  --set image.tag=dev \
  --set config.apisix.baseURL=http://apisix-admin:9180/apisix/admin \
  --set config.apisix.adminKey=edd1c9f034335f136f87ad84b625c8f1 \
  --namespace ingress-apisix
```

The admin key used above is the default one. If you change the admin key configuration when you deployed APISIX, please remember to change it here.

Change the `image.tag` to the apisix-ingress-controller version that you desire. Wait for the correspdoning pods are running.

Now try to create some [resources](../CRD-specification.md) to verify the running status. As a minimalist example, see [proxy-the-httpbin-service](../samples/proxy-the-httpbin-service.md) to learn how to apply resources to drive the apisix-ingress-controller.