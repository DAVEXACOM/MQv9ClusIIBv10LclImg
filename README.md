# Repository under construction

This repository is a work in progress following on from the work in IIB-MQ. The plan is to use this reposity to deliver a HELM release into IBM Cloud Private Kubernetes service via Micro Services Builder such that one container is running IIB with Maven installed MQ Client and the other container is running the MQ Queue Manager. We will explore replicas and persistent volumes for load balancing and recovery

# Origin

This repository is a merge of the original ot4i/iibdocker and ibm-messaging/mq-docker repositories on github

# Overview

This repository contains a Dockerfile and some scripts which demonstrate a way in which you might run [IBM Integration Bus](http://www-03.ibm.com/software/products/en/ibm-integration-bus) and IBM MQ in a [Docker](https://www.docker.com/whatisdocker/) container.

IBM would [welcome feedback](#issues-and-contributions) on what is offered here.

For a Foundational approach to messaging and integration on IBM Cloud Private we have looked at a "traditional ESB" on ICP:
		A 4-way MQ v9.0.n Cluster
		2 Gateway Queue Managers each an MQ cluster repository https://github.com/DAVEXACOM/MQv9ClusGatewayImg
		2 "backend" Queue Managers each hosting a locally bound IIB v10.0.0.n node. https://github.com/DAVEXACOM/MQv9ClusIIBv10LclImg
		All the queues are set up to drop messages into the gateways, have them cross the cluster, be serviced by IIB nodes and returned across the cluster to the gateways.
		The Foundational approach leverages the K8s DNS service and it's naming conventions in order the cluster channels in the MQSC files can resolve.
		The Helm release is coupled to the MQ Gateway Img build Github repository (the image is built and then the helm release driven) and leverages and approach of chart folders with chart folders inside 		chart folders. Helm works its way down.
		The Helm release coupled to the MQ Gateway Img build relies on the MQ Clus IIB Local image (build only, no release or deploy) have been already run and it's image being on the ICP. 

For ICP 3.1.1. With Microclimate you have to customize the jenkins build scripts. The process is documented in the following link.

https://github.com/cloudnativedemo/icp-notes/blob/master/microclimate_notes.md

# Building the image

The image can be built using standard [Docker commands](https://docs.docker.com/userguide/dockerimages/) against the supplied Dockerfile.  For example:

~~~
cd 10.0.0.7
docker build -t iibv10image .
~~~

This will create an image called iibv10image in your local docker registry.

# What the image contains

The built image contains a full installation of [IBM Integration Bus for Developers Edition V10.0](https://ibm.biz/iibdevedn).  

# Running a container

After building a Docker image from the supplied files, you can [run a container](https://docs.docker.com/userguide/usingdocker/) which will create and start an Integration Node to which you can [deploy](http://www-01.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/af03890_.htm) integration solutions.

In order to run a container from this image, it is necessary to accept the terms of the IBM Integration Bus for Developers license.  This is achieved by specifying the environment variable `LICENSE` equal to `accept` when running the image.  You can also view the license terms by setting this variable to `view`. Failure to set the variable will result in the termination of the container with a usage statement.  You can view the license in a different language by also setting the `LANG` environment variable.

In addition to accepting the license, you can optionally specify an Integration Node name using the `NODENAME` environment variable.

The last important point of configuration when running a container from this image, is port mapping.  The Dockerfile exposes ports `4414` and `7800` for IIB and `1414` and `9883` for MQ by default, for Integration Node administration and Integration Server HTTP traffic respectively.  This means you can run with the `-P` flag to auto map these ports to ports on your host.  Alternatively you can use `-p` to expose and map any ports of your choice.

For example:

~~~
docker run --name myNode -e LICENSE=accept -e NODENAME=MYNODE -P iibv10image -e MQ_QMGR_NAME=MQ1
~~~

This will run a container that creates and starts an Integration Node called `MYNODE` and exposes ports `4414` and `7800` on random ports on the host machine. It also creates a queue manager called `MQ1` and starts this queue manager with some default queues defined.
For more information on configuring MQ please see [README.md](https://github.com/ibm-messaging/mq-docker/blob/master/README.md) for the standalone MQ container.

At this point you can use:
~~~
docker port <container name>
~~~

to see which ports have been mapped then connect to the Node's web user interface as normal (see [verification](# Verifying your container is running correctly) section below).

### Running administration commands

You can run any of the Integration Bus
 commands using one of two methods:

##### Directly in the container

Attach a bash session to your container and execute your commands as you would normally:

~~~
docker exec -it <container name> /bin/bash
~~~

At this point you will be in a shell inside the container and can source `mqsiprofile` and run your commands.

##### Using Docker exec

Use Docker exec to run a non-interactive Bash session that runs any of the Integration Bus commands.  For example:

~~~
docker exec <container name> /bin/bash -c mqsilist
~~~

### Accessing logs

This image also configures syslog, so when you run a container, your node will be outputting messages to /var/log/syslog inside the container.  You can access this by attaching a bash session as described above or by using docker exec.  For example:

~~~
docker exec <container id> tail -f /var/log/syslog
~~~

# Verifying your container is running correctly

Whether you are using the image as provided or if you have customised it, here are a few basic steps that will give you confidence your image has been created properly:

1. Run a container, making sure to expose port 4414, 1414 and 9443 to the host - the container should start without error
2. Run mqsilist to show the status of your node as described above - your node should be listed as running
3. Access syslog as descried above - there should be no errors
4. Connect a browser to your host on the port you exposed in step 1 - the Integration Bus web user interface should be displayed.
5. Connect to a browser and connect via HTTPS to port 9443 to run the MQ administration console.

At this point, your container is running and you can [deploy](http://www-01.ibm.com/support/knowledgecenter/SSMKHH_10.0.0/com.ibm.etools.mft.doc/af03890_.htm) integration solutions to it using any of the supported methods.



# License

The Dockerfile and associated scripts are licensed under the [Eclipse Public License 1.0](./LICENSE). IBM Integration Bus for Developers is licensed under the IBM International License Agreement for Non-Warranted Programs. This license may be viewed from the image using the `LICENSE=view` environment variable as described above. Note that this license does not permit further distribution.


IBM MQ Advanced for Developers is licensed under the IBM International License Agreement for Non-Warranted Programs. This license may be viewed from the image using the LICENSE=view environment variable as described above or may be found online. Note that this license does not permit further distribution.
