VDJServer
===================

VDJServer is a next generation immune repertoire analysis portal and platform.

##Deployments

 * Development: <https://vdj-dev.tacc.utexas.edu>
 * Staging: <https://vdj-staging.tacc.utexas.edu>
 * Production: <https://vdjserver.org>

In general, the development instance is meant for testing new features that are not ready to be released yet. The staging instance is meant to test upcoming releases/features that are considered to be stable.

NOTE: the development and staging instances are currently using docker, but production has not migrated to docker yet as of 19/June/2015.

##Components

VDJServer is currently composed of 3 separate components that have been added to this repository as submodules:

 * [vdjserver-web-api]<https://bitbucket.org/vdjserver/vdjserver-web-api/>: a node.js API service that serves as middleware for VDJ clients and Agave.
 * [vdjserver-web-backbone]<https://bitbucket.org/vdjserver/vdjserver-web-backbone/>: a web application that end users can interact with.
 * [vdjserver-web-nginx]<https://bitbucket.org/vdjserver/vdjserver-web-nginx/>: a webserver configured for vdj API/webapp needs.

All 3 components need to be deployed in order for VDJServer to function properly.

##Configuration Procedure

All configuration procedures are the same for dockerized and non-dockerized versions of these apps.

**Configuring vdjserver-web-api**

There are two files that need to be set up to run the API:

 * vdjserver-web-api/app/scripts/config/agaveSettings.js
 * vdjserver-web-api/app/scripts/config/config.js

They can be copied from their default templates:

```
    $ cd vdjserver-web-api/app/scripts/config
    $ cp agaveSettingsDefault.js agaveSettings.js
    $ cp configDefault.js config.js
```

The agaveSettings.js file will need WSO2 app and key information, in addition to service account information. These values are considered to be confidential and are available via stache. Please see a VDJ project team member for access.

** Configuring vdjserver-web-backbone**

There is only one file that needs to be set up for the backbone app to function properly:

 * vdjserver-web-backbone/app/scripts/config/environment-config.js

This file can be copied from its default template:

```
    $ cd vdjserver-web-backbone/app/scripts/config
    $ cp environment-config.js.defaults environment-config.js
```

The default values specified in the template are suitable for production, but should be changed for dev, staging, and local deployments as necessary.

##Deployment Procedure

###Dockerized instances (development and staging)

Dockerized instances may be deployed using the supplied init.d script: vdjserver-web/host/init.d/vdjserverweb. This has already been installed on development and staging systems to /etc/init.d/vdjserverweb. It can be accessed as follows:

```
    [wscarbor@vdj-dev vdjserver-web]$ sudo service vdjserverweb
    Usage: /etc/init.d/vdjserverweb {start|stop|restart|rebuild|rebuild-without-cache}
```

In most cases, a simple restart command is sufficient to bring up vdjserver. The restart command will attempt to stop all running docker-compose instances, and it is generally successful. However, if it encounters any problems then you can just stop instances manually and try it again:

```
[wscarbor@vdj-dev vdjserver-web]$ sudo docker ps
CONTAINER ID        IMAGE                        COMMAND                CREATED             STATUS              PORTS                                      NAMES
fdc7c3119366        vdjserverweb_nginx:latest    "/root/nginx-config-   32 minutes ago      Up 32 minutes       0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   vdjserverweb_nginx_1
adfecbce3e55        vdjserverweb_vdjapi:latest   "/bin/sh -c '/usr/bi   32 minutes ago      Up 32 minutes       8443/tcp                                   vdjserverweb_vdjapi_1

[wscarbor@vdj-dev vdjserver-web]$ sudo docker stop fdc
```

###Non-Dockerized instances (production)

**Nginx**
Production uses a version of nginx that has been installed via yum. Its configuration is located /etc/nginx/nginx.conf.

**Backbone**
Backbone on production has been deployed at /var/www/html/vdjserver-backbone/.
It is served out of /var/www/html/vdjserver-backbone/live-site, and can be built via a standard grunt build process as follows:

```
    $ cd /var/www/html/vdjserver-backbone
    $ grunt build
```

Once a new build has completed, grunt will automatically rotate builds as needed. It will move the old build into live-site-backup and place the new build into live-site so it can be served immediately.

**API**
The VDJ API on production has been deployed at /var/www/node/vdjserver-auth/.
It is a small node.js program that uses forever.js for persistence. There is a small script in place for starting/restarting the VDJ API as follows:

```
    $ cd /var/www/node/vdjserver-auth
    $ ./vdjauth-start.sh
```

This script will automatically stop a currently running instance if it exists, and will immediately start a new instance with correct logging parameters.

##Development Guidelines

**Code Style**

 * Code should roughly follow Google Javascript Style Guide conventions: <https://google-styleguide.googlecode.com/svn/trunk/javascriptguide.xml>.

 * A jscs.rc file (Javascript Code Style Checker) file has been provided in the project repo, and all developers are encouraged to use it.

 * A git pre-commit hook is available via the file pre-commit.sh. To use it, just symlink it as follows: ```ln -s ../../pre-commit.sh .git/hooks/pre-commit```

 * Spaces are preferred over tabs, and indentation is set at 4 spaces.  

 *  Vimrc settings: ```set shiftwidth=4, softtabstop=4, expandtab```

**Git Structure and Versioning Process**

 * This project uses the Git Flow methodology for code management and development: <https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow>.

 * New development and features should be done on branches that are cloned from the **develop** branch, and then merged into this branch when completed. Likewise, new release candidates should be branched from **develop**, and then merged into **master** once they have been tested/verified. Once a release branch is ready for production, it should be merged into **master** and tagged appropriately. Every deployment onto production should be tagged following semantic versioning 2.0.0 standards: <http://semver.org/>.

##Development Setup

You will need to clone down the parent project and all submodules in order to set up a local instance of vdjserver.

```
    - Clone project
        $ git clone git@bitbucket.org:vdjserver/vdjserver-web.git

        cd vdjserver-web

    - Clone submodules
        $ git submodule update --init
        $ git submodule foreach git checkout master
        $ git submodule foreach git pull

    - Follow configuration steps listed above in the "Configuration Procedure" section of this document
```

If you are working on a single component, it may be easier to run it directly on your development machine rather than running it through docker. For example, vdjserver-web-backbone livereload tends to work better when running directly on the host. However, this is your choice so feel free to use whatever works best for you.
