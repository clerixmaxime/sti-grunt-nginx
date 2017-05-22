grunt and nginx - CentOS Docker image
========================================

This repository contains the source for building various versions of
the Grunt.js frontends as a reproducible Docker image using
[source-to-image](https://github.com/openshift/source-to-image).  The
resulting image can be run using [Docker](http://docker.io).

Versions
---------------

CentOS versions currently provided are:
* CentOS7

OpenShift 3 Usage
---------------------

First ensure you have a [working OpenShift 3
environment](http://www.openshift.org/) with the `oc` command in your
path.

```
$ oc new-build https://github.com/clerixmaxime/sti-grunt-nginx.git --context-dir=/1.0 --strategy=docker --to=sti-grunt-nginx -n openshift
```

Wait for that buld to complete. Monitor it
with `oc status`.

Changes
----------------

* Update to Node.js 6.x
* Run as root to avoid change ownership and permissions failures
