---
title: "How to Build Node.js Apps with Paketo Buildpacks"
weight: 200
menu:
  main:
    parent: "howto"
    name: "Node.js"
aliases:
  - /docs/buildpacks/language-family-buildpacks/nodejs/
---

{{% howto_exec_summary bp_name="Paketo Node.js Buildpack" bp_repo="https://github.com/paketo-buildpacks/nodejs" reference_docs_path="/docs/reference/nodejs-reference" %}}

## Build a Sample App
To build a sample app locally with this buildpack using the pack CLI, run

{{< code/copyable >}}
git clone https://github.com/paketo-buildpacks/samples
cd samples/nodejs/npm
pack build my-app --buildpack paketo-buildpacks/nodejs \
  --builder paketobuildpacks/builder:base
{{< /code/copyable >}}

See [samples](https://github.com/paketo-buildpacks/samples/tree/main/nodejs/npm)
for how to run the app.

**NOTE: Though the example above uses the Paketo Base builder, this buildpack is
also compatible with the Paketo Full builder. The Paketo Full builder is
required if your app utilizes common C libraries.**

## Install a Specific Node Engine Version
The Node.js buildpack allows you to specify a version of Node.js to use during
deployment. This version can be specified in a number of ways, including
through the `BP_NODE_VERSION` environment variable, a `package.json`, `.nvmrc` or `.node-version` files. When specifying a
version of the Node.js engine, you must choose a version that is available
within the buildpack. The supported versions can be found on the Paketo Node Engine
component buildpack's [releases page](https://github.com/paketo-buildpacks/node-engine/releases/latest).

The buildpack prioritizes the versions specified in
each possible configuration location with the following precedence, from
highest to lowest: `BP_NODE_VERSION`, `package.json`, `.nvmrc` and `.node-version`.

### Using BP_NODE_VERSION

To configure the buildpack to use Node.js v12.12.0 when deploying your app, set the
following environment variable at build time, either directly (ex. `pack build
my-app --env BP_NODE_VERSION=12.12.0`) or through a
[project.toml](https://github.com/buildpacks/spec/blob/main/extensions/project-descriptor.md)
file:

{{< code/copyable >}}
BP_NODE_VERSION="12.12.0"
{{< /code/copyable >}}

### Using package.json
If your apps use `npm` or `yarn`, you can specify the Node.js version your apps use
during deployment by configuring the `engines` field in the `package.json`
file. To configure the buildpack to use Node.js v12.12.0 when deploying your
app, include the values below in your `package.json` file:

{{< code/copyable >}}
{
  "engines": {
    "node": "12.12.0"
  }
}
{{< /code/copyable >}}

For more information about the `engines` configuration option in the
`package.json` file, see the
[engines](https://docs.npmjs.com/files/package.json#engines) section of the
_npm-package.json_ topic in the NPM documentation.

### Using .nvmrc
Node Version Manager is a common option for managing the Node.js version an app
uses. To specify the Node.js version your apps use during deployment, include a
`.nvmrc` file with the version number. For more information about the contents
of a `.nvmrc` file, see [.nvmrc](https://github.com/nvm-sh/nvm#nvmrc) in the
Node Version Manager repository on GitHub.

### Using .node-version
`.node-version` is another common option that is compatible with Node.js version managers
such as `asdf` and `nodenv`. You can use a `.node-version` file to set the Node.js version
that your apps use during deployment, according to one of the following formats:

{{< code/copyable >}}
12.12.0
{{< /code/copyable >}}

OR

{{< code/copyable >}}
v12.12.0
{{< /code/copyable >}}

OR

{{< code/copyable >}}
12.12
{{< /code/copyable >}}


### Deprecated: Using buildpack.yml

Specifying the Node version through `buildpack.yml` configuration will be deprecated in Node Engine Buildpack v1.0.0.
To migrate from using `buildpack.yml` please set the `$BP_NODE_VERSION` environment variable.

## Enable Heap Memory Optimization
Node.js limits the total size of all objects on the heap. Enabling the
`optimize-memory` feature sets this value to three-quarters of the total memory
available in the container. For example, if your app is limited to 1&nbsp;GB
at run time, the heap of your Node.js app is limited to 768&nbsp;MB.

You can enable memory optimization through the `BP_NODE_OPTIMIZE_MEMORY` environment variable.

### Using `BP_NODE_OPTIMIZE_MEMORY` Environment Variable
To enable memory optimization through the `BP_NODE_OPTIMIZE_MEMORY` environment
variable, set it to `true`.

{{< code/copyable >}}
pack build my-app \
  --buildpack paketo-buildpacks/nodejs \
  --env BP_NODE_OPTIMIZE_MEMORY=true
{{< /code/copyable >}}

### Deprecated: Using buildpack.yml

Enabling memory optimization through your `buildpack.yml` file will be
deprecated in Node Engine Buildpack v1.0.0. To migrate from using
`buildpack.yml` please set the `BP_NODE_OPTIMIZE_MEMORY` environment variable
mentioned above.

## Build an App From Source in a Subdirectory

To specify a subdirectory to be used as the root of the app, please use the
`BP_NODE_PROJECT_PATH` environment variable at build time either directly or
through a [`project.toml`](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor).
This could be useful if your app is a part of a monorepo.

For example, if your project has the following structure:
```
.
├── go-app
│   ├── go.mod
│   └── main.go
└── node-app
    ├── file.js
    ├── index.js
    └── package.json
```
you could then set the following at build time.
```
$BP_NODE_PROJECT_PATH=node-app
```

## Run Scripts During Build Phase
To specify a script or series of scripts to run during build phase, please use the 
`BP_NODE_RUN_SCRIPTS` environment variable at build time either directly or through a 
[`project.toml`](https://buildpacks.io/docs/app-developer-guide/using-project-descriptor). This 
could be useful if your app uses a framework like Angular, React, or Vue where you need to run 
scripts to build your app into a production state.

For example, if your project's `package.json` has the following scripts:
```json
{
  "scripts": { 
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test",
    "lint": "eslint src/**/*.js src/**/*.jsx",
  }
}
```
and your environment variable was set:
```
$BP_NODE_RUN_SCRIPTS=lint,build
```
then the `lint` and then `build` scripts would be run via npm or yarn, during build phase. Note that the 
value for `BP_NODE_RUN_SCRIPTS` must be a comma separated list of script names.

## Build an App that Uses NPM
The Node.js buildpack can detect automatically if an app requires `npm`.

### Configure NPM During the Build
The Node.js buildpack respects native configuration options for NPM. If you would
like to learn more about NPM native configuration please check the NPM
Configuration [documentation](https://docs.npmjs.com/cli/v6/using-npm/config)
and the `.npmrc`
[documentation](https://docs.npmjs.com/cli/v6/configuring-npm/npmrc).

#### Project-level `.npmrc`
Adding an `.npmrc` file in your app's working directory will allow you to provide
[project-level][npmrc/precedence] `npm` configuration.

#### Global `.npmrc`
Some users may prefer not to include an `.npmrc` file in their source code 
and app image (e.g. if an `.npmrc` contains credentials for connecting
to a private registry). The `.npmrc` can be provided via a [service binding][service-binding]
whose `type` is `npmrc`. The binding must contain a file called `.npmrc`.
The Node.js Buildpack will set this binding as the [`NPM_CONFIG_GLOBALCONFIG`][npmrc/precedence] in
the build environment.
To `pack build` with the binding:
{{< code/copyable >}}
pack build myapp \
    --env SERVICE_BINDING_ROOT=/bindings \
    --volume <absolute-path-to-binding-dir>:/bindings/npmrc
{{< /code/copyable >}}

## Build an App that Uses Yarn
The Node.js buildpack can detect automatically if an app requires `yarn`, by
checking for a `yarn.lock` file.

### Configure Yarn During the Build
The Node.js buildpack respects native configuration options for Yarn. If you would
like to learn more about Yarn configuration using `.yarnrc` please visit [the
Yarn documentation](https://classic.yarnpkg.com/en/docs/yarnrc).

#### Project-level `.yarnrc`
Adding an `.yarnrc` file in your app's working directory will allow you to provide
project-level `yarn` configuration.

#### User-level `.yarnrc`
Some users may prefer not to include an `.yarnrc` file in their source code 
and app image. The `.yarnrc` can be provided via a [service binding][service-binding]
whose `type` is `yarnrc`. The binding must contain a file called `.yarnrc`.
The Node.js Buildpack will set this binding as the user-level `.yarnrc` in
the build environment. It will not be present in the launch environment.
To `pack build` with the binding:
{{< code/copyable >}}
pack build myapp \
    --env SERVICE_BINDING_ROOT=/bindings \
    --volume <absolute-path-to-binding-dir>:/bindings/yarnrc
{{< /code/copyable >}}

## Compile Native Extensions with `node-gyp`
If your app requires compilation of native extensions using `node-gyp`, the Node.js buildpack requires that
you use the Paketo Full Builder. This is because `node-gyp` requires `python` which is excluded from
the Paketo Base Builder's stack, and the module may require other shared objects.

### With pack and a Command-Line Flag
When building with the pack CLI, specify the latest Paketo Full Builder at build time
with the `--builder` flag.

{{< code/copyable >}}
pack build my-app --builder paketobuildpacks/builder:full
{{< /code/copyable >}}

## Build an App Without Package Management

The Node.js buildpack supports building apps without `node_modules` or a `package.json`.
It will detect this type of app automatically, by looking for one of these four files in
the root of your application directory:
- `server.js`
- `app.js`
- `main.js`
- `index.js`

### Specify A Custom Entrypoint
If your app's entrypoint file is not one of the four files named above,
you can specify a different file name (or path) by setting the `BP_LAUNCHPOINT`
environment variable at build time.

#### Using BP_LAUNCHPOINT
`BP_LAUNCHPOINT` can be set as follows:


{{< code/copyable >}}
BP_LAUNCHPOINT="./src/launchpoint.js"
{{< /code/copyable >}}

The image produced by the build will run `node src/launchpoint.js`
as its start command.

## Build and Serve a Front-end Framework App

Combining the Paketo Node.js buildpack with a web server buildpack like the
Paketo Apache HTTP Server buildpack or Paketo NGINX buildpack allows you to
easily containerize front-end framework apps (e.g. React, Angular, Vue). The
buildpacks will build source code into production-ready static assets, then
automatically configure a web server to serve those assets. Check the Paketo samples
repository for an example [React app][sample/react-servers] with build instructions.

1. Define a script under the `"scripts"` property of your `package.json` that
   builds your production-ready static assets. Most frameworks bootstrap this
   automatically. For React, it's `"build"`.

2. Find out where static assets are stored after the build script runs. It'll
   be a directory under the root of the app directory. For React, this is
   `./build` by default.

3. Select which web server to use: NGINX or HTTPD.

4. **[NGINX]** If you chose HTTPD, **skip to step 5**. Use the Node.js and
   NGINX buildpacks to build your app. Use environment variables to configure
   the server. `BP_NODE_RUN_SCRIPTS` should be set to the name of the _build
   script_ from step 1. `BP_WEB_SERVER_ROOT` should be set to the build _output
   directory_ from step 2. To (optionally) further adjust the behaviour of the
   NGINX server, see the NGINX How-to [guides][nginx/configure].
<!-- spellchecker-disable -->
{{< code/copyable >}}
pack build frontend-nginx --buildpack paketo-buildpacks/nodejs \
                          --buildpack paketo-buildpacks/nginx \
                          --env BP_NODE_RUN_SCRIPTS=build \
                          --env BP_WEB_SERVER=nginx \
                          --env BP_WEB_SERVER_ROOT=build
{{< /code/copyable >}}
<!-- spellchecker-enable -->

5. **[HTTPD]** If you chose NGINX, **skip to step 6**. Use the Node.js and
   HTTPD buildpacks to build your app. Use environment variables to configure
   the server. `BP_NODE_RUN_SCRIPTS` should be set to the name of the _build
   script_ from step 1. `BP_WEB_SERVER_ROOT` should be set to the build _output
   directory_ from step 2. To (optionally) further adjust the behaviour of the
   HTTPD server, see the HTTPD How-to [guides][httpd/configure].
<!-- spellchecker-disable -->
{{< code/copyable >}}
pack build frontend-httpd --buildpack paketo-buildpacks/nodejs \
                          --buildpack paketo-buildpacks/httpd \
                          --env BP_NODE_RUN_SCRIPTS=build \
                          --env BP_WEB_SERVER=httpd \
                          --env BP_WEB_SERVER_ROOT=build
{{< /code/copyable >}}
<!-- spellchecker-enable -->

6. You're done! The resulting app container will serve your frontend app with
   an automatically generated server configuration.


## Enable Process Reloading
By default, your Node.js server will be the only process running in your app
container at runtime. You can enable restarting the server process
when files in the app's working directory change, which may facilitate a shorter
feedback loop for iterating on code changes. This feature may be used in conjunction with
a dev orchestrator like [Tilt][tilt].

### Using `BP_LIVE_RELOAD_ENABLED`

To enable reloadable processes, set the `$BP_LIVE_RELOAD_ENABLED` environment
variable at build time, either by passing a flag to the
[platform][definition/platform] or by
adding it to your `project.toml`. See the Cloud Native Buildpacks
[documentation][project-file] to learn more about `project.toml` files.

#### With a `pack build` flag
{{< code/copyable >}}
pack build myapp --env BP_LIVE_RELOAD_ENABLED=true
{{< /code/copyable >}}

#### In a `project.toml` file
{{< code/copyable >}}
[[ build.env ]]
  name = 'BP_LIVE_RELOAD_ENABLED'
  value = 'true'
{{< /code/copyable >}}

#### In a `Tiltfile` with the `pack` resource
You can use the Paketo Node.js buildpack with [Tilt][tilt]. This example
uses the [`pack` extension][tilt/pack] for Tilt.
<!-- spellchecker-disable -->
{{< code/copyable >}}
pack('my-app',
  buildpacks=["paketo-buildpacks/nodejs"],
  env_vars=["BP_LIVE_RELOAD_ENABLED=true"],
  live_update=[
    sync('.', '/workspace'),
  ]
)
{{< /code/copyable >}}
<!-- spellchecker-enable -->

## Install a Custom CA Certificate
Node.js Buildpack users can provide their own CA certificates and have them
included in the container root truststore at build-time and runtime by
following the instructions outlined in the [CA
Certificates]({{< ref "/docs/howto/configuration#ca-certificates" >}})
section of our configuration docs.

## Override the Start Process Set by the Buildpack
Node.js Buildpack users can set custom start processes for their app image by
following the instructions in the
[Procfiles]({{< ref "/docs/howto/configuration#procfiles" >}}) section
of our configuration docs.

## Set Environment Variables for App Launch Time
Node.js Buildpack users can embed launch-time environment variables in their
app image by following the documentation for the [Environment Variables
Buildpack](https://github.com/paketo-buildpacks/environment-variables/blob/main/README.md).

## Add Custom Labels to the App Image
Node.js Buildpack users can add labels to their app image by following the
instructions in the [Applying Custom
Labels]({{< ref "/docs/howto/configuration#applying-custom-labels" >}})
section of our configuration docs.

<!-- spellchecker-disable -->
<!-- References -->
[tilt]:https://tilt.dev/
[tilt/pack]:https://github.com/tilt-dev/tilt-extensions/tree/master/pack
[definition/platform]:https://buildpacks.io/docs/concepts/components/platform
[project-file]:https://buildpacks.io/docs/app-developer-guide/using-project-descriptor/
[service-binding]:{{< ref "docs/howto/configuration#bindings" >}}
[npmrc/precedence]:https://docs.npmjs.com/cli/v8/using-npm/config#npmrc-files

[nginx/configure]:{{< ref "docs/howto/web-servers#automatically-generate-an-nginxconf" >}}
[httpd/configure]:{{< ref "docs/howto/web-servers#automatically-generate-an-httpdconf" >}}
[sample/react-servers]:https://github.com/paketo-buildpacks/samples/tree/main/nodejs/react-httpd-nginx
<!-- spellchecker-enable -->
