---
layout: post
title: Hybrid federation on Kubernetes
date: '2017-03-21 18:40:59'
---

Lots of improvements have been made on Kubernetes supporting Federation, specially outside of Google Cloud Engine. Here, we'll setup a Kubernetes federation having the control plane running on a Google Cloud Engine instance and using Google DNS. After that, we'll create a local on-prem cluster (in a VM, in my case) and make it join the Federation.

If you don't know the basics of Kubernetes Federation, you can take a look at this official [doc](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/federation.md).

The details of the process are on this [repository](https://github.com/henriquetruta/hybrid-k8s-federation/)

Hope to be able to have a full on-premises cluster using CoreDNS in a very short future.

Any questions or feedback, please contact me.