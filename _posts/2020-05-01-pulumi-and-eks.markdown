---
layout: post
title:  "Pulumi and EKS"
date:   2020-05-01
categories: k8s pulumi
---

This post documents my (nearly) first-time experience using Pulumi to deploy an EKS cluster using Pulumi Crosswalk.

> [Pulumi Crosswalk](https://www.pulumi.com/docs/guides/crosswalk/aws/eks/
) for AWS is a collection of libraries that use automatic well-architected best practices to make common infrastructure-as-code tasks in AWS easier and more secure.

## A rather smallish TIF - tool installation fest

Since I've not setup anything k8s up to now, let alone [EKS](https://docs.aws.amazon.com/eks/latest/userguide/what-is-eks.html
), this was quit a bit to chew on. My initial goal was to setup an example cluster and deploy *some* containers into it.
But before one can do that, there are tools to install.

    choco install -y kubernetes-cli aws-iam-authenticator

 - The [kubernetes-cli](https://kubernetes.io/de/docs/tasks/tools/install-kubectl/) package installs the tool `kubectl`, which is *the* tool to interact with any K8S cluster at the plumbing level.
  - The [IAM authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html) is used to allow `kubectl` to authenticate using an IAM role. Without it, the EKS cluster cannot be accessed.

## Some Pulumi Code

These <s>two</s> three lines create an EKS cluster with *reasonable defaults*

    import * as eks from "@pulumi/eks";
    const cluster = new eks.Cluster("my-cluster");
    export const kubeconfig = cluster.kubeconfig;


I really like that *export* pattern. You can just `pulumi stack output kubeconfig > kubeconfig.yml` to get the kube config to allow loging into the cluster.

    pulumi stack output kubeconfig > kubeconfig.yml
    set KUBECONFIG=./kubeconfig.yml
    kubectl get pods

As a sidenote, after logging into the AWS console and checking the resources manually, AWS EKS told me that the deployed cluster was of version `1.15`, and directed me to the [cluster upgrade docs](https://docs.aws.amazon.com/eks/latest/userguide/update-cluster.html).

I followed the Pulumi page a bit more and deployed 2 NGINX containers into the cluster. Actually, you can do that using Pulumi, too. The SDK contains types for K8S "types" (like *Deployment* and *Service*). It's ok, however I would separate AWS infrastructure from K8S configuration, because the latter is actually independent of anything AWS.

## Infrastructure Visualization

As another sidenote, Pulumi has a `graph` feature that [Export a stack’s dependency graph to a file](https://www.pulumi.com/docs/reference/cli/pulumi_stack_graph/). Briefly, I hoped that I get a nice overview diagram of all resources. The result for the small example cluster was an actual dependency graph (that could be visualized using `dot` from graphviz), however it only showed internal Pulumi and AWS resource names and not the nice visualization I hoped it would be.

For reference, I've used graphviz inside an alpine linux container, after generating the file *graphfile* using `pulumi stack graph graphfile.dot`

    FROM alpine:3.11
    RUN apk add --update --no-cache graphviz ttf-freefont
    COPY graphfile.dot .

Inside the container, I ran `dot -Tpng -ooutput.png graphfile`, and while leaving the container running, extracted `output.png` using [docker cp](https://docs.docker.com/engine/reference/commandline/cp/).

Also a future topic: How to generate visually pleasing diagrams of one's AWS infrastructure. After a 5-minute google search, the best candidate is [cloudmapper](https://github.com/duo-labs/cloudmapper). There are *a lot* of half-backed tools out there, and I want something that *does not* call to any webservice.

## OOPS - deletion failed?!

    Destroying (aws-eks-dev2):
         Type                      Name                          Status                  Info
         pulumi:pulumi:Stack       awsk8ssample-aws-eks-dev2     **failed**              1 error
     -   └─ aws:ec2:SecurityGroup  my-cluster-nodeSecurityGroup  **deleting failed**     1 error

That security group can't be deleted since there was a *Network Interface* using it.
I've deleted it manually and reran `pulumi destroy`. This time it worked. I hope I cannot reproduce this behaviour.

## Other noteworthy stuff I came across

 - There is a specialized tool to interact with AWS EKS clusters called [eksctl](https://github.com/weaveworks/eksctl).
 - There is a [public roadmap](https://github.com/aws/containers-roadmap/projects/1) for all things containers inside AWS.
