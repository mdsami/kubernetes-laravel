# Laravel + EKS + Helm

## What is Helm and Why ?

* Building k8s cluster with declarative configuration files (YAML) is good. 
* But how you rolling updates / rollback ? `kubectl apply` for every configuration file. What ?!
* Using environment variables to templatize it ? Not supported.

There are discussion about these issues and suggestions on tools to templatize the manifest files:  
https://github.com/kubernetes/kubernetes/issues/23896#issuecomment-31354485  
  
So I decided to give it a try on [Helm](https://helm.sh/).  
  
Basically, Helm uses [Go templates](https://godoc.org/text/template) for templating
your manifest files (YAML). While ships with several built-in functions and other additonal features.  

Helm process those template with the values spcified in `values.yaml`. 
You can have multiple `values.yaml` file for different environments. 
Just like the concept in [Docker Compose](https://docs.docker.com/compose/extends/#compose-documentation).  
  
Understand [three big concepts](https://docs.helm.sh/using_helm/#three-big-concepts) in Helm.  


## Setup Helm

1. Install [Helm](https://docs.helm.sh/using_helm/#install-helm)

2. 