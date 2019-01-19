# Zalenium

[Zalenium](https://opensource.zalando.com/zalenium/) is a flexible and scalable container based Selenium Grid with video recording, live preview, basic auth & dashboard.

## Setup With Helm

We will install the [Zalenium](https://opensource.zalando.com/zalenium/) using their [chart](https://github.com/zalando/zalenium/tree/master/charts/zalenium) with [Helm](https://docs.helm.sh) (aka The package manager for Kubernetes).
To install `helm cli`, please follow the guideline [here](https://github.com/arnaud-deprez/cicd-openshift/blob/master/README.md).

Once it is done, you can run the following: 

```sh
# Clone the project
git clone https://github.com/zalando/zalenium.git
cd zalenium
# deploy
helm template --name zalenium \
    --set hub.tag=3.141.59g \
    --set persistence.video.enabled=true \
    --set persistence.video.size=10Gi \
    --set hub.ingress.enabled=true \
    --set hub.ingress.hostname="<ingress_hostname>" \
    charts/zalenium | oc apply -f -
```

For example, with `minishift` you can setup the hostname with:

```sh
helm template --name zalenium \
    --set hub.tag=3.141.59g \
    --set persistence.video.enabled=true \
    --set persistence.video.size=10Gi \
    --set hub.ingress.enabled=true \
    --set hub.ingress.hostname="zalenium-cicd.$(minishift ip).nip.io" \
    charts/zalenium | oc apply -f -
```
