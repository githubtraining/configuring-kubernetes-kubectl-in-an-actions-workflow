# How To Deploy A Container App To Kubernetes

GitHub Actions Workflow allows users to take advantage of a vast selection of open source tooling and solutions. Since the Actions Workflow 'runners' are in themselves Virtual Machines (VMs) running on cloud servers, the operating system choice for the runner dictates the syntax of run commands, the operating system commands available, and the third-party applications available for us on any particular runner.

In this example we build upon prior examples and utilize docker to containerize an application, and push it to dockerhub.com. Then we call the Kubernetes kubectl utility to create or update a pod deployment using the container we pushed to dockerhub.com.

## The GitHub Actions Workflow Architecture

To illustrate the manner through which a GitHub Actions Workflow might be employed for a CI/CD pipeline, it is important to understand the high level architecture of the GitHub Workflow runner.

The following illustration is a basic workflow runner and shows the external public cloud systems that might be utilized to host the application on Kubernetes. The role of dockerhub.com has also been added to the architecture.

<table>
<tr><td>
<img width="1204" alt="Screen Shot 2019-09-23 at 3 06 02 PM" src="https://user-images.githubusercontent.com/43185011/65454711-b8f01f00-de13-11e9-90d6-613e3000db75.png">
</td></tr>
<tr><td>
The GitHub Actions Workflow 'Runner' is a VM that launches on each workflow trigger and terminates upon workflow completion. The above architecture shows an Amazon EC2 server instance that has launched previously and has been configured to run Kubernetes. In this example, the java application is imaged into a docker container that is then pushed to docker hub. The kubectl command is then run from the 'runner' VM instance to deploy the pod to pull and run the container image.
</td></tr>
</table>

### IMPORTANT NOTE ABOUT THIS STEP
> #### The following steps would have already been performed in another lab or phase of the process. They are provided here for your review only. This learning lab's template repository contains the workflow that completes these tasks. It is not necessary for you to create any files, or edit any files for this step. This is provided for review context only.

### Containerizing a Java Application

#### Building the Container Image

After an applications has been compiled with a build tool such as gradle or maven, there is a java archive that is a .jar, .war or .ear file format dependent upon the build configuration. In our example we have built a .war (web archive) file that contains the java bytecode for a Spring Boot application. 

One way to create a docker image that may be deployed is to use a Dockerfile. In this case we have placed one in the root directory of our repository. The Dockerfile simply contains the necessary commands to create the image we need to run the very simple Spring Boot application. The example Dockerfile is shown below:

<table>
<tr><td>
<img width="1111" alt="Screen Shot 2019-09-23 at 3 14 51 PM" src="https://user-images.githubusercontent.com/43185011/65455294-edb0a600-de14-11e9-96e3-7db3f66113a5.png">
</td></td>
</table>

This Dockerfile starts with a base image called java:8-jdk-alpine. It then copies the .war file created by the gradle build into our new image file. It establishes the working directory as /usr/app, exposes the web port 8080, and establishes an 'entrypoint' that is a command to start the application once deployed.

The command to build the container image is:

```
docker build -t nasa-picture .
```
This command simply tells the docker runtime in the runner to execute the Dockerfile, and tag the image with the name 'nasa-picture'. The . at the end is telling the build to look in the present working directory for the Dockerfile spec.

#### Pushing the Container Application to Docker Hub

To push the application to Docker Hub, authentication is required. The following command is used in the workflow to pull the docker username and password from the secrets vault and authenticate to the docker hub API endpoint.

```
docker login -u ${{secrets.dhuser}} -p ${{secrets.dhpassword}}
```

####

To take the most recently tagged image named 'nasa-picture' and tell docker where to push the image to in docker hub, you would use this command:

```
docker tag nasa-picture:latest githubtraining/universe:np-${{github.actor}}
```
This tells docker to take the nasa-picture image and tag it with the external tag that will place it in the 'githubtraining' repository, within a folder named 'universe' on docker hub and name it np-<GitHub Username>.

The last step is to push the image to docker hub with the following command:

```
docker push githubtraining/universe:np-${{github.actor}}
```

Each of these individual command s are run as a disctinct command within the workflow. The entire set of commands looks as follows in the workflow yaml.

<table>
<tr><td>
<img width="1111" alt="Screen Shot 2019-09-23 at 3 29 07 PM" src="https://user-images.githubusercontent.com/43185011/65456263-ee4a3c00-de16-11e9-94a6-94f94de92730.png">
</td><tr>
</table>

### Step Completion

> #### Having completed the review of this step, you may close this issue to proceed in this lab.
