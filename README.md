# CI/CD infrastructure setup on Openshift

This repository contains all the components needed to setup the CI/CD infrastructure.
This has been tested with [Minishift](https://github.com/minishift/minishift) and [CDK](https://developers.redhat.com/products/cdk/overview) for Rhel based images.

## CI/CD architecture in Openshift

Application in Openshift will be segregated in many projects, each project is isolated from other.

![CI/CD Architecture](http://www.plantuml.com/plantuml/proxy?src=https://raw.githubusercontent.com/arnaud-deprez/cicd-openshift/master/doc/images/architecture.puml "CI/CD Architecture")

**Openshift project**
> This is the default project that comes by default with a classic Openshift installation.
It contains images and templates supported by RedHat.

**My-Openshift project**
> The idea of this project is to mimic what is Openshift project but for our specific needs.
It will contains base image, templates, and secrets for private registry.

**Tiller project**
> It will contain the helm server part `Tiller` to manage our release across the cluster.
Tiller will be responsible to apply (install, upgrade or delete) a release in a namespace.

**CICD project**
> It will contain the entire CI/CD infrastructure such as `Jenkins`, `Nexus`, `Sonarqube`, a `Selenium grid` and whatever else is necessary to build, 
test and deliver our applications and improve our code quality.

**Dev project**
> Dev project is where the released image for each cell will be pushed and tagged from each cell project
when developer feel confident with.
This project will also be used by developers to make some tests in an integrated environment.

**Staging project**
> This is where business e2e tests are run. If tests passed, the cell can be promoted to OPT.

**Prod project**
> This project will be used for user acceptance before going to OPS.

## Setup

### My project

Similarly to the Openshift project, we will create a my-openshift project to store our custom templates and eventually our images from our private registry.

```sh
oc new-project my-openshift
oc secrets new-dockercfg my-registry --docker-server="<url>" --docker-username="<username>" --docker-password="<password>" --docker-email="<email>"
oc secrets link default my-registry --for=pull
# to build an image
oc secrets link builder my-registry
```

Then when we want to import an image from our private registry, we will use the reference policy `local`, so other pod will not fetch image
from it directly, instead the image will be cached in the openshift internal registry and the other pod will retrieve their image from it.

This configuration allow us to not provide docker secret in each project and so we can enforce which service account has the
right to pull an image from our private registry without sharing the secret in every project.

```sh
oc import-image ${image}:${version} --from=${image}:${version} -n my-openshift -reference-policy=local --confirm
```

Once we have imported the image, we have to allow other project to pull image from the `my-openshift` project.

To simplify this management, we can also add the ability to all service account in project `project-a` to pull image from project `my-openshift`:

```sh
oc policy add-role-to-group system:image-puller system:serviceaccounts:project-a -n my-openshift
```

### Helm installation (Tiller project)

First, you need to download and install helm client: https://github.com/kubernetes/helm/blob/master/docs/install.md

Then we will install tiller in a specific project `tiller` with appropriate roles:

```sh
export TILLER_NAMESPACE=tiller
oc new-project $TILLER_NAMESPACE
oc create serviceaccount $TILLER_NAMESPACE -n $TILLER_NAMESPACE
oc adm policy add-cluster-role-to-user cluster-admin -z $TILLER_NAMESPACE -n $TILLER_NAMESPACE
oc policy add-role-to-user edit system:serviceaccount:cicd:jenkins -n $TILLER_NAMESPACE
helm init --service-account $TILLER_NAMESPACE
```

By giving these roles to `tiller` service account in `tiller` project, we will be able to create projects and mostly every possible resources in them from helm.
Of course this can be more fine grained depending on your needs and your security policies.

If tiller is already installed but helm client is not initialized, you can simply run the following:

```sh
export TILLER_NAMESPACE=tiller
helm init --client-only
```

### CI/CD project

We will create a cicd project to install our Jenkins, Nexus and SonarQube and whatever else is necessary for our CI/CD infrastructure.
The cicd project should be able to retrieve image from this project.

```sh
oc new-project cicd
oc policy add-role-to-group system:image-puller system:serviceaccounts:cicd -n my-openshift
```

The rest of this documentation assumes you run command in the project cicd. To ensure it, run: `oc project cidc`

#### Components installation

1. [Jenkins](https://github.com/arnaud-deprez/jenkins-docker-openshift)
1. [Nexus 3](https://github.com/arnaud-deprez/nexus3-docker)
1. [SonarQube](https://github.com/arnaud-deprez/sonarqube-docker)
1. [Selenium Grid](https://github.com/arnaud-deprez/selenium-grid-docker)