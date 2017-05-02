Lab Guide- Creating your own S2I Builder


Creating your own S2I Builder

In this lab you will learn to create S2I (Source to Image) builder images using RHEL base image. This is a simple example that creates a Lighttpd S2I image to run on OpenShift. You can follow a similar approach for other technologies.
The code used created in this lab is available on github here
https://github.com/RedHatWorkshops/lighttpd-rhel
This document can also be accessed from the browser by using this bit.ly link
http://bit.ly/s2i-lab 
Assumptions

You know OpenShift and you should have used OpenShift S2I to deploy your applications. You are here to learn how to create S2I builders.
You should know to use git apis, basic Linux skills, and using vi editor.
Prerequisites

This lab requires a RHEL box as we will be using RHEL base image in this use case. In order to build using a RHEL base image you will need a RHEL box; for other base images, it is not required. In this lab we will be using RHEL VM by clicking on the “View s2i” icon on the desktop.
This rhel box should be subscribed to Red Hat Network. Hence you are given a RHEL VM that is pre-subscribed with the Red Hat Network. If you want to repeat this lab at home, please check on developer subscriptions.
Make sure that docker is installed and is running on this RHEL box (yum install docker -y). It is already installed and running for you.
In order to use this image, you will need an OpenShift cluster. It can be as simple as `oc cluster up`. OpenShift is already running on the machine given to you.
S2I tools are available to download for the latest linux version from https://github.com/openshift/source-to-image/releases/.  We already downloaded the S2I tools for you and they are in /usr/local/bin
Using S2I Tools to create S2I Builder

This section takes you step-by-step to create the lighttpd S2I builder image.
 
Step 1: Verify the S2I tool and Local OpenShift

Note: You are running the rest of the steps from the VM that was opened by clicking on the  “View s2i” icon on your desktop. This documentation can be opened in that VM by opening http://bit.ly/s2i-lab  in the browser.
Verify that the s2i tool works on the box by running
$ s2i
Source-to-image (S2I) is a tool for building repeatable docker images.

A command line interface that injects and assembles source code into a docker image.
Complete documentation is available at http://github.com/openshift/source-to-image

Usage:
 s2i [flags]
 s2i [command]

Available Commands:
 version           Display version
 build             Build a new image
 rebuild           Rebuild an existing image
 usage             Print usage of the assemble script associated with the image
 create            Bootstrap a new S2I image repository
 genbashcompletion Generate Bash completion for the s2i command

Flags:
     --ca="": Set the path of the docker TLS ca file
     --cert="": Set the path of the docker TLS certificate file
 -h, --help[=false]: help for s2i
     --key="": Set the path of the docker TLS key file
     --loglevel=0: Set the level of log output (0-5)
 -U, --url="unix:///var/run/docker.sock": Set the url of the docker socket to use

Use "s2i [command] --help" for more information about a command.
The machine provided to you is running a local OpenShift cluster. To verify OpenShift is running, open Firefox browser by clicking on on the top menu Applications->Firefox Web Browser

The browser has been configured to log into OpenShift running locally by default. If not try accessing https://ocp.127.0.0.1.nip.io:8443/console from the browser.

You will see a warning that the Connection is not secure. Add an Exception by clicking on the Add Exception button on the browser and on the next page press the button  Confirm Security Exception


Now it should take you to the OpenShift Web Console to Login.
The username to login is developer and the password is developer

Once logged in you will find a project named myproject already created for you. We will be using this project a little later.

Step 2 Create the S2I Builder Working Structure

S2I tools provide the following syntax to create a default working structure for you. You can edit the files to create the S2I builder image from this structure.
s2i create <future-builder-image-name> <builder-image-src-folder>
Run the following command to create a directory named s2i-lighttpd where the artifacts for the S2I builder will be added. The target S2I builder image name will be lighttpd-rhel
$ s2i create lighttpd-rhel s2i-lighttpd
Run ls s2i-lighttpd and observe the following folder structure
Dockerfile – This is a standard Dockerfile where we’ll define the builder image
Makefile – a helper script for building and testing the builder image
test/
run – test script to test if the builder image works correctly
test-app/ – directory for your test application
s2i/bin/
assemble – script responsible for building the application
run – script responsible for running the application
save-artifacts – script responsible for incremental builds
usage – script responsible for printing the usage of the builder image

Step 3 Update Assemble and Run Scripts

