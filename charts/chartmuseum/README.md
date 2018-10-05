# ChartMuseum Helm Chart

Deploy your own private ChartMuseum.
Please also see https://github.com/kubernetes-helm/chartmuseum

## Deployment with helm

```sh
# fetch template
helm fetch --untar --untardir build --repo https://kubernetes-charts.storage.googleapis.com --version 1.6.2 chartmuseum
# deploy
helm template --name chartmuseum -f charts/chartmuseum/values.yaml build/chartmuseum | oc apply -f -
```