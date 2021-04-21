# Flask Sample Application

This repository provides a sample Python web application implemented using the Flask web framework and hosted using ``gunicorn``. It is intended to be used to demonstrate deployment of Python web applications to OpenShift 4.

## Implementation Notes

This sample Python application relies on the support provided by the default S2I builder for deploying a WSGI application using the ``gunicorn`` WSGI server. The requirements which need to be satisfied for this to work are:

* The WSGI application code file needs to be named ``wsgi.py``.
* The WSGI application entry point within the code file needs to be named ``application``.
* The ``gunicorn`` package must be listed in the ``requirements.txt`` file for ``pip``.

In addition, the ``.s2i/environment`` file has been created to allow environment variables to be set to override the behaviour of the default S2I builder for Python.

* The environment variable ``APP_CONFIG`` has been set to declare the name of the config file for ``gunicorn``.

## Deployment Steps

The deployment was test using *Red Hat CodeReady Containers*:

* [Introducing Red Hat CodeReady Containers](https://code-ready.github.io/crc/);
* [Red Hat OpenShift 4 on your laptop: Introducing Red Hat CodeReady Containers](https://developers.redhat.com/blog/2019/09/05/red-hat-openshift-4-on-your-laptop-introducing-red-hat-codeready-containers/);
* [Red Hat CodeReady Containers / Install OpenShift on your laptop](https://developers.redhat.com/products/codeready-containers/overview);

To obtain the default CRC to get the ``kubeadmin`` password, run ``crc console --credentials``.

```bash
$ oc login -u kubeadmin -p <password> https://api.crc.testing:6443
$ oc whoami                                                    # kubeadmin
$ oc new-project sample-flask-s2i                              # create OCP project
$ oc new-app https://github.com/sjfke/ocp-sample-flask-s2i.git # s2i deploy direct from git repo
$ oc expose service/ocp-sample-flask-s2i                       # make accessible outside OCP.
```

Once the application deployment is finished then it will be accessible as [ocp-sample-flask-s2i](http://ocp-sample-flask-s2i-sample-flask-s2i.apps-crc.testing).

### Output of ``oc new-app``

```
--> Found image 8ec6f0d (6 weeks old) in image stream "openshift/python" under tag "3.8-ubi8" for "python"

    Python 3.8 
    ---------- 
    Python 3.8 available as container is a base platform for building and running various Python 3.8 applications and frameworks. Python is an easy to learn, powerful programming language. It has efficient high-level data structures and a simple but effective approach to object-oriented programming. Python's elegant syntax and dynamic typing, together with its interpreted nature, make it an ideal language for scripting and rapid application development in many areas on most platforms.

    Tags: builder, python, python38, python-38, rh-python38

    * The source repository appears to match: python
    * A source build using source code from https://github.com/sjfke/ocp-sample-flask-s2i.git will be created
      * The resulting image will be pushed to image stream tag "ocp-sample-flask-s2i:latest"
      * Use 'oc start-build' to trigger a new build

--> Creating resources ...
    imagestream.image.openshift.io "ocp-sample-flask-s2i" created
    buildconfig.build.openshift.io "ocp-sample-flask-s2i" created
    deployment.apps "ocp-sample-flask-s2i" created
    service "ocp-sample-flask-s2i" created
--> Success
    Build scheduled, use 'oc logs -f buildconfig/ocp-sample-flask-s2i' to track its progress.
    Application is not exposed. You can expose services to the outside world by executing one or more of the commands below:
     'oc expose service/ocp-sample-flask-s2i' 
    Run 'oc status' to view your app.

```

### Output of ``oc status`` while deploying

```bash
$ oc status
In project sample-flask-s2i on server https://api.crc.testing:6443

svc/ocp-sample-flask-s2i - 10.217.5.126:8080
  deployment/ocp-sample-flask-s2i deploys istag/ocp-sample-flask-s2i:latest <-
    bc/ocp-sample-flask-s2i source builds https://github.com/sjfke/ocp-sample-flask-s2i.git on openshift/python:3.8-ubi8 
      build #1 pending for 20 seconds
    deployment #1 running for 20 seconds - 0/1 pods growing to 1


1 info identified, use 'oc status --suggest' to see details.

```

### Output of ``oc status`` after application is deployed.

```bash
$ oc status
In project sample-flask-s2i on server https://api.crc.testing:6443

http://ocp-sample-flask-s2i-sample-flask-s2i.apps-crc.testing to pod port 8080-tcp (svc/ocp-sample-flask-s2i)
  deployment/ocp-sample-flask-s2i deploys istag/ocp-sample-flask-s2i:latest <-
    bc/ocp-sample-flask-s2i source builds https://github.com/sjfke/ocp-sample-flask-s2i.git on openshift/python:3.8-ubi8 
    deployment #2 running for 10 minutes - 1 pod
    deployment #1 deployed 14 minutes ago


1 info identified, use 'oc status --suggest' to see details.

$ oc get routes
NAME                   HOST/PORT                                                PATH   SERVICES               PORT       TERMINATION   WILDCARD
ocp-sample-flask-s2i   ocp-sample-flask-s2i-sample-flask-s2i.apps-crc.testing          ocp-sample-flask-s2i   8080-tcp                 None
```