Lighttpd is a httpd server. The only thing we need to do during assembly is to copy the source files into a directory from which lighttpd will serve.
By default , the S2I builder will copy the source code into /tmp/src. If you want, this location can be changed by setting the io.openshift.s2i.destination label or passing --destination flag, in which case the sources will be placed in the src subdirectory of the directory you specified.
In the assemble code below we will copy the source from this /tmp/src to the working directory ,which is /opt/app-root/src. You will see that when we create the container image.
But s2i is meant to do more than that. As an example, if you were using a language such as java, you can compile your code here to produce a binary. Just to give a taste of some additional magic, let us make our lighttpd image a little smart. We will add some functionality to unzip a compressed file if your htmls are provided as a zip file.
Edit s2i/bin/assemble script to be as below:
#!/bin/bash -e
#
# S2I assemble script for the 'lighttpd-rhel' image.
# The 'assemble' script builds your application source so that it is ready to run.
#
# For more information refer to the documentation:
#   https://github.com/openshift/source-to-image/blob/master/docs/builder_image.md
#
if [[ "$1" == "-h" ]]; then
    # If the 'lighttpd-rhel' assemble script is executed with '-h' flag,
    # print the usage.
    exec /usr/libexec/s2i/usage
fi
# Restore artifacts from the previous build (if they exist).
#
if [ "$(ls /tmp/artifacts/ 2>/dev/null)" ]; then
  echo "---> Restoring build artifacts..."
  mv /tmp/artifacts/. ./
