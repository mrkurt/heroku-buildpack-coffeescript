Heroku buildpack: Coffeescript
==============================

This is a [Heroku buildpack](http://devcenter.heroku.com/articles/buildpacks) for [Coffeescript](http://coffeescript.org/) apps. It uses [NPM](http://npmjs.org/) and [SCons](http://www.scons.org/). It is based on the [Heroku Node.js buildpack](https://github.com/heroku/heroku-buildpack-nodejs).

Usage
-----

Example usage:

    $ ls
    Procfile  package.json  src

    $ ls src
    app.coffee

    $ heroku create --stack cedar --buildpack https://github.com/aergonaut/heroku-buildpack-coffeescript.git

    $ git push heroku master
    ...
    -----> Heroku receiving push
    -----> Fetching custom buildpack
    -----> Coffeescript app detected
    -----> Vendoring node 0.4.7
    -----> Installing dependencies with npm 1.0.8
           coffee-script@1.3.3 ./node_modules/coffee-script

           express@2.1.0 ./node_modules/express
           ├── mime@1.2.2
           ├── qs@0.3.1
           └── connect@1.6.2
           Dependencies installed
    -----> Compiling coffee source
           Compiled to target directory

The buildpack will detect your app as Coffeescript if it detects a file matching `src/*.coffee` in your project root.  It will use NPM to install your dependencies, and vendors a version of the Node.js runtime into your slug.  The `node_modules` directory will be cached between builds to allow for faster NPM install time.

You must include Coffeescript in your `package.json`. The buildpack does not install Coffeescript automatically in order to allow you to specify your own Coffeescript version.

Compiled javascript is written to the `target` directory in the slug. Your `Procfile` should reference these compiled files like so:

    web: node target/app.js

Node.js and npm versions
------------------------

You can specify the versions of Node.js and npm your application requires using `package.json`

    {
      "name": "myapp",
      "version": "0.0.1",
      "engines": {
        "node": ">=0.4.7 <0.7.0",
        "npm": ">=1.0.0"
      }
    }

To list the available versions of Node.js and npm, see these manifests:

http://heroku-buildpack-nodejs.s3.amazonaws.com/manifest.nodejs
http://heroku-buildpack-nodejs.s3.amazonaws.com/manifest.npm

Hacking
-------

To use this buildpack, fork it on Github.  Push up changes to your fork, then create a test app with `--buildpack <your-github-url>` and push to it.

To change the vendored binaries for Node.js, NPM, and SCons, use the helper scripts in the `support/` subdirectory.  You'll need an S3-enabled AWS account and a bucket to store your binaries in.

For example, you can change the default version of Node.js to v0.6.7.

First you'll need to build a Heroku-compatible version of Node.js:

    $ export AWS_ID=xxx AWS_SECRET=yyy S3_BUCKET=zzz
    $ s3 create $S3_BUCKET
    $ support/package_nodejs 0.6.7

Open `bin/compile` in your editor, and change the following lines:

    DEFAULT_NODE_VERSION="0.6.7"
    S3_BUCKET=zzz

Commit and push the changes to your buildpack to your Github fork, then push your sample app to Heroku to test.  You should see:

    -----> Vendoring node 0.6.7
