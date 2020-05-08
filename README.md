### Summary

This repo helps to build and deploy a nodejs app via an OpenShift templates and Helm.

### Helm vs OCP template

* [Helm](https://helm.sh/) is best way to find, share, and use software built for Kubernetes. It allows describing the application structure through convenient helm-charts and managing it with simple commands.
[OCP-Template](https://docs.openshift.com/container-platform/3.5/dev_guide/templates.html) can describe a set of objects, for example services, build configurations, and deployment configurations. These can be processed to create anything you have permission to create within a project
* Templates are way more static compared to Helm. Helm provides values.yaml file which can be used to parameterize the entire helm chart. These values are easy to override for multiple runs use different values file or using console input.
* Helm can deal with many life cycle methods which templates lack.
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
 4. Run below command to check if the repos are added successfully and Helm is able to access the build and deploy charts.
 ```bash
 helm search repo chartrepo
 ```

### How to use the equivalent Helm version of OCP template

In a Helm chart the customization happens in the values.yaml used. For each run you can use a different values file. These files will hold those fields that are made dynamic in the corresponding charts. Following table gives an overview of different values you can set in values file for both build and deploy

| Name | Description | Default |
| ------ | ------ | ------ |
| NAME_OVERRIDE  | Value that overrides release name for your created k8s resource in Openshift.| Helm run Release name  |
| NAMESPACE | Namespace in which ImageStream exists  | openshift |
| NODEJS_VERSION | Version of nodejs used | 8 |
| MEMORY_LIMIT | Maximum amount of memory the container can use | 512Mi |
| GIT_SECRET | Name of secret created in OpenShift to read from source repository. Check prerequisite step for details  | git-secret |
| SOURCE_REPOSITORY_URL | The URL of the repository with your application source code. You can download this sample code from default value and host it in your personal git repo | https://gitlab.consulting.redhat.com/bsubhaku/helloworldnodeapp.git |
| SOURCE_REPOSITORY_REF | Set this to a branch name, tag or other ref of your repository if you | master |
| CONTEXT_DIR  | Set this to the relative path to your project if it is not in the root | nil |
| APPLICATION_DOMAIN  | The exposed hostname that will route to the Node.js service, if left  | nil |
| NPM_MIRROR | The custom NPM mirror URL | nil |


####  Build 
1. Update the [values.yaml](helm/nodeapp-build/values.yaml) for Build with your required values.
2. Stay in root folder. If logged in to the OpenShift cluster via ```oc```, run the following command to create the BuildConfig and an ImageStream via Helm. This will build the application source specified by source values in [/helm/nodeapp-build/values.yaml](helm/nodeapp-build/values.yaml) and push to the ImageStream.
```bash 
helm upgrade --install <a-release-name> chartrepo/nodeapp-build --values helm/nodeapp-build/values.yaml
```
 > Note: The release-name can be any other name.
 
 3. Run below command to make sure a build pod is running
 ```bash
 oc get pods
 ```

 #### Deploy
1. Update the [values.yaml](helm/nodeapp-deploy/values.yaml) for Deploy  with your required values.
2. Stay in root folder. If logged in to the OpenShift cluster via ```oc```, run the following command to create the Deployment, Route and Service via Helm. This will pull the previously built image in previous Build step. Update values in [/helm/nodeapp-deploy/values.yaml](helm/nodeapp-deploy/values.yaml) before running below command.
```bash 
helm upgrade --install <a-release-name> chartrepo/nodeapp-deploy --values helm/nodeapp-deploy/values.yaml
```
> Note: The release-name can be any other name.

 3. Run below command to get the exposed route information and run the url in your browser when it is ready to see the NodeJS application.
 ```bash
 oc get route -l app=<used-release-name-in-previous-step>
 ```
 4. If you run into any issue always login to your cluster and check the pod logs.