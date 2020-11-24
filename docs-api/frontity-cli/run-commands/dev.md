## `dev`

Starts a development server.

```text
npx frontity dev [options]
```

### Arguments

#### **`[options]`**

| Option | Description |
| :---: | :--- |
| `--production` | Builds the project for production. Related environment variable: [`FRONTITY_DEV_PRODUCTION`](https://docs.frontity.org/frontity-cli/environment-variables#frontity_dev_production) |
| `--port <port>` | Runs the server on a custom port. Default is 3000. Related environment variable: [`FRONTITY_DEV_PORT`](https://docs.frontity.org/frontity-cli/environment-variables#frontity_dev_port) |
| `--https` | Runs the server using https. Related environment variable: [`FRONTITY_DEV_HTTPS`](https://docs.frontity.org/frontity-cli/environment-variables#frontity_dev_https) |
| `--dont-open-browser` | Don't open a browser window with the localhost. Related environment variable: [`FRONTITY_DEV_DONT_OPEN_BROWSER`](https://docs.frontity.org/frontity-cli/environment-variables#frontity_dev_dont_open_browser) |
| `--target <target>` | Create bundles with `es5` or `module`. Default target is `module`. Related environment variable: [`FRONTITY_DEV_TARGET`](https://docs.frontity.org/frontity-cli/environment-variables#frontity_dev_target) |
| [`--publicPath <path>`](https://docs.frontity.org/frontity-cli/build-commands#the-publicpath-option) | Set the [public path](https://webpack.js.org/guides/public-path/) for static assets. Default path is `/static/`. Related environment variable: [`FRONTITY_DEV_PUBLIC_PATH`](https://docs.frontity.org/frontity-cli/environment-variables#frontity_dev_public_path). |
| `--help` | Output usage information |

**Examples**

* Starts a server in development mode using https and port 3002

```text
npx frontity dev --https --port 3002
```

* Starts a server in development mode using the folder `assets` as the path for statics

```text
npx frontity dev --public-path="/assets"
```

**The --production option**

This flag correspond to [webpack’s mode parameter](https://webpack.js.org/configuration/mode/) so it will run webpack in the production mode as described [there](https://webpack.js.org/configuration/mode/) before launching the development server.

So, if you do:

```text
npx frontity dev --production
```

The webpack bundler internally will do things like..

* Enable certain webpack-specific optimizations and minify the code
* Also disable hot-module reloading (HMR)
* Not create source maps
* Append hashes to filenames so for caching purposes

Normally, you would always use the development server in development mode, but sometimes you may want to check that everything works in production mode, or check the bundle analyzer (the files at `/build/analyze`) for the production bundle.
