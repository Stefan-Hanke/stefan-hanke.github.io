---
layout: post
title:  "Random Rumblings - A plan, IS4 admin template, AWS-Vault"
date:   2020-05-07
categories: identityserver pulumi k8s aws-vault
---

Random Rumblings - stuff that I came across while surfing along.

## Infrastructure planning

The plan in bullet points:

 - Pulumi as IaaS Language
 - K8S as container orchestrator, described by Pulumi
 - IS4 deployment with static configuration
 - A sample app that is using the IS4 for authn/authz (two users - normal/admin)
 - Usage of the AWS WAF to not directly expose K8S nodes to the public internet.

## IdentityServer4 admin template

I've installed the `is4admin` template and ran it.
The Admin UI is basically an UI management tool for all things identity.
With this, you can configure the IdentityServer via an UI instead of hardcoding the configuration inside a `Config.cs`.
This is how *Rock Solid Knowledge* (the company behind IdentityServer4) is generating revenue. The AdminUI is not free, it's intended audience are companies (prices start at £500).

## AWS-Vault - create temporary AWS credentials

I'm still searching an offline, visually pleasing, text-based visualization method... and as always, while browsing the readme of the [cloudmapper](https://github.com/duo-labs/cloudmapper) project, I've come across another interesting project. This time, it's the [aws-vault](https://github.com/99designs/aws-vault).

This project securely stores AWS credentials on the computer, and allows to generate [STS temporary credentials](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp.html) for running commands. The temporary credentials then are passed to a sub-process either via environment variables or a local EC2 metadata server on `169.254.169.254:80`. The program is re-using an OS-specific secure credential store to handle the actual storage (e.g. the [Credential Manager](https://support.microsoft.com/en-au/help/4026814/windows-accessing-credential-manager) on Windows).

Unfortunately, the aws-vault choco package contains unfortunately malware.

    C:\Windows\system32>choco install -y aws-vault
    Chocolatey v0.10.15
    Installing the following packages:
    aws-vault
    By installing you accept licenses for the packages.
    Progress: Downloading aws-vault 5.4.2... 100%
    aws-vault not installed. An error occurred during installation:
     Operation did not complete successfully because the file contains a virus or potentially unwanted software.
    
    aws-vault package files install completed. Performing other installation steps.
    The install of aws-vault was NOT successful.
    aws-vault not installed. An error occurred during installation:
     Operation did not complete successfully because the file contains a virus or potentially unwanted software.
    
    
    Chocolatey installed 0/1 packages. 1 packages failed.
     See the log for details (C:\ProgramData\chocolatey\logs\chocolatey.log).
    
    Failures
     - aws-vault (exited 1) - aws-vault not installed. An error occurred during installation:
     Operation did not complete successfully because the file contains a virus or potentially unwanted software.

    C:\Windows\system32>

I was notified by Windows Defender and now am running a full scan on this machine.

### Intermezzo `169.254.169.254:80` WAT?!

I did know there are [private addresses](https://de.wikipedia.org/wiki/Private_IP-Adresse) in both current IP versions. However I did not recognize the `169` prefixed ones. These are so-called *link-local* ones, which are only valid in networks that are physically directly attached (i.e. all computers connected to a switch or to one WLAN).

It looks like that link local addresses are mainly used to implement [zeroconf](https://en.wikipedia.org/wiki/Zero-configuration_networking), IP networking that works without any other server (e.g. DHCP, or DNS).

What I *do love* about the aws-vault project is that [reference links](https://github.com/99designs/aws-vault#references-and-inspiration) section at the end of it's readme. It's like the reference section of a scientific paper, providing pointers to other interesting projects or documentation.