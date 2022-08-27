# Deploy loki-distributed with kustomize and skaffold to Kubernetes
This repo contains loki-distributed manifest that will be customize using kustomize and deployed with skaffold for each environments using profile.

## Tech Stack
* Kustomize (v4.5.7)
* Pre-commit (v2.20.0)
* Skaffold (v1.39.2)
* Helm (v3.9.1)
* Kubectl (v1.24.2)
* Loki distributed (v2.6.1)

### Loki-distributed architecture
Grafana Loki is a set of components that can be composed into a fully featured logging stack. This repo will install loki with distributed services which mean there are some services that need to install instead of standalone binary loki. Here are the services will be installed with this repo.
* gateway
* ingester
* distributor
* querier
* query-frontend
* table-manager
* ruler
![loki-distributed-architecture](https://grafana.com/static/assets/img/blog/2019-04-15-cortex-architecture.png)

### Directory structure
Here is the directory structure to separate the base and overlays (both development and production).
```bash
.
├── README.md
├── base
│   ├── custom-labels.yaml
│   ├── gateway-svc.yaml
│   ├── kustomization.yaml
│   ├── lokidistributed-base.yaml
│   └── priorityclass.yaml
├── overlays
│   ├── development
│   │   ├── distributor-deploy.yaml
│   │   ├── gateway-deploy.yaml
│   │   ├── ingester-ss.yaml
│   │   └── kustomization.yaml
│   └── production
│       ├── distributor-deploy.yaml
│       ├── gateway-deploy.yaml
│       ├── ingester-ss.yaml
│       └── kustomization.yaml
├── skaffold.yaml
└── values.yaml
```

### How to...
Here are the step by step to configure and deploy it.

#### Take the base manifest from helm charts.
1. Prepare the helm values before generate template from helm charts because we need to enable some services before it can be generated.
2. Use the helm template from the helm charts(I use helm charts with version 0.55.1), we'll use it as the base manifest to be customize for both development and production environment.
```bash
$ helm repo add grafana https://grafana.github.io/helm-charts
$ helm template grafana/loki-distributed -f values.yaml --version 0.55.1 > base/lokidistributed-base.yaml
```

#### Deploy using skaffold and kustomize.
1. First of all, make sure the patched files are already put in the base and overlays(dev and prod).
```bash
### To ensure the customization is as expected.
$ kustomize build overlays/development

### To ensure the customization is as expected.
$ kustomize build overlays/production
```
2. I separate the deployment for development and production using the profile in the skaffold config file.
```bash
### Make sure to use the correct cluster's credentials
$ KUBECONFIG=/path/to/kubeconfig skaffold deploy -p development ### --kube-context minikube

### Make sure to use the correct cluster's credentials
$ KUBECONFIG=/path/to/kubeconfig skaffold deploy -p production ### --kube-context prod-k8s-cluster
```


##### References
* [https://grafana.com/blog/2019/04/15/how-we-designed-loki-to-work-easily-both-as-microservices-and-as-monoliths/](https://grafana.com/blog/2019/04/15/how-we-designed-loki-to-work-easily-both-as-microservices-and-as-monoliths/)
* [https://github.com/grafana/helm-charts/tree/main/charts/loki-distributed](https://github.com/grafana/helm-charts/tree/main/charts/loki-distributed)
* [https://grafana.com/docs/loki/latest/installation/microservices-helm/](https://grafana.com/docs/loki/latest/installation/microservices-helm/)
* [https://kubectl.docs.kubernetes.io/guides/config_management/](https://kubectl.docs.kubernetes.io/guides/config_management/)
* [https://skaffold.dev/docs/pipeline-stages/](https://skaffold.dev/docs/pipeline-stages/)
