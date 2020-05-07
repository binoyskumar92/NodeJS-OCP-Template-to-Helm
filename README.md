### Summary

This repo helps to build and deploy a nodejs app via an OpenShift templates and Helm.

### Helm vs OCP template

* [Helm](https://helm.sh/) is best way to find, share, and use software built for Kubernetes. It allows describing the application structure through convenient helm-charts and managing it with simple commands.
[OCP-Template](https://docs.openshift.com/container-platform/3.5/dev_guide/templates.html) can describe a set of objects, for example services, build configurations, and deployment configurations. These can be processed to create anything you have permission to create within a project
* Templates are way more static compared to Helm. Helm provides values.yaml file which can be used to parameterise the entire helm chart. These values are easy to override for multiple runs use different vlaues file or using console input.
* Helm can deal with many lifecycles methods which templates lack.
* Helm can be built around a solid testing framework for making it viable for large projects.

### How to run the OCP template

1. First login to your OpenSH=hift cluster via the ```oc login``` command.
2. Make sure you crea the secret used in the template(```git-secret```). This secret is needed to access the source from source repository like gitlab/github.

```bash
oc create secret generic git-secret --from-literal=username=<git-username> --from-literal=password=<git-passcode/password> --type=kubernetes.io/basic-auth
```
3. Now run
```bash
oc process -f nodeapp-ocp-template.yaml | oc apply -f -
```