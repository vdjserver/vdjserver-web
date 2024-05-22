VDJServer
===================

VDJServer is a next generation immune repertoire analysis portal and platform.

## Deployments

 * Development: <https://vdj-dev.tacc.utexas.edu>
 * Staging: <https://vdj-staging.tacc.utexas.edu>
 * Production: https://vdj-prod.tacc.utexas.edu -> <https://vdjserver.org>

In general, the development instance is meant for testing new features that are not ready to be released yet. The staging instance is meant to test upcoming releases/features that are considered to be stable.

## Components

VDJServer is currently composed of 4 separate components:

 * [vdjserver-web-api](https://github.com/vdjserver/vdjserver-web-api/): a node.js API service that serves as middleware for VDJ clients and Tapis.
 * [vdjserver-web-backbone](https://github.com/vdjserver/vdjserver-web-backbone/): a web application that end users can interact with.
 * [vdjserver-web-nginx](https://github.com/vdjserver/vdjserver-web-nginx/): a webserver configured for vdj API/webapp needs.
 * [vdjserver-plumber](https://github.com/vdjserver/vdjserver-plumber/): R visualization server.

All 4 components need to be deployed in order for VDJServer to function properly.

## Configuration Procedure

You will need to clone the parent project and all submodules in order to set up an instance of VDJServer.

```
# Clone project
$ git clone https://github.com/vdjserver/vdjserver-web.git

$ cd vdjserver-web

# Clone submodules
$ git submodule update --init --recursive
```

All configuration procedures are based upon docker containers of the components.

### Configuring vdjserver-web-api

Sometimes there are issues with permissions on the redis directory, which
will prevent builds and/or startups. If such happens, completely delete the
directory then mkdir with the appropriate user.

```
$ rm -rf vdjserver-web-api/redis
$ mkdir vdjserver-web-api/redis
```

There is one configuration file that needs to be set up to run the API:

 * vdjserver-web-api/.env

It can be copied from its default template:

```
$ cd vdjserver-web-api
$ cp .env.defaults .env
$ emacs .env
```

Please consult a VDJServer admin for proper settings.

### Configuring vdjserver-web-backbone

There is only one file that needs to be set up for the backbone app to function properly:

 * environment-config.js

This file can be copied from its default template, edit to make any hostname changes:

```
$ cd vdjserver-web-backbone
$ mkdir docker/environment-config/v2
$ cp docker/environment-config/environment-config.js.defaults docker/environment-config/v2/environment-config.js

$ emacs docker/environment-config/v2/environment-config.js
```

The default values specified in the template are suitable for production, but should be changed for dev, staging, and local deployments as necessary.

## Deployment Procedure

### SSL

VDJServer does not handle SSL certificates directly, and is currently configured to run HTTP internally on port 8080. It must be deployed behind a reverse proxy in order to allow SSL connections.

Review the [VDJServer administration guide](https://vdjserver.readthedocs.io/en/latest/admin/admin.html) for details about configuring nginx with SSL.

### Docker compose build

There are a set of docker compose files for building and starting all of the components.

```
$ cd docker-compose/v2
$ docker compose build
```

If everything has build properly, then can manually bring up the system with:

```
$ docker compose up
```

Verify no unexpected error messages. Verify you can access the website GUI. Manually bring down the system with:

```
$ docker compose down
```

### Configuring systemd

You will need to set up the VDJServer systemd service file on your host machine in order to have the VDJ web infrastructure automatically restart when the host machine reboots.
Verify the paths point to the appropriate docker-compose directory.

```
sudo cp host/systemd/vdjserver.service /etc/systemd/system/vdjserver.service

sudo systemctl daemon-reload

sudo systemctl enable docker

sudo systemctl enable vdjserver

sudo systemctl start vdjserver
```

## Development Guidelines

### Code Style

 * Code should roughly follow Google Javascript Style Guide conventions: <https://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml>.

 * A jscs.rc file (Javascript Code Style Checker) file has been provided in the project repo, and all developers are encouraged to use it.

 * A git pre-commit hook is available via the file pre-commit.sh. To use it, just symlink it as follows: ```ln -s ../../pre-commit.sh .git/hooks/pre-commit```

 * Spaces are preferred over tabs, and indentation is set at 4 spaces.

 *  Vimrc settings: ```set shiftwidth=4, softtabstop=4, expandtab```

### Git Structure and Versioning Process

 * This project uses the Git Flow methodology for code management and development: <https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow>.

 * New development and features should be done on branches that are cloned from the **develop** branch, and then merged into this branch when completed. Likewise, new release candidates should be branched from **develop**, and then merged into **master** once they have been tested/verified. Once a release branch is ready for production, it should be merged into **master** and tagged appropriately. Every deployment onto production should be tagged following semantic versioning 2.0.0 standards: <http://semver.org/>.

