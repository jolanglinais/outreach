# PNK Stack Service

![Node Template Logo Header Image][headerimg]

**P**ostgreSQL, **N**ode, and **K**oa

This acts as a guide for creating a relatively production ready backend micro-service, which can be customized to your needs.

The reference template repository can be found at [`irmerk/pnk-stack`][repo]

*ℹ️ Note*: This guide does not currently include PostgreSQL, but will in the future when I get back to it.

## Stack

- [Node.js][node] with [TypeScript][ts]
    - [Koa][koa]
- [PostgreSQL][psql]
    - [Objection][objection]

## Setup

All of this page assumes that you are using a Mac and that your software is up to date. If not up to date you may run into install issues when you install the packages below.

### Shell

macOS now ships with [Zsh][zsh] as the default shell. If you want more customization and handy shortcuts, you can install [oh my zsh][omz].

### Terminal Emulator

You can use the Terminal app that ships with macOS, but [iTerm2][iterm] is a popular and recommended alternative with more features and following. For information regarding prompt customization, see [Zsh Prompt][shprompt].

A helpful setting can also allow you to [use Alt/Cmd + Right/Left Arrow in iTerm][itermshort].

### Text Editor / IDE

Use whatever text editor / integrated development environment (IDE) you prefer. At the time of this writing, [Visual Studio Code][vsc] is an extremely popular one with a built-in plugin manager for easily extending and customizing functionality.

For helpful VSCode keyboard shortcuts to incorporate into your workflow, see their [Tips and Tricks][vsctt].

My suggestions for [VSCode extensions][vscext] and [keyboard shortcuts][vscsht].

### Command Line Tools

Install the [Homebrew][brew] package manager for easy and relatively safe installation and upgrades.

Recommended tools:

#### [`nvm`][nvm]

Manages installations of Node.js and npm:

```sh
brew install nvm \
&& mkdir ~/.nvm \
&& echo 'export NVM_DIR="$HOME/.nvm"' >> ~/.zprofile \
&& echo '[ -s "/opt/homebrew/opt/nvm/nvm.sh" ] && \. "/opt/homebrew/opt/nvm/nvm.sh" # This loads nvm' >> ~/.zprofile \
&& source ~/.zprofile
```

#### [Node.js][node]

[Node.js][node] and [npm][npm] could be installed directly via [Homebrew][brew] as well, but doing so makes it challenging to switch between versions. Some projects each require specific versions of [Node.js][node] and [npm][npm] versions to work properly, so nvm installations are strongly preferred.

```sh
nvm install node
```

#### [Git][git]

```sh
brew install git
```

Set your name and email address explicitly:

```sh
git config --global user.name "<YOUR_NAME>"
```

```sh
git config --global user.email <YOUR_EMAIL>
```

#### [Docker][dockerg]

# Rationale

## `Makefile`
- [Used for clarity and speed][make].
## [Koa][koa]

- I chose Koa because it is newer, less opinionated, lighter weight, and easier to customize than the standard of Express. Koa is modular and customizable because it requires modules for routing, the templating engine, and JSONP. This way, I can create middleware ourselves or use the built-in middleware.
- Koa focuses more on modern features of the JavaScript ES6 language, like [generators][generator], [async functions][async], and the [Node.js][node] runtime. Moreover, it uses promise-based flows and async-await syntax to get rid of callbacks, making the code more manageable, cleaner, and more readable. Because it uses generator syntax instead of callbacks, I can use the `yield` keyword to exit and then reenter.
- Koa uses a `context` object, which encapsulates `req/res` objects into one, helping us develop more efficiently by using several helpful methods. Finally, Koa uses cascading middleware. Thanks to asynchronous functions, middleware will run in a cascading fashion until the last middleware is reached.

## [Axios][axios]

- HTTP Client Library for [Node.js][node]
- [Comparison to other libraries][gotc]
- I chose axios instead of [got][got] because “[the got] package is native [ESM][esm] and no longer provides a CommonJS export. If your project uses CommonJS, you will have to [convert to ESM][sindresorhus] or use the [dynamic][dynamic] `import()` function. Please don't open issues for questions regarding CommonJS / ESM.”
- GOT has issues with linting, TypeScript, ESM/CJS, and JSON.

## Logging

The [logging transport][winston] is configured for logging to the console. This happening in a service set up in a cloud service can be configured to be picked up from `STDOUT`

## Linting

[Why I add a new line to the end of a file.][unix]

## GitHub Actions

`npx`: When I run a command with `npx`, it will check whether the command exists in the local `node_modules/.bin` directory or in the system-wide installed packages. If it doesn't find the command, it will attempt to temporary install the package and execute it. This makes `npx` an ideal tool to run the locally installed service being called.

## [Dockerfile][dockerbuild] & [Docker Compose][dockercompose]

- `Dockerfile`
    - Dockerizing a service allows for it to behave the same regardless of the platform on which it is run. A [Dockerfile][dockerbuild] is a blueprint on which the Docker image is built. When the built image is running, it is called a container. Essentially, the container starts as a Dockerfile that has the instructions on how to build the Docker image. The same image can be used to spin up one or even hundreds of containers, which is why Docker is so useful for software scalability.
    - I am using a slim production stage and a more feature-rich, development-focused dev stage.
    - Setting `NODE_ENV` to `production` [can increase performance by 3x][express].
    - Notice some of our commands are put together with an `&&`, creating [fewer Docker layers][dockerlayers], which is good for build caching.
    - Debian-based images because [it may be better in some cases than the more standard alpine linux image][martinheinz].
- `docker-compose.yml`
    - Docker Compose is a way to more easily setup multiple services running with Docker in an ecosystem. The *big* advantage of using Compose is you can define your application stack in a file, keep it at the root of your project repo (it’s now version controlled).
