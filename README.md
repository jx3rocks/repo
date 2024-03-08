:wq
## Chart Repository

[Helm](https://helm.sh) must be installed to use the charts.
Please refer to Helm's [documentation](https://helm.sh/docs/) to get started.

### Searching for charts

Once Helm is set up properly, add the repo as follows:

    helm repo add myrepo https://jx3rocks.github.io/repo/

you can search the charts via:

    helm search repo aks04

## View the YAML

You can have a look at the underlying charts YAML at: [index.yaml](index.yaml)

You can manually update the index.yaml using the commands:
## Extract chart from another repo
curl -LO https://chartmuseum.fyi.hedraios.xyz/charts/node-https-aks02-0.0.1.tgz
## Update the index.yaml
helm repo index .
## Update repo
git commit -a -m "index update"
git push origin gh-pages

