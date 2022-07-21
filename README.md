# tce-tutorial-1
Deploying my first application via [Tanzu Community Edition](https://tanzucommunityedition.io/docs/v0.12/)

This page walks you through the following steps:

1. Install [Tanzu Community Edition](https://tanzucommunityedition.io/docs/v0.12/cli-installation/).
2. Create an [Unmanaged cluster](https://tanzucommunityedition.io/docs/v0.12/getting-started-unmanaged/).
3. Install [kpack package](https://tanzucommunityedition.io/docs/v0.12/package-readme-kpack-0.5.3/).
4. Install [kpack-dependencies package](https://tanzucommunityedition.io/docs/v0.12/package-readme-kpack-dependencies-0.0.27/).
5. Build an OCI compliant image and Publish the image to your registry.
6. Install and Deploy with [Knative serving](https://tanzucommunityedition.io/docs/v0.12/package-readme-knative-serving-1.0.0/).


### Step 1 - Install [Tanzu Community Edition](https://tanzucommunityedition.io/docs/v0.12/cli-installation/)

```shell
brew install vmware-tanzu/tanzu/tanzu-community-edition
```
### Step 1a. Once the installation is completed, initialize all plugins required by Tanzu Community Edition

```shell
/opt/homebrew/Cellar/tanzu-community-edition/v0.12.1/libexec/configure-tce.sh
```

### Step 2 - Create an Unmanaged Cluster in Tanzu Community Edition

```shell
tanzu unmanaged-cluster create <CLUSTER_NAME> -p 80:80 -p 443:443
```

### Step 3 - Create Registry Secret (This is required to publish images to your registry)

```shell
kubectl create secret docker-registry tutorial-registry-credentials \
--docker-username=my-dockerhub-username \
--docker-password=my-dockerhub-password \
--docker-server=https://index.docker.io/v1/ \
-n default
```

### Step 4 - Install [Kpack package](https://tanzucommunityedition.io/docs/v0.12/package-readme-kpack-0.5.3/)

```shell
tanzu package install kpack \
--package-name kpack.community.tanzu.vmware.com \
-f config/kpack/kpack-values.yaml \
--version 0.5.3 
```

### Step 5 - Install [Kpack Dependencies package](https://tanzucommunityedition.io/docs/v0.12/package-readme-kpack-dependencies-0.0.27/)

```shell
tanzu package install kpack-dependencies \
--package-name kpack-dependencies.community.tanzu.vmware.com \
-f config/kpack_dependencies/kpack-dependencies-values.yaml \
--version 0.0.27 
```

### Step 6 - Install [Contour package](https://tanzucommunityedition.io/docs/v0.12/package-readme-contour-1.20.1/)

```shell
tanzu package install contour \
--package-name contour.community.tanzu.vmware.com \
-f config/contour/contour-values.yaml \
--version 1.20.1
```

### Step 7 - Install [Knative Serving package](https://tanzucommunityedition.io/docs/v0.12/package-readme-knative-serving-1.0.0/)

```shell
tanzu package install knative-serving \
--package-name knative-serving.community.tanzu.vmware.com \
-f config/knative_serving/knative-serving-values.yaml \
--version 1.0.0
```


### Step 8 - Verify kpack, kpack-dependencies and knative-serving packages are installed, and status is "Reconcile succeeded"

```shell
tanzu package installed list -A
```

### 9. Verify kpack-dependencies are installed on your cluster

```shell
kubectl get clusterstore -n default
kubectl get clusterstack -n default
kubectl get clusterbuilder -n default
```

## Building our OCI compliant image 

### 1. Create Service Account
This service account is used to reference the registry secret created in prerequisite step 1 using the manifest service-account.yaml file.

```shell
kubectl apply -f service-account.yaml
```

### Step 2 - Build OCI Image and Publish

The image.yaml manifest file defines your image source, builder and tag needed to build and publish your OCI image.
To learn how cloud native build packs work visit [BuildPacks Docs](https://buildpacks.io/docs/)

_Make sure you replace <DOCKER_USERNAME> with your username_
### Apply the image to your cluster
```shell
kubectl apply -f image.yaml
```
### Step 2a - View the logs of how kpack builds the image

```shell
kubectl logs hello-world-build-1-build-pod -c detect
kubectl logs hello-world-build-1-build-pod -c prepare
kubectl logs hello-world-build-1-build-pod -c build
kubectl logs hello-world-build-1-build-pod -c export
```

### Step 2b - Get the latest image built by Kpack and make sure READY is True
```shell
kubectl get images hello-world
```

### Step 3 - Deploy your application to our cluster using knative. (Note your image will be different)
To learn more on Knative visit [Knative Docs](https://knative.dev/docs/getting-started/)

```shell
kn service create hello-world \
--image index.docker.io/ynamadi676/tutorial1@sha256:3889ad138b7739159a015e9e71d810fccf6e707186a3412a5917ea82f36313ad \
--pull-secret tutorial-registry-credentials
```

### Step 3a - Verify you are able to access the deployed image
```shell
open -a Safari.app "http://hello-world.default.127-0-0-1.nip.io/"
```