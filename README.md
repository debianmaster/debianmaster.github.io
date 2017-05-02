<div>

<span class="c22 c21"></span>

<a id="t.6629782ea3036bb634eef6e2d5d50ac6de33dcd0"></a><a id="t.27"></a>

<table class="c43">

<tbody>

<tr class="c15">

<td class="c40" colspan="1" rowspan="1">

<span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 98.56px; height: 50.50px;">![](images/image9.png)</span>

</td>

<td class="c33" colspan="1" rowspan="1">

<span class="c22 c21"></span>

</td>

<td class="c56" colspan="1" rowspan="1">

<span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 75.50px; height: 35.23px;">![](images/image10.png)</span>

</td>

</tr>

</tbody>

</table>

<span class="c22 c21">                                                                </span>

</div>

# <span class="c21 c29 c58">Creating your own S2I Builder</span>

<span class="c22 c21"></span>

<span class="c21 c19 c29">In this lab you will learn to create S2I (Source to Image) builder images using RHEL base image. This is a simple example that creates a Lighttpd S2I image to run on OpenShift. You can follow a similar approach for other technologies.</span>

<span class="c21 c19 c29">The code used created in this lab is available on github here</span>

<span class="c41 c52">[https://github.com/RedHatWorkshops/lighttpd-rhel](https://www.google.com/url?q=https://github.com/RedHatWorkshops/lighttpd-rhel&sa=D&ust=1493768895335000&usg=AFQjCNHLOYmhG-ZEW_6DfyPeLAdmvwH66g)</span>

<span class="c21 c19 c29">You can git clone this and copy the code from here if you wish.  Tips to do this</span>

1.  <span class="c21 c19 c29">Open a separate terminal window</span>
2.  <span class="c19">Create a directory and change to this directory by running</span>

<a id="t.00dc38c99817d87390d162b38da048bd8462c3c9"></a><a id="t.0"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c4">$ mkdir codefromgit</span>

<span class="c8">$ cd codefromgit</span>

</td>

</tr>

</tbody>

</table>

<span class="c21 c19 c29"></span>

1.  <span class="c19">Run git clone. This will create a lighttpd-rhel directory with your code inside.</span>

<a id="t.ebca4886e18678868d7517179271338fedee4b2a"></a><a id="t.1"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c4">$ git clone https://github.com/RedHatWorkshops/lighttpd-rhel</span>

</td>

</tr>

</tbody>

</table>

<span class="c21 c19 c29"></span>

<span class="c19">Keep this terminal open to copy and paste your code from.</span>

## <span class="c21 c47 c29">Assumptions</span>

1.  <span class="c22 c21">You know OpenShift and you should have used OpenShift S2I to deploy your applications. You are here to learn how to create S2I builders.</span>
2.  <span class="c22 c21">You should know to use git apis, basic Linux skills, and using vi editor.</span>

## <span class="c21 c29 c47">Prerequisites</span>

*   <span class="c21 c19 c29">This lab requires a RHEL box as we will be using RHEL base image in this use case. In order to build using a RHEL base image you will need a RHEL box; for other base images, it is not required.</span>
*   <span class="c19">This rhel box should be subscribed to Red Hat Network. Hence you are given a RHEL machine that is pre-subscribed with the Red Hat Network. If you want to repeat this lab at home, please check on</span> <span class="c41 c52">[developer subscriptions](https://www.google.com/url?q=https://www.redhat.com/en/about/press-releases/red-hat-expands-red-hat-developer-program-no-cost-red-hat-enterprise-linux-developer-subscription&sa=D&ust=1493768895344000&usg=AFQjCNGBxSvg_6HV69vh40S5Lcrz0GO9qQ)</span><span class="c21 c29 c39">.</span>
*   <span class="c19">Make sure that docker is installed and is running on this RHEL box (</span><span class="c5 c6">yum install docker -y</span><span class="c21 c19 c29">). It is already installed and running for you.</span>
*   <span class="c21 c19 c29">In order to use this image, you will need an OpenShift cluster. It can be as simple as `oc cluster up`. OpenShift is already running on the machine given to you.</span>
*   <span class="c19">S2I tools are available to download for the latest</span> <span class="c17">linux version from</span> <span class="c52 c53 c54">[https://github.com/openshift/source-to-image/releases/](https://www.google.com/url?q=https://github.com/RedHatWorkshops/openshiftv3-advanced-workshop/blob/master&sa=D&ust=1493768895347000&usg=AFQjCNG1Jrz5TQCvM4yZvKSU7kdGEipq_A)</span><span class="c19">.  We already downloaded the S2I tools for you and they are in</span> <span class="c5 c21 c6">/usr/local/bin</span>

<span class="c5 c21 c6"></span>

* * *

<span class="c5 c21 c6"></span>

## <span class="c21 c47 c29">Using S2I Tools to create S2I Builder</span>

<span class="c22 c21">This section takes you step-by-step to create the lighttpd S2I builder image.</span>

<span class="c22 c21"></span>

<span> </span><span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 702.52px; height: 294.50px;">![](images/image7.png)</span>

<span class="c22 c21"></span>

### <span class="c20">Step 1: Verify the S2I tool and Local OpenShift</span>

<span class="c17">Verify that the</span> <span class="c5">s2i</span><span class="c17"> tool works on the box by running</span>

<a id="t.5a223cd7f17cb77a0df07795ada5379aec9df9de"></a><a id="t.2"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ s2i  
</span><span class="c0">Source</span><span class="c2">-</span><span class="c8">to</span><span class="c2">-</span><span class="c8">image</span> <span class="c2">(</span><span class="c8">S2I</span><span class="c2">)</span><span class="c8"> </span><span class="c8 c10">is</span><span class="c8">a tool</span> <span class="c8 c10">for</span><span class="c8"> building repeatable docker images</span><span class="c2">.</span><span class="c8">A command line</span> <span class="c8 c10">interface</span><span class="c8">that injects</span> <span class="c8 c10">and</span><span class="c8">assembles source code</span> <span class="c8 c10">into</span><span class="c8"> a docker image</span><span class="c2">.</span><span class="c8">  
</span><span class="c0">Complete</span><span class="c8">documentation</span> <span class="c8 c10">is</span><span class="c8"> available at http</span><span class="c2">:</span><span class="c8 c27">//github.com/openshift/source-to-image</span><span class="c8">  

</span><span class="c0">Usage</span><span class="c2">:</span><span class="c8">s2i</span> <span class="c2">[</span><span class="c8">flags</span><span class="c2">]</span><span class="c8">s2i</span> <span class="c2">[</span><span class="c8">command</span><span class="c2">]</span><span class="c8">  

</span><span class="c0">Available</span><span class="c8"> </span><span class="c0">Commands</span><span class="c2">:</span><span class="c8">version</span> <span class="c0">Display</span><span class="c8">version  
 build</span> <span class="c0">Build</span><span class="c8">a</span> <span class="c8 c10">new</span><span class="c8">image  
 rebuild</span> <span class="c0">Rebuild</span><span class="c8">an existing image  
 usage</span> <span class="c0">Print</span><span class="c8">usage of the assemble script associated</span> <span class="c8 c10">with</span><span class="c8"> the image  
 create            </span><span class="c0">Bootstrap</span><span class="c8">a</span> <span class="c8 c10">new</span><span class="c8">S2I image repository  
 genbashcompletion</span> <span class="c0">Generate</span><span class="c8"> </span><span class="c0">Bash</span><span class="c8">completion</span> <span class="c8 c10">for</span><span class="c8"> the s2i command  

</span><span class="c0">Flags</span><span class="c2">:</span><span class="c8">  
     </span><span class="c2">--</span><span class="c8">ca</span><span class="c2">=</span><span class="c3">""</span><span class="c2">:</span><span class="c8"> </span><span class="c0">Set</span><span class="c8"> the path of the docker TLS ca file  
     </span><span class="c2">--</span><span class="c8">cert</span><span class="c2">=</span><span class="c3">""</span><span class="c2">:</span><span class="c8"> </span><span class="c0">Set</span><span class="c8"> the path of the docker TLS certificate file  
 </span><span class="c2">-</span><span class="c8">h</span><span class="c2">,</span><span class="c8"> </span><span class="c2">--</span><span class="c8">help</span><span class="c2">[=</span><span class="c8 c10">false</span><span class="c2">]:</span><span class="c8">help</span> <span class="c8 c10">for</span><span class="c8"> s2i  
     </span><span class="c2">--</span><span class="c8">key</span><span class="c2">=</span><span class="c3">""</span><span class="c2">:</span><span class="c8"> </span><span class="c0">Set</span><span class="c8"> the path of the docker TLS key file  
     </span><span class="c2">--</span><span class="c8">loglevel</span><span class="c2">=</span><span class="c8 c12">0</span><span class="c2">:</span><span class="c8"> </span><span class="c0">Set</span><span class="c8">the level of log output</span> <span class="c2">(</span><span class="c8 c12">0</span><span class="c2">-</span><span class="c8 c12">5</span><span class="c2">)</span><span class="c8">  
 </span><span class="c2">-</span><span class="c8">U</span><span class="c2">,</span><span class="c8"> </span><span class="c2">--</span><span class="c8">url</span><span class="c2">=</span><span class="c3">"unix:///var/run/docker.sock"</span><span class="c2">:</span><span class="c8"> </span><span class="c0">Set</span><span class="c8">the url of the docker socket to</span> <span class="c8 c10">use</span><span class="c8">  

</span><span class="c0">Use</span><span class="c8"> </span><span class="c3">"s2i [command] --help"</span><span class="c8"> </span><span class="c8 c10">for</span><span class="c8"> more information about a command.</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span>The machine provided to you is running a local OpenShift cluster. To verify OpenShift is running, open Firefox browser by clicking on on the top menu</span> <span class="c5 c21 c6">Applications->Firefox Web Browser</span>

<span style="overflow: hidden; display: inline-block; margin: -0.00px -0.00px; border: 2.67px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 455.92px; height: 82.50px;">![](images/image11.png)</span>

<span class="c22 c21"></span>

<span>The browser has been configured to log into OpenShift running locally by default. If not try accessing</span> <span class="c41">[https://ocp.127.0.0.1.nip.io:8443/console](https://www.google.com/url?q=https://ocp.127.0.0.1.nip.io:8443/console&sa=D&ust=1493768895389000&usg=AFQjCNHEyZbkCMF4Q03FDI6DOrnOREWVCw)</span><span class="c22 c21"> from the browser.</span>

<span style="overflow: hidden; display: inline-block; margin: -0.00px 0.00px; border: 2.67px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 566.50px; height: 275.08px;">![](images/image2.png)</span>

<span>You will see a warning that the</span> <span class="c5 c6">Connection is not secure</span><span>. Add an Exception by clicking on the</span> <span class="c5 c6">Add Exception</span><span> button on the browser and on the next page press the button  </span><span class="c5 c6">Confirm Security Exception</span>

<span style="overflow: hidden; display: inline-block; margin: -0.00px 0.00px; border: 2.67px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 624.00px; height: 262.67px;">![](images/image8.png)</span>

<span class="c22 c21"></span>

<span style="overflow: hidden; display: inline-block; margin: -0.00px -0.00px; border: 2.67px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 455.78px; height: 391.50px;">![](images/image1.png)</span>

<span class="c22 c21"></span>

<span class="c22 c21">Now it should take you to the OpenShift Web Console to Login.</span>

<span>The username to login is</span> <span class="c5 c6">developer</span><span>and the password is</span> <span class="c5 c6">developer</span>

<span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 0.00px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 624.00px; height: 316.00px;">![](images/image4.png)</span>

<span class="c22 c21"></span>

<span>Once logged in you will find a project named</span> <span class="c5 c6">myproject</span><span class="c22 c21"> already created for you. We will be using this project a little later.</span>

<span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 2.67px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 624.00px; height: 192.00px;">![](images/image5.png)</span>

### <span class="c20">Step 2 Create the S2I Builder Working Structure</span>

<span class="c22 c21">S2I tools provide the following syntax to create a default working structure for you. You can edit the files to create the S2I builder image from this structure.</span>

<span class="c22 c21"></span>

<span class="c5 c6">s2i create <future-builder-image-name> <builder-image-src-folder></span>

<span class="c21 c19 c29"></span>

<span class="c19">Run the following command to create a directory named</span> <span class="c5">s2i-lighttpd</span><span class="c19">where the artifacts for the S2I builder will be added. The target S2I builder image name will be</span> <span class="c5">lighttpd-rhel</span>

<a id="t.39f315d1d5994809a32e45667b21b61cfb69a3d6"></a><a id="t.3"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ s2i create</span> <span class="c8">lighttpd</span><span class="c2">-</span><span class="c8">rhel</span><span class="c8"> s2i</span><span class="c2">-</span><span class="c8">lighttpd</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">Run</span> <span class="c5">ls s2i-lighttpd</span><span class="c21 c19 c29"> and observe the following folder structure</span>

*   <span class="c5">Dockerfile</span><span class="c21 c19 c29"> – This is a standard Dockerfile where we’ll define the builder image</span>
*   <span class="c5">Makefile</span><span class="c21 c19 c29"> – a helper script for building and testing the builder image</span>
*   <span class="c5 c21">test/</span>

*   <span class="c5">run</span><span class="c21 c19 c29"> – test script to test if the builder image works correctly</span>
*   <span class="c5">test-app/</span><span class="c21 c19 c29"> – directory for your test application</span>

*   <span class="c5 c21">s2i/bin/</span>

*   <span class="c5">assemble</span><span class="c21 c19 c29"> – script responsible for building the application</span>
*   <span class="c5">run</span><span class="c21 c19 c29"> – script responsible for running the application</span>
*   <span class="c5">save-artifacts</span><span class="c21 c19 c29"> – script responsible for incremental builds</span>
*   <span class="c5">usage</span><span class="c21 c19 c29"> – script responsible for printing the usage of the builder image</span>

### <span class="c20"></span>

### <span>Step 3 Update Assemble and Run Scripts</span>

<span class="c21 c19 c29">Lighttpd is a httpd server. The only thing we need to do during assembly is to copy the source files into a directory from which lighttpd will serve.</span>

<span class="c19">By default , the S2I builder will copy the source code into</span> <span class="c5">/tmp/src</span><span class="c19">. If you want, this location can be changed by setting the</span> <span class="c5">io.openshift.s2i.destination</span><span class="c19">label or passing</span> <span class="c5">--destination</span><span class="c19">flag, in which case the sources will be placed in the</span> <span class="c5">src</span> <span class="c21 c19 c29">subdirectory of the directory you specified.</span>

<span class="c19">In the</span> <span class="c5">assemble</span><span class="c19">code below we will copy the source from this</span> <span class="c5">/tmp/src</span><span class="c19"> to the working directory ,which is</span><span class="c5"> /opt/app-root/src</span><span class="c19">. You will see that when we create the container image.</span>

<span class="c21 c19 c29">But s2i is meant to do more than that. As an example, if you were using a language such as java, you can compile your code here to produce a binary. Just to give a taste of some additional magic, let us make our lighttpd image a little smart. We will add some functionality to unzip a compressed file if your htmls are provided as a zip file.</span>

<span class="c19">Edit</span> <span class="c5">s2i/bin/assemble</span><span class="c19"> script to be as below:</span>

<a id="t.14608d3ebca07e5b8cef52e7e417179e01f0f52a"></a><a id="t.4"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c8 c27">#!/bin/bash -e</span>

<span class="c4">#</span>

<span class="c8 c27"># S2I assemble script for the 'lighttpd-rhel' image.</span>

<span class="c8 c27"># The 'assemble' script builds your application source so that it is ready to run.</span>

<span class="c4">#</span>

<span class="c8 c27"># For more information refer to the documentation:</span>

<span class="c8 c27">#   https://github.com/openshift/source-to-image/blob/master/docs/builder_image.md</span>

<span class="c4">#</span>

<span class="c4"></span>

<span class="c8 c10">if</span><span class="c8"> </span><span class="c2">[[</span><span class="c8"> </span><span class="c3">"$1"</span><span class="c8"> </span><span class="c2">==</span><span class="c8"> </span><span class="c3">"-h"</span><span class="c8"> </span><span class="c2">]];</span><span class="c8"> </span><span class="c8 c10">then</span>

<span class="c8"></span> <span class="c8 c27"># If the 'lighttpd-rhel' assemble script is executed with '-h' flag,</span>

<span class="c8"></span> <span class="c8 c27"># print the usage.</span>

<span class="c8"></span> <span class="c8 c10">exec</span><span class="c8"> </span><span class="c2">/</span><span class="c8">usr</span><span class="c2">/</span><span class="c8">libexec</span><span class="c2">/</span><span class="c8">s2i</span><span class="c2">/</span><span class="c4">usage</span>

<span class="c8 c10">fi</span>

<span class="c4"></span>

<span class="c8 c27"># Restore artifacts from the previous build (if they exist).</span>

<span class="c4">#</span>

<span class="c8 c10">if</span><span class="c8"> </span><span class="c2">[</span><span class="c8"> </span><span class="c3">"$(ls /tmp/artifacts/ 2>/dev/null)"</span><span class="c8"> </span><span class="c2">];</span><span class="c8"> </span><span class="c8 c10">then</span>

<span class="c8">echo</span> <span class="c3">"---> Restoring build artifacts..."</span>

<span class="c8">mv</span> <span class="c2">/</span><span class="c8">tmp</span><span class="c2">/</span><span class="c8">artifacts</span><span class="c2">/.</span><span class="c8"> </span><span class="c2">./</span>

<span class="c8 c10">fi</span>

<span class="c4"></span>

<span class="c8">echo</span> <span class="c3">"---> Installing application source..."</span>

<span class="c8">cp</span> <span class="c2">-</span><span class="c0">Rf</span><span class="c8"> </span><span class="c2">/</span><span class="c8">tmp</span><span class="c2">/</span><span class="c8">src</span><span class="c2">/.</span><span class="c8"> </span><span class="c2">./</span>

<span class="c4"></span>

<span class="c8">echo</span> <span class="c3">"---> Building application from source..."</span>

<span class="c8 c27"># Unzip if any compressed files</span>

<span class="c8">files</span><span class="c2">=(</span><span class="c8">$</span><span class="c2">(</span><span class="c8">ls</span> <span class="c2">-</span><span class="c8 c12">1</span><span class="c8"> </span><span class="c2">/</span><span class="c8">tmp</span><span class="c2">/</span><span class="c8">src</span><span class="c2">))</span>

<span class="c8 c10">for</span><span class="c8">file</span> <span class="c8 c10">in</span><span class="c8"> $</span><span class="c2">{</span><span class="c8">files</span><span class="c2">[*]}</span>

<span class="c8 c10">do</span>

<span class="c8"></span> <span class="c8 c10">if</span><span class="c8"> </span><span class="c2">[</span><span class="c8"> $</span><span class="c2">{</span><span class="c8">file</span><span class="c8 c27">##*.} = "zip" ]; then</span>

<span class="c8">        unzip $</span><span class="c2">{</span><span class="c8">file</span><span class="c2">}</span>

<span class="c8"></span> <span class="c8 c10">fi</span>

<span class="c8 c10">done</span>

</td>

</tr>

</tbody>

</table>

<span>  
So the code above will copy the sources to the home directory, and if there are any zip files in the source code it will also unzip the same into home.</span>

<span class="c19">Now, change the</span> <span class="c5">s2i/bin/run</span><span class="c19"> script to start Lighttpd server as shown below:</span>

<a id="t.e5ce48384364b2594bc1a96b45d00328a2d4785b"></a><a id="t.5"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c8 c27">#!/bin/bash -e</span><span class="c8">  
</span><span class="c8 c27">#</span><span class="c8">  
</span><span class="c8 c27"># S2I run script for the 'lighttpd-rhel' image.</span><span class="c8">  
</span><span class="c8 c27"># The run script executes the server that runs your application.</span><span class="c8">  
</span><span class="c8 c27">#</span><span class="c8">  
</span><span class="c8 c27"># For more information see the documentation:</span><span class="c8">  
</span><span class="c8 c27">#   https://github.com/openshift/source-to-image/blob/master/docs/builder_image.md</span><span class="c8">  
</span><span class="c8 c27">#</span><span class="c8">  

</span><span class="c8 c27">#exec <start your server here></span><span class="c8">  
</span><span class="c8 c10">exec</span><span class="c8">lighttpd</span> <span class="c2">-</span><span class="c8">D</span> <span class="c2">-</span><span class="c8">f</span> <span class="c2">/</span><span class="c8">opt</span><span class="c2">/</span><span class="c8">app</span><span class="c2">-</span><span class="c8">root</span><span class="c2">/</span><span class="c8">etc</span><span class="c2">/</span><span class="c8">lighttpd</span><span class="c2">.</span><span class="c4">conf</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">Note The above</span> <span class="c5">exec</span><span class="c19">command refers to a</span> <span class="c5">lighttpd.conf</span><span class="c21 c19 c29"> configuration file. We will create that in the next step.</span>

<span class="c19">Let’s understand a little bit about</span> <span class="c5">s2i/bin/save-artifacts</span><span class="c19">although we won’t use it in this example.</span> <span class="c19">This script is run to copy any previously downloaded dependencies into the new build so that they are not re-downloaded again when you are using incremental builds. As an example, if we were running maven builds, if you want to save on downloading the dependencies each time the build is run, you can use incremental builds. If incremental builds are used, this</span> <span class="c5">save-artifacts</span><span class="c19"> script is invoked. Since it doesn’t apply here, let’s delete the same.</span>

<a id="t.da93afd811f76c3598d76a3a7040b404abc82377"></a><a id="t.6"></a>

<table class="c13">

<tbody>

<tr class="c24">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ rm s2i/bin/save-artifacts</span>

</td>

</tr>

</tbody>

</table>

<span class="c8 c21 c10"></span>

<span class="c19">Next, let us change the permissions for all the script files by running</span>

<a id="t.a8bdb68b6dbf6b862dd64d7efcea91e9a4579732"></a><a id="t.7"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ chmod 755 s2i/bin/*</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c21 c19 c29">Now the S2I scripts are ready.</span>

### <span class="c20">Step 4 Add Lighttpd Configuration File</span>

<span class="c5 c21 c6"></span>

<a id="t.ca675117e19f0a6773b535430879c518815996d2"></a><a id="t.8"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ mkdir ./etc</span>

</td>

</tr>

</tbody>

</table>

<span class="c21 c19 c29"></span>

<span class="c19">Lighttpd requires a configuration file to run this server. Let us add this file</span> <span class="c5">s2i-lighttpd/</span><span class="c5">etc/lighttpd.conf</span><span class="c19"> with minimal content for Lighttpd to run.</span>

<a id="t.59417338c9b1f5e3131ce8f8174efdc4acffc5e2"></a><a id="t.9"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c8 c27"># directory where the documents will be served from</span><span class="c8">  
server</span><span class="c2">.</span><span class="c8">document</span><span class="c2">-</span><span class="c8">root</span> <span class="c2">=</span><span class="c8"> </span><span class="c3">"/opt/app-root/src"</span><span class="c8">  

</span><span class="c8 c27"># port the server listens on</span><span class="c8">  
server</span><span class="c2">.</span><span class="c8">port</span> <span class="c2">=</span><span class="c8"> </span><span class="c8 c12">8080</span><span class="c8">  

</span><span class="c8 c27"># default file if none is provided in the URL</span><span class="c8">  
index</span><span class="c2">-</span><span class="c8">file</span><span class="c2">.</span><span class="c8">names</span> <span class="c2">=</span><span class="c8"> </span><span class="c2">(</span><span class="c8"> </span><span class="c3">"index.html"</span><span class="c8"> </span><span class="c2">)</span><span class="c8">  

</span><span class="c8 c27"># configure specific mimetypes, otherwise application/octet-stream will be used for every file</span><span class="c8">  
mimetype</span><span class="c2">.</span><span class="c8">assign</span> <span class="c2">=</span><span class="c8"> </span><span class="c2">(</span><span class="c8">  
 </span><span class="c3">".html"</span><span class="c8"> </span><span class="c2">=></span><span class="c8"> </span><span class="c3">"text/html"</span><span class="c2">,</span><span class="c8">  
 </span><span class="c3">".txt"</span><span class="c8"> </span><span class="c2">=></span><span class="c8"> </span><span class="c3">"text/plain"</span><span class="c2">,</span><span class="c8">  
 </span><span class="c3">".jpg"</span><span class="c8"> </span><span class="c2">=></span><span class="c8"> </span><span class="c3">"image/jpeg"</span><span class="c2">,</span><span class="c8">  
 </span><span class="c3">".png"</span><span class="c8"> </span><span class="c2">=></span><span class="c8"> </span><span class="c3">"image/png"</span><span class="c4">  
)</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c14">Note</span><span class="c21 c19 c29">:</span>

*   <span class="c19">In this config file, we have set the</span> <span class="c5">server.document-root</span><span class="c19">to</span> <span class="c5">/opt/app-root/src</span><span class="c21 c19 c29"> the same place where we copied the sources in the assemble script</span>
*   <span class="c19">While we are placing this configuration file in the</span> <span class="c5">etc</span><span class="c19"> directory, this file is copied to an appropriate location in the container image specs ( you will see that in the next step)</span>

### <span class="c20">Step 5 Edit the Container Image Specification</span>

<span class="c19">While you can start your builder from any base image, In this example we will start with a RHEL base image, so that you understand how to create S2I builders from an operating system image.</span> <span class="c19">Modify the</span> <span class="c5">Dockerfile</span><span class="c21 c19 c29"> for the following changes:</span>

*   <span class="c19">We will use RHEL based builder image from Red Hat's registry. This is the just</span> <span class="c41 c52">[enough version of RHEL7](https://www.google.com/url?q=http://rhelblog.redhat.com/2017/03/13/introducing-the-red-hat-enterprise-linux-atomic-base-image/&sa=D&ust=1493768895684000&usg=AFQjCNHMgX48NxpNhewoQ49vgh4QlZDd_g)</span><span class="c19">that you can use as base image for applications</span> <span class="c5 c21">registry.access.redhat.com/rhel7-atomic</span>
*   <span class="c21 c19 c29">Add your name as the Maintainer</span>
*   <span class="c21 c19 c29">Add an environment variable to include LIGHTTPD version. The rest of the environment variables are to define where the S2I scripts are sourced from, the HOME directory on the container, and sets the PATH on the container.</span>
*   <span class="c21 c19 c29">Then add the k8s and openshift labels to describe your builder image</span>
*   <span class="c5">rhel7-atomic</span><span>is a minimal image. So, we will add all the packages needed by your container. This image doesn’t include</span> <span class="c5">yum</span><span>. We will use</span> <span class="c5">microdnf</span><span>for installing packages and clean up the packages after the install. For the image to be efficient, we minimize the number of layers as well.</span> <span class="c19">In order to install Lighttpd, you will need to install epel. Hence both are included in the</span> <span class="c5 c21">microdnf install</span>
*   <span>We will add a default user 1001, create the home directory and change the ownership to user 1001</span>
*   <span class="c19">Copy the Lighttpd configuration file from</span> <span class="c5">/etc</span><span class="c19">into</span> <span class="c5 c21">/opt/app-root/etc</span>
*   <span class="c21 c19 c29">Change the ownership of `/opt/app-root' to user 1001 and set the Docker USER to 1001</span>
*   <span class="c21 c19 c29">We will set the directory with sources as the working directory</span>
*   <span class="c19">The label we defined before defines the location for S2I scripts</span> <span class="c5">io.openshift.s2i.scripts-url=image:///usr/libexec/s2i.</span> <span class="c19">Copy the s2i scripts from the  </span><span class="c19">s2i/bin to</span> <span class="c5 c21">/usr/libexec/sti</span>
*   <span>Copy the lighttpd configuration file from</span> <span class="c5">etc</span><span>to</span> <span class="c5">/opt/app-root/etc</span> <span class="c21 c19 c29">on the container</span>
*   <span class="c19">Run the container as user 1001</span>
*   <span class="c21 c19 c29">This container would expose port 8080</span>
*   <span class="c19">If the container is started by user point to</span> <span class="c5">usage</span><span class="c21 c19 c29"> for help</span>

<span class="c19">The resultant code is also shown below. Edit slowly and DOUBLE CHECK. :-)</span>

<a id="t.23bbb2a62f3979da92b5ac9b785aa039e4d32ebe"></a><a id="t.10"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c8 c27"># lighttpd-rhel</span>

<span class="c8">FROM registry</span><span class="c2">.</span><span class="c8">access</span><span class="c2">.</span><span class="c8">redhat</span><span class="c2">.</span><span class="c8">com</span><span class="c2">/</span><span class="c8">rhel7</span><span class="c2">-</span><span class="c4">atomic</span>

<span class="c4"></span>

<span class="c8 c27"># TODO: Put the maintainer name in the image metadata</span>

<span class="c8">MAINTAINER</span> <span class="c0">Veer</span><span class="c8"> </span><span class="c0">Muchandi</span><span class="c2"><</span><span class="c8">veer@redhat</span><span class="c2">.</span><span class="c4">com></span>

<span class="c4"></span>

<span class="c8 c27"># TODO: Rename the builder environment variable to inform users about application you provide them</span>

<span class="c8 c27"># ENV BUILDER_VERSION 1.0</span>

<span class="c4">ENV \</span>

<span class="c8">LIGHTTPD_VERSION</span><span class="c2">=</span><span class="c8 c12">1.4</span><span class="c2">.</span><span class="c8 c12">35</span><span class="c4"> \</span>

<span class="c8">STI_SCRIPTS_URL</span><span class="c2">=</span><span class="c8">image</span><span class="c2">:</span><span class="c8 c27">///usr/libexec/s2i \</span>

<span class="c8">STI_SCRIPTS_PATH</span><span class="c2">=</span><span class="c3">/usr/</span><span class="c8">libexec</span><span class="c2">/</span><span class="c4">s2i \</span>

<span class="c8">HOME</span><span class="c2">=</span><span class="c3">/opt/</span><span class="c8">app</span><span class="c2">-</span><span class="c8">root</span><span class="c2">/</span><span class="c4">src \</span>

<span class="c8">PATH</span><span class="c2">=</span><span class="c3">/opt/</span><span class="c8">app</span><span class="c2">-</span><span class="c8">root</span><span class="c2">/</span><span class="c8">src</span><span class="c2">/</span><span class="c8">bin</span><span class="c2">:</span><span class="c3">/opt/</span><span class="c8">app</span><span class="c2">-</span><span class="c8">root</span><span class="c2">/</span><span class="c8">bin</span><span class="c2">:</span><span class="c3">/usr/</span><span class="c8 c10">local</span><span class="c2">/</span><span class="c8">sbin</span><span class="c2">:</span><span class="c3">/usr/</span><span class="c8 c10">local</span><span class="c2">/</span><span class="c8">bin</span><span class="c2">:</span><span class="c3">/usr/</span><span class="c8">sbin</span><span class="c2">:</span><span class="c3">/usr/</span><span class="c8">bin</span><span class="c2">:</span><span class="c3">/sbin:/</span><span class="c4">bin</span>

<span class="c4"></span>

<span class="c8 c27"># TODO: Set labels used in OpenShift to describe the builder image</span>

<span class="c8">LABEL io</span><span class="c2">.</span><span class="c8">k8s</span><span class="c2">.</span><span class="c8">description</span><span class="c2">=</span><span class="c3">"Platform for serving static HTML Pages"</span><span class="c4"> \</span>

<span class="c8">          io</span><span class="c2">.</span><span class="c8">k8s</span><span class="c2">.</span><span class="c8">display</span><span class="c2">-</span><span class="c8">name</span><span class="c2">=</span><span class="c3">"Lighttpd 1.4.35"</span><span class="c4"> \</span>

<span class="c8">          io</span><span class="c2">.</span><span class="c8">openshift</span><span class="c2">.</span><span class="c8">expose</span><span class="c2">-</span><span class="c8">services</span><span class="c2">=</span><span class="c3">"8080:http"</span><span class="c4"> \</span>

<span class="c8">          io</span><span class="c2">.</span><span class="c8">openshift</span><span class="c2">.</span><span class="c8">tags</span><span class="c2">=</span><span class="c3">"builder,html,lighttpd"</span><span class="c4"> \</span>

<span class="c8">          io</span><span class="c2">.</span><span class="c8">openshift</span><span class="c2">.</span><span class="c8">s2i</span><span class="c2">.</span><span class="c8">scripts</span><span class="c2">-</span><span class="c8">url</span><span class="c2">=</span><span class="c8">image</span><span class="c2">:</span><span class="c8 c27">///usr/libexec/s2i</span>

<span class="c4"></span>

<span class="c8 c27"># TODO: Install required packages here:</span>

<span class="c8 c27"># RUN yum install -y ... && yum clean all -y</span>

<span class="c8">RUN microdnf install</span> <span class="c2">-</span><span class="c8">y</span> <span class="c2">--</span><span class="c8">enablerepo</span><span class="c2">=</span><span class="c8">rhel</span><span class="c2">-</span><span class="c8 c12">7</span><span class="c2">-</span><span class="c8">server</span><span class="c2">-</span><span class="c8">rpms</span> <span class="c2">--</span><span class="c8">enablerepo</span><span class="c2">=</span><span class="c8">rhel</span><span class="c2">-</span><span class="c8 c12">7</span><span class="c2">-</span><span class="c8">server</span><span class="c2">-</span><span class="c8">extras</span><span class="c2">-</span><span class="c8">rpms</span> <span class="c2">--</span><span class="c8">enablerepo</span><span class="c2">=</span><span class="c8">rhel</span><span class="c2">-</span><span class="c8 c12">7</span><span class="c2">-</span><span class="c8">server</span><span class="c2">-</span><span class="c8">optional</span><span class="c2">-</span><span class="c4">rpms \</span>

<span class="c8">automake gettext git lsof make tar unzip wget which</span> <span class="c2">&&</span><span class="c4"> \</span>

<span class="c8">rpm</span> <span class="c2">-</span><span class="c0">Uvh</span><span class="c8"> https</span><span class="c2">:</span><span class="c8 c27">//dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm && \</span>

<span class="c8">microdnf install</span> <span class="c2">-</span><span class="c8">y</span> <span class="c2">--</span><span class="c8">enablerepo</span><span class="c2">=</span><span class="c8">rhel</span><span class="c2">-</span><span class="c8 c12">7</span><span class="c2">-</span><span class="c8">server</span><span class="c2">-</span><span class="c8">rpms</span> <span class="c2">--</span><span class="c8">enablerepo</span><span class="c2">=</span><span class="c8">rhel</span><span class="c2">-</span><span class="c8 c12">7</span><span class="c2">-</span><span class="c8">server</span><span class="c2">-</span><span class="c8">extras</span><span class="c2">-</span><span class="c8">rpms</span> <span class="c2">--</span><span class="c8">enablerepo</span><span class="c2">=</span><span class="c8">rhel</span><span class="c2">-</span><span class="c8 c12">7</span><span class="c2">-</span><span class="c8">server</span><span class="c2">-</span><span class="c8">optional</span><span class="c2">-</span><span class="c8">rpms lighttpd</span> <span class="c2">&&</span><span class="c4"> \</span>

<span class="c8">microdnf clean all</span> <span class="c2">-</span><span class="c8">y</span> <span class="c2">&&</span><span class="c4"> \</span>

<span class="c8">mkdir</span> <span class="c2">-</span><span class="c8">p</span> <span class="c2">/</span><span class="c8">opt</span><span class="c2">/</span><span class="c8">app</span><span class="c2">-</span><span class="c8">root</span> <span class="c2">&&</span><span class="c4"> \</span>

<span class="c8">useradd</span> <span class="c2">-</span><span class="c8">u</span> <span class="c8 c12">1001</span><span class="c8"> </span><span class="c2">-</span><span class="c8">r</span> <span class="c2">-</span><span class="c8">g</span> <span class="c8 c12">0</span><span class="c8"> </span><span class="c2">-</span><span class="c8">d $</span><span class="c2">{</span><span class="c8">HOME</span><span class="c2">}</span><span class="c8"> </span><span class="c2">-</span><span class="c8">s</span> <span class="c2">/</span><span class="c8">sbin</span><span class="c2">/</span><span class="c4">nologin \</span>

<span class="c8"> </span><span class="c2">-</span><span class="c8">c</span> <span class="c3">"Default Application User"</span><span class="c8"> </span><span class="c8 c10">default</span><span class="c8"> </span><span class="c2">&&</span><span class="c4"> \</span>

<span class="c8">chown</span> <span class="c2">-</span><span class="c8">R</span> <span class="c8 c12">1001</span><span class="c2">:</span><span class="c8 c12">0</span><span class="c8"> </span><span class="c2">/</span><span class="c8">opt</span><span class="c2">/</span><span class="c8">app</span><span class="c2">-</span><span class="c4">root</span>

<span class="c4"></span>

<span class="c8 c27"># Directory with the sources is set as the working directory so all STI scripts</span>

<span class="c8 c27"># can execute relative to this path.</span>

<span class="c8">WORKDIR $</span><span class="c2">{</span><span class="c4">HOME}</span>

<span class="c4"></span>

<span class="c4"></span>

<span class="c8 c27"># Copy the S2I scripts to /usr/libexec/s2i</span>

<span class="c8">COPY</span> <span class="c2">./</span><span class="c8">s2i</span><span class="c2">/</span><span class="c8">bin</span><span class="c3">/ /</span><span class="c8">usr</span><span class="c2">/</span><span class="c8">libexec</span><span class="c2">/</span><span class="c4">s2i</span>

<span class="c4"></span>

<span class="c8 c27"># Copy the lighttpd configuration file</span>

<span class="c8">COPY</span> <span class="c2">./</span><span class="c8">etc</span><span class="c3">/ /</span><span class="c8">opt</span><span class="c2">/</span><span class="c8">app</span><span class="c2">-</span><span class="c8">root</span><span class="c2">/</span><span class="c4">etc</span>

<span class="c4"></span>

<span class="c8 c27"># Drop the root user and make the content of /opt/app-root owned by user 1001</span>

<span class="c8">RUN chown</span> <span class="c2">-</span><span class="c8">R</span> <span class="c8 c12">1001</span><span class="c2">:</span><span class="c8 c12">1001</span><span class="c8"> </span><span class="c2">/</span><span class="c8">opt</span><span class="c2">/</span><span class="c8">app</span><span class="c2">-</span><span class="c4">root</span>

<span class="c4"></span>

<span class="c8 c27"># This default user is created in the openshift/base-centos7 image</span>

<span class="c8">USER</span> <span class="c8 c12">1001</span>

<span class="c4"></span>

<span class="c8 c27"># TODO: Set the default port for applications built using this image</span>

<span class="c8">EXPOSE</span> <span class="c8 c12">8080</span>

<span class="c4"></span>

<span class="c8 c27"># TODO: Set the default CMD for the image</span>

<span class="c8">CMD</span> <span class="c2">[</span><span class="c3">"/usr/libexec/s2i/usage"]</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

### <span class="c20">Step 6 Build the S2I Builder Image</span>

<span class="c21 c19 c29"></span>

<span class="c19">Change to</span> <span class="c5">s2i-lighttpd</span><span class="c19"> directory and run</span>

<a id="t.fdc0769432855a1c7183a366e50445850d7b758b"></a><a id="t.11"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c4">$ sudo make build</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">This step will invoke "docker build" using the Dockerfile above to create a docker image with name</span> <span class="c5">lighttpd-rhel</span><span class="c21 c19 c29">.</span>

<span class="c19">Upon successful execution of</span> <span class="c5">sudo make build</span><span class="c19"> run</span>

<a id="t.ee0d35e3578482b3f3ccce319d8a205683289eff"></a><a id="t.12"></a>

<table class="c13">

<tbody>

<tr class="c15">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ docker images</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">to verify the existence of the</span> <span class="c5">lighttpd-rhel</span><span class="c21 c19 c29"> image on your rhel box.</span>

### <span>Step 7 Testing the S2I Image Locally</span>

<span class="c19">Now it’s time to test our builder image. Let's create an</span> <span class="c5 c6">index.html</span><span class="c19">file in the</span> <span class="c5 c6">s2i-lighttpd/test/test-app</span><span class="c19"> directory with the following contents:</span>

<a id="t.1c359c1893878df493e9714e9c6e9ce81b7e1147"></a><a id="t.13"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c0"><!doctype html></span><span class="c8">  
</span><span class="c8 c10"><html></span><span class="c8">  
 </span><span class="c8 c10"><head></span><span class="c8">  
   </span><span class="c8 c10"><title></span><span class="c8">Hello World!</span><span class="c8 c10"></title></span><span class="c8">  
 </span><span class="c8 c10"></head></span><span class="c8">  
</span><span class="c8 c10"><body></span><span class="c8">  
 </span><span class="c8 c10"><h1></span><span class="c8">Hello World from Lighttpd!</span><span class="c8 c10"></h1></span><span class="c8">  
</span><span class="c8 c10"></body></span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">Note from the</span> <span class="c5 c6">lighttpd.conf</span><span class="c19">file above that</span> <span class="c5 c6">index.html</span><span class="c21 c19 c29"> is the default</span>

<span class="c19">Now let us do the first build. Run the following command from the s2i-lighttpd directory:</span>

<a id="t.947fc576d2c659cdeeb505456cd6ce4a3519df77"></a><a id="t.14"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ sudo</span> <span class="c2">/</span><span class="c8">usr</span><span class="c2">/</span><span class="c8 c10">local</span><span class="c2">/</span><span class="c8">bin</span><span class="c2">/</span><span class="c8">s2i build test</span><span class="c2">/</span><span class="c8">test</span><span class="c2">-</span><span class="c8">app</span><span class="c2">/</span><span class="c8"> lighttpd</span><span class="c2">-</span><span class="c8">rhel sample</span><span class="c2">-</span><span class="c4">app</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">This invokes the S2I build process using the</span> <span class="c5">assemble</span><span class="c19">script in the lighttpd-rhel image. The build creates an application image for the test application with name</span> <span class="c5">sample-app</span><span class="c21 c19 c29">.</span>

<span class="c19">Once the image is created you can check the existence by running</span> <span class="c5">docker images</span><span class="c19">again to find</span> <span class="c5">sample-app</span><span class="c21 c19 c29">.</span>

<span class="c19">Now to test the</span> <span class="c5">sample-app</span><span class="c19"> image run it with docker</span>

<a id="t.04f1875d47462b41a7b323f0e6d432ab6d02bb1b"></a><a id="t.15"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ docker run</span> <span class="c2">-</span><span class="c8">p</span> <span class="c8 c12">8080</span><span class="c2">:</span><span class="c8 c12">8080</span><span class="c8"> sample</span><span class="c2">-</span><span class="c4">app</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c6">  
</span><span class="c19">and test it from the browser using</span> <span class="c41 c52">[http://localhost:8080](https://www.google.com/url?q=http://localhost:8080&sa=D&ust=1493768895920000&usg=AFQjCNHIqt4g1KaHo52CsRi9VIT4W7KVEg)</span><span class="c19">. You should see a response.</span>

<span style="overflow: hidden; display: inline-block; margin: 0.00px -0.00px; border: 2.67px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 496.50px; height: 134.47px;">![](images/image6.png)</span>

<span class="c22 c21"></span>

<span>Let us run the next test with a zip file (Remember our lighttpd-rhel s2i builder can also handle zip files!!).  Follow the steps in the code snippet below. We will create a new test directory by copying the previous test contents. We will edit the</span> <span class="c5">index.html</span><span>file. Then create a zip file using the</span> <span class="c5">index.html</span><span>and remove the</span> <span class="c5">index.html</span><span class="c22 c21"> file. So at the end the contents of the test case is a zip file.</span>

<span class="c22 c21"></span>

<a id="t.a4ba19eed9b032b6b4d522b84ffa1431fe39eb3f"></a><a id="t.16"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c2">[</span><span class="c8">student@ocp</span><span class="c2">-</span><span class="c8">lab s2i</span><span class="c2">-</span><span class="c8">lighttpd</span><span class="c2">]</span><span class="c8">$ cp</span> <span class="c2">-</span><span class="c8">R test</span><span class="c2">/</span><span class="c8">test</span><span class="c2">-</span><span class="c8">app</span><span class="c2">/</span><span class="c8"> test</span><span class="c2">/</span><span class="c8">test</span><span class="c2">-</span><span class="c8">zip</span><span class="c2">-</span><span class="c4">app</span>

<span class="c2">[</span><span class="c8">student@ocp</span><span class="c2">-</span><span class="c8">lab s2i</span><span class="c2">-</span><span class="c8">lighttpd</span><span class="c2">]</span><span class="c8">$ vi test</span><span class="c2">/</span><span class="c8">test</span><span class="c2">-</span><span class="c8">zip</span><span class="c2">-</span><span class="c8">app</span><span class="c2">/</span><span class="c8">index</span><span class="c2">.</span><span class="c4">html</span>

<span class="c2">[</span><span class="c8">student@ocp</span><span class="c2">-</span><span class="c8">lab s2i</span><span class="c2">-</span><span class="c8">lighttpd</span><span class="c2">]</span><span class="c8">$ cat test</span><span class="c2">/</span><span class="c8">test</span><span class="c2">-</span><span class="c8">zip</span><span class="c2">-</span><span class="c8">app</span><span class="c2">/</span><span class="c8">index</span><span class="c2">.</span><span class="c4">html</span>

<span class="c0"><!doctype html></span>

<span class="c8 c10"><html></span>

<span class="c8"></span> <span class="c8 c10"><head></span>

<span class="c8"></span> <span class="c8 c10"><title></span><span class="c8">Hello World!</span><span class="c8 c10"></title></span>

<span class="c8"></span> <span class="c8 c10"></head></span>

<span class="c4"></span>

<span class="c8"></span> <span class="c8 c10"><body></span>

<span class="c8"></span> <span class="c8 c10"><h1></span><span class="c8">Hello from ZIP file test!</span><span class="c8 c10"></h1></span>

<span class="c8"></span> <span class="c8 c10"></body></span>

<span class="c8 c10"></html></span>

<span class="c4"></span>

<span class="c4">[student@ocp-lab s2i-lighttpd]$ cd test/test-zip-app</span>

<span class="c4">[student@ocp-lab test-zip-app]$ zip test.zip index.html</span>

<span class="c4">  adding: index.html (deflated 27%)</span>

<span class="c4">[student@ocp-lab test-zip-app]$ rm index.html</span>

<span class="c8">[student@ocp-lab test-zip-app]$ cd ../..</span>

<span class="c2">[</span><span class="c8">student@ocp</span><span class="c2">-</span><span class="c8">lab s2i</span><span class="c2">-</span><span class="c8">lighttpd</span><span class="c2">]</span><span class="c8">$ ls test</span><span class="c2">/</span><span class="c8">test</span><span class="c2">-</span><span class="c8">zip</span><span class="c2">-</span><span class="c4">app/</span>

<span class="c8">test</span><span class="c2">.</span><span class="c4">zip</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c22 c21">Let us run a new build. BTW, if you see a warning like below, ignore it.</span>

<span class="c22 c21"></span>

<a id="t.e7143dbb8980004ed50c7c582a6739da65a3b5a2"></a><a id="t.17"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c2">[</span><span class="c8">student@ocp</span><span class="c2">-</span><span class="c8">lab s2i</span><span class="c2">-</span><span class="c8">lighttpd</span><span class="c2">]</span><span class="c8">$ sudo</span> <span class="c2">/</span><span class="c8">usr</span><span class="c2">/</span><span class="c8 c10">local</span><span class="c2">/</span><span class="c8">bin</span><span class="c2">/</span><span class="c8">s2i build test</span><span class="c2">/</span><span class="c8">test</span><span class="c2">-</span><span class="c8">zip</span><span class="c2">-</span><span class="c8">app</span><span class="c2">/</span><span class="c8"> lighttpd</span><span class="c2">-</span><span class="c8">rhel sample</span><span class="c2">-</span><span class="c8">zip</span><span class="c2">-</span><span class="c4">app</span>

<span class="c2">---></span><span class="c8"> </span><span class="c0">Installing</span><span class="c8"> application source</span><span class="c2">...</span>

<span class="c2">---></span><span class="c8"> </span><span class="c0">Building</span><span class="c8">application</span> <span class="c8 c10">from</span><span class="c8"> source</span><span class="c2">...</span>

<span class="c0">Archive</span><span class="c2">:</span><span class="c8">  test</span><span class="c2">.</span><span class="c4">zip</span>

<span class="c8">inflating</span><span class="c2">:</span><span class="c8"> </span><span class="c3">/tmp/</span><span class="c8">src</span><span class="c2">/</span><span class="c8">index</span><span class="c2">.</span><span class="c4">html</span>

<span class="c8">warning</span><span class="c2">:</span><span class="c8"> </span><span class="c0">Failed</span><span class="c8">to kill container</span> <span class="c3">"69e13d3b7e30934a071bd844e7eb716eb748eccfda0d8b5f86b65d867c0de554"</span><span class="c2">:</span><span class="c8"> </span><span class="c0">Error</span><span class="c8">response</span> <span class="c8 c10">from</span><span class="c8"> daemon</span><span class="c2">:</span><span class="c8"> </span><span class="c2">{</span><span class="c3">"message"</span><span class="c2">:</span><span class="c3">"Cannot kill container 69e13d3b7e30934a071bd844e7eb716eb748eccfda0d8b5f86b65d867c0de554: Container 69e13d3b7e30934a071bd844e7eb716eb748eccfda0d8b5f86b65d867c0de554 is not running"}</span>

</td>

</tr>

</tbody>

</table>

<span class="c4"></span>

<span class="c19">Once the image is created you can check the existence by running</span> <span class="c5">docker images</span><span class="c19">again to find</span> <span class="c5">sample-zip-app</span><span class="c21 c19 c29">.</span>

<span class="c19">Now to test the</span> <span class="c5">sample-app</span><span class="c19"> image run it with docker and check back in the browser.</span>

<a id="t.5572f500e1e1eaaba11855b34e640d2a6ad6d0f4"></a><a id="t.18"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ docker run</span> <span class="c2">-</span><span class="c8">p</span> <span class="c8 c12">8080</span><span class="c2">:</span><span class="c8 c12">8080</span><span class="c8"> sample-zip</span><span class="c2">-</span><span class="c4">app</span>

</td>

</tr>

</tbody>

</table>

<span style="overflow: hidden; display: inline-block; margin: 0.00px 0.00px; border: 2.67px solid #000000; transform: rotate(0.00rad) translateZ(0px); -webkit-transform: rotate(0.00rad) translateZ(0px); width: 584.50px; height: 142.38px;">![](images/image3.png)</span>

### <span class="c20">Step 8 Login to OpenShift registry</span>

<span class="c21 c19 c29"></span>

<span class="c21 c19 c29">Now we are ready to test our S2I Builder on an OpenShift Cluster.</span>

<span class="c22 c21"></span>

* * *

<span class="c22 c21"></span>

<span class="c21 c14 c50">Notes for the Cluster Administrator. Others can read as well</span>

<span class="c19">Pushing to an OpenShift Cluster requires the registry on the cluster to have been exposed (i.e registry service should have been exposed by a route) following the process listed here</span> <span class="c41 c52">[https://docs.openshift.com/container-platform/3.4/install_config/registry/securing_and_exposing_registry.html](https://www.google.com/url?q=https://docs.openshift.com/container-platform/3.4/install_config/registry/securing_and_exposing_registry.html&sa=D&ust=1493768896028000&usg=AFQjCNG_YU2h1cduz78C9RWc2Bu2Lq-ScA)</span>

<span class="c19">Also you need a user that is given</span> <span class="c5">system:registry</span><span class="c19">role to be able to login to this registry as described here</span> <span class="c29 c45">[https://docs.openshift.com/container-platform/3.3/install_config/registry/accessing_registry.html#access-user-prerequisites](https://www.google.com/url?q=https://github.com/RedHatWorkshops/openshiftv3-advanced-workshop/blob/master&sa=D&ust=1493768896031000&usg=AFQjCNG8GgplbzgPKJGA9prtv_ChIUTInA)</span>

<span class="c19">As an example the cluster administrator should have run the following command to allow user</span> <span class="c5 c6">developer</span><span class="c21 c19 c29"> to talk to registry.</span>

<span class="c5 c21 c6">oc adm policy add-role-to-user system:registry developer  
</span>

<span class="c19">Also the user pushing the images should be given</span> <span class="c5">image-builder</span><span class="c21 c19 c29"> role by the administrator to the project into which the image is being pushed. This is not available by default even if it is user's own project. As an example, the administrator should run the following command.</span>

<span class="c5 c21 c6">oc adm policy add-role-to-user system:image-builder developer -n myproject  
</span>

* * *

<span class="c22 c21"></span>

<span class="c5 c21 c6"></span>

<span>For this lab, the openshift user</span> <span class="c5 c6">developer</span><span> has been provided access to push images to a project named  </span><span class="c5 c6">myproject</span><span class="c22 c21">.</span>

<span class="c5 c21 c6"></span>

<span class="c19">Let us first login to openshift local cluster running at</span> <span class="c8 c41">[https://ocp.127.0.0.1.nip.io:8443](https://www.google.com/url?q=https://ocp.127.0.0.1.nip.io:8443&sa=D&ust=1493768896043000&usg=AFQjCNFKLFOcsNyHDn5MHO_4sVPkH1YI1A)</span><span class="c8 c27"> </span><span class="c19">from your command line using username</span> <span class="c5 c6">developer</span><span class="c19">and password</span> <span class="c5 c6">developer</span>

<a id="t.ed88827741417f9408c4043ba34a4ebb155e5596"></a><a id="t.19"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ oc login https</span><span class="c2">:</span><span class="c8 c27">//ocp.127.0.0.1.nip.io:8443</span>

<span class="c0">The</span><span class="c8">server uses a certificate</span> <span class="c8 c10">signed</span><span class="c8"> </span><span class="c8 c10">by</span><span class="c4"> an unknown authority.</span>

<span class="c0">You</span><span class="c8"> can bypass the certificate check</span><span class="c2">,</span><span class="c8">but any data you send to the server could be intercepted</span> <span class="c8 c10">by</span><span class="c4"> others.</span>

<span class="c0">Use</span><span class="c8"> insecure connections</span><span class="c2">?</span><span class="c8"> </span><span class="c2">(</span><span class="c8">y</span><span class="c2">/</span><span class="c8">n</span><span class="c2">):</span><span class="c4"> y</span>

<span class="c4"></span>

<span class="c0">Authentication</span><span class="c8">required</span> <span class="c8 c10">for</span><span class="c8"> https</span><span class="c2">:</span><span class="c8 c27">//ocp.127.0.0.1.nip.io:8443 (openshift)</span>

<span class="c0">Username</span><span class="c2">:</span><span class="c4"> developer</span>

<span class="c0">Password:</span>

<span class="c0">Login</span><span class="c4"> successful.</span>

<span class="c4"></span>

<span class="c0">You</span><span class="c8">have one project on</span> <span class="c8 c10">this</span><span class="c8"> server</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"myproject"</span>

<span class="c4"></span>

<span class="c0">Using</span><span class="c8">project</span> <span class="c3">"myproject".</span>

<span class="c0">Welcome</span><span class="c2">!</span><span class="c8"> </span><span class="c0">See</span><span class="c8"> </span><span class="c3">'oc help'</span><span class="c8">to</span> <span class="c8 c10">get</span><span class="c4"> started.</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">Find the oauth token assigned at login by running the following command. We will use this same token to log into docker registry.</span>

<a id="t.0ccdcdef5afdea08e69b17866f69a65eea871cfc"></a><a id="t.20"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ oc whoami</span> <span class="c2">-t</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c21 c19 c29">Make a note the oauth-token output by the command above. This token will serve as a password to log into the docker registry running on OpenShift.</span>

<span class="c21 c17 c29">Let us now  login to the registry running on OpenShift.</span>

<span class="c21 c17 c29"></span>

<span class="c17">Your OpenShift local cluster has the registry service assigned an ip address of</span> <span class="c5 c6">172.30.1.1</span><span class="c17">. Login to the docker registry using</span> <span class="c5 c6">oauth-token</span><span class="c17">from the last step, using username</span> <span class="c5 c6">developer</span><span class="c17">, and the registry url of</span> <span class="c5 c6">172.30.1.1:5000</span><span class="c21 c17 c29">.  This command will log you into the registry.</span>

<span class="c4"></span>

<a id="t.0aacf74263f3eb17fbef1399c42aa5716eaa25d7"></a><a id="t.21"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ docker login</span> <span class="c2">-</span><span class="c8">u developer</span> <span class="c2">-</span><span class="c8">p</span> <span class="c2"><</span><span class="c8">oauth</span><span class="c2">-</span><span class="c8">token</span><span class="c2">></span><span class="c8"> </span><span class="c8 c12">172.30</span><span class="c2">.</span><span class="c8 c12">1.1</span><span class="c2">:</span><span class="c8 c12">5000</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c14 c53">Note</span><span class="c17">: If you were using a secured registry, in order to connect to the registry, you will need certificate that is trusted by the registry. You would place the certificate in</span> <span class="c5">/etc/docker/certs.d/<registry-url></span><span class="c21 c17 c29"> folder. For this lab, we are using a local OpenShift cluster and hence this step is not required.</span>

<span class="c21 c17 c29"></span>

### <span class="c20">Step 9 Create ImageStream</span>

<span>L</span><span class="c22 c21">et us tag the image we created earlier to push into the OpenShift Cluster.</span>

<span class="c22 c21"></span>

<span class="c5 c21 c6">$ docker tag lighttpd-rhel 172.30.1.1:5000/myproject/lighttpd-rhel</span>

<span>This command will add a new tag for the</span> <span class="c5 c6">lighttpd-rhel</span><span>image that we created earlier to be pushed into the openshift registry into namespace</span> <span class="c5 c6">myproject</span><span class="c22 c21">.</span>

<span class="c21 c22"></span>

<span class="c19">We now need an imagestream where we will declare that this image is a builder image, so that we can use it from the web console via the catalog to build our application.</span> <span class="c19">Create a new json ImageStream file with the following content. Edit to make sure your image name matches with what you noted in the previous step. Let us name it as</span> <span class="c5">lighttpd-rhel-is.json</span><span class="c19">. Also make sure that the</span> <span class="c5">namespace</span><span class="c19"> is pointing to your project name (</span><span class="c5">myproject</span><span class="c21 c19 c29">)</span>

<span class="c4"></span>

<a id="t.13b63dc3cd16c1a290248256d2c14f583f683aa0"></a><a id="t.22"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c2">{</span><span class="c8">  
   </span><span class="c3">"kind"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"ImageStream"</span><span class="c2">,</span><span class="c8">  
   </span><span class="c3">"apiVersion"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"v1"</span><span class="c2">,</span><span class="c8">  
   </span><span class="c3">"metadata"</span><span class="c2">:</span><span class="c8"> </span><span class="c2">{</span><span class="c8">  
       </span><span class="c3">"name"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"lighttpd-rhel"</span><span class="c2">,</span><span class="c8">  
       </span><span class="c3">"namespace"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"myproject"</span><span class="c8">  
   </span><span class="c2">},</span><span class="c8">  
   </span><span class="c3">"spec"</span><span class="c2">:</span><span class="c8"> </span><span class="c2">{</span><span class="c8">  
       </span><span class="c3">"tags"</span><span class="c2">:</span><span class="c8"> </span><span class="c2">[</span><span class="c8">  
           </span><span class="c2">{</span><span class="c8">  
               </span><span class="c3">"name"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"latest"</span><span class="c2">,</span><span class="c8">  
               </span><span class="c3">"annotations"</span><span class="c2">:</span><span class="c8"> </span><span class="c2">{</span><span class="c8">  
                   </span><span class="c3">"description"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"Run HTML"</span><span class="c2">,</span><span class="c8">  
                   </span><span class="c3">"iconClass"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"icon-tomcat"</span><span class="c2">,</span><span class="c8">  
                   </span><span class="c3">"tags"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"builder,lighttpd"</span><span class="c8">  
               </span><span class="c2">},</span><span class="c8">  
           </span><span class="c3">"from"</span><span class="c2">:</span><span class="c8"> </span><span class="c2">{</span><span class="c8">  
             </span><span class="c3">"kind"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"DockerImage"</span><span class="c2">,</span><span class="c8">  
             </span><span class="c3">"name"</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"172.30.1.1:</span><span class="c3">5000</span><span class="c3">/myproject/lighttpd-rhel:latest"</span><span class="c8">  
           </span><span class="c2">}</span><span class="c8">  
           </span><span class="c2">}</span><span class="c8">  
       </span><span class="c2">]</span><span class="c8">  
   </span><span class="c2">}</span><span class="c4">  
}</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">Note the</span> <span class="c5">tags</span><span class="c19">section that uses the</span> <span class="c5">builder</span><span class="c21 c19 c29"> tag. This is required for OpenShift to recognize this imagestream as a builder image and to display on the catalog when you access from the web console.</span>

<span class="c19">Let us now add this imagestream to</span> <span class="c5 c6">myproject</span><span class="c19"> on the OpenShift cluster.</span>

<a id="t.7c88f7a35200edaf99b1331f9f9af2dc78ced2b1"></a><a id="t.23"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ oc create</span> <span class="c2">-</span><span class="c8">f lighttpd</span><span class="c2">-</span><span class="c8">rhel</span><span class="c2">-</span><span class="c8 c10">is</span><span class="c2">.</span><span class="c8">json</span> <span class="c2">-</span><span class="c4">n myproject</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">Verify the image streams in your project by running the following command.</span>

<a id="t.650ad191f8b117f1fcb946e6c38ab0b93187ba2c"></a><a id="t.24"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ oc</span> <span class="c8 c10">get</span><span class="c8"> </span><span class="c8 c10">is</span><span class="c8"> </span><span class="c2">-</span><span class="c4">n myproject</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">and you should see an image with name</span> <span class="c5">lighttpd-rhel</span><span class="c19">. However, the tags should be empty since we did not push the docker image into the registry yet.</span>

### <span class="c20">Step 10 Pushing image to Registry</span>

<span class="c19">N</span><span class="c19">ow push the docker image into the docker registry on your openshift cluster.</span>

<a id="t.e9bb7745ad9fa2efaa3e0186062d15963d68fe1f"></a><a id="t.25"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c8">$ docker push</span> <span class="c8 c12">172.30</span><span class="c2">.</span><span class="c8 c12">1.1</span><span class="c2">:</span><span class="c8 c12">5000</span><span class="c2">/</span><span class="c8">myproject</span><span class="c2">/</span><span class="c8">lighttpd</span><span class="c2">-</span><span class="c4">rhel</span>

</td>

</tr>

</tbody>

</table>

<span class="c5 c21 c6"></span>

<span class="c19">Check the image stream in your project and you should see the</span> <span class="c5">lighthttpd-rhel</span><span class="c19"> image with appropriate tags as shown below:</span>

<a id="t.93d020c85b67f6bc8038ceec15ca767275787c90"></a><a id="t.26"></a>

<table class="c13">

<tbody>

<tr class="c23">

<td class="c26" colspan="1" rowspan="1">

<span class="c2">[</span><span class="c8">student@ocp</span><span class="c2">-</span><span class="c8">lab</span> <span class="c2">~]</span><span class="c8">$ oc</span> <span class="c8 c10">get</span><span class="c8"> </span><span class="c8 c10">is</span><span class="c8"> lighttpd</span><span class="c2">-</span><span class="c8">rhel</span> <span class="c2">-</span><span class="c8">o yaml  
apiVersion</span><span class="c2">:</span><span class="c8"> v1  
kind</span><span class="c2">:</span><span class="c8"> </span><span class="c0">ImageStream</span><span class="c8">  
metadata</span><span class="c2">:</span><span class="c8">  
 annotations</span><span class="c2">:</span><span class="c8">  
   openshift</span><span class="c2">.</span><span class="c8">io</span><span class="c2">/</span><span class="c8">image</span><span class="c2">.</span><span class="c8">dockerRepositoryCheck</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">2017</span><span class="c2">-</span><span class="c8 c12">04</span><span class="c2">-</span><span class="c8 c12">13T22</span><span class="c2">:</span><span class="c8 c12">54</span><span class="c2">:</span><span class="c8 c12">46Z</span><span class="c8">  
 creationTimestamp</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">2017</span><span class="c2">-</span><span class="c8 c12">04</span><span class="c2">-</span><span class="c8 c12">13T22</span><span class="c2">:</span><span class="c8 c12">54</span><span class="c2">:</span><span class="c8 c12">46Z</span><span class="c8">  
 generation</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">2</span><span class="c8">  
 name</span><span class="c2">:</span><span class="c8"> lighttpd</span><span class="c2">-</span><span class="c8">rhel  
 </span><span class="c8 c10">namespace</span><span class="c2">:</span><span class="c8"> myproject  
 resourceVersion</span><span class="c2">:</span><span class="c8"> </span><span class="c3">"5114"</span><span class="c8">  
 selfLink</span><span class="c2">:</span><span class="c8"> </span><span class="c3">/oapi/</span><span class="c8">v1</span><span class="c2">/</span><span class="c8">namespaces</span><span class="c2">/</span><span class="c8">myproject</span><span class="c2">/</span><span class="c8">imagestreams</span><span class="c2">/</span><span class="c8">lighttpd</span><span class="c2">-</span><span class="c8">rhel  
 uid</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">31016623</span><span class="c2">-</span><span class="c8 c12">209c</span><span class="c2">-</span><span class="c8 c12">11e7</span><span class="c2">-</span><span class="c8 c12">9372</span><span class="c2">-</span><span class="c8 c12">0800277e3835</span><span class="c8">  
spec</span><span class="c2">:</span><span class="c8">  
 tags</span><span class="c2">:</span><span class="c8">  
 </span><span class="c2">-</span><span class="c8"> annotations</span><span class="c2">:</span><span class="c8">  
     description</span><span class="c2">:</span><span class="c8"> </span><span class="c0">Run</span><span class="c8"> HTML  
     iconClass</span><span class="c2">:</span><span class="c8"> icon</span><span class="c2">-</span><span class="c8">tomcat  
     tags</span><span class="c2">:</span><span class="c8"> builder</span><span class="c2">,</span><span class="c8">lighttpd  
   </span><span class="c8 c10">from</span><span class="c2">:</span><span class="c8">  
     kind</span><span class="c2">:</span><span class="c8"> </span><span class="c0">DockerImage</span><span class="c8">  
     name</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">172.30</span><span class="c2">.</span><span class="c8 c12">1.1</span><span class="c2">:</span><span class="c8 c12">5000</span><span class="c2">/</span><span class="c8">myproject</span><span class="c2">/</span><span class="c8">lighttpd</span><span class="c2">-</span><span class="c8">rhel</span><span class="c2">:</span><span class="c8">latest  
   generation</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">2</span><span class="c8">  
   importPolicy</span><span class="c2">:</span><span class="c8"> </span><span class="c2">{}</span><span class="c8">  
   name</span><span class="c2">:</span><span class="c8"> latest  
status</span><span class="c2">:</span><span class="c8">  
 dockerImageRepository</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">172.30</span><span class="c2">.</span><span class="c8 c12">1.1</span><span class="c2">:</span><span class="c8 c12">5000</span><span class="c2">/</span><span class="c8">myproject</span><span class="c2">/</span><span class="c8">lighttpd</span><span class="c2">-</span><span class="c8">rhel  
 tags</span><span class="c2">:</span><span class="c8">  
 </span><span class="c2">-</span><span class="c8"> items</span><span class="c2">:</span><span class="c8">  
   </span><span class="c2">-</span><span class="c8"> created</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">2017</span><span class="c2">-</span><span class="c8 c12">04</span><span class="c2">-</span><span class="c8 c12">13T22</span><span class="c2">:</span><span class="c8 c12">55</span><span class="c2">:</span><span class="c8 c12">17Z</span><span class="c8">  
     dockerImageReference</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">172.30</span><span class="c2">.</span><span class="c8 c12">1.1</span><span class="c2">:</span><span class="c8 c12">5000</span><span class="c2">/</span><span class="c8">myproject</span><span class="c2">/</span><span class="c8">lighttpd</span><span class="c2">-</span><span class="c8">rhel@sha256</span><span class="c2">:</span><span class="c8">f83d4c776d5e86a37fa083606d4d314ae6ddce26967dac1cd342660c0d8d011a  
     generation</span><span class="c2">:</span><span class="c8"> </span><span class="c8 c12">2</span><span class="c8">  
     image</span><span class="c2">:</span><span class="c8"> sha256</span><span class="c2">:</span><span class="c8">f83d4c776d5e86a37fa083606d4d314ae6ddce26967dac1cd342660c0d8d011a  
   tag</span><span class="c2">:</span><span class="c4"> latest</span>

</td>

</tr>

</tbody>

</table>

<span class="c21 c19 c29"></span>

### <span class="c20">Step 11 Test deploying an application</span>

<span class="c19">Now it is time to test your</span> <span class="c5">lighttpd-rhel</span><span class="c21 c19 c29"> S2I image from the Web Console.</span>

*   <span class="c19">Login to the Web Console and select the</span> <span class="c5">myproject</span><span class="c21 c19 c29"> project</span>
*   <span class="c19">Select</span> <span class="c5">Add to Project</span><span class="c21 c19 c29"> Button</span>
*   <span class="c19">You can search for</span> <span class="c5">lighttpd-rhel</span><span class="c21 c19 c29"> image in the catalog.</span>
*   <span class="c19">Select this image, give a name and the following git-url to try out a simple webpage to deploy</span> <span class="c45 c29">[https://github.com/VeerMuchandi/simple](https://www.google.com/url?q=https://github.com/RedHatWorkshops/openshiftv3-advanced-workshop/blob/master&sa=D&ust=1493768896247000&usg=AFQjCNEqWtTVZEJQKKgmB3jxI6e3OI7Tjg)</span>

<span class="c21 c19 c29">This should build and deploy the application.</span>

<span class="c19">Once tested, if you want to make this image available across the cluster, handover your imagestream file and the image to the cluster administrator. The administrator would have to repeat steps 7 thru 9 for the</span> <span class="c5">openshift</span> <span class="c21 c19 c29">namespace.</span>

## <span class="c21 c47 c29">Summary</span>

<span class="c19">Congratulations!!, In this lab you have learned how to create, test and use an S2I Builder image of your own. Please try out the technologies of your choice with S2I.</span>

<span class="c22 c21"></span>

<div>

<span class="c22 c21"></span>

</div>
