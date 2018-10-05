# Selenium Grid

[Selenium](https://www.seleniumhq.org/) automates browsers. That's it! What you do with that power is entirely up to you. Primarily, it is for automating web applications for testing purposes, but is certainly not limited to just that. Boring web-based administration tasks can (and should!) be automated as well.

## Setup With Helm

We will install the [Selenium grid]() using [docker-selenium](https://github.com/SeleniumHQ/docker-selenium) with [Helm](https://docs.helm.sh) (aka The package manager for Kubernetes).
To install `helm cli`, please follow the guideline [here](https://github.com/arnaud-deprez/cicd-openshift/blob/master/README.md).

**NOTE**

---
> Since docker-selenium `3.14.0-helium`, the docker images are fully compliant with openshift rules and can run as non root user.
> So there is no need to extend these images anymore and they can be used as-is.
---

Once it is done, you can run the following: 

```sh
# fetch template
helm fetch --untar --untardir build --repo https://kubernetes-charts.storage.googleapis.com --version 0.11.0 selenium
# deploy
helm template --name selenium -f charts/selenium/values.yaml --set hub.ingress.enabled=true --set hub.ingress.hosts={"<ingress_hostname>"} build/selenium | oc apply -f -
```

For example, with `minishift` you can setup the hostname with:

```sh
helm template --name selenium -f charts/selenium/values.yaml --set hub.ingress.enabled=true --set hub.ingress.hosts={"hub-cicd.$(minishift ip).nip.io"} build/selenium | oc apply -f -
```