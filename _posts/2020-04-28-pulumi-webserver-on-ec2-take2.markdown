---
layout: post
title:  "Pulumi - Webserver on AWS EC2 - Take 2"
date:   2020-04-28
categories: iac aws csharp
---

In the [last post]({% post_url 2020-04-28-pulumi-webserver-on-ec2-take1 %}), I've used TypeScript alongside [the tutorial for a python SimpleHTTPServer](https://www.pulumi.com/docs/tutorials/aws/ec2-webserver/). This time, I'm using CSharp.

## problems here also

.. and rather unexpected, the example fails to compile also, with them redesigning the API without adapting the examples.

- Types moved to different namespaces, e.g. `Pulumi.Aws.Inputs.GetAmiFilterArgs` or `Pulumi.Aws.Ec2.Inputs.SecurityGroupIngressArgs`
- Async code did not compile, `var ami = Pulumi.Aws.GetAmi.InvokeAsync(new Pulumi.Aws.GetAmiArgs`

The fixes however were straightforward this time. However, I'm wondering where this SDK is defined; [searching for e.g. GetAmiFilterArgs](https://github.com/pulumi/pulumi/search?q=GetAmiFilterArgs&unscoped_q=GetAmiFilterArgs) does not yield a result. I did not find another repository also.

## yay - pulumi up working

    D:\git\github\stefan.hanke\webserver>pulumi up
    Enter your passphrase to unlock config/secrets
        (set PULUMI_CONFIG_PASSPHRASE to remember):
    Previewing update (webserver-csharp):
         Type                      Name                        Plan        Info
         pulumi:pulumi:Stack       webserver-webserver-csharp
     ~   ├─ aws:ec2:SecurityGroup  webserver-secgrp            update      [diff: ~ingr     
     +-  └─ aws:ec2:Instance       webserver-www               replace     [diff: +user     

    Outputs:
      ~ publicHostName: "ec2-35-158-XX-XX.eu-central-1.compute.amazonaws.com" => output<string>
      ~ publicIp      : "35.158.XX.XX" => output<string>

    Resources:
        ~ 1 to update
        +-1 to replace
        2 changes. 1 unchanged

    Do you want to perform this update?  [Use arrows to move, enter to select, type to filteDo you want to perform this update?  [Use arrows to move, enter to select, type to filteDo you want to perform this update? yes
    Updating (webserver-csharp):
         Type                      Name                        Status       Info
         pulumi:pulumi:Stack       webserver-webserver-csharp
     ~   ├─ aws:ec2:SecurityGroup  webserver-secgrp            updated      [diff: ~ing     
     +-  └─ aws:ec2:Instance       webserver-www               replaced     [diff: +use     

    Outputs:
      ~ publicHostName: "XXX.eu-central-1.compute.amazonaws.com" => "XXX.eu-central-1.compute.amazonaws.com"
      ~ publicIp      : "35.158.XX.XX" => "18.156.XX.XX"

    Resources:
        ~ 1 updated
        +-1 replaced
        2 changes. 1 unchanged

    Duration: 1m47s

    Permalink: file:///C:/Users/Stefan/.pulumi/stacks/webserver-csharp.json

    D:\git\github\stefan.hanke\webserver>

Interesting tidbit: note that all stacks by default are put into `%USERPROFILE%/.pulumi/stacks`. So each stack on the machine must have a unique name across all projects. Maybe there is an option to group them e.g. per project.

## AWS EC2 UserData

The user data consists of the following 4 lines:

    var userData = // <-- ADD THIS DEFINITION
    @"#!/bin/bash
    echo ""Hello, World!"" > index.html
    nohup python -m SimpleHTTPServer 80 &";

[AWS EC2 UserData](https://docs.aws.amazon.com/de_de/AWSEC2/latest/UserGuide/user-data.html) allows to run a script on EC2 instance startup. This is used to run a small python-based HTTP server with a small static site.