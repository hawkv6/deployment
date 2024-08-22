# HawkV6 Deployment Guide

## Overview
This repository provides the necessary files to deploy the core components of the HawkV6 project on a Kubernetes cluster. The following elements are included:

- **[Jalapeno](https://github.com/cisco-open/jalapeno):** Used for network data aggregation, processing, and storage.
- **[Jalapeno API Gateway (JAGW)](https://github.com/jalapeno-api-gateway):** Facilitates easy retrieval and subscription of data stored in Jalapeno.
- **[Consul](https://github.com/hashicorp/consul):** Serves as a service registry for network services.

Additionally, this repository includes Grafana dashboards stored in the `grafana-dashboards` folder. These dashboards are not automatically included in the Jalapeno Grafana instance but can be easily imported and used.

## Setup Instructions

### Prerequisites
Before proceeding, ensure that you've cloned this repository and initialized the Helm submodules (`jagw-helm` and `jalapeno-helm`):

```bash
git clone https://github.com/hawkv6/deployment.git
git submodule init
git submodule update
```

### Jalapeno Deployment
Follow these steps to install Jalapeno in the `hawkv6-jalapeno` namespace:

1. **Prepare Configuration Files:**
   - Modify the `jalapeno-values.yaml` file according to your requirements.
   - Copy the modified content to the `values.yaml` file within the `jalapeno-helm` submodule:

	```bash
	cp jalapeno-values.yaml jalapeno-helm/jalapeno-helm/values.yaml
	```

2. **Configure Telegraf Ingress**
- This action ensures the correct Kafka topics are used for data transfer to the `clab-telemetry-linker`.
- Copy the `collector-telegraf-ingress-cm.yaml` content into the corresponding ConfigMap in the `jalapeno-helm` submodule.:

	```bash
	cp collector-telegraf-ingress-cm.yaml jalapeno-helm/jalapeno-helm/templates/collectors/telegraf-ingress/collector-telegraf-ingress-cm.yaml
	```
The following Kafka topics are used for data transfer:
- `hawkv6.telemetry.unprocessed`: Contains unprocessed performance measurement telemetry data, utilized by the `clab-telemetry-linker`.
- `hawkv6.openconfig`: Holds OpenConfig interface status and related IPv6 addresses, used by the `generic-processor`.
- `jalapeno.telemetry`: Handles all other telemetry data.

3. **Configure Telegraf Egress**
- This action ensures data is forwarded to InfluxDB.
- Copy the `processors-telegraf-egress-cm.yaml` content into the corresponding ConfigMap in the `jalapeno-helm` submodule:

	```bash
	cp processors-telegraf-egress-cm.yaml jalapeno-helm/jalapeno-helm/templates/processors/telegraf-egress/processors-telegraf-egress-cm.yaml
	```

These Kafka topics are used for data transfer to InfluxDB:

- `hawkv6.telemetry.processed`: Contains the processed performance measurement telemetry data. The `clab-telemetry-linker` writes to this topic.
- `jalapeno.telemetry`: Handles all other telemetry data.
- `hawkv6.telemetry.normalized`: Contains normalized telemetry data. The `generic-processor` writes to this topic.


4. **Deploy Jalapeno**
- Install Jalapeno in the `hawkv6-jalapeno` namespace using Helm:

	```bash
	cd jalapeno-helm
	helm install jalapeno ./jalapeno-helm -n hawkv6-jalapeno --create-namespace
	```

5. **Import Grafana Dashboards** (Optional)
- Import the dashboards found in the `grafana-dashboards` folder into the Jalapeno Grafana instance.


### Jalapeno API Gateway Deployment
Follow these steps to install the Jalapeno API Gateway in the `hawkv6-jagw` namespace:

1. **Configure Jalapeno API Gateway:**
   - Modify the `jagw-values.yaml` file according to your requirements.
   - Copy the updated content to the `values.yaml` file in the `jagw-helm` submodule:

	```bash
	cp jagw-values.yaml jagw-helm/jagw/values.yaml
	```

2. **Install the API Gateway:**
   - Deploy the Jalapeno API Gateway in the `hawkv6-jagw` namespace using Helm:

	```bash
	cd jagw-helm
	helm install jagw ./jagw -n hawkv6-jagw --create-namespace
	```

### Consul Deployment

Follow these steps to install the Consul instance in the `hawkv6-consul` namespace.

1. **Configure Consul:**
   - Modify the `consul-values.yaml` file according to your requirements.

2. **Add HashiCorp Helm Repository:**
   - Add the HashiCorp Helm repository to your Helm configuration:

	```bash
   	helm repo add hashicorp https://helm.releases.hashicorp.com
   	"hashicorp" has been added to your repositories
	```

3. **Verify Consul Chart Access:**
   - Ensure that you have access to the Consul chart:

	```bash
   	helm search repo hashicorp/consul
   	NAME                CHART VERSION   APP VERSION DESCRIPTION
   	hashicorp/consul    1.0.1           1.14.1      Official HashiCorp Consul Chart
	```
4. **Label Worker Node (Optional):**
   - If external services cannot reach your internal pod, you may need to work with hostPorts. To do this, label a worker node to run the Consul server:

	```bash
	kubectl label nodes dev-w-0 hawkv6-consul-server=true
	```

 5. **Label Namespace:**
   - Since Consul only accepts non-K8S client agents with Host Ports, label the namespace accordingly:

	```bash
	kubectl create namespace hawkv6-clab
	kubectl label ns hawkv6-clab --overwrite \
    	pod-security.kubernetes.io/enforce=privileged \
    	pod-security.kubernetes.io/warn=privileged
	``` 
6. **Install Consul:**
   - Deploy Consul in the `hawkv6-consul` namespace using Helm:

	```bash
	helm install consul hashicorp/consul --namespace hawkv6-consul --values consul-values.yaml
	```