## Deploy using Helm and K8s

If you have a running Kubernetes cluster available, you can deploy the BPN Discovery using our Helm Chart, which is located under `charts/bpndiscovery`. In case
you don't have a running cluster, you can set up one by yourself locally, using [minikube](https://minikube.sigs.k8s.io/docs/start/). In the following, we will
use a minikube cluster for reference.

Before deploying the BPN Discovery, enable a few add-ons in your minikube cluster by running the following commands:

`minikube addons enable storage-provisioner`

`minikube addons enable default-storageclass`

`minikube addons enable ingress`

Fetch all dependencies by running `helm dep up charts/bpndiscovery`.

In order to deploy the helm chart, first create a new namespace "discovery": `kubectl create namespace discovery`.

Then run `helm install bpndiscovery -n discovery charts/bpndiscovery`. This will set up a new helm deployment in the discovery namespace. By default, the
deployment contains the BPN Discovery instance itself, and a Postgresql.

Check that the two containers are running by calling `kubectl get pod -n discovery`.

To access the BPN Discovery API from the host, you need to configure the `Ingress` resource. By default, the `Ingress` is disabled.

If you enable the `Ingress`, the BPN Discovery exposes the API on https://minikube/discovery/bpndiscovery. For that to work, you need to append `/etc/hosts`
by running `echo "minikube $(minikube ip)" | sudo tee -a /etc/hosts`.

For automated certificate generation, use and configure [cert-manager](https://cert-manager.io/). By default, authentication is deactivated, please
adjust `bpndiscovery.authentication` if needed.

## Configuration with External Keycloak and PostgreSQL

If you want to use an external Keycloak instance for authentication and an external PostgreSQL database instead of the bundled ones, please refer to our comprehensive [Configuration Guide](docs/CONFIGURATION_GUIDE.md).

The guide includes:
- Step-by-step instructions in both Russian and English
- Example configuration files
- Security best practices
- Troubleshooting tips

Quick example for external setup:

```bash
# Use the provided example values file
helm install bpndiscovery charts/bpndiscovery \
  -n discovery \
  -f charts/bpndiscovery/values-external-keycloak-postgres.yaml \
  --set bpndiscovery.dataSource.url=jdbc:postgresql://your-db-host:5432/bpndiscovery \
  --set bpndiscovery.dataSource.user=your-db-user \
  --set bpndiscovery.dataSource.password=your-db-password \
  --set bpndiscovery.idp.issuerUri=https://your-keycloak/auth/realms/your-realm
```

## Parameters

The Helm Chart can be configured using the following parameters. For a full overview, please see the [values.yaml](./charts/bpndiscovery/values.yaml).

### BPN Discovery parameters and PostgreSQL parameters

Please have a look into the [README.md](charts/bpndiscovery/README.md).

### Prerequisites

- Kubernetes 1.19+ 
- Helm 3.2.0+ 
- PV provisioner support in the underlying infrastructure