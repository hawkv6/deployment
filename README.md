### General
This repository is used to install the following elements on a Kubernetes cluster needed for the hawkv6 project.
- [Jalapeno](https://github.com/cisco-open/jalapeno)
- [Jalapeno API Gateway] https://github.com/jalapeno-api-gateway also called JAGW
- [Consul](https://github.com/hashicorp/consul)

#### Jalapeno Helm
The `jalapeno-helm-values.yaml` is used with https://github.com/jalapeno-api-gateway/jalapeno-helm to deploy Jalapeno.
It is added via submodule.

#### Jalapeno API Gatewat Helm
The `jagw-helm-values.yaml` is used with https://github.com/jalapeno-api-gateway/jagw-helm/tree/main to deploy the Jalapeno API Gateway.
It is added via submodule.

#### Consul
More information about installing Consul on Kubernetes with helm here: https://developer.hashicorp.com/consul/docs/k8s/installation/install

### Deployment
Before you can start clone this repository with `git clone` and update the helm submodules (jagw-helm and jalapeno-helm) with `git submodule init`.

### Deploy Jalapeno
The hawkv6 Jalapeno will be installed in the `hawkv6-jalapeno` namespace.
To deploy the cluster, follow these steps.
1. Adapt the `jalapeno-values.yaml` file to your needs and copy the file's content into the `values.yaml` file in the folder `jalapeno-helm` of the `jalapeno-helm` submodule.
	```
	cp jalapeno-values.yaml  jalapeno-helm/jalapeno-helm/values.yaml
	``` 
2. Copy the content of the `collector-telegraf-ingress-cm.yaml` file into the config map used for the Telegraf ingress service. This will ensure that the correct topics in Kafka will be used to transfer data to the [clab-telemetry-linker](https://github.com/hawkv6/clab-telemetry-linker)
	```
	cp collector-telegraf-ingress-cm.yaml jalapeno-helm/jalapeno-helm/templates/collectors/telegraf-ingress/collector-telegraf-ingress-cm.yaml
	```
3. Copy the content of the `processors-telegraf-egress-cm.yaml` file into the config map used for the Telegraf egress service. This will ensure that the data are forwarded to the InfluxDB:
	```
	cp processors-telegraf-egress-cm.yaml jalapeno-helm/jalapeno-helm/templates/processors/telegraf-egress/processors-telegraf-egress-cm.yaml
	```
4. Install Jalapeno with the following command in the `hawkv6-jalapeno` namespace:
	```
	cd jalapeno-helm && helm install jalapeno ./jalapeno-helm -n hawkv6-jalapeno --create-namespace
	```
5. (Optional) Import the dashboards found in the folder `grafana-dashboards`

### Deploy Jalapeno API Gateway
The hawkv6 Jalapeno API Gateway will be installed in the `hawkv6-jagw` namespace.
To deploy the application, follow these steps.
1. Adapt the `jagw-values.yaml` file to your needs and copy the file's content into the `values.yaml` file in the folder `jagw` of the `jagw-helm` submodule.
	```
	cp jagw-values.yaml jagw-helm/jagw/values.yaml
	``` 
2. Install the Jalapeno API Gateway with the following command in the `hawkv6-jagw` namespace:
	```
    cd jagw-helm && helm install jagw jagw -n hawkv6-jagw --create-namespace
	``` 


### Deploy Consul
The hawkv6 Consul instance will be installed in the `hawkv6-consul` namespace.
To deploy the application, follow these steps.
1. Adapt the `consul-values.yaml` file to your needs.
2. Add the HashiCorp Helm Repository:
	```
	$ helm repo add hashicorp https://helm.releases.hashicorp.com
	"hashicorp" has been added to your repositories
	```
3. Verify that you have access to the consul chart:
	```
	$ helm search repo hashicorp/consul
	NAME                CHART VERSION   APP VERSION DESCRIPTION
	hashicorp/consul    1.0.1           1.14.1      Official HashiCorp Consul Chart
	```
4. If the external services can not reach your internal pod you have to work with hostPorts. Thus, it makes sense to let the server run only on one worker node. To do so we label one worker with  `hawkv6-consul-server=true`:
	```
	kubectl label nodes dev-w-0 hawkv6-consul-server=true
	```
5. Because Consul does only accept non K8S client agents with Host Ports, we have to label the namespace too. To do so create the namespace and label it:
	```
	kubectl create namespace hawkv6-clab
	kubectl label ns hawkv6-clab --overwrite \
  	pod-security.kubernetes.io/enforce=privileged \
  	pod-security.kubernetes.io/warn=privileged
	```
6. Install Consul with the following command in the `hawkv6-consul` namespace:
	```
	helm install consul hashicorp/consul --namespace hawkv6-consul --values consul-values.yaml
	``` 
