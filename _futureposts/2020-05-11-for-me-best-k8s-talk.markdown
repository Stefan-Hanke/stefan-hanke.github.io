---
layout: post
title:  "How does AWS implement K8S?"
date:   2020-05-10
categories: k8s yt aws
---

On the integration of k8s into aws.

## What do I get using AWS EKS?

AWS EKS creates an k8s cluster on top of AWS resources. The k8s structure should fit in a good way, since AFAIK everything in k8s can be plugged.

Clusters created by AWS EKS

 - are using IAM for authentication. Now how could this possibly work? AWS guys have written a plugin that forwards auth requests directed to the k8s cluster to IAM.

