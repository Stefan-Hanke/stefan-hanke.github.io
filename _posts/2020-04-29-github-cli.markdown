---
layout: post
title:  "GitHub CLI"
date:   2020-04-29
categories: github cli
---

GitHub offers a CLI client to interact with its service from a command line.

I've installed it using *chocolatey* - `choco install gh` and you can start. I really like the `refreshenv` script that comes bundled with chocolatey. It updates `PATH` in the current terminal, no need to start a new one.

The source of the client is - how could that be different - [hosted on GitHub](https://github.com/cli/cli).
The CLI is written in the *command style* (or whatever this style of CLI argument passing is called).


    D:\git\github\stefan.hanke>gh repo create -h
    flag needs an argument: 'h' in -h
    Usage:
      gh repo create [<name>] [flags]

    Flags:
      -d, --description string   Description of repository
          --enable-issues        Enable issues in the new repository (default true)
          --enable-wiki          Enable wiki in the new repository (default true)
      -h, --homepage string      Repository home page URL
          --public               Make the new repository public
      -t, --team string          The name of the organization team to be granted access

    Global Flags:
          --help              Show help for command
      -R, --repo OWNER/REPO   Select another repository using the OWNER/REPO format

Once the CLI has access to the GitHub account, it allows very easy interaction with it.

    D:\git\github\stefan.hanke>gh repo create pulumi-apigateway -d "pulumi - api gateway and lambda"
    Notice: authentication required
    Press Enter to open github.com in your browser...
    vAuthentication complete. Press Enter to continue...

    ✓ Created repository Stefan-Hanke/pulumi-apigateway on GitHub
    ? Create a local project directory for Stefan-Hanke/pulumi-apigateway? Yes
    Initialized empty Git repository in D:/git/github/stefan.hanke/pulumi-apigateway/.git/
    ✓ Initialized repository in './pulumi-apigateway/'


## CLI in general


I'm curious whether there is some term that describes this kind of CLI.
It certainly has a property that I personally really like - [*discoverability*](https://en.wikipedia.org/wiki/Discoverability).
Sure, you can have man pages, websites with documentation etc. however, I like being able to attach `-h` to some command or subcommand.

    D:\git\github\stefan.hanke>dotnet -h
    .NET Core SDK (3.1.100)
    Usage: dotnet [runtime-options] [path-to-application] [arguments]

    Execute a .NET Core application.

    [flags, options etc]

    SDK commands:
      add               Add a package or reference to a .NET project.
      build             Build a .NET project.
      build-server      Interact with servers started by a build.
      clean             Clean build outputs of a .NET project.
      [more commands]

    [more output]

    D:\git\github\stefan.hanke>dotnet build -h
    Usage: dotnet build [options] <PROJECT | SOLUTION>

    Arguments:
      <PROJECT | SOLUTION>   The project or solution file to operate on. If a file is not specified, the command will search the current directory for one.

    Options:
      -h, --help                            Show command line help.
      -o, --output <OUTPUT_DIR>             The output directory to place built artifacts in.
      -f, --framework <FRAMEWORK>           The target framework to build for. The target framework must also be specified in the project file.
      [more options]

    D:\git\github\stefan.hanke>


There are pages on Wikipedia. The [CLI page](https://en.wikipedia.org/wiki/Command-line_interface) reads like a history lesson, to be expected. At least the [overview](https://en.wikipedia.org/wiki/Comparison_of_command_shells) gives clues as to what CLIs could do.

The [oclif](https://oclif.io/) project seems promising - it is a CLI generator for the language - *wait for it* - JavaScript. Kind of funny to see so much momentum inside this community.