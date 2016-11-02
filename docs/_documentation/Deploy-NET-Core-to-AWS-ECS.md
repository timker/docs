---
slug: deploy-netcore-docker-aws-ecs
title: Deploy .NET Core with Docker to EC2 Container Service
---

![](http://docs.servicestack.net/images/aws/ecs/ecs-banner.png)

One of the primary benefits of .NET Core's first-class support for Linux is being able to leverage 
the thriving ecosystem that's formed around automating, deploying and hosting Server Apps on Linux. 
Whilst there have been a number of deployment strategies that have evolved over the years, the 
current state-of-the-art is the standardization around the Docker container format which hits the 
sweet spot of providing isolation within in a minimal footprint that encapsulates your App and its 
dependencies which is contrast to an entire Operating System when using VM's. In addition to being 
lightweight and efficient to run, it's also enables fast, reproducible builds where your entire
App's Docker Image can be rebuilt by your CI Server on each check-in - bringing the same benefits
of version control and continuous integration on your code to your Server infrastructure.

Once built, your Docker App can be treated like an opaque Image where the fact it's running .NET Core 
instead of a pure LAMP stack becomes a transparent implementation detail that doesn't have any impact
on the utility of the Docker App, which are both equally able to benefit from Docker's rich ecosystem. 
E.g. Your Docker App can be uploaded to any public Docker repository where it can be easily shared and 
run on any [supported Linux Distrobution](https://docs.docker.com/engine/installation/linux/).
In addition to being able to run any published Docker image on any compatible Linux Server, you're also
able to use it as a the source image for your own custom Docker Image, in fact you'll this feature to 
build your own .NET Core Docker App which will be based on Microsoft's 
[microsoft/dotnet](https://hub.docker.com/r/microsoft/dotnet/) Docker Image that's published on 
[Docker's public repository](https://hub.docker.com/explore/).

## Native Docker Integration in the Cloud

As the premier Container format for packaging Server Apps, all major cloud providers include native
support for Docker which goes beyond the basic hosting Docker Images to also include registry management, 
monitoring, inspection and orchestration which lets you easily (or automatically) scale your Docker App on 
a managed cluster of VMs. The 

 - **AWS** - [Amazon EC2 Container Service](https://aws.amazon.com/ecs/) 
 - **Azure** - [Azure Container Service](https://azure.microsoft.com/en-us/services/container-service/) 
 - **Google Cloud** - [Google Container Engine](https://cloud.google.com/container-engine/) 

Whilst other popular Cloud providers like [Digital Ocean](https://www.digitalocean.com) makes it trivial to 
[setup a Docker Server in seconds](https://www.digitalocean.com/products/one-click-apps/docker/).

## AWS EC2 Container Service 

For this guide we'll walk through how we've deployed our 
[.NET Core Live Demos](https://github.com/NetCoreApps/LiveDemos) using the popular power combo of:

 - Using [travis-ci.org](https://travis-ci.org/) (free for OSS projects) for running CI scripts to rebuild Docker App Images on every check-in
 - Using [AWS EC2 Container Service](https://aws.amazon.com/ecs/) for managing Docker images and instance deployments
 - Using [nginx-proxy](https://github.com/jwilder/nginx-proxy) setting up an nginx reverse proxy and automatically bind virtual hosts to Docker Instances

We'll also look at covering [Azure Container Service](https://azure.microsoft.com/en-us/services/container-service/) 
in future as it's another popular option for deploying and hosting .NET Core Docker Apps.

## Deploying to AWS Container Service Overview

In this guide we'll through setting up your AWS Account to enable Docker deployments using AWS EC2 Container 
Service and have them deploy to a single EC2 Instance for optimal value and minimal cost. There's 
[no additional cost for using EC2 Container Service](https://aws.amazon.com/ecs/pricing/), you're only
charged for the EBS and EC2 resources for storing and running your Docker Containers. Thanks to the 
small footprint of Docker Containers and .NET Core's lean profile you'll be able to host a few small 
Docker Apps on a T2 Micro instance at less than **<$10 per month** or if you haven't created an account with
AWS before, you can run this **free for 12 months** courtesy of [AWS Free Tier](https://aws.amazon.com/free/).

### Deploying fork of redis-geo .NET Core App

We'll be deploying a fork of the .NET Core [Redis GEO Live Demo](https://github.com/NetCoreApps/redis-geo) 
as it's a good representative of a small multi-project .NET Core solution that requires an external Redis 
Server infrastructure dependency which we'll deploy alongside our App in a separate Docker container. 

We'll use [Travis CI](https://travis-ci.org/) to do the actual build and deployments which effectively just 
runs plain bash scripts that lives with your code in the same 
[Github Repo](https://github.com/NetCoreApps/redis-geo) to build and deploy your Docker App. Most of the 
bash scripts are generic scripts and can be reused to deploy different projects with only minor config 
changes to specify which solution to build and what AWS Account to deploy to, with any sensitive information 
maintained in Travis-CI private Environment variables to keep them out of public Github repositories. 

### Running build scripts locally

A nice property of using plain Bash scripts to build Docker images is being able to run them locally letting 
you run the resulting Docker Image natively on Linux, but also on OSX with
[Docker for Mac](https://docs.docker.com/engine/installation/mac/) which is nicely integrated courtesy of
OS X's built-in Bash and Terminal.app. Whilst less integrated, it's also possible to run them with 
[Docker for Windows](https://docs.docker.com/docker-for-windows/).

### Walkthrough Overview

The basic steps to deploy your .NET Core App from scratch that are covered in this guide include:

 1. Create `ecsInstanceRole` that your EC2 Instance will run as
 2. Create a new `ecsDemoUser` for running the AWS CLI to deploy to your AWS Account
 3. Starting a new EC2 Docker Server Instance with `ecsInstanceRole`
 4. Configuring DNS and associating Elastic IP to new EC2 Instance
 5. Configure the new EC2 Instance with `nginx-proxy`
 6. Forking the `redis-geo` Github project
 7. Configure **Travis CI** to deploy your `redis-geo` fork to your AWS Account
 8. Play your deployed .NET Core Docker App!

This may seem like a lot of effort to deploy a Docker App but most of the above steps only need to be done 
once - only **steps 6-8** are repeatable for each new project that you want to deploy to your EC2 
Container Instance.

At the end of the rainbow an instance of **your fork** of the redis-geo .NET Core App will be running on your
AWS EC2 Instance, re-deployed on every check-in by, triggering **Travis CI** into action to rebuild and 
package the latest version of your App in a new Docker App Image that it also publishes to AWS ECS. 
A complete history of every Docker App Image and related task definition that was deployed is also maintained 
in your private AWS ECS Docker Repository. You'll also be able to inspect your running EC2 Instance to see 
how much CPU and memory is used which will provide a good indication of how many more Docker Apps you'll be 
able to pack into your EC2 Instance.

Now we know what we're getting, let's jump in!

## 1. Create the `ecsInstanceRole` Role

Our first port of call is to create a new `ecsInstanceRole` in IAM, this is a 
[special Role for EC2 Container Instances](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/instance_IAM_role.html)
that also needs the necessary permissions for your EC2 Instance to communicate with the ECS API. 
When launching a new EC2 Instance with the `ecsInstanceRole`, it automatically creates the 
**default cluster** in ECS and registers itself to the available set of EC2 instances that can run Docker Apps 
that are deployed to that cluster.

Creating new Roles are done within **Identity & Access Management (IAM)** in your AWS dashboard under the **Roles** 
tab which you can jump directly to with the deep link:
[https://console.aws.amazon.com/iam/home#roles](https://console.aws.amazon.com/iam/home#roles) then by 
clicking on **Create New Role** button to start the Create Role Wizard:

Type `ecsInstanceRole` for the **Role Name** then click **Next Step**:

![](http://docs.servicestack.net/images/aws/ecs/role-01.png)

Select the **Amazon EC2 Role for EC2 Container Service** Role Type:

![](http://docs.servicestack.net/images/aws/ecs/role-02.png)

You'll also need to attach the `AmazonEC2ContainerServiceforEC2Role` policy which you can quickly find by 
pasting it in the **Policy Type** filter then checking the only result, then proceed to the **Next Step**:

![](http://docs.servicestack.net/images/aws/ecs/role-03.png)

The last page in the Wizard lets you review details of the Role you'll creating after clicking **Create Role**:

![](http://docs.servicestack.net/images/aws/ecs/role-04.png)

The `ecsInstanceRole` will now be listed as a Role in IAM which should look like:

![](http://docs.servicestack.net/images/aws/ecs/role-05.png)

## 2. Create a new IAM User 

We now need to create a new IAM User to allow the remote Servers we want permission to access our 
ECS Container Service on our behalf which **Travis CI** needs when using the 
[AWS CLI](https://aws.amazon.com/cli/) to deploy our Docker App to our ECS **default cluster**.

Similar to creating a role, to create a new User select the **Users** tab then click **Create New Users**.

On the next screen add the preferred Username in the first textbox then click **Create**. The remainder 
of this tutorial uses the `ecsDemoUser` Username:

![](http://docs.servicestack.net/images/aws/ecs/user-01.png)

You'll need to copy both the **Access Key ID** and **Secret Access Key** which is **only available** 
from the next screen. These credentials will need to be saved as they're required for **Travis CI** to 
authenticate as this new User. 

![](http://docs.servicestack.net/images/aws/ecs/user-02.png)

We also need to assign the following permissions to the new User to give it access to our ECS Service.
The easiest way to select these policies is to copy each into the **Policy Type** Filter then select
the filtered result, you'll need to do this for each of the policies below:

 1. `AmazonEC2ContainerRegistryFullAccess`
 2. `AmazonEC2ContainerServiceFullAccess`
 3. `AmazonEC2ContainerServiceRole`

After clearing the filter you should see 3 Policies selected, then click **Attach Policy** to add them 
to the User.

![](http://docs.servicestack.net/images/aws/ecs/user-03.png)

As the names were cropped in the previous screen, double-check that you have the same policies selected 
as below:

![](http://docs.servicestack.net/images/aws/ecs/user-04.png)

## Checkout AWS Container Service

At this point we to check out the **EC2 Container Service** by clicking on its name in the AWS dashboard or 
following the direct link: [https://console.aws.amazon.com/ecs/home](https://console.aws.amazon.com/ecs/home)

We're not going to do anything within ECS at the moment other than observe its emptiness :) The first time
you open it you'll visit the splash page below which you can skip on through by clicking **Get Started**:

![](http://docs.servicestack.net/images/aws/ecs/ecs-01.png)

In the next screen AWS tries to guide you into populating your ECS Service with something, but as we're 
after a blank slate, you'll want to click on the **Cancel** link:

![](http://docs.servicestack.net/images/aws/ecs/ecs-02.png)

Which will arrive us to our destination, complete emptiness :)

![](http://docs.servicestack.net/images/aws/ecs/ecs-03.png)

But we needn't worry, everything in our ECS Service is going to be created outside by our **new EC2 Instance**
and **Travis CI** build.

## 3. Launch new EC2 Docker Server Instance

At this point we now have everything we need to light up some bits where our Docker Apps will run on.
We want the EC2 Instances in the Container Service to use the purpose-specific 
**Amazon ECS-Optimized Amazon Linux AMI** which you can quickly find by going to the **AWS Marketplace** tab
and adding `ecs-optimized` in the filter then **Select** the first Amazon AMI result:

![](http://docs.servicestack.net/images/aws/ecs/launch-01.png)

Which will start the Launch Instance Wizard where you can select the Instance Type you want to run. 
The **t2.micro** is more than enough for what we need but you can select your preferred instance type 
then click **Next: Configure instance Details**:

![](http://docs.servicestack.net/images/aws/ecs/launch-02.png)

The important option in the **Configure Instance Details** screen is to select the `ecsInstanceRole` for the 
EC2 Instance's **IAM Role**, you'll also want to choose `Enable` for the **Auto-assign Public IP** option
if it's not the default already:

![](http://docs.servicestack.net/images/aws/ecs/launch-03.png)

You can skip pass the defaults on the **Add Storage** screen, in the **Tag Instance** screen you'll be able 
to assign a human-friendly label to identify this instance, e.g:

![](http://docs.servicestack.net/images/aws/ecs/launch-04.png)

In the **Configure Security Group** page you can specify the ports you want open. You'll want **SSH** open
so you can SSH into your instance and for a Web App you'll also typically want **HTTP** and **HTTPS** open 
as well. With the warning message AWS is encouraging you to limit your SSH access to a limited IP Range
which if you have a **static IP** from your ISP is a good idea to lock down the **Source** to `My IP`, 
otherwise leaving it open to any IP is fine as we'll be using a Private Key to login to our Instance.

![](http://docs.servicestack.net/images/aws/ecs/launch-05.png)

On the next page you want to either create a new Public/Private Key Pair or use an existing one. If creating 
a new one you need to click **Download Key Pair** as you'll need it to SSH into your instance.

![](http://docs.servicestack.net/images/aws/ecs/launch-06.png)

After clicking **Launch Instances** you'll be redirected to your EC2 **Instances** tab where you can 
watch your Instance starting up:

![](http://docs.servicestack.net/images/aws/ecs/launch-07.png)

## Revisit AWS Container Service

Once **running** go back to your **EC2 Container Service** where you should now see a **default** cluster
was created:

![](http://docs.servicestack.net/images/aws/ecs/ecs-04.png)

Then clicking on **ECS Instances** tab should show your new Instance with an **ACTIVE** state:

![](http://docs.servicestack.net/images/aws/ecs/ecs-05.png)

If you don't see a **default** cluster created with your new Instance attached, terminate your Instance 
then go back and create it again, triple-checking it has the `ecsInstanceRole` **IAM Role** when Configuring
the Instance. If that fails, try re-creating the `ecsInstanceRole` ROLE making sure it has the 
`AmazonEC2ContainerServiceforEC2Role` Policy. 

## 4. Associating Elastic IP to new EC2 Instance

Before connecting to the Instance I prefer to assign it a domain name which gives it a canonical reference 
that makes it easier to remember and refer to later. Before doing this you'll want to assign your instance
an **Elastic IP** so you have a permanent Public IP within your control that you can assign a domain name to. 

To do this, click on the **Elastic IPs** tab then click on **Allocate New Address**, this will issue you a 
new **Elastic IP** you can assign to your new instance, e.g:

![](http://docs.servicestack.net/images/aws/ecs/elasticip-01.png)

After which you should see show up in the **Elastic IPs** tab:

![](http://docs.servicestack.net/images/aws/ecs/elasticip-02.png)

## Assign a Domain Name to your Elastic IP

This is going to different for each Domain Registrar but you'll want to assign a new **A Record** to 
point to your new **Elastic IP**. If you're using Google Domains you can refer to their 
[Google Domains Help Documentation](https://support.google.com/domains/answer/3290309?hl=en&ref_topic=3251230).

For the purposes of this tutorial we've used the `ecsdemo.netcore.io` domain name which resolves to our 
Elastic IP Address.

## 5. Log into the new EC2 Instance

With the new Instance up and running we can login to it, but you'll need access to a `ssh` client to make 
that happen. If you're fortunate to have a Unix-based OS at your disposal you're already set, if you're 
on Windows 10 you can enable [Bash on Ubuntu on Windows](https://msdn.microsoft.com/en-us/commandline/wsl/about)
otherwise you'll need to install a [SSH Client for Windows](http://www.putty.org/).

When you're ready open a Terminal and copy the **EC2 Instance Key Pair** that you downloaded after creating
the new EC2 Instance and copy it into a permanent location, e.g `~/pem/ecs-demo.pem`.

Before we can use the Private Key we need to lock down the permissions so only we can read it which you 
can do with:

    $ chmod 400 ~/pem/ecs-demo.pem

Once locked down you can use it to SSH into your instance using the `-i` flag, logging in as the `ec2-user`:

    $ ssh -i ~/pem/ecs-demo.pem ec2-user@ecsdemo.netcore.io

Replace `ecsdemo.netcore.io` with your domain name or Elastic IP. If everything went to plan you should now
be logged into your EC2 Instance:

![](http://docs.servicestack.net/images/aws/ecs/ssh-01.png)

We're only assigned one task while we're here which is to run an instance of the `jwilder/nginx-proxy` 
Docker App using the command below:

    $ docker run -d -p 80:80 -v /var/run/docker.sock:/tmp/docker.sock:ro jwilder/nginx-proxy

Which will pull the [jwilder/nginx-proxy](https://hub.docker.com/r/jwilder/nginx-proxy/) Docker Image from 
Dockers public repository [https://hub.docker.com/explore/](https://hub.docker.com/explore/) which sets up 
a new Docker container running [nginx](https://nginx.org/en/) and 
[docker-gen](https://github.com/jwilder/docker-gen) which is what will enable our no-touch deployments where 
it will generate the reverse proxy configs for nginx each time we deploy a new Docker App using its 
`VIRTUAL_HOST` Environment Variable that it was created with. 

It's not important to 
[know exactly how it works](http://jasonwilder.com/blog/2014/03/25/automated-nginx-reverse-proxy-for-docker/) 
only that it's responsible for setting up nginx to reverse proxy requests from our App's custom `VIRTUAL_HOST` 
(e.g. ecsdemo.netcore.io) to our .NET Core Kestrel HTTP Server that's listening on port **5000**.

If successful you'll get an output similar to:

![](http://docs.servicestack.net/images/aws/ecs/ssh-02.png)

For the more curious you can run `docker ps` to see the running Docker Containers which should include 
this nginx-proxy and AWS's ECS agent.

## 6. Fork redis-geo Demo

With all the prep work for setting up our Docker Server out of the way, we can now get to the fun part 
of deploying Docker Apps to it. Let's get started by forking the 
[https://github.com/NetCoreApps/redis-geo](https://github.com/NetCoreApps/redis-geo) project:

![](http://docs.servicestack.net/images/aws/ecs/fork-01.png)

This will create our own fork of the `redis-geo` Github project into our own Github User Account. We then
need to **Clone** our local fork which we can do from the [Github Desktop App](https://desktop.github.com/)
to a local folder:

![](http://docs.servicestack.net/images/aws/ecs/fork-02.png)

### Inspect Scripts

Lets open the folder in VS code to inspect what deployment scripts we have:

    C:\src\redis-geo>code .

The `.travis.yml` is used to configure the environment which **Travis CI** will execute your build under.
Here we're telling Travis we want a **csharp** environment for our solution located at **src/RedisGeo.sln**
using an Ubuntu 14.04 Linux distribution with .NET Core installed. The **script** section tells Travis how 
it should build our project, which is to just run the build scripts in the repo:

![](http://docs.servicestack.net/images/aws/ecs/code-01.png)

The `DockerFile` tells Docker how to build our Docker App Image which: 

 - is based on the **microsoft/dotnet:latest** Docker Image on the public Docker Repository 
 - includes a copy of our .NET Core solution files
 - runs `dotnet restore` on to pull down all NuGet dependencies
 - builds the .NET Core Solutions **Host Project** using `dotnet build`
 - notifies Docker our App will run a TCP Service listening on port `5000` that it should expose
 - sets the `ASPNETCORE_URLS` environment variables
 - tells Docker what to execute to run our App

This is a fairly generic template for running a .NET Core multi-project solution which typically only
requires changing the **WORKDIR** to point to your Solutions **Host Project**:

![](http://docs.servicestack.net/images/aws/ecs/code-02.png)

The primary config changes you'll need to make are in `deploy-env.sh` which contains public deployment and 
project-specific info. Here you'll want to change `AWS_DEFAULT_REGION` with the AWS region where your 
ECS Service is located. You'll also need to change `AWS_VIRTUAL_HOST` with your `VIRTUAL_HOST`:

![](http://docs.servicestack.net/images/aws/ecs/code-03.png)

The `task-definition.json` is specific to AWS ECS which lets you declaratively define an **atomic unit**
of Docker Containers ECS should run when it deploys this "Task". It's essentially a thin wrapper around
[docker run](https://docs.docker.com/engine/reference/run/) where most features can be easily mapped 
to its command-line switches. 

The `task-definition.json` is where we tell ECS what Docker Containers our App needs. Here in addition to 
our Docker App we also need an instance of `redis:3.2` that's made available to our App via Docker's
`--link` feature which lets us connect to the redis-server instance using the `ecsdemo-redisgeo-redis` 
Hostname that we tell our App about with the `REDIS_HOST` Environment variable:

![](http://docs.servicestack.net/images/aws/ecs/code-04.png)

## Create project in Travis-CI

After making the necessary changes in `deploy-env.sh` we can register our project in **Travis CI**, 
fortunately for us Travis's UX is streamlined making this effortless, starting by clicking on **Sign Up** 
on its home page [https://travis-ci.org](https://travis-ci.org):

![](http://docs.servicestack.net/images/aws/ecs/travis-01.png)

Then click on the `+` link next to **My Repositories**:

![](http://docs.servicestack.net/images/aws/ecs/travis-02.png)

This takes you to your Github Profile view where you just need to click on the the switch icon next to your
**redis-geo fork**, e.g:

![](http://docs.servicestack.net/images/aws/ecs/travis-03.png)

After enabling your project it will appear in your repositories list. Here we need to click on 
the **Settings** Menu Item in the **More Options** drop down menu in order to specify the sensitive 
`ecsDemoUser` credentials that we don't want to include in a public Github Repository. At a minimum
you'll need to add Environment variables for the `ecsDemoUser` IAM User you copied above:

 - `AWS_ACCOUNT_NUMBER`
 - `AWS_ACCESS_KEY_ID`
 - `AWS_SECRET_ACCESS_KEY`

The **Settings** page is where you can define any sensitive information you want your Docker App to have
access to. E.g. although it's not needed for this Demo you can specify your `SERVICESTACK_LICENSE` here
and inject it into your Docker App using the **"environment"** Key/Value list in your projects 
`task-definition.json`. If your App requires a lot of confidential information it can be easier to instead
register a connection string to a [Remote AppSettings data source](http://docs.servicestack.net/appsettings) 
that way you can have a centralized location for managing keys and connection strings for all your Apps.

![](http://docs.servicestack.net/images/aws/ecs/travis-04.png)

If it's not on hand, you can get the `AWS_ACCOUNT_NUMBER` for your `ecsDemoUser` from the number embedded 
in its **User ARN**:

![](http://docs.servicestack.net/images/aws/ecs/travis-05.png)

After adding your `ecsDemoUser` credentials above, commit and push your changes to `deploy-env.sh` in order 
to trigger a new **Travis CI** build. Or if you've previously committed them you can click **Restart build**
to rerun the build using your latest Environment Variables. This will trigger a new build which if 
everything is in place should result in a successful green build:

![](http://docs.servicestack.net/images/aws/ecs/travis-06.png)

## Inspect AWS Container Service

Now we can go back into **EC2 Container Service** and have a look at what the generic `deploy.sh` script 
created for us. Clicking the **Repositories** tab will display your Apps 
[private ECR Repository](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ECS_Console_Repositories.html)
which contains a list of all Docker Images that were deployed to ECS with the current running Docker Container 
based on the Image tagged with the **latest** Image tag.

![](http://docs.servicestack.net/images/aws/ecs/ecs-06.png)

In the **Clusters** tab you'll see that your **default** Cluster contains an 
[ECS Service](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) 
aptly named `ecsdemo-redisgeo-service`: 

![](http://docs.servicestack.net/images/aws/ecs/ecs-07.png)

[ECS Service](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/ecs_services.html) 
are what manages your running Tasks where if your Docker App goes down for whatever reason, its Service will 
restart new Task Instances based on its 
[Task Definition](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/task_defintions.html) 
until the **Desired count** of instances is reached.  

![](http://docs.servicestack.net/images/aws/ecs/ecs-08.png)

If your Service contains a **RUNNING** Task that means your time here was a success which we can further
verify by SSH'ing back into our EC2 Instance and running `docker ps` to see our running Docker Containers:

![](http://docs.servicestack.net/images/aws/ecs/ssh-03.png)

Where we should see **4 running** Docker Containers:

 1. AWS ECS agent
 2. nginx-proxy
 3. redis-server
 4. Our .NET Core Docker App!

## 8. Play your deployed .NET Core Docker App!

With all the pieces in place we can visit our virtual host [http://ecsdemo.netcore.io](http://ecsdemo.netcore.io) 
to see the fruits of our Labor - our .NET Core Docker App in action!

![](http://docs.servicestack.net/images/aws/ecs/app-01.png)

## Things to try

### 1. Make a change to your App

With your .NET Core Docker mission officially a success the first thing you're going to want to do 
is committing a change to your fork whilst watching your **Travis CI** bot working away so you can see your
Continuous Docker deployments in action :)

### 2. Go through this tutorial again

If you've stuttered hazily through parts of this tutorial I recommend going through it again from scratch as
things should become a lot clearer and faster the next time around. After doing this a few times everything 
will appear intuitive and you won't need to refer to this tutorial again.

### 3. Copy the build scripts to Dockerize own .NET Core Apps

Customizing the deploy scripts to your own Apps will help familiarize yourself with Docker and identify 
the areas which are customizable.

### 4. Exec build scripts to Create and Run your Docker App locally 

Install [Docker for Mac](https://docs.docker.com/engine/installation/mac/) or 
[Docker for Windows](https://docs.docker.com/engine/installation/windows/) and try building and running the
Docker Images locally. Since you're running locally you'll need to set any required **Travis CI** Environment 
Variables then have a look at converting what ECS does with `task-definition.json` into docker commands. 
This example is a little tricky due to the use of Docker's `--link` feature but with some trial and error and
the [Docker run reference](https://docs.docker.com/engine/reference/run/) to guide you, you'll have your 
Docker App running locally - acquainting you with one of Docker's killer features!

