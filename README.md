### Summary

This repo helps to build and deploy a nodejs app via an OpenShift templates and Helm.

### Helm vs OCP template

* [Helm](https://helm.sh/) is best way to find, share, and use software built for Kubernetes. It allows describing the application structure through convenient helm-charts and managing it with simple commands.
[OCP-Template](https://docs.openshift.com/container-platform/3.5/dev_guide/templates.html) can describe a set of objects, for example services, build configurations, and deployment configurations. These can be processed to create anything you have permission to create within a project
* Templates are way more static compared to Helm. Helm provides values.yaml file which can be used to parameterise the entire helm chart. These values are easy to override for multiple runs use different vlaues file or using console input.
* Helm can deal with many lifecycles methods which templates lack.
* Helm can be built around a solid testing framework for making it viable for large projects.

### Prerequisites for OCP Template
1. User has access to an Openshift Cluster 3.11 or 4. 
2. User has installed [OpenShift Client Tools](https://docs.openshift.com/enterprise/3.0/cli_reference/get_started_cli.html#installing-the-cli) and has access to the cluster via ```bash oc```.
3. First login to your OpenShift cluster via the ```oc login``` command. Make sure you have a working namespace. If not, create a new one.
4. Update the [source information](https://github.com/binoyskumar92/NodeJS-OCP-Template-to-Helm/blob/a84a7b2a7d8d65e985843924b600867fc2cb660d/nodeapp-ocp-template.yaml#L54) in the template to point to the Nodejs app repo you would like to build and deploy
5. Run below command to create the secret used in the template [git-secret](https://github.com/binoyskumar92/NodeJS-OCP-Template-to-Helm/blob/a47f994559ec29a97699de1ebaee5a44dce44ebb/nodeapp-ocp-template.yaml#L57) in your OpenShift namespace. This secret is needed to read/clone the source code from source repository like gitlab/github you specified above.
```bash
oc create secret generic git-secret --from-literal=username=<git-username> --from-literal=password=<git-passcode/password> --type=kubernetes.io/basic-auth
```
### How to run the OCP template

1. Now run the following command to process the template and create it in OpenShift namespace you are currently using. 
```bash
oc process -f nodeapp-ocp-template.yaml | oc apply -f -
```
2. Run below command to make sure all resources in the template file is created
```bash
oc get all -l app=nodeapp
```

### Prerequisites for Helm

1. Go through the prerequisites for ocp template except step 4.
2. Make sure [Helm client](https://github.com/helm/helm/releases) v3.x.x is installed and added to your path.
3. Add the Helm chart repo by running below command. The [docs](docs) folder has packaged charts for build and deploy.
```bash
 helm repo add chartrepo https://binoyskumar92.github.io/NodeJS-OCP-Template-to-Helm/
 ```
 *Note: The name chartrepo can be any other name*
 4. Run below command to check if the repos are added successfully and it is able to access the build and deploy charts.
 ```bash
 helm search repo chartrepo
 ```

### How to use the equivalent Helm version of OCP template

1. Go through the prerequisites and make sure [Helm client](https://github.com/helm/helm/releases) v3.x.x is installed.
2. Update the [values.yaml]() for Build with your required values.
3. Navigate to [helm/build](helm/nodeapp-build) folder. If logged in to the OpenShift cluster via ```oc```, run the following command to create the BuildConfig and an ImageStream via Helm. This will build the application source specified by source values in [/helm/build/values.yaml](/helm/nodeapp-build/values.yaml) and push to the ImageStream.
```bash 
helm upgrade --install <a-release-name> chartrepo/nodeapp-build 
```