# Flask Application deployed with S2I 

This repository provides a sample Python web application implemented using the Flask web framework and hosted using ``gunicorn``. It is intended to be used to demonstrate deployment of Python web applications to OpenShift 4 using [source-to-image](https://docs.openshift.com/enterprise/3.0/creating_images/s2i.html#creating-images-s2i).

This is derived from [Sample Python Flask application for testing OpenShift 3 - by Graham Dumpleton](https://github.com/OpenShiftDemos/os-sample-python).

## Implementation Notes

This sample Python application relies on the support provided by the default S2I builder for deploying a WSGI application using the ``gunicorn`` WSGI server. The requirements which need to be satisfied for this to work are:

* The WSGI application code file needs to be named ``wsgi.py``.
* The WSGI application entry point within the code file needs to be named ``application``.
* The ``gunicorn`` package must be listed in the ``requirements.txt`` file for ``pip``.

In addition, the ``.s2i/environment`` file has been created to allow environment variables to be set to override the behaviour of the default S2I builder for Python.

* The environment variable ``APP_CONFIG`` has been set to declare the name of the config file for ``gunicorn``.

The example is based on [Getting Started with Flask](https://scotch.io/tutorials/getting-started-with-flask-a-python-microframework) but has 
been modified to work [Green Unicorn - WSGI sever](https://docs.gunicorn.org/en/stable/) and the content of the web-site 
changed to provide some [Lorem Ipsum](https://en.wikipedia.org/wiki/Lorem_ipsum) pages from [Lorem IPsum Generators - The 14 Best](https://digital.com/lorem-ipsum-generators/), 
and `isalive` and `isready` probe pages have been added for Kubernetes.


## Deployment Steps

The deployment was tested using *Red Hat CodeReady Containers* (CRC) details of which can be found here:

* [Introducing Red Hat CodeReady Containers](https://code-ready.github.io/crc/);
* [Red Hat OpenShift 4 on your laptop: Introducing Red Hat CodeReady Containers](https://developers.redhat.com/blog/2019/09/05/red-hat-openshift-4-on-your-laptop-introducing-red-hat-codeready-containers/);
* [Red Hat CodeReady Containers / Install OpenShift on your laptop](https://developers.redhat.com/products/codeready-containers/overview);

To obtain the default CRC ``kubeadmin`` password, run ``crc console --credentials``.

```bash
$ oc login -u kubeadmin -p <password> https://api.crc.testing:6443
$ oc whoami               # kubeadmin
$ oc new-project work00   # create work00 OCP project
$ oc project              # check project is work00
Using project "work00" on server "https://api.crc.testing:6443".

$ oc new-app https://github.com/sjfke/ocp-sample-flask-s2i.git # s2i deploy direct from git repo

# Takes time to download and build, so monitor the build log
$ oc get pods
NAME                           READY   STATUS     RESTARTS   AGE
ocp-sample-flask-s2i-1-build   0/1     Init:0/2   0          52s

$ oc logs -f ocp-sample-flask-s2i-1-build # tail -f the build log
....
Writing manifest to image destination
Storing signatures
Successfully pushed image-registry.openshift-image-registry.svc:5000/work00/ocp-sample-flask-s2i@sha256:d64948a51e28587dbc8742033e50d1ff20943cd7bef9ebc137149a03334804fa
Push successful

$ oc get pods  # build has completed and the pod is running, NOTE: the 1/1 in the READY column
NAME                                    READY   STATUS      RESTARTS   AGE
ocp-sample-flask-s2i-1-build            0/1     Completed   0          10m
ocp-sample-flask-s2i-86d575b4f7-kgdqs   1/1     Running     0          4m38s

$ oc expose service/ocp-sample-flask-s2i  # make accessible outside OCP.
```

Once the application deployment is finished then it will be accessible as [ocp-sample-flask-s2i](http://ocp-sample-flask-s2i-work00.apps-crc.testing).

```bash
$ oc get all | egrep "HOST/PORT|route.route" # HOST/PORT column provides the URL
NAME                                            HOST/PORT                                      PATH   SERVICES               PORT       TERMINATION   WILDCARD
route.route.openshift.io/ocp-sample-flask-s2i   ocp-sample-flask-s2i-work00.apps-crc.testing          ocp-sample-flask-s2i   8080-tcp                 None

$ curl http://ocp-sample-flask-s2i-work00.apps-crc.testing
$ firefox http://ocp-sample-flask-s2i-work00.apps-crc.testing
```
Checking the pod from OpenShift command-line:

```bash
$ oc get pods
NAME                                       READY   STATUS    RESTARTS   AGE
ocp-sample-flask-docker-7f54d777d8-lxlpj   1/1     Running   0          3m32s

$ oc logs ocp-sample-flask-s2i-86d575b4f7-kgdqs         # get pod log
$ oc describe pod ocp-sample-flask-s2i-86d575b4f7-kgdqs # get pod description
$ oc rsh ocp-sample-flask-s2i-86d575b4f7-kgdqs          # login shell on pod
$ oc rsh ocp-sample-flask-s2i-86d575b4f7-kgdqs ps -ef   # run 'ps -ef' on pod, note 4x gunicorn/wsgi
UID          PID    PPID  C STIME TTY          TIME CMD
1000640+       1       0  0 15:36 ?        00:00:00 /opt/app-root/bin/python3.8 /opt/app-root/bin/gunicorn wsgi --bind=0.0.0.0:8080 --ac
1000640+      27       1  0 15:36 ?        00:00:00 /opt/app-root/bin/python3.8 /opt/app-root/bin/gunicorn wsgi --bind=0.0.0.0:8080 --ac
1000640+      28       1  0 15:36 ?        00:00:00 /opt/app-root/bin/python3.8 /opt/app-root/bin/gunicorn wsgi --bind=0.0.0.0:8080 --ac
1000640+      29       1  0 15:36 ?        00:00:00 /opt/app-root/bin/python3.8 /opt/app-root/bin/gunicorn wsgi --bind=0.0.0.0:8080 --ac
1000640+      65       0  0 16:00 pts/0    00:00:00 ps -ef
$
```

From the OpenShift Console WebUI:

* From *Builds* check the build log file, to see what is happening
* From *Services* link, you can access the Pod, and even open a terminal on the Pod.


## Undeployment Steps

```bash
$ oc get all --selector app=ocp-sample-flask-s2i     # list everything associated with the app
$ oc delete all --selector app=ocp-sample-flask-s2i  # delete everything associated with the app
$ oc delete project work00                           # delete the work00 project
```

## Example output from various commands

### Output of ``oc new-app https://github.com/sjfke/ocp-sample-flask-s2i.git``

```bash
$ oc new-app https://github.com/sjfke/ocp-sample-flask-s2i.git
--> Found image 7c90c99 (3 months old) in image stream "openshift/python" under tag "3.8-ubi8" for "python"

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
In project work00 on server https://api.crc.testing:6443

svc/ocp-sample-flask-s2i - 10.217.5.6:8080
  deployment/ocp-sample-flask-s2i deploys istag/ocp-sample-flask-s2i:latest <-
    bc/ocp-sample-flask-s2i source builds https://github.com/sjfke/ocp-sample-flask-s2i.git on openshift/python:3.8-ubi8 
      build #1 running for 7 seconds - 406fa64: Merge pull request #2 from sjfke/dev (Geoff Collis <34105187+sjfke@users.noreply.github.com>)
    deployment #1 running for 7 seconds - 0/1 pods growing to 1


1 info identified, use 'oc status --suggest' to see details.
```

### Output of ``oc status`` after application is deployed.

```bash
$ oc status
In project work00 on server https://api.crc.testing:6443

http://ocp-sample-flask-s2i-work00.apps-crc.testing to pod port 8080-tcp (svc/ocp-sample-flask-s2i)
  deployment/ocp-sample-flask-s2i deploys istag/ocp-sample-flask-s2i:latest <-
    bc/ocp-sample-flask-s2i source builds https://github.com/sjfke/ocp-sample-flask-s2i.git on openshift/python:3.8-ubi8 
    deployment #2 running for 3 minutes - 1 pod
    deployment #1 deployed 4 minutes ago


1 info identified, use 'oc status --suggest' to see details.
```

### Output of ``oc get routes`` after ``oc expose service/ocp-sample-flask-s2i``

```bash
$ oc get routes
NAME                   HOST/PORT                                      PATH   SERVICES               PORT       TERMINATION   WILDCARD
ocp-sample-flask-s2i   ocp-sample-flask-s2i-work00.apps-crc.testing          ocp-sample-flask-s2i   8080-tcp                 None
```

### Output of ``oc get all --selector app=ocp-sample-flask-s2i``

``` bash
$ oc get routes
NAME                   HOST/PORT                                      PATH   SERVICES               PORT       TERMINATION   WILDCARD
ocp-sample-flask-s2i   ocp-sample-flask-s2i-work00.apps-crc.testing          ocp-sample-flask-s2i   8080-tcp                 None
[gcollis@morpheus ocp-sample-flask-s2i]$ oc get all --selector app=ocp-sample-flask-s2i
NAME                           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
service/ocp-sample-flask-s2i   ClusterIP   10.217.5.6   <none>        8080/TCP   6m45s

NAME                                   READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/ocp-sample-flask-s2i   1/1     1            1           6m45s

NAME                                                  TYPE     FROM   LATEST
buildconfig.build.openshift.io/ocp-sample-flask-s2i   Source   Git    1

NAME                                              TYPE     FROM          STATUS     STARTED         DURATION
build.build.openshift.io/ocp-sample-flask-s2i-1   Source   Git@406fa64   Complete   6 minutes ago   58s

NAME                                                  IMAGE REPOSITORY                                                                      TAGS     UPDATED
imagestream.image.openshift.io/ocp-sample-flask-s2i   default-route-openshift-image-registry.apps-crc.testing/work00/ocp-sample-flask-s2i   latest   5 minutes ago

NAME                                            HOST/PORT                                      PATH   SERVICES               PORT       TERMINATION   WILDCARD
route.route.openshift.io/ocp-sample-flask-s2i   ocp-sample-flask-s2i-work00.apps-crc.testing          ocp-sample-flask-s2i   8080-tcp                 None
```