- `.dockerignore`
    - Just like I wouldn’t use Git without .`gitignore`, I use a `.dockerignore` file to ignore files that I don’t want to land in our Docker image. It helps to keep the Docker image small and keep the build cache more efficient by ignoring irrelevant file changes.
- `dumb-init`
    - I use `dumb-init` because it will be the first process (`PID 1`) and will handle the signal forwarding and the reaping of the zombie processes. It is a simple process supervisor and init system that helps to handle these situations.
    - Docker containers usually run a single application, and for that application, it is normal to expect it to receive signals from the Linux environment. However, when running in the background (as a daemon), the application may not correctly handle the Linux signals, which can lead to zombie processes and not graceful termination.
- Multiple Stages
    - Optimizing layers with multiple stages (build and production) in the `Dockerfile` results in a drastically smaller image size:

```sh
REPOSITORY      TAG     IMAGE ID        CREATED             SIZE
single-stage    latest  8ceb85286547    4 seconds ago       402MB
multi-stage     <none>  ec0680eefe42    About a minute ago  300MB
```

# Reference

- [emmanuelnk/RESTful-Typescript-Koa][emmanuelnk]
- [posquit0/koa-rest-api-boilerplate][posquit0]
- [inadarei/maikai][inadarei]
- [banzaicloud/service-tools][banzaicloud]
- [transitive-bullshit/koa-micro][transitive]
- [Koa Js : Part 1 - How to make a Koa server in 10 minutes!][kachiic]
- [Sentry's Koa Guide][sentryk]
- Health check
    - [dyhpoon/koa-simple-healthcheck][dyhpoon]
    - [rictorres/koa-ping-healthcheck][rictorres]
- Docker
    - [Best practices for writing Dockerfiles][dockerpractice]
    - [Snyk's 10 best practices to containerize Node.js web applications with Docker][snykdocker]
    - [Top 8 Docker Best Practices for using Docker in Production ✅][techworld_with_nana]
    - [hexops/dockerfile][hexops]

[headerimg]: ../images/nodeTemplate.png
[repo]: https://github.com/irmerk/pnk-stack

<!--Back-->
[node]: https://nodejs.org/en
[ts]: https://www.typescriptlang.org/
[koa]: https://koajs.com/
[psql]: https://www.postgresql.org/
[objection]: https://vincit.github.io/objection.js/

<!--Terminal Emulator-->
[iterm]: https://iterm2.com/
[shprompt]: ./shell-profile
[itermshort]: https://apple.stackexchange.com/questions/136928/using-alt-cmd-right-left-arrow-in-iterm

<!--Shell-->
[zsh]: https://zsh.sourceforge.io/
[omz]: https://ohmyz.sh/

<!--Text Editor / IDE-->
[vsc]: https://code.visualstudio.com/
[vsctt]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[sub]: https://www.sublimetext.com/
[neo]: https://neovim.io/
[emac]: https://www.spacemacs.org/
[ehlix]: https://helix-editor.com/
[vscsht]: ../reference/vscode.md#keyboard-shortcuts
[vscext]: ../reference/vscode.md#extensions

<!--Command Line Tools-->
[brew]: https://brew.sh/
[nvm]: https://github.com/nvm-sh/nvm
[npm]: https://www.npmjs.com/
[git]: https://git-scm.com/
[dockerg]: https://www.docker.com/get-started/
[terra]: https://developer.hashicorp.com/terraform
[terrad]: https://developer.hashicorp.com/terraform/downloads

<!--Rationale-->
[make]: https://spin.atomicobject.com/2021/03/22/makefiles-vs-package-json-scripts/
[axios]: https://github.com/axios/axios
[gotc]: https://github.com/sindresorhus/got#comparison
[got]: https://github.com/sindresorhus/got#install
[esm]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules
[sindresorhus]: https://gist.github.com/sindresorhus/a39789f98801d908bbc7ff3ecc99d99c
[dynamic]: https://v8.dev/features/dynamic-import
[winston]: https://github.com/winstonjs/winston/blob/master/docs/transports.md
[dockerbuild]: https://docs.docker.com/engine/reference/builder/
[dockercompose]: https://docs.docker.com/compose/
[express]: https://expressjs.com/en/advanced/best-practice-performance.html#set-node_env-to-production
[dockerlayers]: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#minimize-the-number-of-layers
[martinheinz]: https://martinheinz.dev/blog/92
[unix]: https://unix.stackexchange.com/a/18789
[generator]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Generator
[async]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/async_function

<!--Reference-->
[emmanuelnk]: https://github.com/emmanuelnk/RESTful-Typescript-Koa
[posquit0]: https://github.com/posquit0/koa-rest-api-boilerplate
[inadarei]: https://github.com/inadarei/maikai
[banzaicloud]: https://github.com/banzaicloud/service-tools
[transitive-bullshit]: https://github.com/transitive-bullshit/koa-micro
[kachiic]: https://dev.to/kachiic/koa-js-part-1-how-to-make-a-koa-server-in-10-minutes-3og9
[sentryk]: https://docs.sentry.io/platforms/node/guides/koa/
[dyhpoon]: https://github.com/dyhpoon/koa-simple-healthcheck
[rictorres]: https://github.com/rictorres/koa-ping-healthcheck
[dockerpractice]: https://docs.docker.com/develop/develop-images/dockerfile_best-practices/
[snykdocker]: https://snyk.io/blog/10-best-practices-to-containerize-nodejs-web-applications-with-docker/
[techworld_with_nana]: https://dev.to/techworld_with_nana/top-8-docker-best-practices-for-using-docker-in-production-1m39
[hexops]: https://github.com/hexops/dockerfile