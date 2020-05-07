### Summary

This repo helps to build and deploy a nodejs app via an OpenShift templates and Helm.

### Helm vs OCP template

* [Helm](https://helm.sh/) is best way to find, share, and use software built for Kubernetes. It allows describing the application structure through convenient helm-charts and managing it with simple commands.
[OCP-Template](https://docs.openshift.com/container-platform/3.5/dev_guide/templates.html) can describe a set of objects, for example services, build configurations, and deployment configurations. These can be processed to create anything you have permission to create within a project
* Templates are way more static compared to Helm. Helm provides values.yaml file which can be used to parameterise the entire helm chart. These values are easy to override for multiple runs use different vlaues file or using console input.
* Helm can deal with many lifecycles methods which templates lack.
* Helm can be built around a solid testing framework for making it viable for large projects.

### Prerequisites

1. First login to your OpenSH=hift cluster via the ```oc login``` command. Make sure you have a working namespace.
2. Update the [source information](https://github.com/binoyskumar92/NodeJS-OCP-Template-to-Helm/blob/a84a7b2a7d8d65e985843924b600867fc2cb660d/nodeapp-ocp-template.yaml#L54) to point to the nodejs app you want to build and deploy
2. Make sure you create the secret used in the template(```git-secret```). This secret is needed to read/clone the source code from source repository like gitlab/github you specified above.
```bash
oc create secret generic git-secret --from-literal=username=<git-username> --from-literal=password=<git-passcode/password> --type=kubernetes.io/basic-auth
```
### How to run the OCP template

1. Now run the following command to process the template and create it in OpenShift namespace you are currently logged in. 
```bash
oc process -f nodeapp-ocp-template.yaml | oc apply -f -
```
2. Run below command to make sure all resources in the template file is created
```bash
oc get all -l app=nodeapp
```