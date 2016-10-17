+++
tags = [
  "direnv",
  "Environment Variable",
]
categories = [
  "Productivity",
]
image = "/img/defaulf-post-bg.jpg"
description = "Direnv is a tool to manage environment variables per directory"
date = "2016-10-16T22:35:06-07:00"
title = "Managing Environment Variables Using direnv"
draft = false

+++

## Trouble with Managing Environment Variables

In this container era, environment variables are the most used way of passing parameters. When you are going to try some software you may need to setup many environment variables. if you repeatedly going to setup things you may have to export environment variables repeatedly or you may have to write a script to export the required environment variables. In this case, [direnv](https://direnv.net/) is the cool tool to help you to free from managing environment variables on your own.

## Install and Setup `direnv`

Follow the guide in [home page of the project](https://direnv.net/) to intall direnv and properly add a hook to your shell. For example i am using zsh shell.
I have to add following to `~/.zshrc` file after installing binary.

````
eval "$(direnv hook zsh)"
````

## Example Usage

Recently i happened to setup [kubernetes](http://kubernetes.io/) using kubernetes local setup [guide](https://github.com/kubernetes/kubernetes/blob/master/docs/devel/running-locally.md).
In the process of setting up a local cluster, I have to export following environment variables to modify the setup according to my requirement.
````
KUBE_ENABLE_CLUSTER_DNS=true
API_HOST=172.17.42.1
KUBELET_HOST=172.17.42.1
HOSTNAME_OVERRIDE=172.17.42.1

````

First couple of times I have manually exported the variables. Then I have written a script and called the script before starting the setup script. I just realized I have many scripts likes this for each project I am working on. I was searching a way to automatically export  required variables when I `cd` into the project or something similar to ease my life. I found `direnv` .It was the  perfect candidate for my requirements.

I have created `.envrc` file in kubernetes directory and added follwing contents to the file.

````
export KUBE_ENABLE_CLUSTER_DNS=true
export API_HOST=172.17.42.1
export KUBELET_HOST=172.17.42.1
export HOSTNAME_OVERRIDE=172.17.42.1
````

First time i had to run following command to allow the `direnv` to securly configure the environment variables.
````
direnv allow .
````

After that when i `cd` into a folder the required environment variables were automatically exported.

````
➜  ~ cd go/src/k8s.io/kubernetes/
direnv: loading .envrc
direnv: export +API_HOST +HOSTNAME_OVERRIDE +KUBELET_HOST +KUBE_ENABLE_CLUSTER_DNS
````
I just descibed a single usecase.Please refer `Usage` from [direnv](https://direnv.net/) project page for further usecases.
