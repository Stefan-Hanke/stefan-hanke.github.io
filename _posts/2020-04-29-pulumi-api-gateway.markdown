---
layout: post
title:  "Pulumi - AWS API Gateway"
date:   2020-04-29
categories: iac aws csharp
---

Another Pulumi example - this time, an AWS API Gateway serving content via an AWS lambda callback ([source on GH](https://github.com/Stefan-Hanke/pulumi-apigateway)).

    D:\git\github\stefan.hanke\pulumi-apigateway>pulumi stack output
    Current stack outputs (1):
        OUTPUT  VALUE
        url     https://9abh5l1tx3.execute-api.eu-central-1.amazonaws.com/stage/

## AWS API Gateway

I've wondered about that *stage* path. It turns out this is an [API Gateway term](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-basic-concept.html).

An [*AWS API Gateway*](https://docs.aws.amazon.com/apigateway/latest/developerguide/welcome.html) defines an HTTP API, and one can specify what should happen when a client hits an endpoint. It puts the **API** in the foreground, and models anything that is necessary for it work after it.

After 5 minutes spent on the site, some features include
- direct support for OIDC or OAUTH2 in HTTP APIs
- *staging* APIs to enable changes without breaking existing clients
- objectifies the API to stand for itself, with its own representation

## What Would Pulumi Do?

Let's play a little bit and observe what Pulumi creates when I change the infrastructure code.

### Dropping the lambda route

    // Create a public HTTP endpoint (using AWS APIGateway)
    const endpoint = new awsx.apigateway.API("hello", {
        routes: [
            // Serve static files from the `www` folder (using AWS S3)
            {
                path: "/",
                localPath: "www",
            },

            // // Serve a simple REST API on `GET /name` (using AWS Lambda)
            // {
            //     path: "/source",
            //     method: "GET",
            //     eventHandler: (req, ctx, cb) => {
            //         cb(undefined, {
            //             statusCode: 200,
            //             body: Buffer.from(JSON.stringify({ name: "AWS" }), "utf8").toString("base64"),
            //             isBase64Encoded: true,
            //             headers: { "content-type": "application/json" },
            //         });
            //     },
            // },
        ],
    });

results in:

    D:\git\github\stefan.hanke\pulumi-apigateway>pulumi preview
    Enter your passphrase to unlock config/secrets
        (set PULUMI_CONFIG_PASSPHRASE to remember):
    Previewing update (apigateway-dev):
         Type                                Name                       Plan
     +   pulumi:pulumi:Stack                 myproject-apigateway-dev   create
     +   └─ aws:apigateway:x:API             hello                      create
     +      ├─ aws:iam:Role                  hello4c238266              create
     +      ├─ aws:s3:Bucket                 hello                      create
     +      ├─ aws:iam:RolePolicyAttachment  hello4c238266              create
     +      ├─ aws:apigateway:RestApi        hello                      create
     +      ├─ aws:s3:BucketObject           hello4c238266/index.html   create
     +      ├─ aws:s3:BucketObject           hello4c238266/favicon.png  create
     +      ├─ aws:apigateway:Deployment     hello                      create
     +      └─ aws:apigateway:Stage          hello                      create

    Resources:
        + 10 to create

    Permalink: file:///C:/Users/Stefan/.pulumi/stacks/apigateway-dev.json

    D:\git\github\stefan.hanke\pulumi-apigateway>

### Dropping the static route

    // Create a public HTTP endpoint (using AWS APIGateway)
    const endpoint = new awsx.apigateway.API("hello", {
        routes: [
            // // Serve static files from the `www` folder (using AWS S3)
            // {
            //     path: "/",
            //     localPath: "www",
            // },

            // Serve a simple REST API on `GET /name` (using AWS Lambda)
            {
                path: "/source",
                method: "GET",
                eventHandler: (req, ctx, cb) => {
                    cb(undefined, {
                        statusCode: 200,
                        body: Buffer.from(JSON.stringify({ name: "AWS" }), "utf8").toString("base64"),
                        isBase64Encoded: true,
                        headers: { "content-type": "application/json" },
                    });
                },
            },
        ],
    });

results in:

    D:\git\github\stefan.hanke\pulumi-apigateway>pulumi preview
    Enter your passphrase to unlock config/secrets
        (set PULUMI_CONFIG_PASSPHRASE to remember):
    Previewing update (apigateway-dev):
         Type                                Name                      Plan
     +   pulumi:pulumi:Stack                 myproject-apigateway-dev  create
     +   └─ aws:apigateway:x:API             hello                     create
     +      ├─ aws:iam:Role                  hello66382ec5             create
     +      ├─ aws:iam:RolePolicyAttachment  hello66382ec5-32be53a2    create
     +      ├─ aws:lambda:Function           hello66382ec5             create
     +      ├─ aws:apigateway:RestApi        hello                     create
     +      ├─ aws:apigateway:Deployment     hello                     create
     +      ├─ aws:lambda:Permission         hello-f61e7551            create
     +      └─ aws:apigateway:Stage          hello                     create
     
    Resources:
        + 9 to create

    Permalink: file:///C:/Users/Stefan/.pulumi/stacks/apigateway-dev.json

    D:\git\github\stefan.hanke\pulumi-apigateway>
