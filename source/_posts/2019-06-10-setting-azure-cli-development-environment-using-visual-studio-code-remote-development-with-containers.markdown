---
layout: post
title: "Setting Azure CLI Development Environment Using Visual Studio Code Remote Development with  Containers"
date: 2019-06-10 19:22:11 +0800
comments: true
categories: 
---

I have tested new Visual Studio Code Remote Development feature with Azure CLI. Now you can start developing Azure CLI in a few simple steps:

- Install Visual Studio Code Insiders Edition
- Clone Azure CLI Repository
- Launch Visual Studio Code and wait for a few minutes
- Set a breakpoint and press F5

To replicate this you need do download Visual Studio Code from here:

https://code.visualstudio.com/insiders/

Clone my fork of Azure CLI repository and switch to **adding-devcontainer-setup** branch.

```bash
git clone https://github.com/zikalino/azure-cli.git
cd azure-cli
git checkout adding-devcontainer-setup
```

Then run Visual Code from cloned repository:

```bash
code-insiders .
```

Visual Studio Code should detect that **.devcontainer** folder is present, and will ask whether you want to use container.

![Reopen in Container](/images/devcontainer-reopen.png)

When you choose **Reopen in Container**, VSC will build the image from docker file, that may take a few minutes, but it will be done only once (for current folder).

![Build Container](/images/devcontainer-build.png)

When container is ready, just set a breakpoint, for instance in **azure-cli/src/azure-cli/__main__.py** and press **F5**.

![Run Container](/images/devcontainer-run.png)

## More Details on the Container

My Dockerfile is pretty simple.
As a base I used Ubuntu 16.04, and:

- Installed Python 3.6
- Cloned Azure CLI repo (just to install dependencies)
- Created virtual environment located at **/env** (outside Azure CLI repo)
- Installed **azdev** using **pip**
- Installed Azure CLI dependencies using **azdev setup**
- Made sure that **/env** is activated every time **bash** is started by Visual Studio Code.

```
# start from Ubuntu 16.04
FROM ubuntu:xenial

# install python 3.6, pip, venv and git
RUN apt-get update
RUN apt-get -y install software-properties-common
RUN add-apt-repository ppa:deadsnakes/ppa
RUN apt-get update
RUN apt-get -y install python3.6 python3-pip python3-venv git

# clone azure cli into /workspaces/azure-cli
RUN mkdir /workspaces; cd /workspaces; git clone https://github.com/Azure/azure-cli.git

# create environment in /env and set up 
RUN python3 -m venv env
RUN /bin/bash -c "source /env/bin/activate; pip3 install azdev; azdev setup --cli /workspaces/azure-cli" 

# remove azure-cli as it will be used 
RUN rm -rf ./workspaces/azure-cli

# add environment activation to .bashrc
RUN echo "source /env/bin/activate" >> ~/.bashrc
```

## And Configuration File

Configuration file **devcontainer.json** is pretty simple. It specifies **Dockerfile** and a path to Python interpreter inside **/env**.

Please note that instead of specifying **dockerFile** it's possible to specify existing **image**.

```
{
	"name": "Azure CLI Dev",
	"dockerFile": "Dockerfile",
	"context": ".",
	"extensions": [
		"ms-python.python"
	],
	"settings": {
		"python.pythonPath": "/env/bin/python3"
	},
	"postCreateCommand": "bash"
}
```


## Other Considerations

(1) It's possible to specify existing image instead of dockerfile. If we have existing development image that would make setup even faster.

(2) Perhaps it would be a good idea to clone all **azure-cli**, **azure-cli-dev-tools**, **azure-cli-extensions** under workspaces. Then it should be possible to map all three of them when starting the container.

(3) Perhaps leaving **azure-cli** (I currently delete it) would be a good idea, this way the container could be used as a standalone dev environment with source code cloned inside.

(4) Instead of cloning **azure-cli** it's possible to map external repository. I am not doing it right now, as there's **.dockerignore** file that would need to be modified in order to make this solution to work, and that could affect

## Documents

Development environment setup instructions:

https://github.com/Azure/azure-cli-dev-tools

About VSC Remote Development

https://code.visualstudio.com/docs/remote/remote-overview
