---
layout: post
title:  "Pulumi - Webserver on AWS EC2 - Take 1"
date:   2020-04-28
categories: iac aws typescript
---

A report on failing a typescript-based Pulumi project for the Python SimpleHTTPServer.

The task seemd straightforward. Using [this tutorial](https://www.pulumi.com/docs/tutorials/aws/ec2-webserver/), I wanted to just play around with Pulumi a bit more. Turns out, things are always neither trivial nor straightforward.

## node-gyp failed to compile

Node-modules may have C++ code embedded that is compiled on installing the module.
This typically does not work with newer versions of Node. An example error looks like

    c:\users\stefan\appdata\local\node-gyp\cache\14.0.0\include\node\openssl\ossl_typ.h(91): error C2371: 'EVP_MD': redefinition; different basic types (compiling source file ..\deps\grpc\src\boringssl\err_data.c) [D:\git\github\stefan.hanke\webserver\node_modules\grpc\build\boringssl.vcxproj]

The solution was to switch from V14.0.0 to the latest Node LTS version V12.16.2. I've refrained to use [nvm-windows](https://github.com/coreybutler/nvm-windows) since it looks like not being maintained anymore.

As an interesting tidbit, running `npm audit` find 9 vulnerabilities with a low score on *prototype pollution*, a rather JS-specific problem.

## ts failed to compile the code

    D:\git\github\stefan.hanke\webserver>pulumi preview
    Enter your passphrase to unlock config/secrets
        (set PULUMI_CONFIG_PASSPHRASE to remember):
    Previewing update (webserver-dev):
         Type                 Name                     Plan       Info                                                                                            +   pulumi:pulumi:Stack  webserver-webserver-dev  create     1 error                                                                                        
    Diagnostics:
      pulumi:pulumi:Stack (webserver-webserver-dev):
        error: Running program 'D:\git\github\stefan.hanke\webserver' failed with an unhandled exception:
        TSError: ⨯ Unable to compile TypeScript:
        index.ts(22,14): error TS2339: Property 'id' does not exist on type 'Promise<GetAmiResult>'.

            at createTSError (D:\git\github\stefan.hanke\webserver\node_modules\ts-node\src\index.ts:261:12)
            at getOutput (D:\git\github\stefan.hanke\webserver\node_modules\ts-node\src\index.ts:367:40)
            at Object.compile (D:\git\github\stefan.hanke\webserver\node_modules\ts-node\src\index.ts:558:11)
            at Module.m._compile (D:\git\github\stefan.hanke\webserver\node_modules\ts-node\src\index.ts:439:43)
            at Module._extensions..js (internal/modules/cjs/loader.js:1176:10)
            at Object.require.extensions.<computed    [as .ts] (D:\git\github\stefan.hanke\webserver\node_modules\ts-node\src\index.ts:442:12)
            at Module.load (internal/modules/cjs/loader.js:1000:32)
            at Function.Module._load (internal/modules/cjs/loader.js:899:14)
            at Module.require (internal/modules/cjs/loader.js:1042:19)
            at require (internal/modules/cjs/helpers.js:77:18)

The problem is that the Pulumi API was changed to be async in nature, which by itself is ok. However the whole example unfortunately breaks down for me. One cannot use the newer TypeScript 3.8 release that has support for [top-level await](https://www.infoq.com/news/2020/02/typescript-3-8-release/), since *target* and *module* options are hardcoded values inside pulumi for typescript. This was [reported](https://github.com/pulumi/pulumi/issues/2673) and no fix in sight.

       if (typeScript) {
           tsnode.register({
               typeCheck: true,
               skipProject: skipProject,
               compilerOptions: {
                   target: "es6",
                   module: "commonjs",
                   moduleResolution: "node",
                   sourceMap: "true",
               },
           });
       }

I'm switching to CSharp in Take 2.