fi
echo "---> Installing application source..."
cp -Rf /tmp/src/. ./
echo "---> Building application from source..."
# Unzip if any compressed files
files=($(ls -1 /tmp/src))
for file in ${files[*]}
do
  if [ ${file##*.} = "zip" ]; then
        unzip ${file}
  fi
done

So the code above will copy the sources to the home directory, and if there are any zip files in the source code it will also unzip the same into home.
Now, change the s2i/bin/run script to start Lighttpd server as shown below:
#!/bin/bash -e
#
# S2I run script for the 'lighttpd-rhel' image.
# The run script executes the server that runs your application.
#
# For more information see the documentation:
#   https://github.com/openshift/source-to-image/blob/master/docs/builder_image.md
#

#exec <start your server here>
exec lighttpd -D -f /opt/app-root/etc/lighttpd.conf
Note The above exec command refers to a lighttpd.conf configuration file. We will create that in the next step.
Let’s understand a little bit about s2i/bin/save-artifacts although we won’t use it in this example. This script is run to copy any previously downloaded dependencies into the new build so that they are not re-downloaded again when you are using incremental builds. As an example, if we were running maven builds, if you want to save on downloading the dependencies each time the build is run, you can use incremental builds. If incremental builds are used, this save-artifacts script is invoked. Since it doesn’t apply here, let’s delete the same.
$ rm s2i/bin/save-artifacts
Next, let us change the permissions for all the script files by running
$ chmod 755 s2i/bin/*
Now the S2I scripts are ready.
Step 4 Add Lighttpd Configuration File

$ mkdir ./etc
Lighttpd requires a configuration file to run this server. Let us add this file s2i-lighttpd/etc/lighttpd.conf with minimal content for Lighttpd to run.
# directory where the documents will be served from
server.document-root = "/opt/app-root/src"

# port the server listens on
server.port = 8080

# default file if none is provided in the URL
index-file.names = ( "index.html" )

# configure specific mimetypes, otherwise application/octet-stream will be used for every file
mimetype.assign = (
 ".html" => "text/html",
 ".txt" => "text/plain",
 ".jpg" => "image/jpeg",
 ".png" => "image/png"
)
Note:
In this config file, we have set the server.document-root to /opt/app-root/src the same place where we copied the sources in the assemble script
While we are placing this configuration file in the etc directory, this file is copied to an appropriate location in the container image specs ( you will see that in the next step)
Step 5 Edit the Container Image Specification

While you can start your builder from any base image, In this example we will start with a RHEL base image, so that you understand how to create S2I builders from an operating system image. Modify the Dockerfile for the following changes:
We will use RHEL based builder image from Red Hat's registry. This is the just enough version of RHEL7 that you can use as base image for applications registry.access.redhat.com/rhel7-atomic
Add your name as the Maintainer
Add an environment variable to include LIGHTTPD version. The rest of the environment variables are to define where the S2I scripts are sourced from, the HOME directory on the container, and sets the PATH on the container.
Then add the k8s and openshift labels to describe your builder image
rhel7-atomic is a minimal image. So, we will add all the packages needed by your container. This image doesn’t include yum. We will use microdnf for installing packages and clean up the packages after the install. For the image to be efficient, we minimize the number of layers as well. In order to install Lighttpd, you will need to install epel. Hence both are included in the microdnf install
We will add a default user 1001, create the home directory and change the ownership to user 1001
Copy the Lighttpd configuration file from /etc into /opt/app-root/etc
Change the ownership of `/opt/app-root' to user 1001 and set the Docker USER to 1001
We will set the directory with sources as the working directory
The label we defined before defines the location for S2I scripts io.openshift.s2i.scripts-url=image:///usr/libexec/s2i. Copy the s2i scripts from the  s2i/bin to /usr/libexec/sti
Copy the lighttpd configuration file from etc to /opt/app-root/etc on the container
Run the container as user 1001
This container would expose port 8080
If the container is started by user point to usage for help
The resultant code is also shown below. Edit slowly and DOUBLE CHECK. :-)
# lighttpd-rhel
FROM registry.access.redhat.com/rhel7-atomic
# TODO: Put the maintainer name in the image metadata
MAINTAINER Veer Muchandi<veer@redhat.com>
# TODO: Rename the builder environment variable to inform users about application you provide them
# ENV BUILDER_VERSION 1.0
ENV \
LIGHTTPD_VERSION=1.4.35 \
STI_SCRIPTS_URL=image:///usr/libexec/s2i \
STI_SCRIPTS_PATH=/usr/libexec/s2i \
HOME=/opt/app-root/src \
PATH=/opt/app-root/src/bin:/opt/app-root/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
# TODO: Set labels used in OpenShift to describe the builder image
LABEL io.k8s.description="Platform for serving static HTML Pages" \
          io.k8s.display-name="Lighttpd 1.4.35" \
          io.openshift.expose-services="8080:http" \
          io.openshift.tags="builder,html,lighttpd" \
          io.openshift.s2i.scripts-url=image:///usr/libexec/s2i
# TODO: Install required packages here:
# RUN yum install -y ... && yum clean all -y
RUN microdnf install -y --enablerepo=rhel-7-server-rpms --enablerepo=rhel-7-server-extras-rpms --enablerepo=rhel-7-server-optional-rpms \
automake gettext git lsof make tar unzip wget which && \
rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && \
microdnf install -y --enablerepo=rhel-7-server-rpms --enablerepo=rhel-7-server-extras-rpms --enablerepo=rhel-7-server-optional-rpms lighttpd && \
microdnf clean all -y && \
mkdir -p /opt/app-root && \
useradd -u 1001 -r -g 0 -d ${HOME} -s /sbin/nologin \
 -c "Default Application User" default && \
chown -R 1001:0 /opt/app-root
# Directory with the sources is set as the working directory so all STI scripts
# can execute relative to this path.
WORKDIR ${HOME}
# Copy the S2I scripts to /usr/libexec/s2i
COPY ./s2i/bin/ /usr/libexec/s2i
# Copy the lighttpd configuration file
COPY ./etc/ /opt/app-root/etc
# Drop the root user and make the content of /opt/app-root owned by user 1001
RUN chown -R 1001:1001 /opt/app-root
# This default user is created in the openshift/base-centos7 image
USER 1001
# TODO: Set the default port for applications built using this image
EXPOSE 8080
# TODO: Set the default CMD for the image
CMD ["/usr/libexec/s2i/usage"]
Step 6 Build the S2I Builder Image

Change to s2i-lighttpd directory and run
$ sudo make build
This step will invoke "docker build" using the Dockerfile above to create a docker image with name lighttpd-rhel.
Upon successful execution of sudo make build run
$ docker images
to verify the existence of the lighttpd-rhel image on your rhel box.
Step 7 Testing the S2I Image Locally

Now it’s time to test our builder image. Let's create an index.html file in the s2i-lighttpd/test/test-app directory with the following contents:
<!doctype html>
<html>
 <head>
   <title>Hello World!</title>
 </head>
<body>
 <h1>Hello World from Lighttpd!</h1>
</body>
Note from the lighttpd.conf file above that index.html is the default
Now let us do the first build. Run the following command from the s2i-lighttpd directory:
$ sudo /usr/local/bin/s2i build test/test-app/ lighttpd-rhel sample-app
This invokes the S2I build process using the assemble script in the lighttpd-rhel image. The build creates an application image for the test application with name sample-app.
Once the image is created you can check the existence by running docker images again to find sample-app.
Now to test the sample-app image run it with docker
$ docker run -p 8080:8080 sample-app

and test it from the browser using http://localhost:8080. You should see a response.

Let us run the next test with a zip file (Remember our lighttpd-rhel s2i builder can also handle zip files!!).  Follow the steps in the code snippet below. We will create a new test directory by copying the previous test contents. We will edit the index.html file. Then create a zip file using the index.html and remove the index.html file. So at the end the contents of the test case is a zip file.
[student@ocp-lab s2i-lighttpd]$ cp -R test/test-app/ test/test-zip-app
[student@ocp-lab s2i-lighttpd]$ vi test/test-zip-app/index.html
[student@ocp-lab s2i-lighttpd]$ cat test/test-zip-app/index.html
<!doctype html>
<html>
    <head>
            <title>Hello World!</title>
    </head>
    <body>
            <h1>Hello from ZIP file test!</h1>
    </body>
</html>
[student@ocp-lab s2i-lighttpd]$ cd test/test-zip-app
[student@ocp-lab test-zip-app]$ zip test.zip index.html
  adding: index.html (deflated 27%)
[student@ocp-lab test-zip-app]$ rm index.html
[student@ocp-lab test-zip-app]$ cd ../..
[student@ocp-lab s2i-lighttpd]$ ls test/test-zip-app/
test.zip
Let us run a new build. BTW, if you see a warning like below, ignore it.
[student@ocp-lab s2i-lighttpd]$ sudo /usr/local/bin/s2i build test/test-zip-app/ lighttpd-rhel sample-zip-app
---> Installing application source...
---> Building application from source...
Archive:  test.zip
inflating: /tmp/src/index.html
warning: Failed to kill container "69e13d3b7e30934a071bd844e7eb716eb748eccfda0d8b5f86b65d867c0de554": Error response from daemon: {"message":"Cannot kill container 69e13d3b7e30934a071bd844e7eb716eb748eccfda0d8b5f86b65d867c0de554: Container 69e13d3b7e30934a071bd844e7eb716eb748eccfda0d8b5f86b65d867c0de554 is not running"}
Once the image is created you can check the existence by running docker images again to find sample-zip-app.
Now to test the sample-app image run it with docker and check back in the browser.
$ docker run -p 8080:8080 sample-zip-app

Step 8 Login to OpenShift registry

Now we are ready to test our S2I Builder on an OpenShift Cluster.
YOU DON’T HAVE TO EXECUTE THESE STEPS IN THIS LAB. THESE ARE FOR READING
Notes for the Cluster Administrator. Others can read as well
Pushing to an OpenShift Cluster requires the registry on the cluster to have been exposed (i.e registry service should have been exposed by a route) following the process listed here https://docs.openshift.com/container-platform/3.4/install_config/registry/securing_and_exposing_registry.html
Also you need a user that is given system:registry role to be able to login to this registry as described here https://docs.openshift.com/container-platform/3.3/install_config/registry/accessing_registry.html#access-user-prerequisites
As an example the cluster administrator should have run the following command to allow user developer to talk to registry.
oc adm policy add-role-to-user system:registry developer
Also the user pushing the images should be given image-builder role by the administrator to the project into which the image is being pushed. This is not available by default even if it is user's own project. As an example, the administrator should run the following command.
oc adm policy add-role-to-user system:image-builder developer -n myproject
For this lab, the openshift user developer has been provided access to push images to a project named  myproject.
Let us first login to openshift local cluster running at https://ocp.127.0.0.1.nip.io:8443 from your command line using username developer and password developer
$ oc login https://ocp.127.0.0.1.nip.io:8443
The server uses a certificate signed by an unknown authority.
You can bypass the certificate check, but any data you send to the server could be intercepted by others.
Use insecure connections? (y/n): y
Authentication required for https://ocp.127.0.0.1.nip.io:8443 (openshift)
Username: developer
Password:
Login successful.
You have one project on this server: "myproject"
Using project "myproject".
Welcome! See 'oc help' to get started.
Find the oauth token assigned at login by running the following command. We will use this same token to log into docker registry.
$ oc whoami -t
Make a note the oauth-token output by the command above. This token will serve as a password to log into the docker registry running on OpenShift.
Let us now  login to the registry running on OpenShift.
Your OpenShift local cluster has the registry service assigned an ip address of 172.30.1.1. Login to the docker registry using oauth-token from the last step, using username developer, and the registry url of 172.30.1.1:5000.  This command will log you into the registry.
$ docker login -u developer -p <oauth-token> 172.30.1.1:5000
Note: If you were using a secured registry, in order to connect to the registry, you will need certificate that is trusted by the registry. You would place the certificate in /etc/docker/certs.d/<registry-url> folder. For this lab, we are using a local OpenShift cluster and hence this step is not required.
Step 9 Create ImageStream

Let us tag the image we created earlier to push into the OpenShift Cluster.
$ docker tag lighttpd-rhel 172.30.1.1:5000/myproject/lighttpd-rhel
This command will add a new tag for the lighttpd-rhel image that we created earlier to be pushed into the openshift registry into namespace myproject.
We now need an imagestream where we will declare that this image is a builder image, so that we can use it from the web console via the catalog to build our application. Create a new json ImageStream file with the following content. Edit to make sure your image name matches with what you noted in the previous step. Let us name it as lighttpd-rhel-is.json. Also make sure that the namespace is pointing to your project name (myproject)
{
   "kind": "ImageStream",
   "apiVersion": "v1",
   "metadata": {
       "name": "lighttpd-rhel",
       "namespace": "myproject"
   },
   "spec": {
       "tags": [
           {
               "name": "latest",
               "annotations": {
                   "description": "Run HTML",
                   "iconClass": "icon-tomcat",
                   "tags": "builder,lighttpd"
               },
           "from": {
             "kind": "DockerImage",
             "name": "172.30.1.1:5000/myproject/lighttpd-rhel:latest"
           }
           }
       ]
   }
}
Note the tags section that uses the builder tag. This is required for OpenShift to recognize this imagestream as a builder image and to display on the catalog when you access from the web console.
Let us now add this imagestream to myproject on the OpenShift cluster.
$ oc create -f lighttpd-rhel-is.json -n myproject
Verify the image streams in your project by running the following command.
$ oc get is -n myproject
and you should see an image with name lighttpd-rhel. However, the tags should be empty since we did not push the docker image into the registry yet.
Step 10 Pushing image to Registry

Now push the docker image into the docker registry on your openshift cluster.
$ docker push 172.30.1.1:5000/myproject/lighttpd-rhel
Check the image stream in your project and you should see the lighthttpd-rhel image with appropriate tags as shown below:
[student@ocp-lab ~]$ oc get is lighttpd-rhel -o yaml
apiVersion: v1
kind: ImageStream
metadata:
 annotations:
   openshift.io/image.dockerRepositoryCheck: 2017-04-13T22:54:46Z
 creationTimestamp: 2017-04-13T22:54:46Z
 generation: 2
 name: lighttpd-rhel
 namespace: myproject
 resourceVersion: "5114"
 selfLink: /oapi/v1/namespaces/myproject/imagestreams/lighttpd-rhel
 uid: 31016623-209c-11e7-9372-0800277e3835
spec:
 tags:
 - annotations:
     description: Run HTML
     iconClass: icon-tomcat
     tags: builder,lighttpd
   from:
     kind: DockerImage
     name: 172.30.1.1:5000/myproject/lighttpd-rhel:latest
   generation: 2
   importPolicy: {}
   name: latest
status:
 dockerImageRepository: 172.30.1.1:5000/myproject/lighttpd-rhel
 tags:
 - items:
   - created: 2017-04-13T22:55:17Z
     dockerImageReference: 172.30.1.1:5000/myproject/lighttpd-rhel@sha256:f83d4c776d5e86a37fa083606d4d314ae6ddce26967dac1cd342660c0d8d011a
     generation: 2
     image: sha256:f83d4c776d5e86a37fa083606d4d314ae6ddce26967dac1cd342660c0d8d011a
   tag: latest
Step 11 Test deploying an application

Now it is time to test your lighttpd-rhel S2I image from the Web Console.
Login to the Web Console and select the myproject project
Select Add to Project Button
You can search for lighttpd-rhel image in the catalog.
Select this image, give a name and the following git-url to try out a simple webpage to deploy https://github.com/VeerMuchandi/simple
This should build and deploy the application.
Once tested, if you want to make this image available across the cluster, handover your imagestream file and the image to the cluster administrator. The administrator would have to repeat steps 7 thru 9 for the openshift namespace.
Summary

Congratulations!!, In this lab you have learned how to create, test and use an S2I Builder image of your own. Please try out the technologies of your choice with S2I.
Published by Google Drive–Report Abuse–Updated automatically every 5 minutes
