---
layout: post
title:  "editly - programmatically generate video"
date:   2020-04-30
categories: commandline editly
---

I tried to play around with [editly](https://github.com/mifi/editly), ended up in a session of linux dependency management.

> Editly is a tool and framework for declarative NLE (non-linear video editing) using Node.js and ffmpeg. Editly allows you to easily and programmatically create a video from set of clips, images and titles, with smooth transitions between and music overlaid.

Well, the given option with `npm i editly -g` did not work out. I'm using Windows and Docker4Windows, and thus opted for generating a Dockerfile that gives me access to editly.
Installing `node` in `debian:buster` was straightforward enough, follow along instructions found on the [node homepage](https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions-enterprise-linux-fedora-and-snap-packages) and done.
Then I basically iterated [installing packages found here](https://github.com/stackgl/headless-gl#system-dependencies) until editly finally was working without errors like:

    root@46ed422ea4e6:/editly/examples# editly --json subtitle.json5 
    640x640 25fps
    createFrameSource gl clip 0 layer 0
    Loop failed TypeError: Cannot set property 'lastAttribCount' of null
        at new Shader (/usr/local/share/.config/yarn/global/node_modules/gl-shader/index.js:13:27)
        at createShader (/usr/local/share/.config/yarn/global/node_modules/gl-shader/index.js:253:16)
        at createGlFrameSource (/usr/local/share/.config/yarn/global/node_modules/editly/sources/glFrameSource.js:26:18)
        at async /usr/local/share/.config/yarn/global/node_modules/p-map/index.js:57:22

The Dockerfile is part of [this repository](https://github.com/Stefan-Hanke/playing-with-editly). `editly` uses `ffmpeg` for video generation and `headless-gl`, which in turn requires an X11 server (`xvfg`) with OpenGL support (`mesa`). Quite some dependencies!

`xvfb` provides a `xvfb-run` command. It opens up a X11 display at `:99.0` (see [here](https://gerardnico.com/ssh/x11/display) for an explanation of the DISPLAY string). Then *just* run

> xvfb-run -s "-ac -screen 0 1280x1024x24" editly --json subtitle.json5

Unfortunately (but also understandable, unfortunately), editly does not prepack necessary assets to actually run the examples. I've bundled the font *Patua One* from [font squirrel](https://www.fontsquirrel.com/license/patua-one), and actually took the time to read its license and also bundled it.